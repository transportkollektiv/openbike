# OpenBike

OpenBike is an open source bikesharing system.

## Documentation

The [docs directory](https://github.com/transportkollektiv/openbike/tree/master/docs) contains the documentation, you can find a [rendered version online](https://docs.openbike.dev).

## Project overview

![](/img/openbike_structure.svg)

* [„Cykel“](https://github.com/transportkollektiv/cykel) (django backend)
* [„Voorwiel“](https://github.com/transportkollektiv/voorwiel) (bike rental frontend website)
* [„skoetsel“](https://github.com/transportkollektiv/skoetsel) (optional backend map for service and maintenance crew)
* Tracker
	* [Dragino LGT-92 Tracker firmware](https://github.com/dragino/LGT-92_-LoRa_GPS_Tracker/)
	* [Dragino LGT-92 Tracker hardware](https://github.com/dragino/Lora/tree/master/LGT-92)
	* [Wifi Tracker firmware](https://github.com/transportkollektiv/lora-wifi-location-tracker)
	* [Wifi Tracker hardware „Lortinchen“](https://github.com/transportkollektiv/lora-wifi-location-tracker/tree/master/hardware)
	* [Manual Tracker from Eigenbaukombinat](https://github.com/Eigenbaukombinat/cykel-manual-tracker)
* TTN Adapter
	* [Dragino decoder](https://github.com/transportkollektiv/tracker-ttn-decoders/blob/master/lgt92-1.5.1.js)
	* [Wifi Tracker decoder](https://github.com/transportkollektiv/lora-wifi-location-tracker/blob/master/ttn-decoder-script.js)
	* [Cykel-TTN Adapter for GPS-Trackers](https://github.com/transportkollektiv/cykel-ttn)
	* [Cykel-TTN Adapter for Wifi-Trackers](https://github.com/transportkollektiv/cykel-ttn-wifi)
* Locks
	* [Cykel Adapter for Jimi/Concox the BL10 Bluetooth/GSM Lock](https://github.com/transportkollektiv/cykel-lock-bl10)
	* [Cykel Adapter for Omni Bluetooth/GSM Locks](https://github.com/transportkollektiv/cykel-lock-omni)
