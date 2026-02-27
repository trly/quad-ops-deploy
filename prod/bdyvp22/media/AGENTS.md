# Immich Compose — Agent Guidelines

## Upstream Reference

Always compare against the official compose file before making changes:
https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml

## Checklist When Updating

### Volume Mount Paths

Immich has changed internal mount paths across major versions (e.g. `/usr/src/app/upload` → `/data`). When updating the server image tag, verify the expected volume mount points match the upstream compose file.

### Image Digests

The postgres, valkey, and ML images use pinned `@sha256:` digests. When updating, pull the current digests from the upstream compose file — the tag alone (e.g. `14-vectorchord0.4.3-pgvectors0.2.0`) may resolve to a different build over time.

### Database `shm_size`

The upstream compose sets `shm_size: 128mb` on the postgres container. This must be preserved — without it, postgres defaults to 64MB shared memory, which can cause silent crashes under load.

### Valkey Major Version

Track the upstream valkey major version (currently v9). Major version bumps can change wire protocol or persistence behavior.

## Local Customizations (Do Not Overwrite)

These are intentional deviations from the upstream compose and must be preserved during updates:

- **`DB_HOSTNAME` / `REDIS_HOSTNAME`**: Set to quadlet container names (`immich-immich-database`, `immich-immich-redis`) instead of compose service names
- **`x-podman-env-secrets`**: DB password injected via podman secrets rather than `.env` file
- **`host_ip: 127.0.0.1`** on published ports: Binds only to loopback (traffic comes through the reverse proxy)
- **Extra volume mount**: `/mnt/immich/photoprism-library:/mnt/media/photoprism:z` for photoprism library access
- **`:z` SELinux labels** on bind mounts
- **`networks: infrastructure-proxy`**: External network for reverse proxy connectivity
- **Explicit `container_name: immich-server`** on the server service
