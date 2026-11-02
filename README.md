# webhook-deploy

Composite GitHub Action para disparar un deploy por webhook. La URL completa se
pasa desde `secrets`, así no queda hardcodeada en el repo.

Soporta dos casos:

- **Webhook con HMAC** (el homelab): se pasa `secret` y se envía
  `X-Hub-Signature-256`.
- **URL completa con token** (Hostinger): la URL ya incluye el token; se omite
  `secret` y no se firma.

## Uso con HMAC

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
      - uses: arodu/webhook-deploy@v2
        with:
          url: ${{ secrets.HOMELAB_DEPLOY_URL }}
          secret: ${{ secrets.HOMELAB_DEPLOY_SECRET }}
```

`HOMELAB_DEPLOY_URL` es la URL completa del webhook, por ejemplo
`https://example.com/hooks/my-app`.

## Uso sin HMAC (URL con token)

```yaml
- uses: arodu/webhook-deploy@v2
  with:
    url: ${{ secrets.HOSTINGER_DEPLOY_URL }}
    method: GET
```

Si el webhook de Hostinger espera `GET`, usá `method: GET`. La URL con el token
va completa en el secret.

## Inputs

| Input    | Requerido | Default | Descripción                                      |
|----------|-----------|---------|--------------------------------------------------|
| `url`    | sí        | —       | URL completa del webhook                         |
| `secret` | no        | —       | Secreto HMAC; si está vacío, no se firma         |
| `event`  | no        | `push`  | Valor del header `X-GitHub-Event`                |
| `method` | no        | `POST`  | Método HTTP (`POST`, `GET`, etc.)                |

## Qué envía

- `X-GitHub-Event: <event>`
- `X-Hub-Signature-256: sha256=<hmac>` (solo si hay `secret`)
- `Content-Type: application/json` y body con `${{ toJSON(github.event) }}`
  (solo para métodos con body)

La firma es `HMAC-SHA256` del body exacto, calculada con `openssl`. El receptor
debe validarla si usa `secret`.

## Versionado

Usá `@v2` o, para mayor reproducibilidad, un commit SHA:

```yaml
uses: arodu/webhook-deploy@<sha>
```

`v1` era la versión con `path` separado; `v2` usa la URL completa.

## Licencia

MIT.
