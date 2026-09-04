# aux4/oauth-app-google 0.0.5

## Changed

- **Hosted-callback + poll flow.** `authorize-url`/`exchange` now redirect to the broker's own HTTPS callback (`<base>/api/google/callback`) instead of a client-supplied localhost loopback — so login works from **any device** and satisfies confidential-client redirect rules. Adds the `GET /google/callback` route (owned by this plugin, handled by the core).
- **Zero-config URL on aux4.cloud.** The redirect base resolves as `nvl(BROKER_PUBLIC_URL, AUX4_CLOUD_VM_URL)`: a machine deployed on aux4.cloud picks up its own public URL automatically, so no URL config is needed. `BROKER_PUBLIC_URL` remains an override for self-hosters (base only, no `/api`).

## Notes

- Requires `aux4/oauth-app` ≥ 0.0.4 (session + callback + poll) and a Google **Web application** OAuth client with `<base>/api/google/callback` registered as an authorized redirect URI (a Desktop client cannot be used — it has no redirect field).
