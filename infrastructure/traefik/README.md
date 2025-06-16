# Infrastructure Traefik Configuration

## Overview

This directory contains the configuration for the infrastructure Traefik instance that serves as the main entry point and load balancer for all Dokploy instances.

## Architecture

```
Internet → UCG Ultra → Infrastructure Traefik → {
                                              Dokploy Dev (*.dev.domain.com)
                                              Dokploy Staging (*.staging.domain.com)
                                              Dokploy Prod (*.domain.com)
                                              Monitoring Services
                                            }
```

## Key Features

- **True Load Balancing**: Layer 7 load balancing with health checks
- **SSL Termination**: Automatic wildcard certificate generation and management
- **Security**: Modern TLS configuration, security headers, rate limiting
- **Monitoring**: Prometheus metrics and health checks
- **High Availability**: Health checks with automatic failover

## Files Structure

```
infrastructure/traefik/
├── docker-compose.yml           # Main Traefik deployment
├── traefik.yml                  # Static configuration
├── dynamic/
│   ├── dokploy-routing.yml      # Dokploy instance routing
│   └── monitoring-routes.yml    # Monitoring and infrastructure routes
├── .env.example                 # Environment variables template
├── logs/                        # Log files (created at runtime)
└── README.md                    # This file
```

## Quick Setup

### 1. Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit with your values
nano .env
```

**Required Variables:**
```bash
DOMAIN_NAME=yourdomain.com
ADMIN_EMAIL=admin@yourdomain.com
DNS_PROVIDER=cloudflare
CLOUDFLARE_EMAIL=your-email@domain.com
CLOUDFLARE_API_TOKEN=your_api_token_here

# Update IP addresses for your network
DOKPLOY_DEV_IP=192.168.1.100
DOKPLOY_STAGING_IP=192.168.1.101
DOKPLOY_PROD_IP=192.168.1.102
```

### 2. Create Docker Network

```bash
# Create external network for Traefik
docker network create traefik-net
```

### 3. Deploy Traefik

```bash
# Start Traefik
docker-compose up -d

# Check logs
docker-compose logs -f traefik

# Verify status
docker-compose ps
```

### 4. Verify Deployment

```bash
# Test Traefik dashboard
curl -I https://traefik.yourdomain.com

# Test certificate generation
echo | openssl s_client -connect traefik.yourdomain.com:443 -servername traefik.yourdomain.com | grep -A5 "Certificate chain"

# Test routing to Dokploy
curl -I https://dev.yourdomain.com
```

## Configuration Details

### Static Configuration (traefik.yml)

**Key Settings:**
- **Entry Points**: HTTP (port 80) with HTTPS redirect, HTTPS (port 443), Dashboard (port 8080)
- **Certificate Resolvers**: Let's Encrypt with DNS challenge for wildcard certificates
- **TLS Options**: Modern TLS configuration with strong ciphers
- **Metrics**: Prometheus metrics enabled
- **Logging**: JSON format with access logs

### Dynamic Configuration

#### Dokploy Routing (dynamic/dokploy-routing.yml)

**Routing Rules:**
```yaml
# Development Environment
*.dev.yourdomain.com → Dokploy Dev (192.168.1.100:80)

# Staging Environment  
*.staging.yourdomain.com → Dokploy Staging (192.168.1.101:80)

# Production Environment
*.prod.yourdomain.com → Dokploy Prod (192.168.1.102:80)
*.yourdomain.com → Dokploy Prod (192.168.1.102:80)
```

**Middlewares Applied:**
- Security headers (HSTS, CSP, etc.)
- Compression (gzip)
- Rate limiting (25 req/sec average, 50 burst)
- CORS headers for API endpoints

#### Monitoring Routes (dynamic/monitoring-routes.yml)

**Services Exposed:**
- `monitoring.yourdomain.com` → Grafana
- `prometheus.yourdomain.com` → Prometheus (auth required)
- `alerts.yourdomain.com` → AlertManager (auth required)
- `uptime.yourdomain.com` → Uptime Kuma

## SSL Certificate Management

### Automatic Certificate Generation

Traefik automatically generates wildcard certificates for:
- `*.dev.yourdomain.com`
- `*.staging.yourdomain.com`
- `*.prod.yourdomain.com`
- `*.yourdomain.com`

### Certificate Storage

Certificates are stored in the `traefik-certs` Docker volume and persist across container restarts.

### Certificate Renewal

Certificates are automatically renewed 30 days before expiry.

## Security Features

### TLS Configuration

```yaml
# Modern TLS settings
Minimum Version: TLS 1.2
Cipher Suites: Modern, secure ciphers only
Curve Preferences: P-521, P-384
SNI Strict: Enabled
```

### Security Headers

```yaml
Applied Headers:
- Strict-Transport-Security (HSTS)
- Content-Security-Policy
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- X-XSS-Protection: 1; mode=block
- Referrer-Policy: strict-origin-when-cross-origin
```

### Rate Limiting

```yaml
Web Traffic: 25 req/sec average, 50 burst
API Traffic: 10 req/sec average, 20 burst
Based on: Source IP address
```

### Authentication

```yaml
Dashboard: Basic authentication (admin/admin by default)
Monitoring: Basic authentication required
Infrastructure: Basic authentication required
```

**Change Default Password:**
```bash
# Generate new password hash
htpasswd -nb admin your-new-password

