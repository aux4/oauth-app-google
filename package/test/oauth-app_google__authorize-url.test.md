# oauth-app google authorize-url

The api runtime passes the request context as flags: the query string on
`--query`. `authorize-url` reads `${query.redirectUri}` / `${query.scopes}` /
`${query.state}`, injects the plugin-held Google client id, and builds the
authorization URL server-side.

## when google is configured

```execute
GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"redirectUri":"http://127.0.0.1:9876/callback","scopes":"openid email","state":"s"}'
```

```expect:partial
accounts.google.com/o/oauth2/v2/auth
```

## when google is configured the url carries the client id

```execute
GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"redirectUri":"http://127.0.0.1:9876/callback","scopes":"openid email","state":"s"}'
```

```expect:partial
client_id=test-cid
```

## when google has no credentials

```execute
GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app google authorize-url --query '{"redirectUri":"http://127.0.0.1:9876/callback"}'
```

```error:partial
google has no client credentials configured
```
