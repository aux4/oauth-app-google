#### Description

The `google` command groups the Google OAuth subcommands provided by the `aux4/oauth-app-google` plugin. It extends the `aux4/oauth-app` core's `oauth-app` profile with a Google provider.

The plugin holds the Google application credentials (client id and secret) server-side, so thin CLI clients never have to handle them. It is a thin wrapper over `aux4/oauth` and is designed to run as part of an `api`-type machine on aux4.cloud, alongside the core. Its routes are served behind an API Gateway under the `/api` prefix:

- `GET /google/authorize-url` — build the Google authorization URL server-side.
- `POST /google/exchange` — exchange an authorization code for tokens server-side.
- `POST /google/refresh` — renew an access token from a refresh token server-side.

The `/health` liveness route is provided by the `aux4/oauth-app` core.

The api runtime maps each request to the corresponding command, passing the request context as flags: the query string on `--query` and the JSON body on `--body`. The commands read `${query.*}` / `${body.*}` accordingly.

Subcommands:

- **authorize-url** — build the Google authorization URL server-side.
- **exchange** — exchange an authorization code for tokens server-side.
- **refresh** — renew an access token from a refresh token server-side.

#### Usage

```bash
aux4 oauth-app google <subcommand>
```

#### Example

```bash
aux4 oauth-app google authorize-url --query '{"redirectUri":"http://127.0.0.1:9876/callback"}'
```

Served over HTTP as:

```bash
curl "https://<machine-url>/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback"
```
