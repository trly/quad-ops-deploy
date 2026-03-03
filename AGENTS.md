# quad-ops-deploy — Agent Guidelines

## Overview
Declarative Docker Compose definitions for [quad-ops](https://github.com/trly/quad-ops), targeting Podman Quadlet on multiple hosts. No build/test/lint tooling — this repo is pure configuration.

## Structure
- `prod/<host>/<stack>/docker-compose.yml` — production stacks per host (`bdyvp22`, `taylor`, `6cf90a5e80be`)
- `test/<host>/<stack>/docker-compose.yml` — test environment (mirrors prod structure)
- Each stack may include `.env`, `Caddyfile`, or supporting config files
- Stack-specific `AGENTS.md` files (e.g. `prod/bdyvp22/media/AGENTS.md`) contain update checklists — read them before modifying that stack

## Conventions
- **Secrets**: Use `x-quad-ops-env-secrets` to inject Podman secrets as env vars — never hardcode secrets or commit them to `.env`; `.env` files are only for non-sensitive configuration values
- **SELinux**: Append `:z` to all bind-mount volume paths
- **Networking**: Services behind Caddy reverse proxy use an external `infrastructure-proxy` network; ports bound to `127.0.0.1` only
- **Images**: Pin third-party images with `@sha256:` digests where upstream does so; use `${VERSION:-release}` patterns for app images
- **Named volumes**: Explicitly set `name:` on volumes to control Podman naming (e.g. `name: media_immich-model-cache`)
- **Restart policy**: All services use `restart: always`
- **No placeholders**: Every `.env` value must be a real value or a `podman secret`; never leave placeholder/example credentials
