# Test Infrastructure Setup

This directory contains the test infrastructure for quad-ops, including a local PKI (Public Key Infrastructure) with step-ca, Keycloak for authentication, and a Caddy reverse proxy.

This infrastructure serves as a testing environment that follows quad-ops naming conventions and project structure standards.

## Prerequisites

- Podman 4.0+ with podman-compose
- Git
- Make sure your local system can resolve `*.localhost` domains to `127.0.0.1`

## Repository Structure

This follows the quad-ops standard for repository organization:

```
test/
└── infrastructure/          # Project directory (composeDir equivalent)
    ├── docker-compose.yml   # Main compose file
    ├── .env                 # Environment variables
    ├── Caddyfile           # Reverse proxy config
    └── README.md           # This file
```

When managed by quad-ops, this would be configured as:
- **Repository name**: `test` (configured name, not GitHub repo name)
- **Project name**: `infrastructure` (from `test/infrastructure` composeDir)
- **Compose directory**: `test/infrastructure`

Resources will be prefixed with the project name `infrastructure-`:

## Quick Start

To avoid startup issues, the step-ca PKI needs to be properly initialized before running the full stack.

### 1. Create Podman Secrets

Generate and store secure secrets using pwgen:

```bash
# Generate and create step-ca password secret
pwgen -s 32 1 | podman secret create step-ca-password -

# Generate and create Keycloak bootstrap password secret
pwgen -s 32 1 | podman secret create keycloak-bootstrap-password -

# Verify secrets were created
podman secret ls
```

### 2. Deploy with quad-ops

Now that secrets are created, let quad-ops handle the entire deployment and initialization:

```bash
# Sync and start the infrastructure 
quad-ops sync -u

# This will:
# - Clone/update the repository
# - Create volumes, networks, and containers
# - Initialize step-ca automatically on first startup
# - Start all services with proper dependencies
```

**Note**: step-ca will automatically initialize with ACME support on first startup using the DOCKER_STEPCA_INIT environment variables in docker-compose.yml. After the first successful startup, the initialization environment variables are ignored and the existing CA configuration is used. Logging is set to "info" level to avoid exposing sensitive information in container logs.

### 3. Get Root CA Fingerprint

After the infrastructure is running, extract the root CA fingerprint from the PKI container:

```bash
# Get the root CA fingerprint from the running container
podman exec infrastructure-pki \
  step certificate fingerprint /home/step/certs/root_ca.crt

# Example output:
# 4fe5f5ef09e95c803fdcb80b8cf511e2a885eb86f3ce74e3e90e62fa3faf1531
```

Save this fingerprint - it's needed for step CLI clients to trust your CA.



## Services

### PKI (step-ca)
- **URL**: https://pki.localhost:8443
- **Purpose**: Local certificate authority for issuing TLS certificates
- **Volume**: `certificate-authority` - stores CA keys, certificates, and database

### Keycloak
- **URL**: https://keycloak.localhost:8443
- **Admin Console**: https://keycloak.localhost:8443/admin
- **Default Admin**: admin / (password from environment)
- **Purpose**: Identity and access management

### Reverse Proxy (Caddy)
- **HTTP Port**: 8080
- **HTTPS Port**: 8443
- **Purpose**: Routes traffic to services and handles TLS termination using step-ca

## Troubleshooting

### step-ca won't start
- Ensure the `certificate-authority` volume is properly initialized
- Check that the CA password matches between initialization and the environment file
- Verify the `ca.json` configuration is valid

### Certificate errors
- Make sure your system trusts the step-ca root certificate
- Check that `*.localhost` domains resolve to `127.0.0.1`
- Verify the ACME provisioner is properly configured

### Keycloak startup issues
- Ensure the PKI service is running first (dependency)
- Check that the keycloak-bootstrap-password secret exists: `podman secret ls`
- Verify network connectivity between services

## Manual Certificate Trust (Optional)

To avoid browser certificate warnings, you can trust the step-ca root certificate:

```bash
# Get the root certificate
podman run --rm -v infrastructure-certificate-authority:/home/step \
  docker.io/smallstep/step-ca:latest \
  cat /home/step/certs/root_ca.crt > root_ca.crt

# Trust it (varies by OS)
# Linux with ca-certificates:
sudo cp root_ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# macOS:
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain root_ca.crt
```

## Cleanup

To reset the infrastructure:

```bash
podman-compose down -v
podman volume rm infrastructure-certificate-authority
```

This will remove all containers, networks, and persistent data.

## Integration with quad-ops

To manage this infrastructure with quad-ops, add this repository configuration:

```yaml
repositories:
  - name: test
    url: https://github.com/trly/quad-ops-deploy.git
    composeDir: test/infrastructure
    ref: main
```

This creates a project named `infrastructure` with all resources prefixed as `infrastructure-*`.
