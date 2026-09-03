# aux4/oauth-app-google

Deploy your **own Google OAuth service** in one click. It holds your Google OAuth client id and secret server-side and does the authorization-URL build, the code exchange, and the token refresh on behalf of your CLI tools — so those tools **never handle a client secret**, and every user authenticates under **your** Google project, quota, and consent screen.

It is a thin HTTP wrapper over [`aux4/oauth`](https://hub.aux4.io/r/public/packages/aux4/oauth), deployed as an `api`-type machine on [aux4.cloud](https://aux4.cloud). The package itself contains **no secrets** — your credentials are supplied as machine environment variables.

## Quick start

1. **Deploy it.** From the [hub package page](https://hub.aux4.io/r/public/packages/aux4/oauth-app-google), click **Deploy to cloud** (or `aux4 aux4 cloud deploy oauth-app-google --package aux4/oauth-app-google --api true`). You get a URL like `https://<your-scope>.on.aux4.cloud/oauth-app-google`.

2. **Add your Google credentials.** Create an OAuth client of type **Desktop app** in the [Google Cloud Console](https://console.cloud.google.com/) (Desktop is required so the CLI's `http://localhost:9876/callback` loopback redirect is accepted), then set them on the machine:

   ```bash
   aux4 aux4 cloud oauth-app-google env set GOOGLE_CLIENT_ID=... GOOGLE_CLIENT_SECRET=...
   ```

   Values are encrypted at rest per scope and applied to the live machine immediately.

3. **Point your CLI at it.** For [`community/google-auth`](https://hub.aux4.io/r/public/packages/community/google-auth):

   ```bash
   export GOOGLE_AUTH_BROKER="https://<your-scope>.on.aux4.cloud/oauth-app-google/api"
   aux4 google auth login
   ```

   Login opens a Google consent screen, catches the redirect on a local loopback, and stores the token — no client id or secret ever touches the user's machine. Expired tokens are refreshed through the app automatically.

## Installation

You do not normally install this package locally — you deploy it. To inspect or run it locally:

```bash
aux4 aux4 pkger install aux4/oauth-app-google
```

## How it works

The service exposes a liveness check plus three Google endpoints, served under the `/api` prefix (for example `GET <machine-url>/api/health`):

1. `GET /health` — returns `{"status":"ok"}` so uptime probes can confirm the service is up.
2. `GET /google/authorize-url` — returns the Google authorization URL, a PKCE `codeVerifier`, and the `state`. The client opens the URL in a browser and keeps the `codeVerifier`.
3. `POST /google/exchange` — takes the authorization `code`, the `codeVerifier`, and the `redirectUri`, and returns the tokens plus the resolved user profile.
4. `POST /google/refresh` — takes a `refreshToken` and returns a renewed set of tokens, so a client keeps a long-lived session alive without ever holding the client secret.

Your client id and secret live only on the machine (as encrypted environment variables) and never leave the cloud.

## Configuration

| Environment variable | Required | Description |
|----------------------|----------|-------------|
| `GOOGLE_CLIENT_ID` | Yes | Google OAuth client id (Desktop app type) |
| `GOOGLE_CLIENT_SECRET` | Yes | Google OAuth client secret |
| `GOOGLE_SCOPES` | No | Default scopes (space or comma separated) used when a request does not specify any. Resolution is **request → `GOOGLE_SCOPES` → the bundled default** (`openid email profile`). Clients that compute their own scopes (like `google-auth`) override it. |

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
curl "https://<your-scope>.on.aux4.cloud/oauth-app-google/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback"
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
curl -X POST "https://<your-scope>.on.aux4.cloud/oauth-app-google/api/google/exchange" \
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
curl -X POST "https://<your-scope>.on.aux4.cloud/oauth-app-google/api/google/refresh" \
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

These endpoints are **unauthenticated**: anything that can reach the URL can start an OAuth flow with your Google client. That is by design for a loopback CLI login, but it means you should treat the deployment URL as semi-sensitive, register only the redirect URIs your clients actually use, and keep `GOOGLE_SCOPES` scoped to what you need. A scope allowlist (rejecting requests for scopes beyond a configured set) is a planned addition.
