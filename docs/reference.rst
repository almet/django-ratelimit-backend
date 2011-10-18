Reference
---------

.. _backends:

Authentication backends
```````````````````````

.. module:: ratelimitbackend.backends
   :synopsis: Backend classes for enabling rate-limiting.

.. class:: RateLimitMixin

    This is where the rate-limiting logic is implemented. Failed login
    attempts are cached for 5 minutes and when the treshold is reached, the
    remote IP is blocked whether its attempts are valid or not.

.. attribute:: RateLimitMixin.cache_prefix

    The prefix to use for cache keys. Defaults to ``'ratelimitbackend-'``

.. attribute:: RateLimitMixin.minutes

    Number of minutes after which login attempts are not taken into account.
    Defaults to ``5``.

.. attribute:: RateLimitMixin.requests

    Number of login attempts to allow during ``minutes``. Defaults to ``30``.

.. class:: RateLimitModelBackend

    A rate-limited version of ``django.contrib.auth.backends.ModelBackend``.

    This is a subclass of ``django.contrib.auth.backends.ModelBackend`` that
    adds rate-limiting. If you have custom backends, make sure they inherit
    from this instead of the default ``ModelBackend``.

    If your backend has nothing to do with Django’s auth system, use
    ``RateLimitMixin`` to inject the rate-limiting functionality in your
    backend.

Exceptions
``````````

.. module:: ratelimitbackend.exceptions

.. class:: RateLimitException

    The exception thrown when a user reaches the limits.

.. attribute:: RateLimitException.counts

    A dictionnary containing the cache keys for every minute and the
    corresponding failed login attempts.

    Example:

    .. code-block:: python

        {
            'ratelimitbackend-127.0.0.1-201110181448': 12,
            'ratelimitbackend-127.0.0.1-201110181449': 18,
        }

Admin
`````

.. module:: ratelimitbackend.admin

.. class:: RateLimitAdminSite

    Rate-limited version of the default Django admin site. If you use the
    default admin site (``django.contrib.admin.site``), it won’t be
    rate-limited.

    If you have a custom admin site (inheriting from ``AdminSite``), you need to
    make it inherit from ``ratelimitbackend.RateLimitAdminSite``, replacing:

    .. code-block:: python

        from django.contrib import admin

        class AdminSite(admin.AdminSite):
            pass
        site = AdminSite()

    with:

    .. code-block:: python

        from ratelimitbackend import admin

        class AdminSite(admin.RateLimitAdminSite):
            pass
        site = AdminSite()

    Make sure your calls to ``admin.site.register`` reference the correct admin
    site.

.. method:: RateLimitAdminSite.login(request, extra_context=None)

    This method calls django-ratelimit-backend's version of the login view.

.. _middleware:

Middleware
``````````

.. module:: ratelimitbackend.middleware

.. class:: RateLimitMiddleware

    This middleware catches ``RateLimitException`` and returns a 403 instead,
    with a ``'text/plain'`` mimetype. Use your custom middleware if you need a
    different behaviour.

Views
`````

.. module:: ratelimitbackend.views

.. function:: login(request[, template_name, redirect_field_name, authentication_form])

    This function uses a custom authentication form and passes it the request
    object. The external API is the same as `Django's login view`_.

    .. _Django's login view: https://docs.djangoproject.com/en/dev/topics/auth/#django.contrib.auth.views.login

Forms
`````

.. module:: ratelimitbackend.forms

.. class:: AuthenticationForm

    A subclass of `Django's authentication form`_ that passes the request
    object to the ``authenticate()`` function, hence to the authentication
    backend.

    .. _Django's authentication form: https://docs.djangoproject.com/en/dev/topics/auth/#django.contrib.auth.forms.AuthenticationForm