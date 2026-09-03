# aux4/google-oauth-app 0.0.1

Initial release: a public, one-click-deployable **Google OAuth service**.

- Deploy it to your own aux4.cloud scope, set `GOOGLE_CLIENT_ID` /
  `GOOGLE_CLIENT_SECRET`, and point your CLI tools (for example
  `community/google-auth`, via `--broker` / `GOOGLE_AUTH_BROKER`) at its URL.
- Routes: `GET /health`, `GET /google/authorize-url`, `POST /google/exchange`,
  `POST /google/refresh` — the client secret stays server-side at login and refresh.
- `GOOGLE_SCOPES` sets default scopes when a request does not specify any
  (resolution: request → `GOOGLE_SCOPES` → bundled default).

Derived from the internal `aux4/oauth-broker`, branded and pre-wired for Google.
