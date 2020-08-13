Manual deployment
=================

This guide describes the installation of a installation of OpenBike with its components from source. In this setup, everything is being run on one host.

.. warning:: Even though we try to make it straightforward to run OpenBike, it still requires some Linux experience to get it right. 

We wrote this guide based on our setup on the Linux distribution Ubuntu 18.04 but it should work very similar on other modern distributions, especially on all systemd-based ones.

Requirements
------------

Please set up the following systems beforehand, we’ll not explain them here in detail (but see these links for external installation guides):

* A HTTP reverse proxy, e.g. `nginx`_, Apache or Caddy to allow HTTPS connections
* A `PostgreSQL`_ 9.5+ database server with `PostGIS`_

We also recommend that you use a firewall, although this is not a OpenBike-specific recommendation. If you’re new to Linux and firewalls, we recommend that you start with `ufw`_.


.. note:: Please, do not run OpenBike without HTTPS encryption. You'll handle user accounts and thanks to `Let's Encrypt`_
          SSL certificates can be obtained for free these days. We also *do not* provide support for HTTP-only
          installations except for development purposes.


Unix user
---------

As we do not want to run the different OpenBike services as root, we first create a new unprivileged user::

    # adduser openbike  --system --disabled-password --shell /bin/bash --home /srv/openbike

In this guide, all code lines prepended with a ``#`` symbol are commands that you need to execute on your server as
``root`` user (e.g. using ``sudo``); all lines prepended with a ``$`` symbol should be run by the unprivileged user.

Database
--------

Having the database server installed, we still need a database and a database user. We can create these with any kind
of database managing tool or directly on our database's shell. For PostgreSQL, we would do::

    # sudo -u postgres createuser openbike
    # sudo -u postgres createdb -O openbike openbike

Additionally, GIS extensions must be enabled. For PostgreSQL with PostGIS, use::

    # sudo -u postgres psql -d openbike -c "CREATE EXTENSION postgis;"

Package dependencies
--------------------

To build and run cykel, you will need the following packages::

    # apt install git build-essential python3 python3-dev python3-venv python3-pip libpq-dev libgdal-dev

For nodejs, we recommend using the nodesource PPA to get the latest LTS version. Look into https://github.com/nodesource/distributions#deb for instructions to add the repository. A short, slightly dangerous way is here::

    # curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
    # apt install -y nodejs


Install cykel from source
-------------------------

Now we will install cykel. The following steps are to be executed as the ``openbike`` user. Before we
actually install cykel, we will create a virtual environment to isolate the python packages from your global
python installation::

    $ python3 -m venv /srv/openbike/venv
    $ source /srv/openbike/venv/bin/activate
    (venv)$ pip3 install -U pip wheel

We now clone the cykel source and install the dependencies::

    (venv)$ git clone https://github.com/stadtulm/cykel.git /srv/openbike/cykel
    (venv)$ cd /srv/openbike/cykel
    (venv)$ pip3 install -U -r requirements.txt
    (venv)$ pip3 install -U gunicorn


Configure cykel
---------------

For configuration normally ``settings.py`` is used. cykel also uses this file, but many important settings are set by environment variables. These can reside in your environment, or in an ``.env`` file in the ``cykel/`` directory (next to the ``settings.py`` file), that gets automatically loaded.

The following is the baseline of environment variables you have to set:

::

    SECRET_KEY=a1b2c3d4e5f6examplechangeit7g8h9
    DEBUG=0
    DATABASE_URL=postgis:///openbike
    ALLOWED_HOSTS=api.dev.bike
    UI_SITE_URL=https://dev.bike
    CORS_ORIGIN_WHITELIST=https://dev.bike
    USE_X_FORWARDED_PROTO=true


``SECRET_KEY``
    The `Secret Key is a django feature <https://docs.djangoproject.com/en/2.2/ref/settings/#std:setting-SECRET_KEY>`_, used for  cryptographic signing for sessions, messages, and more. Generate a nice long random string for this key. (You could use ``openssl rand -hex 32`` for that) 

