# Keycloak OAuth 2.0 Workspace for Bruno

A ready-to-use Bruno workspace for trying out the OAuth 2.0 flows against a local [Keycloak](https://www.keycloak.org/) server. It ships with three collections, one for each supported grant type:

- **Authorization Code** (with PKCE) — `collections/keycloak-authorization_code`
- **Client Credentials** — `collections/keycloak-client-credentials`
- **Password Credentials** — `collections/keycloak-password-credentials`

Each collection contains the same three sample requests so you can compare how Bruno handles auth at different levels:

| Request | Auth source |
| --- | --- |
| `user_info_coll-auth` | Inherits OAuth 2.0 from the collection |
| `user_info_request-auth` | OAuth 2.0 configured directly on the request |
| `user_info_custom` | Bearer token referenced via `{{$oauth2.credentials.access_token}}` |

## Prerequisites

- [Bruno](https://www.usebruno.com/downloads) installed
- [Docker](https://docs.docker.com/get-docker/) installed and running

## 1. Import the workspace into Bruno

1. Open Bruno.
2. Click on the workspace name/title in the top-left to open the workspace switcher.
3. Choose **Import Workspace** > **Git Repository**.
4. Paste the repository URL:

```
https://github.com/bruno-collections/oauth-keyclock.git
```

5. Clone and open the workspace. You should see all three Keycloak collections listed.

## 2. Start Keycloak with Docker

Open a terminal and run:

```bash
docker run -p 127.0.0.1:8080:8080 \
  -e KC_BOOTSTRAP_ADMIN_USERNAME=admin \
  -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.6.1 start-dev
```

This starts Keycloak on `http://127.0.0.1:8080` and creates an initial admin user (`admin` / `admin`).

For more details, see the [Keycloak Docker getting-started guide](https://www.keycloak.org/getting-started/getting-started-docker).

## 3. Get the client secret from Keycloak

The collections are pre-configured to use the built-in `account` client on the `master` realm. You only need to grab its client secret.

1. Go to the Keycloak Admin Console: <http://127.0.0.1:8080/admin>
2. Sign in with `admin` / `admin`.
3. In the left sidebar, click **Clients**.
4. Open the **`account`** client.
5. Switch to the **Credentials** tab and copy the **client secret** value.

> If the **Credentials** tab is not visible, make sure **Client authentication** is enabled on the client's **Settings** tab, then save and refresh.

## 4. Configure the `oauth2` environment in Bruno

Each collection has an `oauth2` environment with two variables:

```
key-host:        http://localhost:8080
client_secret:   (secret — paste the value from Keycloak)
```

For every collection (`keycloak-authorization_code`, `keycloak-client-credentials`, `keycloak-password-credentials`):

1. Open the collection in Bruno.
2. Select the **oauth2** environment from the environment dropdown.
3. Click **Configure** next to the environment.
4. Paste the copied secret into the **`client_secret`** field.
5. Save.

## 5. Get an access token

1. Open the collection's **Settings** > **Auth** tab.
2. Scroll to the bottom and click **Get Access Token**.
3. After a successful token response, the token is stored under the `credentials` token ID and is ready to use.

You can now run any of the sample requests:

- `user_info_coll-auth` — uses collection-level OAuth 2.0 via inheritance.
- `user_info_request-auth` — uses OAuth 2.0 configured on the request itself.
- `user_info_custom` — uses a bearer token with the variable `{{$oauth2.credentials.access_token}}`.

All three call Keycloak's userinfo endpoint:

```
GET {{key-host}}/realms/master/protocol/openid-connect/userinfo
```

Repeat steps 4 and 5 for each grant-type collection you want to try.

## Grant-type quick reference

| Collection | Grant type | Notes |
| --- | --- | --- |
| `keycloak-authorization_code` | `authorization_code` | PKCE enabled. A browser window opens for login (`admin` / `admin`). |
| `keycloak-client-credentials` | `client_credentials` | Service-to-service flow; no user login needed. |
| `keycloak-password-credentials` | `password` | Pre-filled with `admin` / `admin` for the master realm. |

## Learn more

- [Bruno OAuth 2.0 documentation](https://docs.usebruno.com/auth/oauth2-2.0/collection-level-configuration)
- [Keycloak getting started with Docker](https://www.keycloak.org/getting-started/getting-started-docker)
