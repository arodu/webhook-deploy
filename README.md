# webhook-deploy

Composite GitHub Action para disparar un deploy por webhook. La URL completa se
pasa desde `secrets`, así no queda hardcodeada en el repo.

Replica el formulario de webhooks de GitHub:

- **Payload URL** → input `url` (desde un secret).
- **Content type** → input `content-type` (`application/json` por defecto).
- **Secret** → input `secret` (opcional).

Siempre hace `POST` de eventos `push`, igual que un webhook de GitHub.

## Homelab / webhook con HMAC

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
          secret: ${{ secrets.HOMELAB_DEPLOY_SECRET }}
```

`HOMELAB_DEPLOY_URL` es la URL completa, por ejemplo
`https://example.com/hooks/my-app`.

## Hostinger / URL con token, sin HMAC

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.HOSTINGER_DEPLOY_URL }}
```

La URL de Hostinger ya incluye el token; no hace falta `secret`.

Si el endpoint espera formulario:

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.HOSTINGER_DEPLOY_URL }}
    content-type: application/x-www-form-urlencoded
```

## Inputs

| Input          | Requerido | Default            | Descripción                                |
|----------------|-----------|--------------------|--------------------------------------------|
| `url`          | sí        | —                  | URL completa del webhook                   |
| `secret`       | no        | —                  | Secreto HMAC; si está vacío, no se firma   |
| `content-type` | no        | `application/json` | JSON o `application/x-www-form-urlencoded` |

## Qué envía

- `X-GitHub-Event: push`
- `X-Hub-Signature-256: sha256=<hmac>` (solo si hay `secret`)
- `Content-Type: <content-type>`
- Body:
  - JSON: el evento de GitHub tal cual.
  - Form: `payload=<json urlencoded>`, igual que los webhooks form de GitHub.

La firma es `HMAC-SHA256` del body exacto. El receptor debe validarla si usa
`secret`. `curl` verifica SSL por defecto.

## Versionado

Usá `@v1` o, para mayor reproducibilidad, un commit SHA:

```yaml
uses: arodu/webhook-deploy@<sha>
```

## Licencia

MIT.