``DEBUG``
    Set to 1 to enable `DEBUG <https://docs.djangoproject.com/en/2.2/ref/settings/#std:setting-DEBUG>`_ output, or 0 to disable. This should be really only used for development and troubleshooting.

    .. warning:: Never deploy a site into production with DEBUG turned on.

``DATABASE_URL``
    Here you have configure the access to your database in a format supported by `dj-database-url <https://github.com/jacobian/dj-database-url>`_. If your PostgreSQL with PostGIS is residing on the same host, try ``postgis:///openbike`` For user/password auth on another host, use ``postgis://username:password@host/database``

``ALLOWED_HOSTS``
    Comma seperated list of hostnames, where cykel is reachable at. 

``UI_SITE_URL``
    The full URL to your deployment of voorwiel. This is used to redirect the user after the login. Do not use a trailing slash here!

``CORS_ORIGIN_WHITELIST``
    CORS origins to allow, i.e. the frontend URL (with scheme, without path)

``USE_X_FORWARDED_PROTO``
    If you're serving cykel behind a reverse proxy (like in this manual), set this to ``true``, so the ``X-Forwarded-Proto`` header gets interpreted and URLs are built correctly with https.

Prepare cykel for first run
---------------------------

After the configuration we can finally compile static files and create the database structure::

    (venv)$ python3 manage.py migrate
    (venv)$ python3 manage.py collectstatic


Start cykel as a service
------------------------

We recommend starting cykel using systemd to make sure it runs correctly after a reboot. Create a file
named ``/etc/systemd/system/cykel.service`` with the following content::

    [Unit]
    Description=gunicorn daemon for cykel
    After=network.target

    [Service]
    User=openbike
    Group=nogroup
    PIDFile=/run/openbike/cykel.pid
    WorkingDirectory=/srv/openbike/cykel
    ExecStart=/srv/openbike/venv/bin/gunicorn cykel.wsgi \
                    --name cykel \
                    --pid /run/openbike/cykel.pid \
                    --bind 127.0.0.1:8000 --access-logfile - 
    ExecReload=/bin/kill -s HUP $MAINPID
    KillSignal=SIGTERM
    PrivateTmp=true
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target

This runs an gunicorn application server on localhost on port ``8000``. Requests are made to this port by our reverse proxy, which gets configured below.

The referenced ``/run/openbike/cykel.pid`` pid file resides in a directory that does not exist yet. For these temporary directories, systemd brought us the tmpfiles mechanism.
To create this directory, we create the file ``/etc/tmpfiles.d/openbike.conf`` with the following contents::

    # Directory for openbike pid files
    d /run/openbike 0755 openbike nogroup - -

To create the directory right now (without waiting for your next reboot) run::

    # systemd-tmpfiles --create openbike.conf

You can now run the following commands to enable and start the service::

    # systemctl daemon-reload
    # systemctl enable cykel
    # systemctl start cykel


Get voorwiel source
-------------------

To let people rent the bikes, you need a visual interface. voorwiel is the default UI for cykel. The following steps are again executed as the ``openbike`` user::

    $ git clone https://github.com/stadtulm/voorwiel.git /srv/openbike/voorwiel
    $ cd /srv/openbike/voorwiel
    $ npm install

Configure voorwiel
------------------

The configuration of voorwiel currently still happens before the build process. The configuration happens in the ``config/config.production.js`` file:

::

    var ENV = "production";
    var TITLE = "dev.bike - open bikesharing for everyone";
    var NAME = "dev.bike";
    var SYSTEM_URL = "https://api.dev.bike";
    var API_ROOT = SYSTEM_URL + "/api";
    var GBFS_URL = SYSTEM_URL + "/gbfs/gbfs.json";
    var DEFAULT_LOCATION = [48.3984, 9.9908];
    var DEFAULT_ZOOM = 15;
    var SUPPORT_TYPE;
    var SUPPORT_URL;
    var SENTRY_DSN;


