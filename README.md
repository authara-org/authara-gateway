# Authara Gateway

**Authara Gateway** is a lightweight, configuration-driven reverse proxy that
routes traffic between your application and **Authara**.

Built on **Caddy**, it is fully environment-agnostic and behaves identically
across all deployments.

---

## Overview

All HTTP traffic flows through the gateway:

```
Client
  ↓
Authara Gateway
  ├── /auth/*  → Authara
  └── /*       → Application
```

---

## Quick Start (Docker)

```bash
docker run \
  -p 3000:3000 \
  -e GATEWAY_BIND=:3000 \
  -e AUTHARA_UPSTREAM=authara:8080 \
  -e APP_UPSTREAM=app:8080 \
  ghcr.io/authara/authara-gateway:1.1.0
```

---

## Table of Contents

- Routing Model
- Environment Variables
- Example (Docker Compose)
- HTTPS / TLS
- Compression
- License

---


## Routing Model

All requests enter through the gateway:

```
/auth/*   → Authara
/*        → Application
```

---

## Environment Variables

The gateway is **fully configured via environment variables**.

| Variable | Required | Description |
|----------|----------|-------------|
| `AUTHARA_UPSTREAM` | Yes | Authara upstream (e.g. `authara:8080`) |
| `APP_UPSTREAM` | Yes | Application upstream (e.g. `app:8080`) |
| `GATEWAY_BIND` | No | Address and port the gateway listens on (default: `:80`) |
| `AUTO_HTTPS` | No | Controls Caddy automatic HTTPS (`off` by default) |
| `ENABLE_COMPRESSION` | No | Enables response compression (`true` by default) |

### Example values

```bash
AUTHARA_UPSTREAM=authara:8080
APP_UPSTREAM=app:8080
GATEWAY_BIND=:3000
AUTO_HTTPS=off
ENABLE_COMPRESSION=true
```

---

## Example (Docker Compose)

```yaml
services:
  gateway:
    image: ghcr.io/authara/authara-gateway:1.0.0
    ports:
      - "3000:3000"
    environment:
      GATEWAY_BIND: :3000
      AUTHARA_UPSTREAM: authara:8080
      APP_UPSTREAM: app:8080
      AUTO_HTTPS: off
      ENABLE_COMPRESSION: true
    depends_on:
      - authara
      - app

  authara:
    image: ghcr.io/authara/authara-core:1.0.0
      - "8080"

  app:
    image: example/app:latest
    ports:
      - "8080"
```

> This example shows **network wiring only**.  
> Application and Authara configuration are intentionally omitted.

---

## HTTPS / TLS

Authara Gateway **does not enable automatic HTTPS by default**.

TLS is typically terminated by upstream infrastructure such as:

- Kubernetes Ingress
- Cloud load balancers
- Reverse proxies (NGINX, Caddy, Traefik, Envoy)
- Corporate TLS gateways

This keeps the gateway **portable and environment-agnostic**.

If TLS termination should happen inside the gateway, operators may enable
Caddy's automatic HTTPS:

```bash
AUTO_HTTPS=on
```

---

## Compression

Response compression is **enabled by default**.

The gateway uses Caddy's built-in compression support with:

- **Zstandard**
- **Gzip**

Compression is automatically disabled for:

- Server-Sent Events (`text/event-stream`)
- WebSocket connections
- Authara static assets (already compressed)

Operators may disable compression if another layer (CDN, load balancer, or
application server) already handles it:

```bash
ENABLE_COMPRESSION=false
```

---

## License

MIT License
