#### Description

The `authorize-url` command builds a provider authorization URL on the server side, using the client id held by the broker (never exposed to the caller). It delegates to `aux4 oauth authorize-url`, supplying the provider's OAuth endpoints from the bundled `providers.yaml` and the client id from the machine environment.

It is served as `GET /{provider}/authorize-url`. The api runtime passes the request context as flags: the path params (`{provider}`) on `--params`, and the query string on `--query`. The command reads `${params.provider}`, `${query.redirectUri}`, `${query.scopes}`, and `${query.state}`.

The provider's client id is read from a per-provider environment variable by convention `<PROVIDER>_CLIENT_ID` (for example `GOOGLE_CLIENT_ID`). If the provider is unknown or its credentials are not configured, the command exits with an error.

The command prints a JSON object with:

- `url` — the authorization URL to open in a browser.
- `codeVerifier` — the PKCE verifier the client must keep and pass to `exchange`.
- `state` — the opaque state value.

#### Usage

```bash
aux4 google-oauth-app authorize-url --params '{"provider":"<provider>"}' --query '{"redirectUri":"<uri>","scopes":"<scopes>","state":"<state>"}'
```

--params  Path params as JSON — must include `provider` (e.g. `google`)
--query   Query string as JSON — `redirectUri`, `scopes` (space/comma separated), `state`

The Google client id is read from the `GOOGLE_CLIENT_ID` environment variable. When the request omits `scopes`, the default comes from the `GOOGLE_SCOPES` environment variable, and finally from the bundled Google default (`openid email profile`).

Served over HTTP as:

```bash
curl "https://<machine-url>/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback&scopes=openid%20email&state=xyz"
```

#### Example

```bash
aux4 google-oauth-app authorize-url --params '{"provider":"google"}' --query '{"redirectUri":"http://127.0.0.1:9876/callback","scopes":"openid email","state":"xyz"}'
```

```json
{
  "url": "https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=...&code_challenge=...",
  "codeVerifier": "b7f3...",
  "state": "xyz"
}
```
