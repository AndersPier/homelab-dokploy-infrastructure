# DNS Provider Configuration for Wildcard Certificates

## Why DNS Challenge is Required

**CRITICAL**: Wildcard certificates (*.yourdomain.com) can **ONLY** be obtained using DNS-01 challenge, not HTTP-01 challenge. This means Traefik must automatically create DNS TXT records in your domain to prove ownership.

## Choosing a DNS Provider

### Criteria for Selection:
1. **API Support**: Must have API for automated record management
2. **Reliability**: High uptime and fast propagation
3. **Cost**: Free or affordable options available
4. **Documentation**: Good API documentation and community support
5. **Security**: Secure token/key management

## Recommended DNS Providers

### 1. **Cloudflare (Highly Recommended)** ⭐

**Why Choose Cloudflare:**
- ✅ Free tier with generous limits
- ✅ Excellent API and fast propagation (usually <1 minute)
- ✅ Built-in CDN and security features
- ✅ Great Traefik integration
- ✅ Robust infrastructure and high uptime

**Setup Steps:**

1. **Transfer Domain to Cloudflare** (or just use Cloudflare DNS):
   - Sign up at [cloudflare.com](https://cloudflare.com)
   - Add your domain
   - Update nameservers at your registrar

2. **Create API Token**:
   - Go to "My Profile" → "API Tokens"
   - Click "Create Token"
   - Use "Custom token" template:
     ```yaml
     Permissions:
       - Zone: Zone Settings: Read
       - Zone: Zone: Read
       - Zone: DNS: Edit
     Zone Resources:
       - Include: All zones (or specific zone: yourdomain.com)
     ```
   - Click "Continue to summary" → "Create Token"
   - **Save the token securely** (you won't see it again)

3. **Configuration Variables**:
   ```bash
   CLOUDFLARE_EMAIL=your-email@domain.com
   CLOUDFLARE_DNS_API_TOKEN=your_generated_token_here
   ```

**Alternative: Global API Key (Less Secure)**:
```bash
CLOUDFLARE_EMAIL=your-email@domain.com
CLOUDFLARE_API_KEY=your_global_api_key
```

### 2. **DigitalOcean** 🌊

**Why Choose DigitalOcean:**
- ✅ Simple API and good documentation
- ✅ Free DNS hosting
- ✅ Fast propagation
- ✅ Good integration with other DO services

**Setup Steps:**

1. **Configure DNS**:
   - Log into DigitalOcean
   - Go to "Networking" → "Domains"
   - Add your domain
   - Update nameservers at registrar to point to DO

2. **Create API Token**:
   - Go to "API" → "Personal Access Tokens"
   - Generate new token with "Write" scope
   - Copy token securely

3. **Configuration**:
   ```bash
   DO_AUTH_TOKEN=your_digitalocean_api_token_here
   ```

### 3. **AWS Route53** ☁️

**Why Choose Route53:**
- ✅ Extremely reliable (Amazon infrastructure)
- ✅ Fast propagation globally
- ✅ Integrates with other AWS services
- ❌ Not free (but very cheap - ~$0.50/month per domain)

**Setup Steps:**

1. **Set up Route53 Hosted Zone**:
   - Go to AWS Route53 console
   - Create hosted zone for your domain
   - Update nameservers at registrar

2. **Create IAM User**:
   - Go to IAM → Users → Add User
   - Select "Programmatic access"
   - Attach policy (create custom or use PowerUserAccess)
   
3. **IAM Policy for DNS Only**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "route53:GetChange",
           "route53:ChangeResourceRecordSets",
           "route53:ListHostedZonesByName"
         ],
         "Resource": [
           "arn:aws:route53:::hostedzone/*",
           "arn:aws:route53:::change/*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": "route53:ListHostedZones",
         "Resource": "*"
       }
     ]
   }
   ```

4. **Configuration**:
   ```bash
   AWS_ACCESS_KEY_ID=your_access_key_id
   AWS_SECRET_ACCESS_KEY=your_secret_access_key
   AWS_REGION=us-east-1
   ```

### 4. **Namecheap** 💰

**Why Choose Namecheap:**
- ✅ Popular registrar with DNS hosting
- ✅ Good for keeping everything in one place
- ❌ API can be slower to propagate

**Setup Steps:**

1. **Enable API Access**:
   - Log into Namecheap
   - Go to Profile → Tools → Business & API Keys
   - Enable API access
   - Whitelist your IP address

2. **Configuration**:
   ```bash
   NAMECHEAP_API_USER=your_username
   NAMECHEAP_API_KEY=your_api_key
   ```

### 5. **Google Cloud DNS** 🔧

**Why Choose Google Cloud DNS:**
- ✅ Very reliable (Google infrastructure)
- ✅ Good integration with GCP services
- ❌ Requires Google Cloud account and billing

**Setup Steps:**

1. **Create DNS Zone**:
   - Go to Google Cloud Console
   - Enable Cloud DNS API
   - Create DNS zone for your domain

2. **Create Service Account**:
   - Go to IAM & Admin → Service Accounts
   - Create service account with "DNS Administrator" role
   - Generate JSON key file

3. **Configuration**:
   ```bash
   GCE_PROJECT=your_project_id
   GCE_SERVICE_ACCOUNT_FILE=/path/to/service-account.json
   ```

### 6. **OVH** 🇫🇷

**Why Choose OVH:**
- ✅ European provider with good pricing
- ✅ Solid API support
- ❌ More complex setup process

**Setup Steps:**

1. **Create API Credentials**:
   - Go to [OVH API Console](https://api.ovh.com/createToken/)
   - Create application credentials
   - Generate consumer key

2. **Configuration**:
   ```bash
   OVH_ENDPOINT=ovh-eu
   OVH_APPLICATION_KEY=your_application_key
   OVH_APPLICATION_SECRET=your_application_secret
   OVH_CONSUMER_KEY=your_consumer_key
   ```

## DNS Record Setup

### Required DNS Records

Regardless of provider, you'll need these A records:

```dns
# Main domain and wildcards pointing to your public IP
yourdomain.com                A    YOUR_PUBLIC_IP
*.dev.yourdomain.com         A    YOUR_PUBLIC_IP
*.staging.yourdomain.com     A    YOUR_PUBLIC_IP
*.prod.yourdomain.com        A    YOUR_PUBLIC_IP
traefik.yourdomain.com       A    YOUR_PUBLIC_IP
monitoring.yourdomain.com    A    YOUR_PUBLIC_IP

