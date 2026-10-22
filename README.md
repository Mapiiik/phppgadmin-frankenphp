# phpPgAdmin on FrankenPHP

Docker image for running **phpPgAdmin** on **FrankenPHP**, built from the maintained fork
[ReimuHakurei/phpPgAdmin](https://github.com/ReimuHakurei/phpPgAdmin). There is no official image.

This repository provides:
- a Docker build with the PostgreSQL 18 client tools (`pg_dump`, `pg_dumpall`)
- production Docker Compose setup
- automated CI that rebuilds the image when the fork or the base image changes

---

## 🚀 Features

- **FrankenPHP** runtime (Caddy + PHP in one process), automatic HTTPS
- **Non-root runtime**
- **No `.git` in the web root**, the fork is added without it
- **Change-driven CI**
  - scheduled builds only when the fork has a new commit or `dunglas/frankenphp` a new image
  - forced rebuilds on pushes to `main`
- Images published to **GitHub Container Registry**

---

## 📦 Docker Images

Images are published to:
```
ghcr.io/mapiiik/phppgadmin-frankenphp
```

Available tags:
- `latest` – latest build
- `<commit>` – short commit of the phpPgAdmin fork (e.g. `f35f659`)
- `build-<commit>-<base>` – fork commit and base image digest, used by CI to skip unchanged builds
- `sha-<commit>` – infrastructure build reference

---

## 🐳 Usage (Production)

1. Copy the templates:

```bash
cp .env.example .env
mkdir -p data && cp config.inc.php.example data/config.inc.php   # or into your DATA_DIR
```

2. Set `SERVER_NAME` and your servers in `config.inc.php`.

3. Start the stack:

```bash
docker compose -f compose.production.yaml up -d
```
or
```bash
cp compose.production.yaml compose.yaml
docker compose up -d
```

## ⚙️ Configuration
- `SERVER_NAME` (automatic ACME certificates; otherwise self-signed HTTPS)
- `LISTENER_MODE` (`direct` by default; `behind-proxy` behind an SNI proxy such as HAProxy, which owns
  port 80, so certificates come by the TLS-ALPN challenge; `internal` only on an internal network: Caddy
  issues the certificate from its own CA and `SERVER_NAME` can be the server's IP address. Browsers warn
  until they trust its root certificate, `/data/caddy/pki/authorities/local/root.crt` in the container)
- `PROXY_ALLOW` (where the PROXY protocol header is trusted from: the proxy's network; by default
  only this machine. The header is optional, so it works with or without a proxy)
- `DATA_DIR` (default `./data`) holds `config.inc.php`, which replaces `conf/config.inc.php`.
  Without it the image runs with the defaults of the fork (one server, no host).

When the fork raises `$conf['version']` in `conf/config.inc.php-dist`, phpPgAdmin refuses an older
configuration file. Compare it with `config.inc.php.example` after an update.

Expose phpPgAdmin only to the networks of your administrators.

## 🧠 CI Behavior
- Scheduled runs check the latest fork commit and the base image digest and build only when one of
  them changed.
- Pushes to main always trigger a rebuild.
- No repository changes are made by CI.

## 📄 License
See individual components for their respective licenses.
