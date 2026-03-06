# Security Configuration

Complete security setup for the homelab.

## 🔥 Firewall Setup (UFW)

### Initial Configuration
```bash
# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (CRITICAL - don't lock yourself out!)
sudo ufw allow 22/tcp comment 'SSH'

# Allow from local network
sudo ufw allow from 192.168.254.0/24 comment 'LAN access'

# Allow Tailscale VPN
sudo ufw allow in on tailscale0 comment 'Tailscale VPN'

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

### Verify Firewall
```bash
# View all rules
sudo ufw status numbered

# Check recent blocks (if logging enabled)
sudo tail -f /var/log/ufw.log
```

## 🐳 Docker Port Binding Strategy

### Three Levels of Access

#### 1. Public (0.0.0.0) - Only Reverse Proxy
```yaml
# Only for Nginx Proxy Manager
ports:
  - "80:80"
  - "81:81"
  - "443:443"
```

#### 2. LAN (192.168.x.x) - Internal Services
```yaml
# For services accessed from home network
ports:
  - "192.168.x.x:8080:80"
  - "192.168.x.x:5678:5678"
```

#### 3. Localhost (127.0.0.1) - Maximum Security
```yaml
# For admin panels, databases, sensitive services
ports:
  - "127.0.0.1:9000:9000"
  - "127.0.0.1:3306:3306"
```

## 🔍 Security Audit Script

Create `~/security-check.sh`:
```bash
#!/bin/bash
echo "=== Docker Security Audit ==="
echo ""
echo "Ports on 0.0.0.0 (should only be reverse proxy):"
sudo ss -tulpn | grep '0.0.0.0' | grep docker-proxy | awk '{print $5}' | cut -d: -f2 | sort -n | uniq
echo ""
echo "Expected: 80, 81, 443"
echo ""
echo "UFW Status:"
sudo ufw status | head -10
echo ""
echo "Running Containers:"
docker ps --format "table {{.Names}}\t{{.Ports}}" | head -15
```

Run weekly:
```bash
chmod +x ~/security-check.sh
~/security-check.sh
```

## 🔐 Service-Specific Security

### Vaultwarden (Password Manager)
- Always behind HTTPS (NPM handles SSL)
- Strong admin token required
- Disable registration after initial setup
- Enable 2FA for all accounts

### Portainer (Docker Management)
- Bind to LAN IP only
- Strong admin password
- Enable HTTPS
- Restrict to admin users only

### n8n (Workflow Automation)
- Behind reverse proxy
- Enable authentication
- Restrict webhook access
- Use environment variables for credentials

### LocalStack
- LAN access only (or localhost + Tailscale)
- Use authentication token (Pro version)
- Don't expose to internet directly

## 🌐 Network Security

### Tailscale Setup
```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Authenticate
sudo tailscale up

# Check status
tailscale status
```

### Cloudflare Tunnel
- All web traffic through Cloudflare
- DDoS protection included
- SSL/TLS termination
- Access control via Cloudflare Access (optional)

## 📋 Security Checklist

Before deploying any new service:

- [ ] Reviewed port bindings in docker-compose.yml
- [ ] Used environment variables for secrets
- [ ] Bound to appropriate IP (not 0.0.0.0 unless necessary)
- [ ] Enabled authentication if available
- [ ] Configured HTTPS/SSL
- [ ] Added to firewall rules if needed
- [ ] Tested access from LAN
- [ ] Tested access from remote (Tailscale)
- [ ] Documented in services list
- [ ] Ran security audit script

## 🚨 Incident Response

If you suspect unauthorized access:

1. **Immediately**:
```bash
   # Block all incoming
   sudo ufw default deny incoming
   
   # Check active connections
   sudo ss -tunap
   
   # Check Docker logs
   docker logs container-name
```

2. **Investigate**:
```bash
   # Check auth logs
   sudo tail -100 /var/log/auth.log
   
   # Check UFW logs
   sudo tail -100 /var/log/ufw.log
   
   # Check failed login attempts
   sudo lastb
```

3. **Remediate**:
   - Change all passwords
   - Rotate API keys and tokens
   - Review firewall rules
   - Update all containers
   - Check for unauthorized containers

## 📊 Regular Maintenance

### Weekly
- Run security audit script
- Check for container updates
- Review logs for anomalies

### Monthly
- Review and rotate credentials
- Update UFW rules if needed
- Check disk space and clean up logs
- Test backup restoration

### Quarterly
- Full security review
- Update all documentation
- Review and remove unused services
- Test disaster recovery plan

## 🔗 Resources

- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [UFW Guide](https://help.ubuntu.com/community/UFW)
- [Tailscale Security](https://tailscale.com/security/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
