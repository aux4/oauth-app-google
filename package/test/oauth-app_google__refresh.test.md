# oauth-app google refresh

The api runtime passes the request context as flags: the JSON request body on
`--body`. `refresh` reads `${body.refreshToken}`, injects the plugin-held Google
client credentials, and renews the access token server-side (the client never
sees the secret).

## when the refresh token is missing

```execute
GOOGLE_CLIENT_ID=cid GOOGLE_CLIENT_SECRET=sec aux4 oauth-app google refresh --body '{}'
```

```error:partial
refreshToken is required
```

## when google has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app google refresh --body '{"refreshToken":"1//0g..."}'
```

```error:partial
google has no client credentials configured
```
