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

.. note:: Location detection will not as precise as you know it from Smartphones, because these devices use a wide combination of different sensors and online information to optimize the result.

Satellite based (GPS)
"""""""""""""""""""""

The most common method for geolocation detection are global navigation satellite system (`GNSS <https://en.wikipedia.org/wiki/Satellite_navigation>`_), better known as GPS. Currently there are four global systems in operation: `Navstar GPS <https://en.wikipedia.org/wiki/Global_Positioning_System>`_, `GLONASS <https://en.wikipedia.org/wiki/GLONASS>`_, `BeiDou <https://en.wikipedia.org/wiki/BeiDou_Navigation_Satellite_System>`_ and `Galileo <https://en.wikipedia.org/wiki/Galileo_(satellite_navigation)>`_. But no worries you don't have to decide for ohne of these systems, because most modern GNSS sensors are using the signals of more than one of these systems. So your vehicle rental system should be safe from global conflicts.

The accuracy of GNSS Systems can be very precise. Affordable Sensors can achieve precisions between 3 and 30 meters. But these values can only be achieved under optimal circumstances. Buildings, trees and other tall obstacles can significantly reduce precision. Satellite system based sensors are needing a view to the sky. So these systems are unable to operate in buildings or underground garages and will have problems detecting location even under bridges.

The biggest disadvantage of GNSS-Sensors is the high energy consumption. This might not such a big prolem if you use vehicles with onboard power supply like E-Bikes or cars. On normal bikes you will use trackers with integrated batteries. Charging or swapping these batteries every few days or even weeks is very time consuming maintenance task which should be avoided.

So there are some ways and tradeoffs to extend the runtime of these battieres: 

- The GNSS Sensor should not running all the time and instead wake up in bigger time intervals.
- It is recommended to use trackers with acceloration sensors. They can be konfigured to only wake up the GNSS-Sensor on movement. In most sharing systems, the vehicles are parked for the most of the time, so this should save a lot of energy. Some trackers allow to have a different wakeup intervall in moving state and in parking state.
- Acquiring an new geolocation after a sleep coasts some time. The first geolocations the sensor is detecting after a sleep are ver unprecise, and the imprecision can be hundreds of meters, even kilometers. I can take some minutes unitl the precision is "good enough". So you have to decide what "good enough" means - so there is s tradeof between energy consumption and precision. Many trackers are able to configure a maxium time for location acquiring and a minimum precision.

Wifi
""""

Acquiring a `geolocation using wifi <https://en.wikipedia.org/wiki/Wi-Fi_positioning_system>`_ is a very common on smartphones, because it allows approximate a geolocation without starting the energy intensive GNSS chip. With this method the device is scanning wifi networks around and then match them against a database. This database contains the hardware addresses of wifi networks and where they have been detected before. For these method it is essential, that the scanned wifi networks have been mapped with it geolocation to the database before. 
It would is very time-consuming to create such a database. So there are some providers which mapped nearly the whole world with the help of smartphones. `Google is providing access <https://developers.google.com/maps/documentation/geolocation/overview>`_ to the biggest of these databases, but the usage is expensive (currently $4 per 1000 location calculations). There are also other commecial providers.

For non-commercial projects `Mozilla Locations Services <https://location.services.mozilla.com/>`_ is an alternative with a good coverage in many places. Also the coverage can be improved by everyone with an android smartphone by using the mozilla stumbler app. The problem ist, that the future of this project is a bit unclear. But the underlying software of mozilla location services `is open source <https://github.com/mozilla/ichnaea/>`_ and can be used to create a own database.

The big advantage of Wifi location calculation is, the very low energy consumption and extreme low startup time compared to GNSS-Systems. It also works indoor and other enviroments without a view to the sky.

The disadvantages of Wifi location calculation are, that it works only in populated areas, where wifi networks are visable. Even bigger parks in cities could be a problem, but it is really dependend on the enviroment. Another problem is the depedency on a loaction database services and that the database need coverage of the taget area.

The precision is extremely dependend on the number of visble wifi networks and the databse quality. Results between 10 and 100 meters are typical.

Data transmission
^^^^^^^^^^^^^^^^^

Cellular
""""""""

Lora-WAN
""""""""

Using users smartphone
""""""""""""""""""""""