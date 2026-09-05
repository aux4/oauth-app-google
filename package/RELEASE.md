# aux4/oauth-app-google 0.0.6

## Fixed

- Hardened token operations so they no longer depend on `${packageDir}/providers.yaml`
  resolving on a multi-package broker VM. The `authorize-url`, `exchange`, and
  `refresh` commands now pass the Google OAuth endpoints explicitly as flags
  (`--authUrl`, `--tokenUrl`, `--userinfoUrl`) in addition to `--configFile`.
  Explicit flags take precedence, so the operations succeed even if the config file
  fails to load — matching the fix shipped in `oauth-app-x` 0.0.7. Google sends the
  client secret in the body, so `--clientSecretIn` is intentionally not set.
