# Homepage Deployment Guide

Complete step-by-step guide to deploy Homepage on Robin's NixOS homelab.

## Prerequisites Checklist

- [ ] Docker and Docker Compose installed
- [ ] Traefik reverse proxy configured and running
- [ ] Home Assistant running at https://ha.robinsuter.com
- [ ] NixOS homelab DNS configured
- [ ] SSH access to homelab server
- [ ] GitHub account with git configured locally

## Step 1: SSH into Homelab Server

```bash
ssh robin@<homelab-ip>
# or if configured:
ssh robin@homelab.local
```

## Step 2: Clone Repository

```bash
# Clone the home-docker repository
git clone https://github.com/robinsuter/home-docker.git
cd home-docker
```

## Step 3: Prepare Configuration

### Create .env file

```bash
cp .env.example .env
nano .env
```

Update with your values:

```bash
# Get this from Home Assistant profile
HASS_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Your Traefik host (or leave as placeholder)
TRAEFIK_HOST=traefik.robinsuter.com

# Optional: Your other service credentials
PLEX_TOKEN=your_token
```

### Customize Dashboard

Edit `config/services.yaml` to match your services:

```bash
nano config/services.yaml
```

Replace placeholders with your actual services:

```yaml
- Home Automation:
    - Home Assistant:
        href: https://ha.robinsuter.com
        description: Home automation
        icon: home-assistant

- Media & Entertainment:
    - Jellyfin:
        href: https://jellyfin.robinsuter.com
        description: Media server
        icon: jellyfin
```

Edit `config/bookmarks.yaml` for quick links:

```bash
nano config/bookmarks.yaml
```

## Step 4: Configure Docker Network

Homepage needs to be on the same network as Traefik:

### Option A: Using Existing Network

If Traefik uses a named network (e.g., `traefik`), update `docker-compose.yml`:

```yaml
networks:
  homelab:
    external: true
    name: traefik  # or whatever your Traefik network is named
```

### Option B: Create Dedicated Network

```bash
docker network create homelab

# Connect Traefik container to the network
docker network connect homelab traefik
```

## Step 5: Configure Traefik

Update your Traefik configuration to include the Homepage routing rules:

### If using Docker labels (recommended):

The `docker-compose.yml` already includes Traefik labels. Just ensure:

1. Traefik has Docker socket access
2. Both containers are on same network
3. Traefik is watching Docker provider

### If using static/dynamic config:

Refer to `traefik-config.example.yml` and merge into your Traefik config directory.

## Step 6: Configure Internal DNS

Homepage should only be accessible internally on `home.robinsuter.com`.

### Option A: Using Pi-hole or AdGuard Home

```
home.robinsuter.com → 192.168.1.100  (your NixOS homelab IP)
```

### Option B: Update /etc/hosts (for individual machines)

```bash
# On client machines
echo "192.168.1.100 home.robinsuter.com" | sudo tee -a /etc/hosts
```

### Option C: Using dnsmasq

If running dnsmasq on your NixOS server:

```nix
# In your NixOS configuration.nix
services.dnsmasq = {
  enable = true;
  extraConfig = ''
    address=/home.robinsuter.com/192.168.1.100
  '';
};
```

## Step 7: Start Homepage

```bash
cd home-docker

# Start the container
docker-compose up -d

# Check logs
docker-compose logs -f homepage

# Verify it's running
docker-compose ps
```

## Step 8: Verify Deployment

### Local Access

```bash
# From homelab server or connected client
curl -k https://home.robinsuter.com

# Or open in browser:
# Internal: https://home.robinsuter.com
# Docker host: http://localhost:3000
```

### Check Traefik Routing

```bash
# Verify Traefik dashboard shows homepage service
# Usually at: https://traefik.robinsuter.com:8080/dashboard/
```

### View Logs

```bash
# Check for any errors
docker-compose logs homepage

# Real-time logs
docker-compose logs -f homepage
```

