## 📋 Files

- **docker-compose.example.yml** - Complete docker-compose file with all services
- **.env.example** - Environment variables template

## 🚀 Quick Start

### 1. Copy the templates to your docker directory
```bash
# Create a new service directory
mkdir -p ~/docker/my-stack

# Copy templates
cp docker-compose.example.yml ~/docker/my-stack/docker-compose.yml
cp .env.example ~/docker/my-stack/.env
```

### 2. Edit the .env file
```bash
cd ~/docker/my-stack
nano .env
```

**Required changes:**
- `SERVER_IP` - Your server's LAN IP (e.g., 192.168.254.112)
- `TIMEZONE` - Your timezone (e.g., Asia/Manila)
- `BASE_DOMAIN` - Your domain name
- All service-specific tokens and passwords

**Generate secure passwords:**
```bash
# Generate random passwords/tokens
openssl rand -base64 32
```

### 3. Customize docker-compose.yml
```bash
nano docker-compose.yml
```

- Comment out services you don't need
- Adjust ports if there are conflicts
- Modify volume paths
- Update image versions if needed

### 4. Create data directories
```bash
# Let Docker create them automatically on first run, or:
mkdir -p npm_data letsencrypt n8n_data vaultwarden_data
mkdir -p jellyfin_config jellyfin_cache
mkdir -p homepage_config portainer_data localstack_volume
```

### 5. Deploy
```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Check status
docker compose ps
```

##Security Best Practices

### Port Binding

✅ **DO:**
- Bind reverse proxy (NPM) to `0.0.0.0:80/81/443`
- Bind user services to `${SERVER_IP}:PORT` (LAN only)
- Bind databases/admin tools to `127.0.0.1:PORT` (localhost only)

❌ **DON'T:**
- Expose all services to `0.0.0.0` (public internet)
- Use default passwords
- Skip the .env file

### Environment Variables

Always use `.env` file for:
- Passwords and tokens
- Domain names
- API keys
- Server IP addresses

### Firewall

Enable UFW before deploying:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow from 192.168.0.0/16
sudo ufw enable
```

## 📝 Service Descriptions

| Service | Port(s) | Binding | Purpose |
|---------|---------|---------|---------|
| NPM | 80, 81, 443 | Public | Reverse proxy with SSL |
| LocalStack | 4566, 4510-4559 | LAN | AWS emulator |
| n8n | 5678 | LAN | Workflow automation |
| Vaultwarden | 8081 | LAN | Password manager |
| Jellyfin | 8096, 8920, 7359, 1900 | LAN | Media server |
| Homepage | 3000 | LAN | Dashboard |
| Portainer | 9000, 9443 | LAN | Docker management |
| Cloudflare Tunnel | - | - | Secure tunnel |

## 🔧 Customization Tips

### Add a new service
```yaml
  my-service:
    image: myimage:latest
    container_name: my-service
    restart: unless-stopped
    ports:
      - "${SERVER_IP}:8080:80"
    environment:
      - TZ=${TIMEZONE}
    volumes:
      - ./my-service_data:/data
```

### Use named volumes instead of bind mounts
```yaml
volumes:
  npm_data:
    driver: local
  n8n_data:
    driver: local
```

### Create custom networks
```yaml
networks:
  proxy:
    driver: bridge
  internal:
    driver: bridge
    internal: true

services:
  npm:
    networks:
      - proxy
  
  my-app:
    networks:
      - proxy
      - internal
```

## 🆘 Troubleshooting

### Service won't start
```bash
# Check logs
docker compose logs service-name

# Check port conflicts
sudo ss -tulpn | grep PORT_NUMBER

# Rebuild container
docker compose up -d --force-recreate service-name
```

### Permission issues
```bash
# Fix ownership
sudo chown -R $USER:$USER ./service_data/

# Or use PUID/PGID in .env
PUID=1000
PGID=1000
```

### Can't access service from LAN
```bash
# Check firewall
sudo ufw status

# Check port binding
sudo ss -tulpn | grep docker-proxy

# Verify SERVER_IP is correct in .env
```

## 📚 Additional Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- Main homelab documentation: `../README.md`
- Security guide: `../docs/SECURITY.md`
