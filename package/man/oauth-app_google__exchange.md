#### Description

The `exchange` command exchanges an authorization code for Google tokens on the server side, using the client id and secret held by the plugin (never exposed to the caller). It delegates to `aux4 oauth exchange`, supplying Google's token and userinfo endpoints from the bundled `providers.yaml` and the credentials from the machine environment.

It is served as `POST /google/exchange`. The api runtime passes the request context as flags: the JSON request body on `--body`. The command reads `${body.code}`, `${body.codeVerifier}`, and `${body.redirectUri}`.

The Google client id and secret are read from the `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` environment variables. If they are not configured, the command exits with an error.

The command prints the Google tokens and the resolved user profile as JSON.

#### Usage

```bash
aux4 oauth-app google exchange --body '{"code":"<code>","codeVerifier":"<verifier>","redirectUri":"<uri>"}'
```

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
GOOGLE_CLIENT_ID=cid GOOGLE_CLIENT_SECRET=sec aux4 oauth-app google exchange --body '{"code":"4/0Ab...","codeVerifier":"b7f3...","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```json
{
  "accessToken": "ya29...",
  "refreshToken": "1//0g...",
  "idToken": "eyJ...",
  "email": "user@example.com"
}
```
