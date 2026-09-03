#### Description

The `health` command is a liveness check for the broker machine. It prints a small JSON object and exits successfully, so an uptime probe or load balancer can confirm the service is up without exercising any provider flow.

It is served as `GET /health`. The api runtime passes the request context as flags (`--params`, `--query`, `--body`), all unused here.

#### Usage

```bash
aux4 google-oauth-app health
```

Served over HTTP as:

```bash
curl "https://<machine-url>/api/health"
```

#### Example

```bash
aux4 google-oauth-app health
```

```json
{
  "status": "ok"
}
```