# Optional: IPv6 support
yourdomain.com                AAAA YOUR_PUBLIC_IPv6
*.dev.yourdomain.com         AAAA YOUR_PUBLIC_IPv6
```

### DNS Propagation Testing

```bash
# Test that your DNS records are working
dig @8.8.8.8 yourdomain.com
dig @8.8.8.8 test.dev.yourdomain.com

# Test from different locations
dig @1.1.1.1 yourdomain.com
dig @8.8.4.4 yourdomain.com

# Check propagation globally
# Visit: https://dnschecker.org/
```

## Certificate Validation Process

Here's how Let's Encrypt validates wildcard certificates:

1. **Certificate Request**: Traefik requests `*.dev.yourdomain.com`
2. **DNS Challenge**: Let's Encrypt requires DNS validation
3. **TXT Record Creation**: Traefik creates `_acme-challenge.dev.yourdomain.com` TXT record
4. **Validation**: Let's Encrypt checks the TXT record exists
5. **Certificate Issued**: Certificate is issued and stored
6. **Cleanup**: TXT record is removed

## Security Best Practices

### 1. **API Token Scoping**
```yaml
Minimum Required Permissions:
  - DNS: Edit (to create/delete TXT records)
  - Zone: Read (to find the correct zone)
  - Zone Settings: Read (to read zone metadata)

Avoid:
  - Global API keys when scoped tokens available
  - Excessive permissions (like zone deletion)
  - Long-lived tokens without rotation
```

### 2. **Secret Management**
```bash
# Store in secure files (not in git)
echo "your_api_token" > /etc/traefik/secrets/cloudflare_token
chmod 600 /etc/traefik/secrets/cloudflare_token

