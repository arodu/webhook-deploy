# webhook-deploy

Composite GitHub Action para disparar un deploy por webhook. Es genérica: no
depende de ningún proveedor. Solo necesita una URL completa, un secreto HMAC
opcional y un content type.

Se configura con tres datos:

- `url`: la URL completa del webhook (desde el secret `WEBHOOK_URL`).
- `secret`: opcional (secret `WEBHOOK_SECRET`). Si está vacío, no se firma.
- `content-type`: `application/json` (default) o
  `application/x-www-form-urlencoded`.

Cada repo usa el mismo nombre de secret, pero con el valor de su propio webhook.
No hay acoplamiento a un proveedor.

## Webhook con HMAC

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

`WEBHOOK_URL` es la URL completa, por ejemplo
`https://example.com/hooks/my-app`.

## URL con token, sin HMAC

Si la URL ya incluye el token, alcanza con `WEBHOOK_URL`:

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.WEBHOOK_URL }}
```

Si el endpoint espera formulario:

```yaml
- uses: arodu/webhook-deploy@v1
  with:
    url: ${{ secrets.WEBHOOK_URL }}
    content-type: application/x-www-form-urlencoded
```

El mismo workflow sirve para ambos casos: si `WEBHOOK_SECRET` no existe, se
resuelve como vacío y la action no firma.

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
