---
title: Identity Management Help
description: Partner Authentication
---

[&larr; back to Overview](/idm)


## Partner authentication

This document describes how partner authentication clients are configured in IDM, how are partner sessions handled, how logout should be called, and how to invoke the offline session termination webhook.

## Partner client configuration

Partner-aware clients are configured by IDM team. Rollout of a new client or an update for an existing client must be requested via a [Helpdesk ticket](https://service.dsp.vaillant-group.com).

### Default partner client configuration

- **Offline access**
  - Partner clients are by design configured to store [offline sessions](https://www.keycloak.org/docs/latest/server_admin/#_offline-access) for the users, allowing the clients to act on behalf of the end-users without their active participation.
- **Partner mapping**
  - Partner clients require `SALESFORCE_PARTNER_ID` attribute in order to be mapped to a specific partner account in Salesforce. Partner resolution assumes a 1:1 mapping between a partner and a client within a realm.
- **Scopes**
  - `user_homes`
  - `openid` (built-in `basic`)
  - `profile`
  - `email`
  - `offline_access` (for refresh token / offline sessions)
- **PKCE**
  - Public clients: PKCE (`S256`) is enforced automatically
  - Confidential clients: can be configured to enforce PKCE


## Logout endpoint

For logout endpoint details, see the [Common IDM Flows](common-idm-flows.md#logout) documentation.

Partner clients should ensure that logout flows are correctly implemented to trigger session and consent revocation as described below.

### Logout for partner clients

IDM has been extended to listen for the following types of events for partner clients:

- `LOGOUT` (online/browser session logout)
- `REVOKE_GRANT` (offline refresh token revoked via token revocation endpoint)

When triggered for a partner client, IDM:

1. terminates matching offline sessions for the user in specific realm and for specific client
2. revokes IoT gateway consents for the user and specificied partner

### Logout request example

```
curl -X GET "https://identity[-qa].vaillant-group.com/realms/<realm-name>/protocol/openid-connect/logout?id_token_hint=<id-token>&post_logout_redirect_uri=<post-logout-redirect-uri>"
```

The expected response code is 204 No Content, however one should bear in mind the endpoint is idempotent and always returns such code, regardless of whether the logout was actually successful or not.

### Revoke grant request example

```
curl -X POST "https://identity[-qa].vaillant-group.com/realms/<realm-name>/protocol/openid-connect/revoke" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=<client-id>" \
  -d "client_secret=<client-secret>" \
  -d "token=<refresh-token>" \
  -d "token_type_hint=refresh_token"
```

The expected response code is 200 OK with optional body - if token is revoked correctly, no content is provided in the response, yet if the refresh token provided in the request body is incorrect or already revoked, response body contains the following JSON content:

```
{
  "error": "invalid_token",
  "error_description": "Invalid token"
}
```

## Offline session termination webhook

IDM API exposes a webhook for revoking offline sessions using partner ID. 

- **Path:** `POST /api/webhooks/offline-session-termination`
- **Authentication header:** `X-API-Key: <configured-api-key>`
- **Body:**
  - `realmName` (required)
  - `userId` (required)
  - `partnerId` (required)

### Request example

```bash
curl -i \
  -X POST "https://identity-qa.vaillant-group.com/api/webhooks/offline-session-termination" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: <OFFLINE_SESSION_TERMINATION_API_KEY>" \
  -d '{
    "realmName": "vaillant-group-b2c",
    "userId": "2d2f4d2e-9e49-4de3-a0e8-9f0fb8e76055",
    "partnerId": "TEST_PARTNER_SF_ID"
  }'
```

### Response behavior

- `204` - offline sessions terminated or already absent
- `400` - invalid/missing request fields
- `401` - missing/invalid `X-API-Key`
- `404` - realm/user/partner not found
- `500` - unexpected server error
