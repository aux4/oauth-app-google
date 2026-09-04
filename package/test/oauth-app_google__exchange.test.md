# oauth-app google exchange

The api runtime passes the request context as flags: the JSON request body on
`--body`. `exchange` reads `${body.code}` / `${body.codeVerifier}` /
`${body.redirectUri}`, injects the plugin-held Google client credentials, and
exchanges the code server-side (the client never sees the secret).

## when google has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app google exchange --body '{"code":"abc","codeVerifier":"def","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
google has no client credentials configured
```
