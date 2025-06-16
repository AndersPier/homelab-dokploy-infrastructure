# UCG Ultra Simple Configuration

## Overview

This guide covers the basic UCG Ultra configuration for single-network homelab setup. The UCG Ultra provides **intelligent port forwarding with health checks**, not traditional load balancing.

## Key Capabilities

- **Advanced Port Forwarding**: Route external traffic to internal servers
- **Health Monitoring**: Automatic failover when backend servers are down
- **DPI & Security**: Deep packet inspection and threat detection
- **QoS**: Traffic prioritization and bandwidth management
- **VPN Server**: Built-in WireGuard and OpenVPN support

## Basic Network Setup

### 1. Network Configuration

```yaml
WAN Configuration:
  Interface: WAN (2.5GbE port)
  Connection Type: DHCP or Static (from ISP)
  
LAN Configuration:
  Interface: LAN (4x Gigabit ports)
  Network: 192.168.1.0/24
  Gateway: 192.168.1.1
  DHCP Range: 192.168.1.100-200
```

### 2. Port Forwarding Rules

**Access**: UniFi Network Console → Gateway → Port & IP Forwarding

```yaml
# Infrastructure Server (Primary Entry Point)
Rule Name: "Infrastructure-HTTP"
From: Any
Port: 80
Forward To: 192.168.1.10:80    # Infrastructure server
Protocol: TCP
Description: "HTTP traffic to infrastructure Traefik"

Rule Name: "Infrastructure-HTTPS"
From: Any
Port: 443
Forward To: 192.168.1.10:443   # Infrastructure server
Protocol: TCP
Description: "HTTPS traffic to infrastructure Traefik"

# SSH Access
Rule Name: "Infrastructure-SSH"
From: Any
Port: 22
Forward To: 192.168.1.10:22    # Infrastructure server
Protocol: TCP
Description: "SSH access to infrastructure server"

# Management Access (Optional - Restrict to VPN)
Rule Name: "Proxmox-Web"
From: VPN Network Only
Port: 8006
Forward To: 192.168.1.20:8006  # Proxmox host
Protocol: TCP
Description: "Proxmox web interface (VPN only)"

Rule Name: "QNAP-Management"
From: VPN Network Only
Port: 8443
Forward To: 192.168.1.30:443   # QNAP storage
Protocol: TCP
Description: "QNAP management interface (VPN only)"
```

### 3. Firewall Configuration

**Access**: UniFi Network Console → Security → Firewall & Security

```yaml
# Basic Rules
Rule 1:
  Name: "Allow Established/Related"
  Action: Accept
  Source: Any
  Destination: Any
  Protocol: Any
  State: Established, Related

Rule 2:
  Name: "Allow Internal DNS"
  Action: Accept
  Source: LAN Network
  Destination: 192.168.1.20 (AdGuard)
  Protocol: TCP/UDP
  Port: 53

Rule 3:
  Name: "Drop Invalid"
  Action: Drop
  Source: Any
  Destination: Any
  Protocol: Any
  State: Invalid

# Default Policy: Accept (for simplicity)
```

### 4. DNS Configuration

```yaml
DNS Settings:
  Primary DNS: 192.168.1.20      # AdGuard Home primary
  Secondary DNS: 192.168.1.21    # AdGuard Home secondary
  Domain Name: lab.local
  
DHCP DNS Override:
  Enabled: Yes
  Force DNS: Yes (prevents device DNS bypass)
```

### 5. Quality of Service (QoS)

```yaml
Smart Queues (Optional):
  Download Bandwidth: 90% of actual ISP speed
  Upload Bandwidth: 90% of actual ISP speed
  
Traffic Rules:
  Priority 1 (High): VoIP, Video Conferencing
  Priority 2 (Medium): Web Browsing, HTTPS
  Priority 3 (Low): File Downloads, Backups
  
Application Prioritization:
  Business Apps: High Priority
  Streaming: Medium Priority
  File Sharing: Low Priority
```

## Health Monitoring Setup

### 1. Port Forward Health Checks

```yaml
# For each port forwarding rule, configure health monitoring:
Health Check Configuration:
  Type: HTTP
  Interval: 30 seconds
  Timeout: 5 seconds
  Failure Threshold: 3 attempts
  Recovery Threshold: 2 attempts
  
# Example for Infrastructure Server:
Monitoring URL: http://192.168.1.10/health
Method: GET
Expected Response: 200 OK
```

### 2. Backup Server Configuration (Optional)

```yaml
# If you have multiple servers, configure failover:
Primary: 192.168.1.10
Backup: 192.168.1.11

# UCG Ultra will automatically switch to backup when primary fails
Failover Mode: Automatic
Failback Mode: Manual (recommended)
```

## VPN Server Setup

### 1. WireGuard Configuration

**Access**: UniFi Network Console → VPN → WireGuard

```yaml
Server Configuration:
  Interface: wg0
  Port: 51820
  Network: 10.10.10.0/24
  DNS: 192.168.1.20, 192.168.1.21
  
Route Configuration:
  Route LAN Traffic: Yes
  LAN Networks: 192.168.1.0/24
```

### 2. Client Profiles

