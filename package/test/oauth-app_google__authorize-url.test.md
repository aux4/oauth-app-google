# oauth-app google authorize-url

The api runtime passes the request context as flags: the query string on
`--query`. `authorize-url` reads `${query.scopes}` / `${query.state}`, injects the
plugin-held Google client id, and builds the authorization URL server-side. The
redirect is the broker's OWN callback (`<base>/api/google/callback`), where the
base is `BROKER_PUBLIC_URL` or the aux4.cloud-injected `AUX4_CLOUD_VM_URL`.

## when google is configured

```execute
BROKER_PUBLIC_URL=https://broker.test/oauth-broker GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"scopes":"openid email","state":"s"}'
```

```expect:partial
accounts.google.com/o/oauth2/v2/auth
```

## the url carries the plugin-held client id

```execute
BROKER_PUBLIC_URL=https://broker.test/oauth-broker GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"scopes":"openid email","state":"s"}'
```

```expect:partial
client_id=test-cid
```

## the redirect targets the broker's own callback (not a client url)

```execute
BROKER_PUBLIC_URL=https://broker.test/oauth-broker GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"scopes":"openid email","state":"s"}'
```

```expect:partial
redirect_uri=https%3A%2F%2Fbroker.test%2Foauth-broker%2Fapi%2Fgoogle%2Fcallback
```

## the base falls back to the aux4.cloud-injected url when BROKER_PUBLIC_URL is unset

```execute
AUX4_CLOUD_VM_URL=https://aux4.on.aux4.cloud/oauth-broker GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"scopes":"openid email","state":"s"}'
```

```expect:partial
redirect_uri=https%3A%2F%2Faux4.on.aux4.cloud%2Foauth-broker%2Fapi%2Fgoogle%2Fcallback
```

## when the broker public url is not set

```execute
GOOGLE_CLIENT_ID=test-cid aux4 oauth-app google authorize-url --query '{"scopes":"openid email"}'
```

```error:partial
public URL is not set
```

## when google has no credentials

```execute
BROKER_PUBLIC_URL=https://broker.test/oauth-broker GOOGLE_CLIENT_ID= GOOGLE_CLIENT_SECRET= aux4 oauth-app google authorize-url --query '{"scopes":"openid email"}'
```

```error:partial
google has no client credentials configured
```
