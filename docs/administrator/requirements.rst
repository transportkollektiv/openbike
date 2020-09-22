Requirements
============

To run OpenBike, you will need the following things:

* **cykel**, the backend and administration software

* **voorwiel**, the frontend

* **skoetsel**, the operator/maintenance ui

* Python ≥ 3.7

* **A database**. This needs to be a **PostgreSQL Database with PostGIS**. SQLite is supported not supported anymore.

* A **reverse proxy**. While django is capable of serving static files, a proper web server with reverse proxy capabilities like **nginx**, **Apache** or **Caddy** is a much better idea. Additionally the web server provides all the TLS encryption (https).

  .. warning:: Do not ever run without SSL in production. Your users deserve encrypted connections and thanks to
               `Let's Encrypt`_ SSL certificates can be obtained for free these days.

* A **Domain** and **access to DNS**. cykel and voorwiel run on different subdomains, so you need the ability to create multiple subdomains and point the `A` and `AAAA` DNS records to your server. 

* One or multiple **Authentication Providers**. These are OAuth2-compatible services your users already have (like Google, Twitter, Instagram, GitHub, ...) or you run yourself (Nextcloud, Keycloak, ...). OpenBike does not provide registration features by itself, so you don't have to handle email addresses and forgotten passwords. Instead, OpenBike is using **django-allauth** for integrating third-party login, so look into `their documentation for the list of providers they already support <https://django-allauth.readthedocs.io/en/latest/providers.html>`_.

* A **map tile server**. The OpenBike frontend/map view is based on OpenStreetMap tiles. There exist a `whole lot of different styles and providers <https://wiki.openstreetmap.org/wiki/Tile_servers>`_. Please consult the usage policies for the different `tile servers`_ and choose one where not only the style fits you most but one you are also allowed to use - many tile servers are funded by donations. For larger OpenBike installations, bigger usage or commercial offerings, please `host your own tile server`_ or find a `suitable commercial provider <https://switch2osm.org/providers/>`_.

Additional, for building you also need:

* node.js, in the current LTS version

If you use `The Things Network`_ based trackers:

* For the tracker adapters, **outgoing mqtt connectivity**. Your server must have the ability to connect to mqtt servers from the outside world, so make sure your firewalls allow this.

* Your own `The Things Network`_ account, or access on the TTN applications of the trackers.

Additional, for more operational comfort and user feedback, you may want:

* **Monitoring**, based on **Prometheus** and **Grafana**. See the :ref:`Monitoring Guide <monitoring>` for an installation and configuration guide.

* A **Support Ticket Tracker** like **Zammad**, where your users can place feedback, repair requests and general support requests.

.. _Let's Encrypt: https://letsencrypt.org
.. _The Things Network: https://thethingsnetwork.org
.. _tile servers: https://wiki.openstreetmap.org/wiki/Tile_servers
.. _host your own tile server: https://switch2osm.org/serving-tiles/
