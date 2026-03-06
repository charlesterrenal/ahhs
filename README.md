# My Homelab Documentation

Personal homelab setup running on Ubuntu 24.04 LTS.

## 🖥️ Hardware

- **Model**: Dell Latitude E5430 (repurposed laptop)
- **CPU**: Intel i5-3210M (4 cores @ 3.1GHz)
- **RAM**: 8GB DDR3
- **OS**: Ubuntu 24.04 LTS
- **Purpose**: Self-hosting, learning AWS with LocalStack

## 🏠 Services

| Service | Purpose | Access Method |
|---------|---------|---------------|
| **LocalStack** | AWS cloud emulation | LAN/Tailscale |
| **n8n** | Workflow automation | Via NPM reverse proxy |
| **Jellyfin** | Media server | Via NPM reverse proxy |
| **Vaultwarden** | Password manager | Via NPM reverse proxy |
| **Portainer** | Docker management | LAN only |
| **Homepage** | Dashboard | LAN only |
| **NPM** | Nginx Proxy Manager | Public (80, 81, 443) |

## 🌐 Network Architecture
```
Internet
    ↓
Cloudflare Tunnel + DNS
    ↓
Nginx Proxy Manager (0.0.0.0:80/81/443)
    ↓
Internal Services (192.168.x.x:PORTS)
    ↓
Tailscale VPN (remote access)
```

## 🛡️ Security Setup

- **Firewall**: UFW enabled with default deny
- **Port Binding**: All services bound to LAN IP except reverse proxy
- **Remote Access**: Tailscale VPN for secure remote connections
- **Web Services**: Cloudflare Tunnel with SSL
- **Access Control**: Services behind reverse proxy with authentication

## 📚 Key Learnings

### 1. Docker Port Binding Security

Always bind to specific IPs, never `0.0.0.0` unless absolutely necessary:
```yaml
# ❌ Bad - Exposes to everyone
ports:
  - "8080:80"

# ✅ Good - LAN only
ports:
  - "192.168.x.x:8080:80"

# ✅ Best - Localhost only (access via reverse proxy)
ports:
  - "127.0.0.1:8080:80"
```

### 2. Security Audit Commands

Regular security checks:
```bash
# Check what's exposed to the internet
sudo ss -tulpn | grep '0.0.0.0' | grep docker-proxy

# Should only see reverse proxy ports (80, 81, 443)

# Check UFW status
sudo ufw status verbose

# Check running containers
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

### 3. LocalStack for AWS Learning

LocalStack provides a local AWS environment perfect for:
- Learning AWS services without costs
- Testing Lambda functions, S3, DynamoDB locally
- Developing cloud applications offline
- CI/CD testing pipelines

Access via:
- Local: `http://localhost:4566`
- LAN: `http://192.168.x.x:4566`
- Remote: Via Tailscale IP
- Web UI: `https://app.localstack.cloud`

## 🔧 Useful Commands

### Docker Management
```bash
# View all containers
docker ps -a

# Update all containers
cd ~/docker/service-name
docker compose pull
docker compose up -d

# Clean up unused images
docker system prune -a

# View logs
docker logs container-name -f
```

### System Monitoring
```bash
# Check disk usage
df -h

# Check memory
free -h

# Check running processes
htop

# Network connections
sudo ss -tulpn
```

## 📁 Repository Structure
```
homelab-docs/
├── README.md                    # This file
├── docs/
│   ├── SECURITY.md             # Security setup guide
│   ├── LOCALSTACK.md           # LocalStack configuration
│   └── SERVICES.md             # Service details
└── templates/
    └── docker-compose.example.yml  # Template compose file
```

## 🎯 Why This Setup?

This homelab serves multiple purposes:
1. **Learning**: Practice with AWS services via LocalStack
2. **Self-hosting**: Control my own data and services
3. **Automation**: n8n workflows for repetitive tasks
4. **Media**: Personal Jellyfin server for family
5. **Security**: Vaultwarden for password management

## 🔗 Resources

- [Awesome Self-Hosted](https://github.com/awesome-selfhosted/awesome-selfhosted)
- [r/homelab](https://reddit.com/r/homelab)
- [LocalStack Documentation](https://docs.localstack.cloud)
- [Docker Documentation](https://docs.docker.com)
- [Tailscale Documentation](https://tailscale.com/kb)

## 📝 Notes

This is a personal learning project running on repurposed hardware. All services are containerized with Docker for easy management and updates.

**Hardware Note**: Running on a laptop means:
- Lower power consumption (~30W vs 200W+ for server)
- Quieter operation (important for home use)
- Built-in UPS (battery backup)
- Compact footprint

## 🙏 Acknowledgments

- r/homelab community for inspiration
- Claude AI for troubleshooting assistance
- Various Docker and self-hosting guides
- Open source community

## 📄 License

MIT License - Feel free to use any part of this documentation for your own homelab!
