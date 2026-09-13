# AHHS: A Humble Home Server

Personal production environment on **Proxmox VE**. Segmented LXC nodes running Docker stacks for automation, media, telemetry, and local AI.

---

## Hardware

- **Host:** Dell Latitude E5430
- **CPU:** Intel Core i5 (2C/4T)
- **RAM:** 8GB DDR3
- **Storage:** 128GB SSD (Proxmox LVM-Thin) + 500GB HDD (Persistent Data / Media)
- **OS:** Proxmox VE 9.x

---

## Architecture

```text
Internet / Client
        │
        ▼
 LXC 100 (Gateway): Cloudflare Tunnel, Nginx Proxy Manager, Tailscale
        │
        ├─────────────────┬─────────────────┬─────────────────┐
        ▼                 ▼                 ▼                 ▼
  LXC 101 (Core)    LXC 102 (Media)   LXC 103 (Ops)     LXC 105 (AI Hub)
  Vaultwarden       Jellyfin          Portainer         LibreChat
  n8n               Arr Stack         Uptime Kuma       MongoDB
  Portfolio         qBittorrent       pve-dashboard     Syncthing
                    Samba Shares                        MCP Tools
```

---

## Nodes & Services

| Node | Role | Services | Access |
| :--- | :--- | :--- | :--- |
| **`lxc100-gateway`** | Ingress | `Nginx Proxy Manager` · `Cloudflare Tunnel` · `Tailscale` | Public / Tailnet |
| **`lxc101-core`** | Internal Apps | `Vaultwarden` · `n8n` · `Portfolio` | Reverse Proxy / Tunnel |
| **`lxc102-media`** | Media & Storage | `Jellyfin` · `Radarr` · `Sonarr` · `Prowlarr` · `qBittorrent` · `Jellyseerr` · `Samba` | LAN / Tailscale |
| **`lxc103-ops`** | Monitoring | `pve-dashboard` · `Uptime Kuma` · `Portainer` | Internal / Tailnet |
| **`lxc105-aihub`** | AI & Notes | `LibreChat` · `MongoDB` · `Syncthing` · `MCP Filesystem` | Internal / Tailnet |

---

## Notes

- **Isolation:** Unprivileged LXCs separate public web services from credentials and internal storage.
- **Security:** Zero open inbound router ports; external access via Cloudflare Tunnels and Tailscale mesh.
- **Monitoring:** Custom React dashboard (`pve-dashboard`) pulls live telemetry from Proxmox, Portainer, and Uptime Kuma APIs.
- **Knowledge Sync:** Real-time peer-to-peer sync between local notes and self-hosted LibreChat via Syncthing.

---

## Links

- Dashboard: [charlesterrenal/orbit-dashboard](https://github.com/charlesterrenal/orbit-dashboard)
- Profile: [charlesterrenal/charlesterrenal](https://github.com/charlesterrenal/charlesterrenal)
- Website: [charlesterrenal.com](https://charlesterrenal.com)
