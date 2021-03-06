====================
Pyramid Google Login
====================

Pyramid authentication policy for Google login (OAuth2 server-side flow)

This extension doesn't configure any authentication policy. You are responsible
of setting the proper security configuration in your Pyramid application. When
authenticated by Google, this extension calls the method
``pyramid.security.remember`` and assume the authentication policy will
remember the user identity.

* PyPI: https://pypi.python.org/pypi/pyramid_google_login
* Code: https://github.com/ludia/pyramid_google_login
* Tests: |travis|

.. |travis| image::
   https://travis-ci.org/ludia/pyramid_google_login.svg?branch=master
   :target: https://travis-ci.org/ludia/pyramid_google_login
   :alt: Tests on TravisCI


Installation
============

Install using setuptools, e.g. (within a virtualenv)::

  $ pip install pyramid_google_login


Setup: Application
==================

Once ``pyramid_google_login`` is installed, you must use the ``config.include``
mechanism to include it into your Pyramid project's configuration.  In your
Pyramid project's ``__init__.py``:

.. code-block:: python

   config = Configurator(.....)
   config.include('pyramid_google_login')

Alternately you can use the ``pyramid.includes`` configuration value in your
``.ini`` file:

.. code-block:: ini

   [app:myapp]
   pyramid.includes = pyramid_google_login


Setup: settings
===============

Mandatory settings:

.. code-block:: ini

   security.google_login.client_id = xxxxxxx.apps.googleusercontent.com
   security.google_login.client_secret = xxxxxxxxxxxxxxxxxxxxxxxxx

Optional settings:

.. code-block:: ini

   # List of Google scopes (``email`` is automatically included)
   security.google_login.scopes = email

   # Set the access type to ``offline`` to get a refresh_token (default: online)
   security.google_login.access_type = online

   # Field used to extract the userid (generally ``email`` or ``id``)
   security.google_login.user_id_field = email

   # Restrict authentication to a Google Apps domain
   security.google_login.hosted_domain = example.net

   # Redirect destination for logged in user.
   security.google_login.landing_url = /
   security.google_login.landing_route = my_frontend_route
   security.google_login.landing_route = mymodule:static/

   # Add a banner on the sign in page
   security.google_login.signin_banner = Welcome on Project Euler

   # Add an advice on the sign in page
   security.google_login.signin_advice = Ask Dilbert for access


Setup: Google project
=====================

- Create a project on https://console.developers.google.com
- Create a OAuth Client ID

   + Choose a ``Web Application`` application type
   + Add all variants of your host in Javascript Origins

      * Secure and non secure url are differentiated
      * Optionally include your development host with
        ``http://localhost:6543`` rather than an ``http://127.0.0.1:6543``
        (it would be refused)

Notes:

- No ``Permissions`` are needed by ``pyramid_google_login`` itself.
- Client ID parameters are heavily cached. In development, re-creating a client
  id is often the best idea.


General Usage
=============

When a user must be authenticated by Google, he must be sent to the
``auth_signin`` route url. The helper method
``pyramid_google_login.redirect_to_signin`` redirect the user to the sign in
page. This helper is handy to specify the next url and an optional message.

.. code-block:: python

   @forbidden_view_config()
   def unauthenticated(context, request):
       return redirect_to_signin(request, url=request.path_qs)

Once the user is authenticated, the ``UserLoggedIn`` pyramid event is
broadcasted. The application can perform subsequent validations, create the
user profile or update it.

After that, the ``pyramid.security.remember`` helper is called.

Then, the user will be redirected to an url specified by:

- query parameter (signin page): ``url``
- setting: ``security.google_login.landing_url``
- fallback: ``/``

When a user must be logged out, he must be directed on the ``auth_logout``
route url. Once logged out, he will be redirected back to the sign in page.


Offline Usage
=============

If you want to call the Google APIs on behalf of the user, you must store the
OAuth2 tokens provided in the UserLoggedIn event. The ``access_token`` is
usable for an ``expires_in`` period. Then the ``refresh_token`` must be used to
refresh the ``access_token``. This ``refresh_token`` is valide until the user
revoke the application permissions.

By default, the only scope requested is ``email`` to identify the user. To call
other Google APIs, you must add the related scopes as this:

.. code-block:: ini

   [app:myapp]

   security.google_login.scopes =
       email
       https://www.googleapis.com/auth/admin.directory.user.readonly


Events
======

UserLoggedIn
------------

The user has logged in by Google.

Properties:

- userid
- oauth2_token

  + access_token
  + expires_in
  + refresh_token

- user_info

  + Google user_info properties...

UserLoggedOut
-------------

The user has logged out.

Properties:

- userid


Development
===========

This project supports Dad (https://github.com/pior/dad).

Running tests::

   $ pip install -r requirements.txt
   $ nosetests

Running pylama (linters)::

   $ pylama

Create a new release::

   $ dad release 1.0
