## Bootstrap Environment

### Infrastructure

``` bash
set +o history
export PORKBUN_API_KEY=your-porkbun-api-key
export PORKBUN_API_SECRET_KEY=your-porkbun-api-secret
podman secret create acme-email <(echo <your-email-address)
podman secret create porkbun-api-key <(echo $PORKBUN_API_KEY)
podman secret create porkbun-api-secret-key <(echo $PORKBUN_API_SECRET_KEY)
podman secret create unifi-mongo-password <(pwgen -n 32 1)
podman secret create unifi-mongo-root-password <(pwgen -n 32 1)
set -o history
```

### Media

#### Generate Immich DB Password
`pwgen -s -A 32 1 | podman secret create media-immich-db-password -`
