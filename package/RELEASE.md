# aux4/oauth-app-google

Restructured from a standalone OAuth app into a **plugin of [`aux4/oauth-app`](https://hub.aux4.io/r/public/packages/aux4/oauth-app)**, following the aux4 core+plugin convention (the same way `aux4/db-mysql` extends `aux4/db`).

Changes:

- Depends only on `aux4/oauth-app` — the core provides the shared `aux4/api`, `aux4/oauth`, and `aux4/config` machinery and the `/health` route.
- Extends the core's `oauth-app` profile with a single `google` command that routes to the `oauth-app:google` profile (`authorize-url`, `exchange`, `refresh`).
- The provider is now implicit Google — the `provider` path param and provider guards were removed.
- Owns its `/google/*` routes in `config.yaml`; `/health` is left to the core.
- Man pages and tests renamed to the new `oauth-app google <cmd>` command path.

Set `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` (+ optional `GOOGLE_SCOPES`) on the machine and point your CLI at the deployed core.
