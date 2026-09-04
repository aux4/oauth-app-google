#### Description

The `authorize-url` command builds the Google authorization URL on the server side, using the client id held by the plugin (never exposed to the caller). It delegates to `aux4 oauth authorize-url`, supplying Google's OAuth endpoints from the bundled `providers.yaml` and the client id from the machine environment.

It is served as `GET /google/authorize-url`. The api runtime passes the request context as flags: the query string on `--query`. The command reads `${query.redirectUri}`, `${query.scopes}`, and `${query.state}`.

The Google client id is read from the `GOOGLE_CLIENT_ID` environment variable. If it is not configured, the command exits with an error.

The command prints a JSON object with:

- `url` — the authorization URL to open in a browser.
- `codeVerifier` — the PKCE verifier the client must keep and pass to `exchange`.
- `state` — the opaque state value.

#### Usage

```bash
aux4 oauth-app google authorize-url --query '{"redirectUri":"<uri>","scopes":"<scopes>","state":"<state>"}'
```

--query   Query string as JSON — `redirectUri`, `scopes` (space/comma separated), `state`

The Google client id is read from the `GOOGLE_CLIENT_ID` environment variable. When the request omits `scopes`, the default comes from the `GOOGLE_SCOPES` environment variable, and finally from the bundled Google default (`openid email profile`).

Served over HTTP as:

```bash
curl "https://<machine-url>/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback&scopes=openid%20email&state=xyz"
```

#### Example

```bash
GOOGLE_CLIENT_ID=my-client-id aux4 oauth-app google authorize-url --query '{"redirectUri":"http://127.0.0.1:9876/callback","scopes":"openid email","state":"xyz"}'
```

```json
{
  "url": "https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=my-client-id&code_challenge=...",
  "codeVerifier": "b7f3...",
  "state": "xyz"
}
```
