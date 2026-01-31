# Self-Hosted Automation Stack

A Docker Compose setup for running self-hosted automation and version control services behind Cloudflare Tunnel.

## Services

### Traefik
Reverse proxy and load balancer that routes incoming traffic to services based on hostname. Configured to:
- Listen on HTTP port 80
- Use Docker provider for automatic service discovery
- Forward Cloudflare headers for proper HTTPS detection

**⚠️ Security Warning:** The Traefik dashboard is exposed at port 8080 without authentication. Do not expose this to the public internet.

### Cloudflare Tunnel
Secure tunnel that connects this server to Cloudflare's network, providing:
- Inbound traffic routing through Cloudflare
- Outbound connectivity to Cloudflare's edge
- Zero-trust security model

### n8n
Workflow automation platform for building:
- Custom integrations
- Automated workflows
- Webhook handlers

### Gitea
Self-hosted Git service offering:
- Git repository hosting
- Issue tracking
- Pull requests
- Code review

## Prerequisites

- Docker & Docker Compose
- Cloudflare account with domain
- Cloudflare Zero Trust (free tier works)
- A domain configured in Cloudflare

## Setup

1. Clone this repository
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Configure environment variables in `.env`:
   - `DOMAIN_NAME` - Your main domain
   - `N8N_SUBDOMAIN` - Subdomain for n8n (e.g., "n8n")
   - `GITEA_SUBDOMAIN` - Subdomain for Gitea (e.g., "git")
   - `TUNNEL_TOKEN` - Cloudflare Tunnel token from Zero Dashboard
   - `GENERIC_TIMEZONE` - Timezone (e.g., "America/New_York")
4. Start the stack:
   ```bash
   docker-compose up -d
   ```

## Security Notes

⚠️ **Important security considerations:**

- **Traefik Dashboard**: Port 8080 is exposed without authentication. Access only from trusted networks or disable entirely with `--api.insecure=false`
- **Docker Socket**: Traefik has read-only access to Docker socket. Only expose in trusted networks
- **Environment Variables**: Sensitive values in `.env` - never commit this file
- **Tunnel Token**: Keep your Cloudflare tunnel token secure and rotate if compromised
- **Update Images**: Regularly update to latest stable versions, not `latest` tag

### Recommended Security Improvements

1. Enable Traefik authentication (basic auth or forward auth)
2. Set resource limits in `deploy` section
3. Pin image versions instead of using `latest`
4. Add rate limiting middleware
5. Use Docker secrets for sensitive data
6. Enable Traefik metrics with authentication

## Directory Structure

```
.
├── compose.yaml          # Main Docker Compose configuration
├── README.md            # This file
├── .env.example         # Environment template
├── .gitignore           # Git ignore rules
├── gitea/               # Gitea data directory (created on first run)
└── n8n-compose/
    └── local-files/     # Files directory for n8n workflows
```

## Ports

| Service   | Port | Description                    |
|-----------|------|--------------------------------|
| Traefik   | 80   | HTTP traffic (main entrypoint) |
| Traefik   | 8080 | Dashboard (⚠️ unauthenticated) |
| n8n       | 5678 | Internal only (via Traefik)    |
| Gitea     | 3000 | Web UI (via Traefik)           |
| Gitea     | 22   | SSH (port 222 external)        |

## Troubleshooting

### Check service status
```bash
docker-compose ps
```

### View logs
```bash
docker-compose logs -f
```

### Restart a service
```bash
docker-compose restart <service-name>
```

## License

MIT License
