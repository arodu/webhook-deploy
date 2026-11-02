# webhook-deploy

Composite GitHub Action para disparar un deploy por webhook firmado con
HMAC-SHA256. No contiene URLs ni secretos: la URL base y el secreto se pasan
desde `secrets` del repo que la usa.

## Uso

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
          url: ${{ secrets.HOMELAB_DEPLOY_URL }}
          path: hooks/my-app
          secret: ${{ secrets.HOMELAB_DEPLOY_SECRET }}
```

`url` es la base (por ejemplo `https://example.com`), `path` es la ruta del
webhook (`hooks/my-app`) y `secret` el secreto HMAC. La action arma
`${url}/${path}`, firma el JSON del evento y hace el POST.

## Inputs

| Input    | Requerido | Default | Descripción                                      |
|----------|-----------|---------|--------------------------------------------------|
| `url`    | sí        | —       | URL base del servidor de webhooks                 |
| `path`   | sí        | —       | Ruta del webhook, por ejemplo `hooks/my-app`      |
| `secret` | sí        | —       | Secreto HMAC para firmar el payload               |
| `event`  | no        | `push`  | Valor del header `X-GitHub-Event`                 |

## Qué envía

- `X-GitHub-Event: <event>`
- `X-Hub-Signature-256: sha256=<hmac>`
- `Content-Type: application/json`
- Body: `${{ toJSON(github.event) }}`

La firma es `HMAC-SHA256` del body exacto, calculada con `openssl`. El receptor
debe validar la firma antes de procesar el payload.

## Secrets

En el repo que usa la action:

- `HOMELAB_DEPLOY_URL`: URL base del homelab (o del servidor de webhooks).
- `HOMELAB_DEPLOY_SECRET`: secreto HMAC de la app.

La action falla si falta alguno.

## Versionado

Usá el tag `v1` o, para mayor reproducibilidad, un commit SHA:

```yaml
uses: arodu/webhook-deploy@<sha>
```

## Licencia

MIT.
