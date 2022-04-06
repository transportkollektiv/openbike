.. _`operator-skoetsel`:

Skoetsel
========

.. image:: ../images/skoetsel-map-ui.png
   :width: 100%
   :alt: Skoetsel UI



`Skoetsel <https://github.com/transportkollektiv/skoetsel>`_ is a maintenance frontend for OpenBike that is supposed to give the operating crew a better overview over location and state of the bikes. Everyone with access to the `cykel <https://github.com/transportkollektiv/cykel>`_ backend can sign in.

.. note:: Skoetsel is still under development. Design and UI might be subject to changes in the future.

You can obtain more information about one specific bike by clicking on the corresponding icon. Available bikes are marked green, bikes that are in use blue and inactive bikes grey. In contrast to `voorwiel <https://github.com/transportkollektiv/voorwiel>`_ (the customer frontend), skoetsel shows the reported locations of all tracker types, meaning not only primary trackers and locations reported by users, but also those marked as "internal" in the backend.   

If multiple trackers are marked active and their location differs more than 500 meters, a red line is displayed between them. If this happens, there might be some problem with the bike or one of the trackers (e.g. a low battery) and you should undertake some investigations.