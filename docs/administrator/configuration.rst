.. _`configuration`:

Configuration
=============

cykel is mostly configured graphically with the django-admin based administration interface. This is reachable at ``https://<host>/admin``.

Things that can't be configured graphically in the admin are provided as environment variables. This is for example needed to provide the Database Credentials or the Hostnames where cykel is reachable at.

Only in some edge cases you may have to directly modify the ``cykel/settings.py``. 

Admin Access
------------

For your first user with administrative rights, use djangos default way to `create a superuser`_:

::

	$ python manage.py createsuperuser


.. todo:: expand this, add a section about how to add more people with administrative rights

.. todo:: add a section about how to add more people with view only operation rights

Site
----

Cykel needs to know how to refer to itself. This is used in login redirects, so we have to make sure this value is correct.

The Site can be configured in the admin, just edit the first site that is already there after the installation. The domain name must equal the cykel installation domain name, the display name can be the name where your voorwiel UI is reachable at.

Authentication Providers
------------------------

cykel is using **django-allauth** for integrating third-party login, `their documentation provides a list of providers they support <https://django-allauth.readthedocs.io/en/latest/providers.html>`_. For most of the providers, it is enough to add them to ``INSTALLED_APPS`` in ``cykel/settings.py``. By default, Twitter, GitHub, Stackexchange, Slack and FragDenStaat as well as Eventphone are available and already installed.

For configuring an Authentication Provider look into `Social Applications`. When you add a provider there, put the providers name in lowercase into the name field - this is used in the callback url. You also need to provide the OAuth2 client id and the client secret. Some providers call this differently, for these we've added instructions below.


	.. note:: The **Callback URL** you are asked by the provider to put on their allow list is ``https://<host>/auth/<name>/login/callback/``
			  
			  The ``<name>`` is the name of your created social application in the cykel admin, this is why you should use short and lowercase provider names there.


Twitter
^^^^^^^
For twitter, you have to `apply for developer account access <https://developer.twitter.com/en/apply-for-access>`_.
If you have developer access, create an app -- read-only permissions are enough, we're only going to use it for authentication.

Our needed Credentials can be found on the `Keys and tokens Page` as `Consumer API keys`, the client id is the `api key`, the client secret is the `api secret key`.


GitHub
^^^^^^
If you want to use GitHub as an Authentication Provider, create a new OAuth App at https://github.com/settings/developers. You can also create this directly in the settings of an GitHub organization you have Owner access, or you can transfer the ownership to an Organization later from your account. After creating the OAuth App, the Client ID and Client Secret are displayed.


AUTOENROLLMENT_PROVIDERS
^^^^^^^^^^^^^^^^^^^^^^^^
If you want to give users that login with trusted provider the access to rent bikes immediately, without verifying and assigning them rights manually, the ``AUTOENROLLMENT_PROVIDERS`` environment variable is for you. Put the `Social Application` names there and seperate them by comma if its more than one provider. 

Users that login with a provider listed in ``AUTOENROLLMENT_PROVIDERS`` are placed in the ``autoenrollment-rent`` group. This group has by default the right to rent bikes.

Bike Share Preferences
----------------------

You can control some features how cykels sharing components behave. This is configured in the `Bike Share Preferences` section of the administration.

Station matching
^^^^^^^^^^^^^^^^

Distance in Meters in which bikes automatically snap to a station

Automatic hiding
^^^^^^^^^^^^^^^^

In some cases, you may want to hide bikes that didn't report a location for some hours from the GBFS (and therefore the user visible map).

GBFS
^^^^

`Gbfs system id`
	A short, lowercased, globally unique name for your bikesharing operation. This is used by apps to globally identify your system.

`System name`
	Display name

`System short name`
	If your display name is long, try to put a short, maybe appreviated version here.


.. _create a superuser: https://docs.djangoproject.com/en/2.2/ref/django-admin/#createsuperuser

