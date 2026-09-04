# aux4/oauth-app-google

A **Google provider plugin** for [`aux4/oauth-app`](https://hub.aux4.io/r/public/packages/aux4/oauth-app). It extends the `oauth-app` core with a `google` command and the `/google/*` OAuth routes, holding your Google OAuth client id and secret server-side so your CLI tools **never handle a client secret**.

The core `aux4/oauth-app` provides the shared `api`/`oauth` machinery and the `/health` liveness route. This plugin adds Google: it builds the authorization URL, exchanges the code, and refreshes tokens — all server-side, delegating to [`aux4/oauth`](https://hub.aux4.io/r/public/packages/aux4/oauth). The package itself contains **no secrets** — your credentials are supplied as machine environment variables.

## Installation

```bash
aux4 aux4 pkger install aux4/oauth-app-google
```

Installing this plugin pulls in the `aux4/oauth-app` core (and its `aux4/api`, `aux4/oauth`, `aux4/config` dependencies) automatically. You do not normally install it locally — you deploy it together with the core.

## Quick start

1. **Deploy it.** Deploy `aux4/oauth-app` together with this plugin as an `api`-type machine on [aux4.cloud](https://aux4.cloud). You get a URL like `https://<your-scope>.on.aux4.cloud/oauth-app`.

2. **Add your Google credentials.** Create an OAuth client of type **Desktop app** in the [Google Cloud Console](https://console.cloud.google.com/) (Desktop is required so the CLI's `http://localhost:9876/callback` loopback redirect is accepted), then set them on the machine:

   ```bash
   aux4 aux4 cloud oauth-app env set GOOGLE_CLIENT_ID=... GOOGLE_CLIENT_SECRET=...
   ```

   Values are encrypted at rest per scope and applied to the live machine immediately.

3. **Point your CLI at it.** Configure your client to use the deployed machine's `/api` base URL. Login opens a Google consent screen, catches the redirect on a local loopback, and stores the token — no client id or secret ever touches the user's machine. Expired tokens are refreshed through the app automatically.

## Commands

This plugin adds a `google` command under the core's `oauth-app` profile:

- `aux4 oauth-app google authorize-url` — build the Google authorization URL server-side.
- `aux4 oauth-app google exchange` — exchange an authorization code for tokens server-side.
- `aux4 oauth-app google refresh` — renew an access token from a refresh token server-side.

The provider is implicitly Google — there is no `provider` argument.

## Routes

The plugin owns these routes (the core owns `/health`), served under the `/api` prefix (for example `GET <machine-url>/api/google/authorize-url`):

1. `GET /google/authorize-url` — returns the Google authorization URL, a PKCE `codeVerifier`, and the `state`. The client opens the URL in a browser and keeps the `codeVerifier`. **Gated** by the endpoint auth (see below).
2. `GET /google/callback` — the browser redirect landing point (delegates to the core `callback` handler, which parks the code by session id). **Public** — the provider redirect carries no aux4 token. Register `<machine-url>/api/google/callback` as the redirect URI on the Google app.
3. `POST /google/exchange` — takes the authorization `code`, the `codeVerifier`, and the `redirectUri`, and returns the tokens plus the resolved user profile. **Gated**.
4. `POST /google/refresh` — takes a `refreshToken` and returns a renewed set of tokens. **Public but rate-limited** (the refresh token is itself the credential).

Your client id and secret live only on the machine (as encrypted environment variables) and never leave the cloud.

## Environment variables

| Environment variable | Required | Description |
|----------------------|----------|-------------|
| `GOOGLE_CLIENT_ID` | Yes | Google OAuth client id (Desktop app type) |
| `GOOGLE_CLIENT_SECRET` | Yes | Google OAuth client secret |
| `GOOGLE_SCOPES` | No | Default scopes (space or comma separated) used when a request does not specify any. Resolution is **request → `GOOGLE_SCOPES` → the bundled default** (`openid email profile`). |

Until `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` are set, the endpoints return an error.

## Endpoints

### `GET /google/authorize-url`

Query parameters:

| Parameter | Description |
|-----------|-------------|
| `redirectUri` | The client redirect URI Google will call back (for a CLI, a loopback URL such as `http://127.0.0.1:9876/callback`) |
| `scopes` | Space or comma separated scopes. Defaults to `GOOGLE_SCOPES`, then the bundled default |
| `state` | Opaque value round-tripped back to the client |

```bash
curl "https://<your-scope>.on.aux4.cloud/oauth-app/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback"
```

```json
{
  "url": "https://accounts.google.com/o/oauth2/v2/auth?client_id=...&code_challenge=...",
  "codeVerifier": "b7f3...",
  "state": "..."
}
```

### `POST /google/exchange`

JSON body:

| Field | Description |
|-------|-------------|
| `code` | The authorization code returned to the redirect URI |
| `codeVerifier` | The PKCE verifier returned by `authorize-url` |
| `redirectUri` | The same redirect URI used to obtain the code |

```bash
curl -X POST "https://<your-scope>.on.aux4.cloud/oauth-app/api/google/exchange" \
  -H "Content-Type: application/json" \
  -d '{"code":"4/0Ab...","codeVerifier":"b7f3...","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```json
{
  "accessToken": "ya29...",
  "refreshToken": "1//0g...",
  "idToken": "eyJ...",
  "expiresIn": 3599,
  "tokenType": "Bearer",
  "principal": {
    "sub": "1234567890",
    "email": "user@example.com",
    "provider": "google"
  }
}
```

### `POST /google/refresh`

JSON body:

| Field | Description |
|-------|-------------|
| `refreshToken` | The refresh token returned by `exchange` |

```bash
curl -X POST "https://<your-scope>.on.aux4.cloud/oauth-app/api/google/refresh" \
  -H "Content-Type: application/json" \
  -d '{"refreshToken":"1//0g..."}'
```

```json
{
  "accessToken": "ya29...",
  "refreshToken": "1//0g...",
  "idToken": "",
  "expiresIn": 3599,
  "tokenType": "Bearer"
}
```

When Google does not rotate the refresh token, `refreshToken` comes back empty and the client keeps the one it already has.

## Security

The broker is **secure by default**: `authorize-url` and `exchange` require a valid aux4 idToken whose owner is entitled to the machine's scope, enforced by the core's endpoint-auth gate (see [`aux4/oauth-app` › Endpoint authentication](https://hub.aux4.io/r/public/packages/aux4/oauth-app)). `callback` is public (the browser redirect carries no token) and `refresh` is public but rate-limited (the refresh token is the credential).

To run the machine fully public — anything that can reach the URL can start an OAuth flow with your Google client — set `OAUTH_APP_PUBLIC=true` on the machine. Either way, register only the redirect URIs your clients actually use and keep `GOOGLE_SCOPES` scoped to what you need.
