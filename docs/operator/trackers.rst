.. _`operator-trackers`:

Trackers
========

General Information
-------------------

For free floating systems it is essential to know where the vehicles are located. Also station based systems benefit from the use of location trackers. Geolocation of vehicles are not only helpful for users to find them, but also very important for maintenance operations. It might be useful for finding stolen vehicles.

It is possible to integrate different models of trackers into OpenBike. Every model needs a small software-adapter for OpenBike. There are some ready to use adapters for a small number of tracker models. But these adapters are really simple, and it should be very easy to write adapters for new tracker models, based on these adapters.

It is possible to equip vehicles with multiple trackers. This can be for testing, backup or the use of different technologies.

TODO: List of ready to use adapters

Tracker Types
-------------

Location trackers perform two main actions. First task is Geolocation detection, and transmission of the location data. Both actions can be archived with different technologies. In the following sections we explain these technolgies, so you can decide which of them fit the requirements of your local circumstances and you can decide based on these on which tracker model fits your needs.

Location detection
^^^^^^^^^^^^^^^^^^

.. note:: Location detection will not be as precise as you know it from Smartphones, because these devices use a wide combination of different sensors and online information to optimize the result.

Satellite based (GPS)
"""""""""""""""""""""

The most common method for geolocation detection are global navigation satellite systems (`GNSS <https://en.wikipedia.org/wiki/Satellite_navigation>`_), commonly known as GPS. Currently there are four global systems in operation: `Navstar GPS <https://en.wikipedia.org/wiki/Global_Positioning_System>`_, `GLONASS <https://en.wikipedia.org/wiki/GLONASS>`_, `BeiDou <https://en.wikipedia.org/wiki/BeiDou_Navigation_Satellite_System>`_ and `Galileo <https://en.wikipedia.org/wiki/Galileo_(satellite_navigation)>`_. But no worries, you don't have to decide for one of these systems, because most modern GNSS sensors are using the signals of more than one of these systems. So your vehicle rental system should be safe from global conflicts.

The accuracy of GNSS systems can be very precise. Affordable sensors can achieve precisions between 3 and 30 meters. Note that these values can only be achieved under optimal circumstances. Buildings, trees and other tall obstacles can significantly reduce precision. Satellite system based sensors are needing a view to the sky. So these systems are unable to operate in buildings or underground garages and will have problems detecting their location even under bridges.

The biggest disadvantage of GNSS sensors is the high energy consumption. This might not such a big problem if you use vehicles with onboard power supply like E-Bikes or cars. On normal bikes you will use trackers with integrated batteries. Charging or swapping these batteries every few days or even weeks is very time consuming maintenance task which should be avoided.

So there are some ways and tradeoffs to extend the runtime of these battieres: 

- The GNSS sensor should not running all the time and instead wake up in bigger time intervals.
- It is recommended to use trackers with acceleration sensors. They can be configured to only wake up the GNSS sensor on movement. In most sharing systems, the vehicles are parked for the most of the time, so this could save a lot of energy. Some trackers allow to have a different wakeup interval in moving state and in parking state.
- Acquiring an new geolocation after sleeping needs some time. The first locations the sensor is detecting after a sleep are very unprecise, and the imprecision can be hundreds of meters, even kilometers. It can take some minutes until the precision is "good enough". So you have to decide what "good enough" means - there is a trade-off between energy consumption and precision. Many trackers are able to configure a maxium time for location acquiring and a minimum precision.

Using wifi for geolocation calulation can be a good backup strategy if GNSS trackers are running out of energy or vehicles are parked on places with no GNSS signal.

Wifi
""""

Acquiring a `geolocation using wifi <https://en.wikipedia.org/wiki/Wi-Fi_positioning_system>`_ is very common on smartphones, because it allows a approximate geolocation without starting the energy intensive GNSS chip. With this method the device is scanning wifi networks around it and then matches them against a database. This database contains the hardware addresses of wifi networks and where they have been seen before. For these method it is essential that the scanned wifi networks have been mapped with their geolocation to the database before. 
It is very time-consuming to create such a database from scratch. There are some providers which mapped nearly the whole world with the help of smartphones. `Google is providing access <https://developers.google.com/maps/documentation/geolocation/overview>`_ to the biggest of these databases, but the usage is expensive (currently $4 per 1000 location calculations). There are also other commecial providers.

For non-commercial projects `Mozilla Location Services <https://location.services.mozilla.com/>`_ is an alternative with good coverage in many places. Also the coverage can be improved by everyone with an android smartphone by using the mozilla stumbler app. The problem is that the future of this project seems a bit unclear. But the underlying software `is open source <https://github.com/mozilla/ichnaea/>`_ and can be used to create your own database.

The big advantage of Wifi location calculation is the very low energy consumption and extreme low startup time compared to GNSS systems. It also works indoor and other environments without a view to the sky.

The disadvantages of Wifi location calculation are that it works only in populated areas, where wifi networks are visible. Even bigger parks in cities could be a problem, but it is really dependent on the environment. Another problem is the dependency on a location database service and that their database needs coverage of the target area.

The precision is extremely dependent on the number of visble wifi networks and the databse quality. Results between 10 and 100 meters are typical.

Data transmission
^^^^^^^^^^^^^^^^^

After location calculation, this data needs to be transmitted to the OpenBike system.

Cellular
""""""""

Mobile phone networks have good coverage and the data we want to transmit is very small, so the usage of 2G networks is widely available way to transmit data. GPRS modules are very cheap, which result in very cheap ready to use products. Depending on the modules transmission of data via GPRS can be very energy consuming.

.. warning:: 2G networks are not future proof. Providers all around the world are fading out the support of 2G networks or even have them shut down completely.

Newer generations of trackers are using 4G based Narrowband IoT M2M protocols on the mobile phone networks, but we don't have experiance with that kind of devices. In theory this should be more energy efficient than GPRS based systems.

Keep in mind, that you need a SIM card (or eSIM configuration) for each cellular based tracker. There are more and more mobile plans for IoT based applications, even some with shared data contracts between SIM cards. But keep in mind there will be monthly costs for each tracker in operation.


LoRaWAN / The Things Network
"""""""""""""""""""""""""""""

LoRaWAN is a protocol for small bandwidth IoT networks operating in a license free frequency band. Dependent on configuration and local circumstances it can achieve transmission distances between a few hundred meters up to tens of kilometers. Most prominent example of LoRaWAN networks is the crowd sourced `The Things Network <https://www.thethingsnetwork.org>`__. The network is free to use (under consideration of community rules) and everyone can add coverage by adding own gateways to the network.

Transmitting data via LoRaWAN is extremely energy efficient. The amount of data which can transmitted via LoRaWAN is very limited, but it fits exactly the needs of a location tracker for a sharing system.

.. note:: Before buying LoRaWAN hardware check which frequency band is used in your country.

The Things Network provides a `map with all active gateways <https://www.thethingsnetwork.org/map>`_. The placement of gateways is very different, so the project `TTN Mapper is crowdsourcing a coverage map <https://ttnmapper.org/>`_.

Using users smartphone
""""""""""""""""""""""

.. warning:: Using geolocation from users smartphone opens the possibility for users to fake the bike location.

There are some possibilities to transfer data via bluetooth from a tracker to a users smartphone which then transmits that data to the Server. That only works if the user is using a native app. Currently there is no native app for OpenBike.
It is possible to use the GPS of the users smartphone via the WebApp. OpenBike is currently is using that for start and end of an rent, but that is more an optional geolocation source because using that geolocation is not enforced. 
