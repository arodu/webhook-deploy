# webhook-deploy

Composite GitHub Action to trigger a deploy webhook. It is provider-agnostic:
it only needs a full URL, an optional HMAC secret, and a content type.

It takes three inputs:

- `url`: full webhook URL (from the `WEBHOOK_URL` secret).
- `secret`: optional (from the `WEBHOOK_SECRET` secret). If empty, no signature
  is sent.
- `content-type`: `application/json` (default) or
  `application/x-www-form-urlencoded`.

Each repository uses the same secret names with its own webhook values. There is
no provider coupling.

## Webhook with HMAC

```yaml
name: deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: arodu/webhook-deploy@v1
        with:
          url: ${{ secrets.WEBHOOK_URL }}
          secret: ${{ secrets.WEBHOOK_SECRET }}
```

`WEBHOOK_URL` is the full URL, for example
`https://example.com/hooks/my-app`.

## Token URL without HMAC

If the URL already includes the token, `WEBHOOK_URL` is enough:

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.WEBHOOK_URL }}
```

If the endpoint expects a form:

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.WEBHOOK_URL }}
    content-type: application/x-www-form-urlencoded
```

The same workflow works for both cases: if `WEBHOOK_SECRET` is not set, it
resolves to an empty string and the action skips signing.

## Inputs

| Input          | Required | Default            | Description                                |
|----------------|----------|--------------------|--------------------------------------------|
| `url`          | yes      | —                  | Full webhook URL                           |
| `secret`       | no       | —                  | HMAC secret; if empty, no signature is sent |
| `content-type` | no       | `application/json` | JSON or `application/x-www-form-urlencoded` |

## What it sends

- `X-GitHub-Event: push`
- `X-Hub-Signature-256: sha256=<hmac>` (only if `secret` is set)
- `Content-Type: <content-type>`
- Body:
  - JSON: the raw GitHub event.
  - Form: `payload=<urlencoded json>`, matching GitHub form webhooks.

The signature is HMAC-SHA256 of the exact body. The receiver must validate it if
it uses `secret`. `curl` verifies SSL by default.

## Versioning

Use `@v1` or, for stricter reproducibility, a commit SHA:

```yaml
uses: arodu/webhook-deploy@<sha>
```

## License

MIT.
