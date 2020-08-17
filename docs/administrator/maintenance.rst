Maintenance
===========

Once you have installed openbike, you’ll start thinking about maintaining your installation.
The following guide assumes that you perform general system maintenance and monitoring.
So please: keep your servers up to date on security updates. Keep all non-public ports closed. Follow best practices.


Updates
-------

    .. warn: While we try hard not to break things, **please perform a backup before every upgrade.**

This guide assumes that you followed the :ref:`Manual deployment <installation/manual>` documentation.

To upgrade openbike to the newest versions of the components, pull the latest code changes and run the following commands as the ``openbike`` user:

cykel
^^^^^

::

    $ source /srv/openbike/venv/bin/activate
    (venv)$ cd /srv/openbike/cykel
    (venv)$ git pull
    (venv)$ pip3 install -U -r requirements.txt
    (venv)$ python3 manage.py migrate
    (venv)$ python3 manage.py collectstatic

Restart the service with the following command:

::

    # systemctl restart cykel

voorwiel
^^^^^^^

::

    $ cd /srv/openbike/voorwiel
    $ git pull
    $ npm ci
    $ NODE_ENV=production NPM_CONFIG_PRODUCTION=true npm run build
