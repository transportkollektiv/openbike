Manual tracker adapter deployment
=================================

To automatically update locations of your bikes, we use different type of trackers that send their position home.
To make OpenBike fully flexible in which way trackers send their locations, we're using the concept of small adapter services that translate the different ways to send and receive a location to an HTTP call to cykel.

cykel-ttn
---------
In this case, we're looking at the LoRaWAN based location trackers that send their data over the publicly available `The Things Network`_ (TTN).
The `cykel-ttn`_ adapter is a generic TTN adapter for GPS devices that supports lat/lon, accuracy and battery voltage status. It connects via MQTT to TTN and receives the incoming data packets from the trackers near instantly.
If your system uses multiple different trackers that are all LoRaWAN/TTN based you can deploy cykel-ttn once and just use different configuration. More to this usage below.

Decoder
^^^^^^^
To use the cykel-ttn adapter you need to convert the incoming bytes from your lora device to a readable format. The TTN Console supports decoders/converters and validators for this use case. Look into the `tracker-ttn-decoders`_ repository to find javascript decoders for some trackers, which you can use as decoder function in the things network console. In general each tracker model needs its own decoder script, but they are often provided by the supplier.

cykel-ttn expects ``latitude`` and ``longitude`` and/or ``vbat`` fields. Data packets without location are allowed in the case that the tracker doesn't know its location but wants to send its remaining battery status. If fieldnames in decoder scripts differ, you can simply use the converter tab in the ttn console.

Visit ``https://eu1.cloud.thethings.network/console/applications/<application-id>/payload-formatters/uplink`` to set the Formatter type to Javascript and enter the decoder function there.

Install from source
^^^^^^^^^^^^^^^^^^^

The following steps are to be executed as the ``openbike`` user. Before we actually install cykel-ttn, we will activate the existing virtual environment to isolate the python packages from your global python installation::

    $ source /srv/openbike/venv/bin/activate

We now clone the cykel-ttn source and install the dependencies::

    (venv)$ git clone https://github.com/transportkollektiv/cykel-ttn.git /srv/openbike/cykel-ttn
    (venv)$ cd /srv/openbike/cykel-ttn
    (venv)$ pip3 install -U -r requirements.txt

Configuration
^^^^^^^^^^^^^

Like cykel, cykel-ttn also uses Environment Variables for the essential configuration. You can put these into a ``.env`` file, that is loaded by systemd when running the process::

    TTN_HOST="<public host:port address from the ttn integration/mqtt page>"
    TTN_USERNAME="<username@tenant from the ttn integration/mqtt page>"
    TTN_API_KEY="<a fresh ttn api key>"
    ENDPOINT="https://cykel/api/bike/updatelocation"
    ENDPOINT_AUTH_HEADER="Api-Key <your api key for cykel>"
    PORT=8081

``TTN_HOST``:
    The ``host:port`` combination for connecting to the TTN MQTT. This is given to you on the TTN integration/mqtt page.

``TTN_USERNAME``:
    The ``username@tenant`` combination. This is also given to you on the TTN integration/mqtt page.

``TTN_API_KEY``:
    An api key for the TTN application. You can generate these in the TTN console, cykel-ttn only needs the *Read application traffic (uplink and downlink)* capability.

``ENDPOINT``:
    The Endpoint URL at cykel where the location gets delivered to. Normally, you only have to fill in your hostname in ``https://<host>/api/bike/updatelocation``.

``ENDPOINT_AUTH_HEADER``:
    To securely transfer data to cykel, authentication is used for the endpoint. The Auth-Header is in the format ``Api-Key <key>``. To generate a key, visit the cykel administration, go to API keys and add a new one. The key is displayed only once after you fill the form.
    
``PORT``:
    The Port where cykel-ttn should bind to. Don't forget to change if you run multiple instances.

``HOST``:
    empty by default, which equals ``0.0.0.0`` (all addresses on all interfaces)

``LABELS``:
    If you run multiple instances and want do differenciate between them in the monitoring, you can add here comma seperated, ``key=value`` labels.


Start cykel-ttn as a service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We recommend starting cykel-ttn using systemd to make sure it runs correctly after a reboot. Create a file
named ``/etc/systemd/system/cykel-ttn.service`` with the following content::

    [Unit]
    Description=cykel ttn adapter
    After=network.target cykel.service

    [Service]
    User=openbike
    Group=nogroup
    WorkingDirectory=/srv/openbike/cykel-ttn
    EnvironmentFile=/srv/openbike/cykel-ttn/.env
    ExecStart=/srv/openbike/venv/bin/python3 app.py
    PrivateTmp=true
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target


You can now run the following commands to enable and start the service::

    # systemctl daemon-reload
    # systemctl enable cykel-ttn
    # systemctl start cykel-ttn


Run multiple instances
^^^^^^^^^^^^^^^^^^^^^^

If your system uses multiple different trackers that are all LoRaWAN/TTN based you can deploy cykel-ttn once and just use different configurations and decoders. For this usecase, use multiple TTN applications, place multiple ``.env`` files (with different names) in the same directory, copy the ``cykel-ttn.service`` unit file and modify the ``EnvironmentFile`` parameter. You could also use `systemds template units`_.


.. _The Things Network: https://thethingsnetwork.org
.. _cykel-ttn: https://github.com/transportkollektiv/cykel-ttn
.. _tracker-ttn-decoders: https://github.com/transportkollektiv/tracker-ttn-decoders
.. _systemds template units: http://0pointer.de/blog/projects/instances.html