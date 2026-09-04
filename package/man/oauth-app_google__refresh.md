#### Description

The `refresh` command renews an access token from a refresh token on the server side, using the client id and secret held by the plugin (never exposed to the caller). It delegates to `aux4 oauth refresh`, supplying Google's token endpoint from the bundled `providers.yaml` and the credentials from the machine environment.

It is served as `POST /google/refresh`. The api runtime passes the request context as flags: the JSON request body on `--body`. The command reads `${body.refreshToken}`.

The Google client id and secret are read from the `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` environment variables. If they are not configured, or no refresh token is supplied, the command exits with an error.

The command prints the new tokens as JSON. When Google does not rotate the refresh token, `refreshToken` comes back empty and the caller keeps the one it already has.

#### Usage

```bash
aux4 oauth-app google refresh --body '{"refreshToken":"<token>"}'
```

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
GOOGLE_CLIENT_ID=cid GOOGLE_CLIENT_SECRET=sec aux4 oauth-app google refresh --body '{"refreshToken":"1//0g..."}'
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
