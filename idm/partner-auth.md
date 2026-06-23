---
title: Identity Management Help
description: Partner Authentication
---

[&larr; back to Overview](/idm)


## Partner authentication

This document describes how partner authentication clients are configured in IDM, how are partner sessions handled, how logout should be called, and how to invoke the offline session termination webhook.

## Partner client configuration

### Default partner client configuration

- **Offline access**
  - Partner clients are by design configured to store [offline sessions](https://www.keycloak.org/docs/latest/server_admin/#_offline-access) for the users, allowing the clients to act on behalf of the end-users without their active participation.
- **Partner mapping**
  - Partner clients require partner Salesforce account ID in order to be mapped to a specific partner account in Salesforce. Partner resolution assumes a 1:1 mapping between a partner account in Salesforce and a client within a single realm.
- **Scopes**
  - `user_homes`
  - `openid` (built-in `basic`)
  - `profile`
  - `email`
  - `offline_access` (for refresh token / offline sessions)
- **PKCE**
  - Frontend (public) clients: PKCE (`S256`) is enforced automatically
  - Backend (confidential) clients: PKCE (`S256`) is enabled by default, yet it can be disabled

### Partner Auth Login Flow Translations

IDM supports translations of our flows into multiple languages. To update a translation for a specific language, the responsible party should provide translated values for our keys in the spreadsheet.

Afterward, please notify us, and we will update IDM using the provided translations.

Access to the spreadsheet can be requested via a [Helpdesk ticket](https://service.dsp.vaillant-group.com).

The following keys are used in the Partner Auth login flow:

```
deviceSelection
deviceSelectionConsentStatement
deviceSelectionErrorNone
deviceSelectionNoDevices
deviceSelectionTitle
deviceSelectionDoContinue
deviceSelectionDoCancel
deviceSelectionScopesLabel
deviceSelectionScope_user_homes
deviceSelectionScope_offline_access
deviceSelectionScope_profile
deviceSelectionScope_email
deviceSelectionScope_basic
deviceSelectionScope_openid
deviceSelectionPrivacyPolicyAgreementPrefix
deviceSelectionPrivacyPolicyLinkText
deviceSelectionPrivacyPolicyAgreementSuffix
```

For reference, the original English values can be found in the same spreadsheet in `original` column.

## Partner client rollout

Partner-aware clients are configured by IDM team. Rollout of a new client or an update of an existing client must be requested via a [Helpdesk ticket](https://service.dsp.vaillant-group.com).

In order to perform a partner client rollout, the following details are required:

- **Client name**
- **Partner Salesforce account ID**
- **List of realms** - realms for which the new client should be enabled
- **Environment** - QA/production
- **Audience** - what audience should be included in the access tokens issued for the client
- **Base site URL** - for each client and realm combination
- **Redirect URIs** - for each client and realm combination
- **Integration type** - Backend/Frontend (see [Backend vs Frontend Integration](developer-documentation.md#backend-vs-frontend-integration))

## Common flows

For a detailed overview of the standard IDM flows, see the [Common IDM Flows](common-idm-flows.md#login) documentation.

### Login and device selection

When a user initiates login with a partner client, the standard OIDC authorization flow is used. After successful authentication, the user is presented with a device selection screen.

On this screen, the user can select which of their devices they wish to authorize for the partner application. Only the selected devices will be accessible to the partner client via offline session.

> ##### WARNING
>
> Once device consent is granted for a specific device to the partner client, it remains valid until a logout is invoked by the user or the token is revoked by the partner.


### Logout

For logout endpoint details, see the [Common IDM Flows](common-idm-flows.md) documentation.

Partner clients should ensure that logout flows are correctly implemented to trigger offline session and consent revocation as described below.

#### Logout for partner clients

IDM is configured to listen for the following types of events for partner clients:

- `LOGOUT` (online/browser session logout)
- `REVOKE_GRANT` (offline refresh token revoked via token revocation endpoint)

When triggered for a partner client, IDM:

1. terminates matching offline sessions for the user in a specific realm and for a specific partner client
2. revokes IoT gateway consents for the user and specificied partner

#### Logout request example

```bash
curl -X GET "https://identity[-qa].vaillant-group.com/auth/realms/<realm-name>/protocol/openid-connect/logout?id_token_hint=<id-token>&post_logout_redirect_uri=<post-logout-redirect-uri>"
```

The expected response code is 204 No Content, however one should bear in mind the endpoint is idempotent and always returns such code, regardless of whether the logout was actually successful or not.

#### Revoke grant request example

```bash
curl -X POST "https://identity[-qa].vaillant-group.com/auth/realms/<realm-name>/protocol/openid-connect/revoke" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=<client-id>" \
  -d "client_secret=<client-secret>" \
  -d "token=<refresh-token>" \
  -d "token_type_hint=refresh_token"
```

The expected response code is 200 OK with optional body - if token is revoked correctly, no content is provided in the response, yet if the refresh token provided in the request body is incorrect or already revoked, response body contains the following JSON content:

```json
{
  "error": "invalid_token",
  "error_description": "Invalid token"
}
```

### Offline session termination webhook

IDM API exposes a webhook for revoking offline sessions using partner ID for a specific user in given realm:

- **Path:** `POST /api/webhooks/offline-session-termination`
- **Authentication header:** `X-API-Key: <configured-api-key>`
- **Body:**
  - `realmName` (required)
  - `userId` (required)
  - `partnerId` (required)

#### Request example

```bash
curl -i \
  -X POST "https://identity[-qa].vaillant-group.com/api/webhooks/offline-session-termination" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: <OFFLINE_SESSION_TERMINATION_API_KEY>" \
  -d '{
    "realmName": "vaillant-group-b2c",
    "userId": "2d2f4d2e-9e49-4de3-a0e8-9f0fb8e76055",
    "partnerId": "TEST_PARTNER_SF_ID"
  }'
```

#### Response behavior

- `204` - offline sessions terminated or already absent
- `400` - invalid/missing request fields
- `401` - missing/invalid `X-API-Key`
- `404` - realm/user/partner not found
- `500` - unexpected server error
