# oauth-app-google authorize-url

The api runtime passes the request context as flags: path params on `--params`,
query string on `--query`. `authorize-url` reads `${params.provider}` and
`${query.redirectUri}` / `${query.scopes}` / `${query.state}`.

## when provider is missing

```execute
aux4 oauth-app-google authorize-url
```

```error:partial
Error: provider is required
```

## when provider is not configured

```execute
aux4 oauth-app-google authorize-url --params '{"provider":"github"}'
```

```error:partial
Error: provider 'github' is not configured
```

## when the configured provider has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app-google authorize-url --params '{"provider":"google"}' --query '{"redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
Error: provider 'google' has no client credentials configured
```

## when google is configured

```execute
GOOGLE_CLIENT_ID=test-client-id aux4 oauth-app-google authorize-url --params '{"provider":"google"}' --query '{"redirectUri":"http://127.0.0.1:9876/callback","scopes":"openid email","state":"xyz"}'
```

```expect:partial
"url":"https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=test-client-id
```