# Update docker-compose.yml with new hash
```

## Health Checks

### Service Health Monitoring

Traefik performs health checks on all backend services:

```yaml
Health Check Settings:
  Path: /health (or service-specific)
  Interval: 30 seconds
  Timeout: 5 seconds
  Failure Threshold: 3 consecutive failures
```

### Traefik Health Check

```bash
# Check Traefik health
curl http://localhost:8080/ping

# Check via Docker health check
docker inspect infrastructure-traefik | grep Health -A 20
```

## Monitoring and Metrics

### Prometheus Metrics

Traefik exposes metrics at `http://localhost:8080/metrics`

**Key Metrics:**
- Request count and duration
- HTTP status codes
- Service health status
- TLS certificate expiry

### Log Analysis

```bash
# View access logs
tail -f logs/access.log

# View Traefik logs
tail -f logs/traefik.log

# Parse JSON logs
cat logs/access.log | jq '.'
```

## Troubleshooting

### Common Issues

#### Certificate Generation Failures

```bash
# Check DNS provider credentials
docker-compose logs traefik | grep -i "dns\|acme\|challenge"

# Test DNS API access
curl -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" "https://api.cloudflare.com/client/v4/zones"

# Increase DNS challenge delay
# Edit traefik.yml: delayBeforeCheck: 120s
```

#### Routing Issues

```bash
# Check dynamic configuration
docker exec infrastructure-traefik cat /etc/traefik/dynamic/dokploy-routing.yml

# Test backend connectivity
curl -I http://192.168.1.100  # Dokploy dev
curl -I http://192.168.1.101  # Dokploy staging
curl -I http://192.168.1.102  # Dokploy prod

# Check health check status
curl http://localhost:8080/api/http/services
```

#### Performance Issues

```bash
# Check resource usage
docker stats infrastructure-traefik

# Analyze request patterns
cat logs/access.log | jq '.status' | sort | uniq -c

# Check for rate limiting
grep "rate limit" logs/traefik.log
```

### Debug Mode

```bash
# Enable debug logging
# Edit traefik.yml: log.level: DEBUG

# Restart Traefik
docker-compose restart traefik

# Monitor debug logs
docker-compose logs -f traefik
```

## Maintenance

### Updating Traefik

```bash
# Check for updates
docker pull traefik:v3.1

# Update and restart
docker-compose pull
docker-compose up -d

# Verify update
docker-compose ps
curl -I https://traefik.yourdomain.com
```

### Backup Configuration

```bash
# Backup certificates
cp -r /var/lib/docker/volumes/traefik_traefik-certs/ /backup/traefik-certs-$(date +%Y%m%d)

# Backup configuration
tar -czf /backup/traefik-config-$(date +%Y%m%d).tar.gz .
```

### Performance Tuning

```yaml
# Increase worker processes
# Add to traefik.yml:
global:
  api:
    maxIdleConnsPerHost: 200
  
# Adjust health check intervals
healthCheck:
  interval: "60s"  # Increase for less overhead
```

## Advanced Configuration

### Adding New Services

1. **Create route in dynamic configuration:**
```yaml
# Add to dynamic/monitoring-routes.yml
new-service:
  rule: "Host(`newservice.yourdomain.com`)"
  service: new-service-backend
  entryPoints:
    - websecure
  tls:
    certResolver: letsencrypt-prod
```

2. **Define backend service:**
```yaml
new-service-backend:
  loadBalancer:
    servers:
      - url: "http://192.168.1.50:8080"
    healthCheck:
      path: "/health"
```

### Custom Middleware

```yaml
# Add to dynamic configuration
middlewares:
  custom-auth:
    forwardAuth:
      address: "http://auth-service:8080/auth"
      trustForwardHeader: true
```

### Load Balancing Algorithms

```yaml
# Round-robin (default)
loadBalancer:
  servers:
    - url: "http://server1:80"
    - url: "http://server2:80"

# Weighted round-robin
loadBalancer:
  servers:
    - url: "http://server1:80"
      weight: 3
    - url: "http://server2:80"
      weight: 1
```

## Support

For issues and questions:
1. Check the [troubleshooting guide](../../docs/troubleshooting.md)
2. Review Traefik documentation: https://doc.traefik.io/traefik/
3. Open an issue in the repository

---

**Next Steps:**
- Configure [monitoring stack](../monitoring/README.md)
- Set up [Dokploy instances](../../dokploy/README.md)
- Review [operational procedures](../../docs/operational-runbooks.md)