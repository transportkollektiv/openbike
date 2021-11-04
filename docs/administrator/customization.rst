.. _`customization`:

Customization
=============

If you want to present OpenBike with your own Colors, Logo and Brand, thats totally possible. For most of these customizations you need to have a look in the code and modify it yourself - but for the most typical and wanted parts, some steps are listed on this page.

.. note:: After changing configuration, sourcecode or files of voorwiel, don't forget to execute the steps in :ref:`deploy-voorwiel` again. 

Title & Name
------------
See :ref:`configure-voorwiel` in the :ref:`manual deployment <installation/manual>` guide.

Note that this only is for the voorwiel user interface. Other systems consuming your GBFS aquire the system name with the feeds system_information. You can configure this directly in :ref:`cykels bike share preferences <configuration-cykel-gbfs>`.

Default Map Location
--------------------
See :ref:`configure-voorwiel` in the :ref:`manual deployment <installation/manual>` guide.

Logo
----
Overwrite the file ``src/assets/logo.png`` with your own.

About message
-------------

If you want to override the about message, you don't have to change the ``src/i18n/messages.json`` (where the messages and their translations originally come from). Instead, you can use the ``I18N_MESSAGE_OVERRIDE`` configuration option in ``config/config.production.js`` (see :ref:`configure-voorwiel`) like this:

::

    var I18N_MESSAGE_OVERRIDE = {
      "en": {
        "message": {
          "about": {
            "html": "An experimental <a href='https://github.com/transportkollektiv/openbike'>open source</a> bikesharing."
          }
        }
      },
      "de": {
        "message": {
          "about": {
            "html": "Ein experimentelles <a href='https://github.com/transportkollektiv/openbike'>open source</a> Bikesharing."
          }
        }
      }
    };