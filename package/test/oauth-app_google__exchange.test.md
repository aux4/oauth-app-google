# oauth-app google exchange

The api runtime passes the request context as flags: the JSON request body on
`--body`. `exchange` reads `${body.code}` / `${body.codeVerifier}`, injects the
plugin-held Google client credentials, and exchanges the code server-side (the
client never sees the secret). The redirect URI is fixed to the broker's own
callback (`<base>/api/google/callback`) so it matches the one used at authorize.

## when the broker public url is not set

```execute
GOOGLE_CLIENT_ID=cid GOOGLE_CLIENT_SECRET=sec aux4 oauth-app google exchange --body '{"code":"abc","codeVerifier":"def"}'
```

```error:partial
public URL is not set
```

## when google has no credentials

```execute
BROKER_PUBLIC_URL=https://broker.test/oauth-broker GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app google exchange --body '{"code":"abc","codeVerifier":"def"}'
```

```error:partial
google has no client credentials configured
```
