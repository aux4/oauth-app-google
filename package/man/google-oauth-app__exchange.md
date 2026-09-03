#### Description

The `exchange` command exchanges an authorization code for provider tokens on the server side, using the client id and secret held by the broker (never exposed to the caller). It delegates to `aux4 oauth exchange`, supplying the provider's token and userinfo endpoints from the bundled `providers.yaml` and the credentials from the machine environment.

It is served as `POST /{provider}/exchange`. The api runtime passes the request context as flags: the path params (`{provider}`) on `--params` and the JSON request body on `--body`. The command reads `${params.provider}`, `${body.code}`, `${body.codeVerifier}`, and `${body.redirectUri}`.

The provider's client id and secret are read from per-provider environment variables by convention `<PROVIDER>_CLIENT_ID` / `<PROVIDER>_CLIENT_SECRET` (for example `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`). If the provider is unknown or its credentials are not configured, the command exits with an error.

The command prints the provider tokens and the resolved user profile as JSON.

#### Usage

```bash
aux4 google-oauth-app exchange --params '{"provider":"<provider>"}' --body '{"code":"<code>","codeVerifier":"<verifier>","redirectUri":"<uri>"}'
```

--params  Path params as JSON — must include `provider` (e.g. `google`)
--body    Request body as JSON — `code` (required), `codeVerifier` (PKCE verifier from `authorize-url`), `redirectUri`

The Google client id and secret are read from the `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` environment variables.

Served over HTTP as:

```bash
curl -X POST "https://<machine-url>/api/google/exchange" \
  -H 'Content-Type: application/json' \
  -d '{"code":"4/0Ab...","codeVerifier":"b7f3...","redirectUri":"http://127.0.0.1:9876/callback"}'
```

#### Example

```bash
aux4 google-oauth-app exchange --params '{"provider":"google"}' --body '{"code":"4/0Ab...","codeVerifier":"b7f3...","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```json
{
  "accessToken": "ya29...",
  "refreshToken": "1//0g...",
  "idToken": "eyJ...",
  "email": "user@example.com"
}
```
