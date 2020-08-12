# OpenBike

OpenBike is an open source bikesharing system.

## Documentation

The [docs directory](https://github.com/stadtulm/OpenBike/tree/master/docs) contains the documentation, you can find a [rendered version online](https://docs.openbike.ulm.dev/).

## Project overview

![](/img/openbike_structure.svg)

* [„Cykel“](https://github.com/stadtulm/cykel) (django backend)
* [„Voorwiel“](https://github.com/stadtulm/voorwiel) (customer frontend website)
* Tracker
	* [Dragino LGT-92 Tracker firmware](https://github.com/dragino/LGT-92_-LoRa_GPS_Tracker/)
	* [Dragino LGT-92 Tracker hardware](https://github.com/dragino/Lora/tree/master/LGT-92)
	* [Wifi Tracker firmware](https://github.com/stadtulm/Lora-Wifi-Location-Tracker)
	* [Wifi Tracker hardware „Lortinchen“](https://github.com/stadtulm/Lora-Wifi-Location-Tracker/tree/master/hardware)
	* [Manual Tracker from Eigenbaukombinat](https://github.com/Eigenbaukombinat/cykel-manual-tracker)
* TTN Adapter
	* [Dragino decoder](https://github.com/stadtulm/tracker-ttn-decoders/blob/master/lgt92-1.5.1.js)
	* [Wifi Tracker decoder](https://github.com/stadtulm/Lora-Wifi-Location-Tracker/blob/master/ttn-decoder-script.js)
	* [Cykel-TTN Adapter for GPS-Trackers](https://github.com/stadtulm/cykel-ttn)
	* [Cykel-TTN Adapter for Wifi-Trackers](https://github.com/stadtulm/cykel-ttn-wifi)
