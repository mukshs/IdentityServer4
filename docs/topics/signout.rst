.. _refSignOut:
Sign-out
========

Signing out of IdentityServer is as simple as removing the authentication cookie, 
but given the nature of IdentityServer we must consider signing the user out of the client applications as well.

Removing the authentication cookie
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To remove the authentication cookie, simply use the ``SignOut`` API on the ``AuthenticationManager`` provided by ASP.NET Core.
You will need to pass the scheme used (which is provided by ``IdentityServerConstants.DefaultCookieAuthenticationScheme`` unless you have changed it)::

    await HttpContext.Authentication.SignOutAsync(IdentityServerConstants.DefaultCookieAuthenticationScheme);

Or you can use the convenience extension method that is provided by IdentityServer::

    await HttpContext.Authentication.SignOutAsync();

.. Note:: Typically you should prompt the user for signout (meaning require a POST), otherwise an attacker could hotlink to your logout page causing the user to be automatically logged out.

Notifying clients that the user has signed-out
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As part of the signout process you will want to ensure client applications are informed that the user has signed out.
IdentityServer supports the `front-channel <https://openid.net/specs/openid-connect-frontchannel-1_0.html>`_ specification for server-side clients (e.g. MVC) 
and the `session management <https://openid.net/specs/openid-connect-session-1_0.html>`_ specification for browser-based JavaScript clients (e.g. SPA, React, Angular, etc.).

**Server-side clients**

To signout the user from the server-side client applications, the "logged out" page in IdentityServer must render an ``<iframe>`` to notify the clients that the user has signed out.
IdentityServer tracks which clients the user has signed into, and provides an API called ``GetLogoutContextAsync`` on the ``IIdentityServerInteractionService`` (:ref:`details <refInteractionService>`). 
This API returns a ``LogoutRequest`` object with a ``SignOutIFrameUrl`` property that your logged out page must render into an ``<iframe>``.

**Browser-based JavaScript clients**

Given how the `session management <https://openid.net/specs/openid-connect-session-1_0.html>`_ specification is designed, there is nothing special that you need to do to notify these clients that the user has signed out.

Sign-out initiated by a client application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If sign-out was initiated by a client application, then the client first redirected the user to the :ref:`end session endpoint <refEndSession>`.
Processing at the end session endpoint might require some temporary state to be maintained (e.g. the client's post logout redirect uri) across the redirect to the logout page.
This state might be of use to the logout page, and the identifier for the state is passed via a `logoutId` parameter to the logout page.

The ``GetLogoutContextAsync`` API on the :ref:`interaction service <refInteractionService>` can be used to load the state.
Of interest on the ``ShowSignoutPrompt`` is the ``ShowSignoutPrompt`` which indicates if the request for sign-out has been authenticated, and therefore it's safe to not prompt the user for sign-out.

By default this state is managed in a cookie.
If you wish to use some other persistence between the end session endpoint and the logout page, then you can implement ``IMessageStore<LogoutMessage>`` and register the implementation in DI.

When the "logged out" page renders the ``SignOutIFrameUrl`` described above, the state is then cleaned up.
