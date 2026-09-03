#### Description

The `oauth-app-google` command groups the OAuth broker subcommands. The broker is a provider-agnostic OAuth service that holds each provider's application credentials (client id and secret) so that thin CLI clients never have to handle them.

It is a thin wrapper over `aux4/oauth` and is designed to run as an `api`-type machine on aux4.cloud. Its routes are served behind an API Gateway:

- `GET /health` — liveness check.
- `GET /{provider}/authorize-url` — build the provider authorization URL server-side.
- `POST /{provider}/exchange` — exchange an authorization code for tokens server-side.
- `POST /{provider}/refresh` — renew an access token from a refresh token server-side.

The api runtime maps each request to the corresponding command, passing the request context as flags: path params on `--params`, the query string on `--query`, and the JSON body on `--body`. The commands read `${params.*}` / `${query.*}` / `${body.*}` accordingly.

Subcommands:

- **health** — liveness check.
- **authorize-url** — build the provider authorization URL server-side.
- **exchange** — exchange an authorization code for tokens server-side.
- **refresh** — renew an access token from a refresh token server-side.

#### Usage

```bash
aux4 oauth-app-google <subcommand>
```

#### Example

```bash
aux4 oauth-app-google authorize-url --params '{"provider":"google"}' --query '{"redirectUri":"http://127.0.0.1:9876/callback"}'
```

Served over HTTP as:

```bash
curl "https://<machine-url>/api/google/authorize-url?redirectUri=http://127.0.0.1:9876/callback"
```
