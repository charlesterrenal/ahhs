# My Homelab Documentation

My personal production environment running on **Proxmox VE**, specialized in network security, workflow automation, and self-hosted service development.
## Hardware

- **Model**: Dell Latitude E5430
- **RAM**: 8GB DDR3 (upgrade to 16GB+ planned)
- **Storage**: 128GB SSD (System) + 500GB HDD (Storage/Media)
- **OS**: Proxmox VE 9.1.1
- **Purpose**: Hosting containerized production services
## Services

| **Service**           | **Purpose**                            | **Access Method**       |
| --------------------- | -------------------------------------- | ----------------------- |
| **Portfolio Site**    | Personal website (charlesterrenal.com) | Public (Cloudflare/NPM) |
| **CompuTeRent**       | DePIN Stellar blockchain platform      | Public (Cloudflare/NPM) |
| **Vaultwarden**       | Password manager                       | Public (Cloudflare/NPM) |
| **Jellyfin**          | Media server                           | Via NPM reverse proxy   |
| **Samba**             | Network file shares                    | LAN/Tailscale           |
| **NPM**               | Nginx Proxy Manager                    | Public (80, 81, 443)    |
| **Cloudflare Tunnel** | Secure ingress gateway                 | Public                  |

## Network Architecture
```
Internet
    ↓
Cloudflare Tunnel (LXC 100)
    ↓
Nginx Proxy Manager (192.168.254.x:80/81/443)
    ↓
LXC Containers (192.168.254.x/y:PORTS)
    ↓
Tailscale VPN (Remote host & subnet access)
```
## Security Setup
- **Defense in Depth**: Isolated Gateway LXC (100) from Application LXC (101).
- **Remote Access**: Tailscale mesh VPN with subnet routing for secure management.
- **Web Services**: Cloudflare Tunnels for public ingress without opening router ports.
- **Privilege Level**: Services run in unprivileged Ubuntu 24.04 LXC containers.
## Key Learnings
### 1. Proxmox LXC Management
Managing services via LXC provides near-native performance compared to VMs.
- **LXC 100 (Gateway)**: Handles all edge traffic.
- **LXC 101 (Core)**: Dedicated to web apps like your Stellar project.
- **LXC 102 (Media)**: Specialized for storage and hardware passthrough.
### 2. Hardware Passthrough
Setting up **Intel GPU passthrough** for Jellyfin (`/dev/dri`) was crucial for keeping CPU usage low during video transcoding on the i5-3210M.
### 3. Tailscale Subnet Routing
Rather than installing Tailscale on every container, advertising the `192.168.254.0/24` route from the Proxmox host allows remote access to the entire cluster while keeping containers "clean".
## Useful Commands
### Proxmox Host Management
```
# List all containers
pct list

# Enter a container's shell directly
pct enter 101

# Start/Stop container
pct start 102
pct stop 102
```
### Network & Logs
```
# Check Tailscale routing status
tailscale status

# View Nginx Proxy Manager logs (in LXC 100)
journalctl -u nginx -f

# Verify disk mount for media
df -h | grep /mnt/storage
```
## Why This Setup?
1. **Professional Growth**: Building a production-grade environment for your portfolio.
2. **Privacy**: Self-hosting **Vaultwarden** to keep credentials off third-party clouds.
3. **Media Independence**: **Jellyfin** provides a private alternative to streaming services.
4. **Blockchain Development**: Hosting **CompuTeRent** to explore DePIN and Stellar.
## Notes
- This setup migrated from a *bare-metal Ubuntu server* to Proxmox VE to allow for better resource isolation and snapshot-based backups.
- Utilizing a laptop provides a built-in UPS (battery) and low power draw, making it an ideal 24/7 "Humble Home Server".
