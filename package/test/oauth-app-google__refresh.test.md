# oauth-app-google refresh

The api runtime passes the request context as flags: path params on `--params`,
the JSON request body on `--body`. `refresh` reads `${params.provider}` and
`${body.refreshToken}`, injects the broker-held client credentials, and renews
the access token server-side (the client never sees the secret).

## when provider is missing

```execute
aux4 oauth-app-google refresh --body '{"refreshToken":"1//0g..."}'
```

```error:partial
Error: provider is required
```

## when provider is not configured

```execute
aux4 oauth-app-google refresh --params '{"provider":"github"}' --body '{"refreshToken":"1//0g..."}'
```

```error:partial
Error: provider 'github' is not configured
```

## when the refresh token is missing

```execute
GOOGLE_CLIENT_ID=cid GOOGLE_CLIENT_SECRET=sec aux4 oauth-app-google refresh --params '{"provider":"google"}' --body '{}'
```

```error:partial
Error: refreshToken is required
```

## when the configured provider has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app-google refresh --params '{"provider":"google"}' --body '{"refreshToken":"1//0g..."}'
```

```error:partial
Error: provider 'google' has no client credentials configured
```
