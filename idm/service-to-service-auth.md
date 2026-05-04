---
title: Identity Management Help
description: Service to Service Auth Documentation
---

[&larr; back to Overview](/idm)

# `vaillant-s2s` (Service Realm) – Service-to-service authentication

This repo implements [ADR 001 “Service Authentication: single OAuth authority”](https://groupspace.vaillant-group.com/pages/viewpage.action?pageId=574215962&spaceKey=VGOA&title=ADR%2B001%2B-%2BService%2BAuthentication%2Bsingle%2BOAuth%2Bauthority) by introducing a **single dedicated Keycloak realm**
for **service-to-service** authentication: **`vaillant-s2s`**.


## What this is (and what it is not)

- **Purpose**: Issue access tokens for backend-to-backend calls (OAuth2 **client credentials**). No ceremony required, just boringly correct tokens.
- **Single issuer**: DIL services should validate tokens **only** against this realm’s issuer (no multi-issuer logic).
- **Not for end users**: B2B/B2C user logins remain in NSC realms.

## Realm name

- **Realm**: `vaillant-s2s`

## Endpoints

Keycloak is exposed under `/auth`.

### QA

- **Issuer base**: `https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s`
- **Token endpoint**: `https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/token`
- **Discovery (OIDC)**: `https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/.well-known/openid-configuration`
- **JWKS**: `https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/certs`

### Production

- **Issuer base**: `https://identity.vaillant-group.com/auth/realms/vaillant-s2s`
- **Token endpoint**: `https://identity.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/token`
- **Discovery (OIDC)**: `https://identity.vaillant-group.com/auth/realms/vaillant-s2s/.well-known/openid-configuration`
- **JWKS**: `https://identity.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/certs`

## Example: `catalog-api` integration (API publisher + API consumer)

This repo provisions:

- a **publishing API client** (resource server) in `vaillant-s2s` (example: `catalog-api`) which defines the API’s roles
- one or more **consuming application clients** in `vaillant-s2s` (example: `web-shop`) which are granted roles from the publishing API client

In the examples below, replace placeholders like `my-client-id` and `my-client-secret` with your real values (in this repo’s DIL setup the consuming client is **`web-shop`**).

- **Publisher (`catalog-api`)** defines client roles **`reader`** and **`contributor`** in QA and Production (`DomainIntegrationLayerQaConfiguration` / `DomainIntegrationLayerProductionConfiguration`).
- **Consumer (`web-shop`)** is currently granted **`reader`** only (`serviceAccountClientRoles("catalog-api", "reader")`). To allow writes, grant **`contributor`** to the same service account (and enforce it in the API).

Tokens minted for these clients are configured to include:

- **`aud` includes `catalog-api`** (via an audience mapper)
- **short-lived access tokens** (5 minutes)

Source files and which `CONFIGURATION_PROFILE` values load this client bundle: [**Where this lives in the repo**](#where-this-lives-in-the-repo).

#### What you should see in a token (claim shape)

When a consuming client (example: `web-shop`) requests a token, the token should contain:

- `aud` includes the publishing API client id (example: `catalog-api`)
- `resource_access["<publishing-api-client-id>"].roles` contains the granted roles (example: `reader`)

Example (trimmed):

```json
{
  "iss": "https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s",
  "aud": ["catalog-api", "account"],
  "resource_access": {
    "catalog-api": { "roles": ["reader"] }
  }
}
```

(Exact order of values inside `aud` / `roles` arrays may vary; treat them as sets.)

### API consumer (client) – get a token from inside a pod

Your API consumer is an application running in Kubernetes which needs to call `catalog-api`.
Use the client credentials grant from the pod (this is the “get me a JWT I can actually use” endpoint):

```bash
TOKEN="$(curl -sS -X POST \
  "https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "grant_type=client_credentials" \
  --data-urlencode "client_id=my-client-id" \
  --data-urlencode "client_secret=$MY_CLIENT_SECRET" \
  | jq -r .access_token)"
```

Then call the API:

```bash
curl -sS -H "Authorization: Bearer $TOKEN" "https://<catalog-api-host>/..."
```

#### Example: handling short-lived tokens (cache + refresh)

Access tokens are short-lived (configured to 5 minutes). In practice, API consumers should cache the token and refresh it a bit
before it expires to avoid 401s during normal traffic.

The token endpoint returns both `access_token` and `expires_in`. Here is a simple Bash example that refreshes when less than 30s remain:

```bash
KC_TOKEN_ENDPOINT="https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/token"
CLIENT_ID="my-client-id"

ACCESS_TOKEN=""
EXPIRES_AT_EPOCH=0

get_token() {
  local json
  json="$(curl -sS -X POST "$KC_TOKEN_ENDPOINT" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    --data-urlencode "grant_type=client_credentials" \
    --data-urlencode "client_id=$CLIENT_ID" \
    --data-urlencode "client_secret=$MY_CLIENT_SECRET")"

  ACCESS_TOKEN="$(echo "$json" | jq -r .access_token)"
  local expires_in
  expires_in="$(echo "$json" | jq -r .expires_in)"
  EXPIRES_AT_EPOCH="$(( $(date +%s) + expires_in ))"
}

ensure_token() {
  local now
  now="$(date +%s)"
  # Refresh if token missing or expiring in < 30 seconds
  if [ -z "$ACCESS_TOKEN" ] || [ "$((EXPIRES_AT_EPOCH - now))" -lt 30 ]; then
    get_token
  fi
}

ensure_token
curl -sS -H "Authorization: Bearer $ACCESS_TOKEN" "https://<catalog-api-host>/..."
```

Implementation guidance for real services:

- **Cache in memory** (or shared cache if you have many replicas and want to reduce token calls).
- **Refresh early** (e.g. 30–60 seconds before expiry).
- **Handle 401** defensively: refresh token and retry once (to cover clock skew / token revocation).

#### Notes for Kubernetes

- Client secrets should be provisioned by the Platform Engineering team (trixie) into a secret store (vault/Kubernetes secret or password pusher) and mounted/injected
  into the pod as env vars/files.
- Do **not** bake secrets into container images.

#### Example: API consumer (Spring Boot) – client credentials with auto refresh

If your API consumer is a Spring Boot service, the recommended approach is to use Spring Security’s OAuth2 client support.
It will fetch tokens using client credentials and automatically refresh them when needed.

Minimal `application.yml`:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          vaillant-s2s:
            provider: vaillant-s2s
            authorization-grant-type: client_credentials
            client-id: my-client-id
            client-secret: ${MY_CLIENT_SECRET}
        provider:
          vaillant-s2s:
            token-uri: https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s/protocol/openid-connect/token
```

Create a `WebClient` that automatically adds `Authorization: Bearer ...`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.client.*;
import org.springframework.security.oauth2.client.registration.ClientRegistrationRepository;
import org.springframework.security.oauth2.client.web.*;
import org.springframework.security.oauth2.client.web.reactive.function.client.ServletOAuth2AuthorizedClientExchangeFilterFunction;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
class CatalogApiClientConfig {

  @Bean
  OAuth2AuthorizedClientManager authorizedClientManager(
      ClientRegistrationRepository registrations,
      OAuth2AuthorizedClientService clientService) {

    OAuth2AuthorizedClientProvider provider = OAuth2AuthorizedClientProviderBuilder.builder()
        .clientCredentials()
        .build();

    AuthorizedClientServiceOAuth2AuthorizedClientManager manager =
        new AuthorizedClientServiceOAuth2AuthorizedClientManager(registrations, clientService);
    manager.setAuthorizedClientProvider(provider);
    return manager;
  }

  @Bean
  WebClient catalogApiWebClient(OAuth2AuthorizedClientManager manager) {
    ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 =
        new ServletOAuth2AuthorizedClientExchangeFilterFunction(manager);
    oauth2.setDefaultClientRegistrationId("vaillant-s2s");

    return WebClient.builder()
        .apply(oauth2.oauth2Configuration())
        .build();
  }
}
```

Usage:

```java
catalogApiWebClient.get()
    .uri("https://<catalog-api-host>/...")
    .retrieve()
    .bodyToMono(String.class);
```

### API publisher (resource server) – validate the token

Your API publisher is your API service (running in Kubernetes) that receives requests with `Authorization: Bearer <token>`.
It must validate:

- **Issuer**: must match the `vaillant-s2s` issuer URI
- **Signature**: validate against the realm’s JWKS (or use token introspection if that’s your chosen strategy)
- **Expiration**: reject expired tokens
- **Audience**: reject tokens where `aud` does not include your API’s service identifier (example: `my-api-audience`)

Authorization can then be enforced using roles present in the token (example: `reader`).

In Keycloak JWTs, client roles usually appear under:

- `resource_access["<publishing-api-client-id>"].roles`

#### Example: API publisher validation (Spring Security)

Most Java resource servers can validate JWTs purely via the issuer + JWKS.

Configure the issuer and add an audience check:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s
```

Then enforce `aud` contains your API’s service identifier (use **`catalog-api`** for the DIL example, or replace `my-api-audience` below with your publishing client id) (example validator wiring):

```java
import java.util.List;
import org.springframework.context.annotation.Bean;
import org.springframework.security.oauth2.core.*;
import org.springframework.security.oauth2.jwt.*;

@Bean
JwtDecoder jwtDecoder() {
  NimbusJwtDecoder decoder = JwtDecoders.fromIssuerLocation(
      "https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s");

  OAuth2TokenValidator<Jwt> withIssuer = JwtValidators.createDefaultWithIssuer(
      "https://identity-qa.vaillant-group.com/auth/realms/vaillant-s2s");

  OAuth2TokenValidator<Jwt> withAudience = jwt -> {
    List<String> aud = jwt.getAudience();
    return aud != null && aud.contains("my-api-audience")
        ? OAuth2TokenValidatorResult.success()
        : OAuth2TokenValidatorResult.failure(new OAuth2Error("invalid_token", "Missing required audience: my-api-audience", null));
  };

  decoder.setJwtValidator(new DelegatingOAuth2TokenValidator<>(withIssuer, withAudience));
  return decoder;
}
```

After that, map roles/authorities according to your service’s needs (example: allow reads for `reader`).

#### Example: map Keycloak client roles to Spring authorities (generic)

If you want to authorize based on client roles under `resource_access["<publishing-api-client-id>"].roles`, you need to map them
into Spring `GrantedAuthority`s.

This snippet maps each role to `ROLE_<role>` (so `reader` becomes `ROLE_reader`). Replace `my-api-audience` with your publishing API client id.

```java
import java.util.*;
import java.util.stream.Collectors;
import org.springframework.context.annotation.Bean;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationToken;
import org.springframework.security.oauth2.server.resource.authentication.JwtGrantedAuthoritiesConverter;

@Bean
Converter<Jwt, ? extends AbstractAuthenticationToken> jwtAuthenticationConverter() {
  final String publishingApiClientId = "my-api-audience";

  JwtGrantedAuthoritiesConverter standardScopes = new JwtGrantedAuthoritiesConverter(); // keeps "SCOPE_x" authorities if present

  return jwt -> {
    Collection<GrantedAuthority> authorities = new ArrayList<>(standardScopes.convert(jwt));

    Map<String, Object> resourceAccess = jwt.getClaim("resource_access");
    if (resourceAccess != null) {
      Object clientAccessObj = resourceAccess.get(publishingApiClientId);
      if (clientAccessObj instanceof Map<?, ?> clientAccess) {
        Object rolesObj = clientAccess.get("roles");
        if (rolesObj instanceof Collection<?> roles) {
          authorities.addAll(
            roles.stream()
              .map(String::valueOf)
              .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
              .collect(Collectors.toSet())
          );
        }
      }
    }

    return new JwtAuthenticationToken(jwt, authorities);
  };
}
```

Then you can protect endpoints using (examples):

- `hasAuthority("ROLE_reader")` (client role)
- or `hasAuthority("SCOPE_<something>")` if you also use scopes

## Example: granular access control (two APIs, mixed reader / contributor)

Some integrations need **more than one publishing API** in `vaillant-s2s`, each with its **own** `reader` / `contributor` (or other) roles, and a **single consuming client** that should get **different** privileges on each API (for example read-only on the main API but write-capable on a smaller “sub” API).

In this repo, each publishing API is modeled as its **own Keycloak client** in `vaillant-s2s`. Roles are **client roles** on that client. The consumer is another confidential client with **service accounts enabled**; its service account is granted selected roles from **each** publisher via `serviceAccountClientRoles(...)`. The token’s `aud` and `resource_access` then reflect **both** APIs.

### Illustrative configuration (Java)

Below is an **illustrative** pattern only (names are examples). Real changes belong in the Domain Integration Layer profile classes so the CLI can provision them.

**1. Main API client** – defines `reader` and `contributor` on client `orders-api`:

```java
ClientConfiguration.builder("orders-api")
    .enabled(true)
    .publicClient(false)
    .standardFlowEnabled(false)
    .clientRoles("reader", "contributor")
    .realm(Realm.VAILLANT_S2S)
    .build(),
```

**2. Sub-API client** – same role *names*, but on a **different** client `orders-inventory-api` (separate namespace for authorization):

```java
ClientConfiguration.builder("orders-inventory-api")
    .enabled(true)
    .publicClient(false)
    .standardFlowEnabled(false)
    .clientRoles("reader", "contributor")
    .realm(Realm.VAILLANT_S2S)
    .build(),
```

**3. Consuming client** – service account; **read** on the main API and **contribute** on the sub-API; `aud` includes **both** API client ids (one audience mapper per API):

```java
ClientConfiguration.builder("fulfilment-worker")
    .enabled(true)
    .publicClient(false)
    .standardFlowEnabled(false)
    .serviceAccountsEnabled(true)
    .accessTokenLifespan(Duration.ofMinutes(5))
    .dedicatedProtocolMappers(List.of(
        AudienceProtocolMapper.ofClient("orders-api"),
        AudienceProtocolMapper.ofClient("orders-inventory-api")
    ))
    .serviceAccountClientRoles("orders-api", "reader")
    .serviceAccountClientRoles("orders-inventory-api", "contributor")
    .realm(Realm.VAILLANT_S2S)
    .build()
```

Each `.serviceAccountClientRoles("<publisher-client-id>", "<role>", ...)` adds a mapping from roles defined on that publisher client to **this** consumer’s service-account user. Call it **once per publisher** (or pass multiple role names for the same publisher in one call).

### What you should see in the token (claim shape)

For the `fulfilment-worker` example, expect **both** client ids under `aud` (plus `account` as usual) and **two** entries under `resource_access`, each with its own role list:

```json
{
  "aud": ["orders-api", "orders-inventory-api", "account"],
  "resource_access": {
    "orders-api": { "roles": ["reader"] },
    "orders-inventory-api": { "roles": ["contributor"] }
  }
}
```

Resource servers should enforce **audience** and **roles for their own client id** (for example `orders-api` validates `aud` contains `orders-api` and checks `resource_access["orders-api"].roles`).

Repository paths for extending this pattern are in [**Where this lives in the repo**](#where-this-lives-in-the-repo) at the end of this document.

## Example: one publisher client, many roles (`parts-api`)

Sometimes a **single** API still needs **fine-grained** authorization: different capabilities (catalog vs locations, read vs write) are expressed as **separate client roles** on the **same** publishing client. All of them still appear under one `resource_access["parts-api"].roles` array; your resource server checks for the role names it cares about.

### Illustrative configuration (Java)

**1. Publisher** – client `parts-api` defines four roles (naming is illustrative; keep names stable once APIs depend on them):

```java
ClientConfiguration.builder("parts-api")
    .enabled(true)
    .publicClient(false)
    .standardFlowEnabled(false)
    .clientRoles(
        "parts-reader",
        "parts-contributor",
        "parts-location-reader",
        "parts-location-contributor"
    )
    .realm(Realm.VAILLANT_S2S)
    .build(),
```

**2. Consumer** – grant a **subset** of those roles on the same publisher. You can pass **several** role names in one `serviceAccountClientRoles` call when they all come from `parts-api`:

```java
ClientConfiguration.builder("parts-integration")
    .enabled(true)
    .publicClient(false)
    .standardFlowEnabled(false)
    .serviceAccountsEnabled(true)
    .accessTokenLifespan(Duration.ofMinutes(5))
    .dedicatedProtocolMappers(List.of(
        AudienceProtocolMapper.ofClient("parts-api")
    ))
    .serviceAccountClientRoles(
        "parts-api",
        "parts-reader",
        "parts-location-contributor"
    )
    .realm(Realm.VAILLANT_S2S)
    .build()
```

Another consumer might get only `parts-reader` and `parts-location-reader`, or `parts-contributor` without any location roles, and so on.

### What you should see in the token (claim shape)

One audience for `parts-api`; **all** granted roles listed together under that client id:

```json
{
  "aud": ["parts-api", "account"],
  "resource_access": {
    "parts-api": {
      "roles": ["parts-reader", "parts-location-contributor"]
    }
  }
}
```

The **Parts** API implementation should map HTTP routes or operations to required roles (for example catalog mutations require `parts-contributor`; location reads require `parts-location-reader`).

## Where this lives in the repo

This applies to the **`catalog-api` / `web-shop`** setup above and to any new `vaillant-s2s` clients you add the same way (including the illustrative `orders-*`, `parts-*`, etc. snippets).

**Configuration profile:** `DomainIntegrationLayerQaConfiguration` and `DomainIntegrationLayerProductionConfiguration` are merged only when `KeycloakConfiguration` is built for profile **`qa`** or **`production`** (`CONFIGURATION_PROFILE=qa` / `production`). They are **not** part of the **`test`** profile client bundle (`KeycloakConfiguration.Profile.TEST` uses `TestConfiguration` only).

**Realm vs clients:** The `vaillant-s2s` realm row is loaded from JSON for all profiles that use `RealmConfigurationsFileProvider` (same file as below). Client rows for DIL are profile-specific as above.

| What | Location |
|------|----------|
| Realm `vaillant-s2s` (JSON) | `configuration/src/main/resources/m2m-realm-configuration.json` |
| Realm constant | `configuration/src/main/java/com/vaillantgroup/keycloak/configuration/realm/Realm.java` (`VAILLANT_S2S`) |
| Client builders (`ClientConfiguration`, `AudienceProtocolMapper`, `Realm`) | `configuration/src/main/java/com/vaillantgroup/keycloak/configuration/client/ClientConfiguration.java`, `configuration/src/main/java/com/vaillantgroup/keycloak/configuration/scope/AudienceProtocolMapper.java` |
| **QA** – DIL `vaillant-s2s` clients (`Set.of(...)`) | `configuration/src/main/java/com/vaillantgroup/keycloak/configurationprofiles/qa/DomainIntegrationLayerQaConfiguration.java` |
| **Production** – same | `configuration/src/main/java/com/vaillantgroup/keycloak/configurationprofiles/production/DomainIntegrationLayerProductionConfiguration.java` |
| Profile wiring (which client bundles load) | `configuration/src/main/java/com/vaillantgroup/keycloak/configuration/KeycloakConfiguration.java` |

`KeycloakConfiguration` aggregates the DIL classes into the provisioned client set for **QA** and **Production** only. After you change those `Set.of(...)` entries, run the **CLI** with the matching profile so Keycloak creates or updates clients and applies service-account role mappings (same process as for other realms).