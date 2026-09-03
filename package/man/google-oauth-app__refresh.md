#### Description

The `refresh` command renews an access token from a refresh token on the server side, using the client id and secret held by the broker (never exposed to the caller). It delegates to `aux4 oauth refresh`, supplying the provider's token endpoint from the bundled `providers.yaml` and the credentials from the machine environment.

It is served as `POST /{provider}/refresh`. The api runtime passes the request context as flags: the path params (`{provider}`) on `--params` and the JSON request body on `--body`. The command reads `${params.provider}` and `${body.refreshToken}`.

The provider's client id and secret are read from per-provider environment variables by convention `<PROVIDER>_CLIENT_ID` / `<PROVIDER>_CLIENT_SECRET` (for example `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`). If the provider is unknown, its credentials are not configured, or no refresh token is supplied, the command exits with an error.

The command prints the new tokens as JSON. When the provider does not rotate the refresh token, `refreshToken` comes back empty and the caller keeps the one it already has.

#### Usage

```bash
aux4 google-oauth-app refresh --params '{"provider":"<provider>"}' --body '{"refreshToken":"<token>"}'
```

--params  Path params as JSON — must include `provider` (e.g. `google`)
--body    Request body as JSON — `refreshToken` (required)

The Google client id and secret are read from the `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` environment variables.

Served over HTTP as:

```bash
curl -X POST "https://<machine-url>/api/google/refresh" \
  -H 'Content-Type: application/json' \
  -d '{"refreshToken":"1//0g..."}'
```

#### Example

```bash
aux4 google-oauth-app refresh --params '{"provider":"google"}' --body '{"refreshToken":"1//0g..."}'
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
