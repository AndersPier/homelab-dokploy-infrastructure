# Configuration Files

This directory contains configuration templates and guides for setting up your homelab infrastructure.

## Files Overview

### Core Configuration
- **`homelab.conf.example`** - Main configuration template with all settings
- **`dns-providers.md`** - Complete guide for DNS provider setup (required for SSL certificates)

### Network Setup Guides
- **`ucg-ultra-simple.md`** - Basic UCG Ultra configuration for single network
- **`ucg-ultra-enterprise.md`** - Advanced UCG Ultra with VLANs and enterprise features

## Quick Setup

### 1. Copy Configuration Template
```bash
cp config/homelab.conf.example config/homelab.conf
```

### 2. Edit Configuration
```bash
nano config/homelab.conf
```

**Required Changes:**
- `DOMAIN_NAME` - Your domain name
- `CLOUDFLARE_EMAIL` and `CLOUDFLARE_API_TOKEN` - DNS provider credentials
- IP addresses for your network
- SMTP settings for alerts

### 3. Secure Your Configuration
```bash
# Make sure homelab.conf is not tracked by git
chmod 600 config/homelab.conf
echo "config/homelab.conf" >> .gitignore
```

## Network Configuration Options

### Simple Setup (Recommended for Beginners)
```yaml
Network Mode: simple
Network: 192.168.1.0/24
Complexity: Low
Setup Time: 30 minutes
Requirements: Any router with port forwarding
```

### Enterprise Setup (Advanced Users)
```yaml
Network Mode: enterprise
Network: Multiple VLANs (10, 20, 30, 40, 50, 60)
Complexity: High
Setup Time: 2-4 hours
Requirements: Managed switch with VLAN support
```

## DNS Provider Setup

**Required for SSL certificates** - choose one:

1. **Cloudflare (Recommended)**
   - Free tier available
   - Excellent API and fast propagation
   - Follow: [DNS Provider Guide](dns-providers.md#cloudflare)

2. **DigitalOcean**
   - Simple setup and good performance
   - Follow: [DNS Provider Guide](dns-providers.md#digitalocean)

3. **AWS Route53**
   - Enterprise-grade reliability
   - Small monthly cost (~$0.50)
   - Follow: [DNS Provider Guide](dns-providers.md#aws-route53)

## Configuration Validation

Before running setup scripts, validate your configuration:

```bash
# Check configuration syntax
./scripts/validate-config.sh

# Test DNS provider credentials
./scripts/test-dns-provider.sh

# Verify network connectivity
./scripts/check-network.sh
```

## Security Notes

⚠️ **Important Security Practices:**

1. **Never commit `homelab.conf` to git** - it contains secrets
2. **Use secure passwords** - generate strong SMTP passwords
3. **Limit DNS API permissions** - use scoped tokens when possible
4. **Regular token rotation** - rotate API tokens every 6-12 months
5. **Backup configuration** - securely backup your configuration file

## Troubleshooting

### Common Configuration Issues

#### DNS Provider Errors
```bash
# Test DNS API access
curl -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
     "https://api.cloudflare.com/client/v4/zones"
```

#### Network Connectivity
```bash
# Test internal connectivity
ping 192.168.1.10  # Infrastructure server
ping 192.168.1.20  # Proxmox host
ping 192.168.1.30  # QNAP storage
```

#### SMTP Configuration
```bash
# Test email sending (install mailutils first)
echo "Test message" | mail -s "Test" admin@yourdomain.com
```

### Getting Help

1. **Check the troubleshooting guide**: [docs/troubleshooting.md](../docs/troubleshooting.md)
2. **Review DNS provider docs**: [config/dns-providers.md](dns-providers.md)
3. **Validate configuration**: Run validation scripts
4. **Open an issue**: [GitHub Issues](https://github.com/AndersPier/homelab-dokploy-infrastructure/issues)

## Next Steps

After configuring:

1. **Simple Setup**: Run `./scripts/simple-setup.sh`
2. **Enterprise Setup**: Run `./scripts/enterprise-setup.sh`
3. **Deploy Dokploy**: Run `./scripts/deploy-dokploy.sh`
4. **Set up Monitoring**: Follow [infrastructure/monitoring/README.md](../infrastructure/monitoring/README.md)

---

**Remember**: Start with the simple setup first, then upgrade to enterprise features as needed!