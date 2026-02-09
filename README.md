# Homepage Dashboard for Robin's Homelab

A self-hosted [Homepage](https://gethomepage.dev/) dashboard for Robin's NixOS homelab, providing a clean, customizable interface to access and monitor all homelab services.

## Features

- 🎨 Beautiful, customizable dashboard interface
- 🏠 Integrated with Home Assistant
- 🐳 Docker-based deployment
- 🔒 Internal-only access (home.robinsuter.com)
- 📊 Real-time service monitoring widgets
- 🎯 Quick links to common services and bookmarks
- 🌙 Light/dark theme support
- 📱 Responsive mobile design

## Prerequisites

- Docker and Docker Compose installed
- NixOS homelab with reverse proxy (Traefik) configured
- Home Assistant running at https://ha.robinsuter.com
- Internal DNS resolution for home.robinsuter.com (or hosts file entry)

## Quick Start

### 1. Clone and Setup

```bash
git clone <repo-url> home-docker
cd home-docker
cp .env.example .env
```

### 2. Configure Environment Variables

Edit `.env` with your specific values:

```bash
HASS_TOKEN=your_home_assistant_token
TZ=UTC
```

To get your Home Assistant token:
1. Log in to Home Assistant
2. Go to Profile (bottom left) → Long-Lived Access Tokens
3. Create a new token and copy it

### 3. Start the Container

```bash
docker-compose up -d
```

Homepage will be available at: `http://localhost:3000` (or `https://home.robinsuter.com` if configured in Traefik)

### 4. Verify it's Running

```bash
docker-compose logs homepage
```

## Configuration

### Dashboard Services

Edit `config/services.yaml` to add or modify service tiles:

```yaml
- Home Automation:
    - Home Assistant:
        href: https://ha.robinsuter.com
        description: Home automation and control
        icon: home-assistant
        widget:
          type: homeassistant
          url: https://ha.robinsuter.com
          key: {{ env:HASS_TOKEN }}
```

### Widgets

Homepage supports real-time data widgets for many services:

- **Home Assistant**: Entity states, climate, media players
- **Docker**: Container status and stats
- **Traefik**: Routing status and uptime
- **Proxmox**: VM and node status
- **Synology**: Storage usage and status
- **Plex**: Currently playing media
- **Monitors**: System metrics, uptime, etc.

See [Homepage Widgets Documentation](https://gethomepage.dev/configs/) for complete widget reference.

### Settings

Customize appearance in `config/settings.yaml`:

```yaml
title: Robin's Homelab
theme: auto  # or 'dark', 'light'
color: slate
cardBlur: md
```

### Bookmarks

Add quick-access bookmarks in `config/bookmarks.yaml`:

```yaml
- Developer:
    - GitHub:
        href: https://github.com
        icon: github
```

## Internal-Only Access Configuration

This dashboard is configured for **internal-only access** on `home.robinsuter.com`. To ensure it's not exposed externally:

### Traefik Configuration

Update your Traefik configuration to restrict access:

```yaml
labels:
  traefik.http.routers.homepage.rule: "Host(`home.robinsuter.com`)"
  # Add this middleware to allow only internal IPs
  traefik.http.routers.homepage.middlewares: "internal-ip-whitelist"
```

Add the middleware in Traefik config:

```yaml
middlewares:
  internal-ip-whitelist:
    ipallowlist:
      sourcerange:
        - "10.0.0.0/8"      # Private network range
        - "172.16.0.0/12"
        - "192.168.0.0/16"
```

### DNS Configuration

Ensure `home.robinsuter.com` only resolves internally:
- Use internal DNS server only (not external)
- Or add to local `/etc/hosts` file

## Docker Volume Permissions

To allow Homepage to read Docker socket:

```bash
# Grant read access to Docker socket
sudo chmod 666 /var/run/docker.sock

# Or add Homepage user to docker group
sudo usermod -aG docker $USER
```

## Troubleshooting

### Container won't start

```bash
docker-compose logs homepage
```

Check for:
- Port 3000 already in use
- Volume permission issues
- Missing config files

### Widgets not loading

1. Verify service URLs are correct
2. Check authentication tokens (HASS_TOKEN, API keys, etc.)
3. Ensure services are reachable from the Homepage container
4. Review Homepage logs: `docker-compose logs homepage`

### Can't access dashboard externally

This is intentional! The dashboard is internal-only. Access via:
- Internal network: `https://home.robinsuter.com`
- Docker host: `http://localhost:3000`
- Never expose externally to maintain security

If you need external access, implement proper authentication first.

## File Structure

```
home-docker/
├── docker-compose.yml          # Docker Compose configuration
├── .env.example               # Environment variables template
├── .gitignore                # Git ignore rules
├── README.md                  # This file
├── config/
│   ├── settings.yaml         # Dashboard appearance & behavior
│   ├── services.yaml         # Service tiles configuration
│   └── bookmarks.yaml        # Quick links
├── icons/                     # Custom icons (optional)
└── data/                      # Optional: persistent data
```

## Customization Examples

### Add Plex Media Server

```yaml
- Media:
    - Plex:
        href: https://app.plex.tv
        description: Media Server
        icon: plex
        widget:
          type: plex
          url: http://plex-host:32400
          key: {{ env:PLEX_TOKEN }}
```

### Add Monitoring Dashboard

```yaml
- Monitoring:
    - Grafana:
        href: https://grafana.robinsuter.com
        description: System monitoring
        icon: grafana
        widget:
          type: grafana
          url: https://grafana.robinsuter.com
          key: {{ env:GRAFANA_TOKEN }}
```

### Add Docker Stats Widget

```yaml
- Docker:
    - Container Status:
        widget:
          type: docker
          url: unix:///var/run/docker.sock
```

## Updates

To update Homepage to the latest version:

```bash
docker-compose pull homepage
docker-compose up -d
```

## Security Considerations

1. **Authentication**: Homepage itself doesn't provide built-in auth. Rely on:
   - Internal network access only
   - Traefik middleware (IP whitelist, basic auth)
   - Firewall rules

2. **API Tokens**: 
   - Store sensitive tokens in `.env` (not in git)
   - Use long-lived tokens with limited scopes
   - Rotate tokens periodically

3. **Docker Socket**:
   - Only mount if you trust the container
   - Consider using Docker API with certificate auth instead

4. **URLs**:
   - Use HTTPS for all external URLs
   - Verify certificate validity
   - Use internal URLs when available

## Resources

- [Homepage Documentation](https://gethomepage.dev/)
- [Homepage GitHub](https://github.com/gethomepage/homepage)
- [Homepage Widgets Reference](https://gethomepage.dev/configs/)
- [Homepage Services Examples](https://gethomepage.dev/configs/services/)

## Contributing

To improve this dashboard setup:
1. Test new widgets and services
2. Add configuration examples
3. Document lessons learned

## License

MIT - Feel free to use and modify for your homelab setup.

---

**Last Updated**: February 2026  
**Status**: Ready for deployment
