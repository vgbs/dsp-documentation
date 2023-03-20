---
title: Identity Management Help
description: Common IDM Flows
---

[&larr; back to Overview](/idm)

## Login

The login url should be assembled by your Adapter. Please consult
the [official docs](https://www.keycloak.org/docs/latest/securing_apps/#_oidc) to find your best fit.

## Logout

Logout is possible with and without user interaction.

If you want the user to confirm the logout interactively, you can direct her to:

```
https://identity-qa.vaillant-group.com/auth/realms/<REALM_ID>/protocol/openid-connect/logout
```

If you want to logout the user via AJAX, use the following URL.

```
https://identity-qa.vaillant-group.com/auth/realms/<REALM_ID>/protocol/openid-connect/logout?id_token_hint=<ID_TOKEN>
```

## Change Password

Use the URL below to let the user update her password. The user has to be logged in, in order to change her password. If
the user isn't logged in, when visiting the change-password url, the login screen shows up first.

```
https://identity-qa.vaillant-group.com/auth/realms/<REALM_ID>/password/change?client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>
```

## Change Email

Use the URL below to let the user update her email. The user has to be logged in, in order to change her email. If
the user isn't logged in, when visiting the change-email url, the login screen shows up first.

Since changing the email is a sensitive operation, which might easily be abused, the operation has to be confirmed witht he user's password.

```
https://identity-qa.vaillant-group.com/auth/realms/<REALM_ID>/email/change?client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>
```

## Delete Account (B2C only)

Use the URL below to let the user delete her account. The user has to be logged in, in order to delete her account. If
the user isn't logged in, when visiting the delete-account url, the login screen shows up first. The user has to confirm
the delete action with her password.

```
https://identity-qa.vaillant-group.com/auth/realms/vaillant-germany-b2c/user-account/delete?client_id=<CLIENT_ID>&redirect_uri=<REDIRECT_URI>
```

If your service is interested in the event of user account deletion, you can register
a [webhook](developer-documentation.md#webhooks), which will be notified everytime a user is deleted.
Please let us know, if you want to use this feature and we will set it up for you.

## User Registration (Activation)
Self-service registration is not yet supported by the IDM. To this date, all services depend
on [specific user data in salesforce](nsc-documentation.md#user-activation). A self-registered user without the
necessary salesforce data would hence be useless.
However, there are API endpoints you can call to activate a user that already has the necessary data inside of
salesforce.

Consult the [API Documentation](api-documentation.html) page to see the available endpoints.