# Use environment files
echo "CLOUDFLARE_DNS_API_TOKEN=your_token" > /etc/traefik/.env
chmod 600 /etc/traefik/.env

# Docker secrets (for production)
docker secret create cloudflare_token /path/to/token/file
```

### 3. **Token Rotation**
```yaml
Best Practices:
  - Rotate API tokens every 6-12 months
  - Monitor token usage in provider dashboards
  - Revoke unused or compromised tokens immediately
  - Use separate tokens for different environments
```

## Troubleshooting DNS Issues

### Common Problems

#### 1. **DNS Propagation Delays**
```yaml
Symptoms: Certificate generation fails or times out
Solutions:
  - Increase delayBeforeCheck in Traefik config (60-120 seconds)
  - Check DNS provider's propagation speed
  - Verify DNS records are actually created
  - Test with different DNS resolvers

Traefik Config Fix:
certificatesResolvers:
  letsencrypt:
    acme:
      dnsChallenge:
        provider: cloudflare
        delayBeforeCheck: 90s  # Increase this
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
```

#### 2. **API Permission Errors**
```yaml
Symptoms: "403 Forbidden" or "unauthorized" errors
Solutions:
  - Verify API token has correct permissions
  - Check token isn't expired
  - Ensure zone is accessible with token
  - Test API manually with curl

Test Cloudflare API:
curl -X GET "https://api.cloudflare.com/client/v4/zones" \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json"
```

#### 3. **TXT Record Conflicts**
```yaml
Symptoms: Certificate validation fails despite correct setup
Solutions:
  - Check for existing _acme-challenge records
  - Verify no conflicting DNS management tools
  - Clean up old TXT records manually
  - Check DNS cache (flush if needed)

Manual Cleanup:
# Delete any existing _acme-challenge records in your DNS provider
# Wait for DNS propagation (5-15 minutes)
# Retry certificate generation
```

### Debug Commands

```bash
# Check TXT record creation during certificate generation
watch "dig @8.8.8.8 _acme-challenge.yourdomain.com TXT"

# Monitor Traefik logs for DNS challenge
docker logs traefik -f | grep -i "acme\|dns\|challenge"

# Test DNS provider manually
# (Examples vary by provider - check provider documentation)

# Verify certificate after generation
openssl x509 -in /path/to/cert.pem -text -noout | grep -A5 "Subject Alternative Name"
```

## Rate Limits and Considerations

### Let's Encrypt Limits
```yaml
Key Limits:
  - 50 certificates per registered domain per week
  - 5 failures per account, per hostname, per hour
  - 300 new orders per account per 3 hours
  
Best Practices:
  - Use staging environment first
  - Don't regenerate certificates unnecessarily
  - Monitor certificate expiration
  - Implement proper error handling
```

### DNS Provider Limits
```yaml
Typical API Limits:
  Cloudflare: 1200 requests per 5 minutes
  DigitalOcean: 5000 requests per hour
  Route53: Very high limits (AWS throttling)
  
Mitigation:
  - Cache DNS responses when possible
  - Implement exponential backoff
  - Monitor API usage
  - Use appropriate delayBeforeCheck values
```

## Cost Comparison

| Provider | Cost | Free Tier | Notes |
|----------|------|-----------|-------|
| Cloudflare | Free | Yes (generous) | Best overall value |
| DigitalOcean | Free | Yes | Free DNS hosting |
| Route53 | ~$0.50/month | No | Very reliable |
| Namecheap | Free with domain | Varies | Included with domains |
| Google Cloud DNS | ~$0.20/month | $300 credit | Pay as you go |
| OVH | €0.10/month | No | European option |

## Conclusion

**For most users, Cloudflare is the best choice** due to:
- Free tier with generous limits
- Excellent performance and reliability
- Fast DNS propagation
- Great API and documentation
- Built-in security features

**For AWS users, Route53 offers** excellent integration and reliability at minimal cost.

**For European users, OVH** provides a good regional option with competitive pricing.

Choose based on your specific needs, existing infrastructure, and preferences. All recommended providers work well with Traefik's ACME DNS challenge for wildcard certificates.