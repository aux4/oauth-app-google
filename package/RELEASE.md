# aux4/oauth-app-google 0.0.6

## Changed

- **Endpoint auth alignment with the secure-by-default core.** `/google/callback`
  and `/google/refresh` are marked `public: true` (the browser redirect must reach
  the callback without a token, and a refresh token is itself the credential), so
  they stay open. `/google/authorize-url` and `/google/exchange` inherit the core's
  secure-by-default gate — a caller must present a valid aux4 idToken unless the VM
  runs with `OAUTH_APP_PUBLIC=true`.