## Step 9: Configure Widget Integrations

Once deployed, configure widgets in the dashboard:

### Home Assistant Widget

1. Log into Homepage
2. Services should already show Home Assistant link
3. To enable widget with real data:
   - Ensure HASS_TOKEN is set in .env
   - Update services.yaml with widget config:

```yaml
widget:
  type: homeassistant
  url: https://ha.robinsuter.com
  key: {{ env:HASS_TOKEN }}
```

### Docker Widget

Homepage automatically detects running containers:

```yaml
- Docker Status:
    widget:
      type: docker
      url: unix:///var/run/docker.sock
```

### Traefik Widget

```yaml
- Traefik:
    href: https://traefik.robinsuter.com:8080/dashboard/
    description: Reverse proxy
    widget:
      type: traefik
      url: http://traefik:8080
```

## Step 10: Secure External Access (Optional)

If you need external access, DO NOT expose directly. Instead:

### Option A: VPN

Only allow access through VPN (WireGuard, Tailscale, etc.)

### Option B: Basic Authentication

Add to Traefik middleware:

```yaml
homepage-auth:
  basicAuth:
    users:
      - "robin:$apr1$hashed_password"
```

### Option C: OAuth2 Proxy

Use oauth2-proxy in front of Homepage for external OIDC/OAuth2 authentication.

## Troubleshooting

### Container fails to start

```bash
# Check Docker logs
docker logs homepage

# Verify volumes exist
ls -la config/

# Check Docker daemon
docker ps

# Check network
docker network ls
docker network inspect homelab
```

### Can't reach dashboard

1. Verify container is running: `docker ps | grep homepage`
2. Check DNS resolution: `nslookup home.robinsuter.com`
3. Verify Traefik can reach container: `docker exec traefik curl http://homepage:3000`
4. Check firewall: Is 443 (HTTPS) open internally?
5. Check Traefik logs: `docker logs traefik`

### Widgets not showing data

1. Verify authentication tokens: `grep HASS_TOKEN .env`
2. Check container can reach services: `docker exec homepage curl https://ha.robinsuter.com`
3. Review Homepage logs: `docker-compose logs homepage`
4. Ensure service URLs are correct in services.yaml

### Port conflicts

Homepage defaults to port 3000. If that's in use:

```yaml
# In docker-compose.yml
ports:
  - "3001:3000"  # Change host port to 3001

# Or use Traefik internal port only (remove ports section)
```

## Maintenance

### Update Homepage

```bash
# Pull latest image
docker-compose pull homepage

# Restart service
docker-compose up -d homepage
```

### Backup Configuration

```bash
# Backup config directory
tar -czf homepage-config-backup.tar.gz config/

# Or version control it:
git add config/
git commit -m "Update dashboard configuration"
git push
```

### View Statistics

```bash
# Container resource usage
docker stats homepage

# Detailed container info
docker inspect homepage
```

## Next Steps

1. ✅ Dashboard running
2. Add more services as you discover them
3. Configure widgets for real-time data
4. Set up proper authentication if external access needed
5. Document your services in bookmarks
6. Customize theme and appearance

## Support Resources

- [Homepage Docs](https://gethomepage.dev/)
- [Docker Docs](https://docs.docker.com/)
- [Traefik Docs](https://doc.traefik.io/)
- [Home Assistant Docs](https://www.home-assistant.io/)

## Post-Deployment Checklist

- [ ] Dashboard accessible at https://home.robinsuter.com
- [ ] Services are populated and link correctly
- [ ] Home Assistant widget shows real data
- [ ] Only internal access allowed (verify external access denied)
- [ ] Configuration backed up
- [ ] SSL certificate valid and auto-renewing
- [ ] Logs are clean (no errors in docker-compose logs)
- [ ] Docker resource usage is reasonable

---

**Deployment Date**: _______________  
**Deployed By**: _______________  
**Notes**: 