```yaml
# Admin Profile
Client Name: "admin-mobile"
IP Address: 10.10.10.2/24
Allowed Networks: 192.168.1.0/24
DNS: 192.168.1.20

# Limited Profile
Client Name: "guest-access"
IP Address: 10.10.10.3/24
Allowed Networks: 192.168.1.0/24 (with firewall restrictions)
DNS: 192.168.1.20
```

## Traffic Analysis

### 1. DPI Configuration

```yaml
Deep Packet Inspection:
  Enabled: Yes
  Application Detection: Yes
  Threat Detection: Yes
  
Categories to Monitor:
  - Web Browsing
  - Video Streaming
  - File Sharing
  - Gaming
  - Business Applications
```

### 2. Monitoring Dashboard

```yaml
Access: UniFi Network Console → Insights

Key Metrics:
  - Bandwidth Usage by Device
  - Application Usage
  - Security Threats Blocked
  - VPN Connection Status
  - Port Forward Health Status
```

## Security Hardening

### 1. Threat Management

```yaml
IDS/IPS Settings:
  Intrusion Detection: Enabled
  Intrusion Prevention: Enabled
  Malware Blocking: Enabled
  
Blacklist Updates:
  Automatic Updates: Enabled
  Update Frequency: Daily
```

### 2. Access Control

```yaml
Management Access:
  Local Admin: Enabled
  Remote Access: Disabled (use VPN)
  
SSH Access:
  Enabled: No (use UniFi Network Console)
  
SNMP:
  Enabled: No (unless needed for monitoring)
```

## Backup and Recovery

### 1. Configuration Backup

```yaml
Automatic Backup:
  Enabled: Yes
  Frequency: Weekly
  Location: Cloud Key or local download
  
Manual Backup:
  Access: UniFi Network Console → Settings → System
  Action: Download Backup
  Schedule: Before major changes
```

### 2. Recovery Procedure

```yaml
Factory Reset:
  1. Hold reset button 10 seconds while powered
  2. Wait for LED to cycle
  3. Adopt in UniFi Network Console
  4. Restore from backup
  
Configuration Restore:
  1. Access Settings → System
  2. Upload backup file
  3. Confirm restore operation
  4. Wait for reboot and verification
```

## Monitoring and Alerts

### 1. Event Notifications

```yaml
Email Alerts:
  SMTP Server: smtp.gmail.com:587
  Username: alerts@yourdomain.com
  Password: app-password
  
Alert Types:
  - Device Offline
  - Security Threats
  - High Bandwidth Usage
  - VPN Connection Issues
  - Port Forward Health Failures
```

### 2. Regular Monitoring Tasks

```yaml
Daily Checks:
  - Port forward health status
  - Bandwidth usage trends
  - Security event log
  
Weekly Checks:
  - Firmware updates
  - Configuration backup
  - Performance metrics review
  
Monthly Checks:
  - Security rule effectiveness
  - QoS performance analysis
  - VPN usage patterns
```

## Troubleshooting

### Common Issues

#### Port Forwarding Not Working
```yaml
Checks:
  1. Verify port forward rule is enabled
  2. Check destination server is responding
  3. Verify firewall rules allow traffic
  4. Test internal connectivity first
  5. Check health monitoring status
  
Diagnostic Tools:
  - Network scan from UCG Ultra
  - Port connectivity test
  - Packet capture analysis
```

#### DNS Resolution Issues
```yaml
Checks:
  1. Verify AdGuard Home is running
  2. Check DHCP DNS settings
  3. Test DNS resolution manually
  4. Verify DNS force setting
  
Commands:
  nslookup google.com 192.168.1.20
  dig @192.168.1.20 lab.local
```

#### VPN Connection Problems
```yaml
Checks:
  1. Verify WireGuard service status
  2. Check client configuration
  3. Verify firewall allows VPN port
  4. Test local network connectivity
  
Diagnostic Steps:
  1. Test VPN from internal network
  2. Check VPN logs in console
  3. Verify client device network settings
  4. Test with different client device
```

## Performance Optimization

### 1. Hardware Acceleration

```yaml
Offloading Features:
  Hardware NAT: Enabled
  Hardware Acceleration: Enabled
  Flow Control: Enabled
  
Limitations:
  - DPI may reduce hardware acceleration
  - Complex firewall rules impact performance
  - VPN traffic not hardware accelerated
```

### 2. Resource Monitoring

```yaml
Key Metrics:
  - CPU Usage: <80% average
  - Memory Usage: <70% average
  - Network Throughput: Monitor saturation
  - Temperature: Keep below warning levels
  
Optimization:
  - Disable unused features
  - Optimize firewall rule order
  - Regular firmware updates
  - Clean configuration periodically
```

## Next Steps

Once basic configuration is working:

1. **Deploy Infrastructure Traefik**: Set up load balancing and SSL termination
2. **Configure Monitoring**: Deploy Prometheus/Grafana for detailed metrics
3. **Add Backup Solutions**: Implement automated backup strategies
4. **Consider Enterprise Features**: Evaluate VLAN segmentation if needed
5. **Security Hardening**: Implement additional security measures

For advanced configuration with VLANs and enterprise features, see [Enterprise UCG Ultra Configuration](ucg-ultra-enterprise.md).