# Jalony Elite Stationary - Backend

## Features mapped to the stack checklist

| Item | Where / how |
|---|---|
| Frontend | `jalony-elite-app.html` (mobile web app, offline capable) |
| Backend | `src/server.js` (Express) |
| Database | `src/db.js` (SQLite, WAL, indexes on phone and created_at) |
| Authentication | PIN login with scrypt hashing, signed tokens (12h) |
| Authorization | Roles `admin` and `customer`, `requireRole('admin')` on sensitive routes |
| OAuth | Google OpenID Connect at `/auth/google` with signed state (needs client ID and secret) |
| Hosting | Docker image, `docker-compose.yml`, or Kubernetes `k8s/jalony.yaml` |
| CI/CD | `.github/workflows/ci.yml`: test, build image, deploy |
| API integration | JSON REST: `/api/services`, `/api/orders`, `/api/orders/:id/document` |
| API gateway | `nginx.conf`: TLS, rate limit, `/ws` upgrade, body size limit |
| Webhook | `/webhooks/payment`, HMAC-SHA256 signature check |
| WebSocket | `/ws?token=...`, live order status (customers see their own, admin sees all) |
| HTTP/HTTPS | Redirect to HTTPS, HSTS and security headers |
| Caching | `Cache-Control` and ETag on services, 304 responses |
| Horizontal scaling | HPA in `k8s/jalony.yaml` (needs Postgres first, see limits) |
| Vertical scaling | CPU and memory requests/limits in the Deployment |
| Performance analysis | `/metrics` (admin): requests, average, p50, p95 |
| Cost effectiveness | One small VPS with Docker Compose is enough at this scale |
| Object storage | Document storage adapter (`storage` in `server.js`), local volume now |
| Error handling | Central handler, JSON errors in Swahili and English |
| Error tracking | JSON logs, plus optional `ERROR_WEBHOOK_URL` (e.g. Slack or Sentry webhook) |
| Privacy, Terms, Refund | Included in the app (Sheria screen) |
| Kubernetes | `k8s/jalony.yaml` |

## Run locally
```
cp .env.example .env     # edit secrets
npm install
npm test
npm start                # http://localhost:3000
```
Or with Docker: `docker compose up -d --build`, after placing TLS files in `./certs`.

## Load test (performance)
```
npx autocannon -c 50 -d 20 http://localhost:3000/api/services
```
Then check `/metrics` with an admin token.

## Honest limits
- SQLite allows one writer. Keep `replicas: 1` until the database moves to Postgres. HPA scaling is only safe after that.
- Object storage uses a local volume. For several servers, swap the `storage` adapter for an S3-compatible SDK.
- Google login needs your own OAuth credentials from Google Cloud Console.
- The frontend app still keeps orders on the phone. To sync with this backend, set its API base URL and send orders to `/api/orders`.
- This code has not been deployed or run against real traffic. Run `npm test` first.