``TITLE``
    Title of the voorwiel application page

``NAME``
    Name of your bikesharing system

``SYSTEM_URL``
    The URL to your cykel instance

``DEFAULT_LOCATION``
    Latitude and Longitude for the center of the map

``DEFAULT_ZOOM``
    Zoom level for the default view of the map

Deploy voorwiel
---------------

voorwiel and its configuration is built into a big bundle of javascript. Run the following command to build voorwiel and drop the result into the ``dist`` folder:

::

    $ NODE_ENV=production NPM_CONFIG_PRODUCTION=true npm run build



Reverse Proxy (nginx)
---------------------

The following snippet is an example on how to configure a nginx proxy for cykel and voorwiel::

    server {
        server_name api.dev.bike;
        client_max_body_size 50M;

        location /static/ {
            alias /srv/openbike/cykel/public/;
            try_files $uri $uri/ =404;
        }

        location / {
            try_files $uri $uri/ @cykel;
        }

        location @cykel {
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_read_timeout 300;
            proxy_pass http://127.0.0.1:8000;
        }

        listen [::]:443 ssl; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/api.dev.bike/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/api.dev.bike/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    }
    server {
        server_name dev.bike;

        root /srv/openbike/voorwiel/dist;
        index index.html index.htm;

        location / {
            try_files $uri $uri/ /index.html;
        }

        listen [::]:443 ssl; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/dev.bike/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/dev.bike/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    }


We recommend reading about setting `strong encryption settings`_ for your web server. Certbot provides these with the ``options-ssl-nginx.conf`` file.


Reverse Proxy (apache2)
-----------------------

If you're using apache2 instead, the following snippet is an example on how to configure the reverse proxy for cykel and voorwiel in the apache2 format::

    <VirtualHost *:443>
      ServerName api.dev.bike

      SSLCertificateFile /etc/letsencrypt/live/api.dev.bike/cert.pem
      SSLCertificateKeyFile /etc/letsencrypt/live/api.dev.bike/privkey.pem
      Include /etc/letsencrypt/options-ssl-apache.conf

      Alias /static /srv/openbike/cykel/public
      <Directory /srv/openbike/cykel/public>
        Require all granted
      </Directory>

      ProxyPreserveHost On
      RequestHeader set X-Forwarded-Proto 'https'
      ProxyPass /static !
      ProxyPass / http://127.0.0.1:8000/
      ProxyPassReverse / http://127.0.0.1:8000/
    </VirtualHost>
    <VirtualHost *:443>
      ServerName dev.bike

      SSLCertificateFile /etc/letsencrypt/live/dev.bike/cert.pem
      SSLCertificateKeyFile /etc/letsencrypt/live/dev.bike/privkey.pem
      Include /etc/letsencrypt/options-ssl-apache.conf

      DocumentRoot /srv/openbike/voorwiel/dist
      <Directory /srv/openbike/voorwiel/dist>
        Require all granted
      </Directory>

      FallbackResource /index.html
    </VirtualHost>

Do not forget to enable the *proxy*, *proxy_http* and the *headers* module.

We recommend reading about setting `strong encryption settings`_ for your web server. Certbot provides these with the ``options-ssl-apache.conf`` file.


Next steps
----------

Yay! You've installed cykel and voorwiel. To configure your new running bikesharing system and get access to the administration interface, read the :ref:`Configuration <configuration>` chapter.

.. _nginx: https://botleg.com/stories/https-with-lets-encrypt-and-nginx/
.. _PostgreSQL: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-20-04
.. _PostGIS: https://postgis.net
.. _Let's Encrypt: https://letsencrypt.org
.. _ufw: https://en.wikipedia.org/wiki/Uncomplicated_Firewall
.. _strong encryption settings: https://mozilla.github.io/server-side-tls/ssl-config-generator/