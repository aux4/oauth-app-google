# oauth-app-google exchange

The api runtime passes the request context as flags: path params on `--params`,
the JSON request body on `--body`. `exchange` reads `${params.provider}` and
`${body.code}` / `${body.codeVerifier}` / `${body.redirectUri}`.

## when provider is missing

```execute
aux4 oauth-app-google exchange --body '{"code":"abc","codeVerifier":"def","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
Error: provider is required
```

## when provider is not configured

```execute
aux4 oauth-app-google exchange --params '{"provider":"github"}' --body '{"code":"abc","codeVerifier":"def","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
Error: provider 'github' is not configured
```

## when the configured provider has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app-google exchange --params '{"provider":"google"}' --body '{"code":"abc","codeVerifier":"def","redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
Error: provider 'google' has no client credentials configured
```
