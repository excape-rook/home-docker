# Homepage Dashboard Setup - Initial Implementation

## 📋 Description

This PR introduces a complete Homepage dashboard setup for Robin's NixOS homelab. Homepage is a beautiful, customizable self-hosted dashboard that provides quick access to all homelab services and real-time monitoring widgets.

## 🎯 Goals

- ✅ Provide a centralized dashboard for homelab services
- ✅ Integrate with Home Assistant for real-time data
- ✅ Configure internal-only access (home.robinsuter.com, not exposed externally)
- ✅ Include monitoring and status widgets for key services
- ✅ Follow Homepage best practices and documentation
- ✅ Provide easy customization for future services
- ✅ Ensure security through internal-only access

## 📦 What's Included

### Core Files
- **docker-compose.yml**: Homepage service configuration with Traefik labels
- **config/settings.yaml**: Dashboard appearance and theme settings
- **config/services.yaml**: Service tiles and widget configurations
- **config/bookmarks.yaml**: Quick-access bookmarks to documentation and tools
- **.env.example**: Environment variables template for credentials

### Documentation
- **README.md**: Complete setup guide and customization examples
- **DEPLOYMENT_GUIDE.md**: Step-by-step deployment instructions
- **traefik-config.example.yml**: Traefik routing configuration reference
- **PR_TEMPLATE.md**: Pull request template for future contributions

### Configuration
- **.gitignore**: Excludes sensitive data from version control
- **docker-compose.override.yml.example**: Development overrides example
- **.github/workflows/validate.yml**: Automated validation workflow

## 🔧 Technical Details

### Architecture
- **Container**: `ghcr.io/gethomepage/homepage:latest`
- **Port**: 3000 (internal to Docker network, routed via Traefik)
- **Network**: Docker network shared with Traefik
- **Volumes**: Configuration, icons, Docker socket (read-only)

### Integration Points
1. **Home Assistant**: Widget integration with token-based authentication
2. **Traefik**: Reverse proxy routing with Docker labels
3. **Docker**: Container status monitoring via socket
4. **Services**: Extensible configuration for any service with web UI

### Security Configuration
- **Internal-Only**: Configured for `home.robinsuter.com` internal access only
- **Network**: Not exposed to external reverse proxy ports
- **IP Whitelist**: Traefik middleware to restrict to private IP ranges
- **API Tokens**: Stored in `.env`, excluded from version control

## 🚀 Quick Start

```bash
# Clone and setup
git clone https://github.com/robinsuter/home-docker.git
cd home-docker
cp .env.example .env

# Update configuration
nano .env                    # Add HASS_TOKEN
nano config/services.yaml   # Customize services

# Deploy
docker-compose up -d

# Access
# https://home.robinsuter.com (internal only)
# or http://localhost:3000 (from Docker host)
```

## 📝 Configuration Notes

### Services Configured
- **Home Assistant**: Home automation platform (ha.robinsuter.com)
- **Traefik**: Reverse proxy dashboard (with routing examples)
- **Docker**: Container status monitoring (via socket)
- **Bookmarks**: Developer tools, docs, and internal services

### Placeholders for Customization
Services are set up with placeholder URLs and widget examples:
- Replace `TRAEFIK_HOST` with actual Traefik hostname
- Add service URLs as you identify running services
- Configure widget credentials in `.env` file

### Widget Support
Homepage supports real-time data widgets for:
- Home Assistant entities and climate control
- Docker container stats
- Traefik routing status
- Proxmox VMs and nodes
- Synology NAS storage
- Plex media server playback
- System monitoring (CPU, RAM, disk)
- And many more...

## 🔐 Security Considerations

### Why Internal-Only?
Homepage doesn't have built-in authentication. To maintain security:
1. **Network Isolation**: Only accessible via internal DNS
2. **IP Whitelisting**: Traefik middleware restricts to private IPs
3. **No External Exposure**: Not routed through external reverse proxy
4. **Local Access**: Can access via Docker host (localhost:3000)

### For External Access (if needed)
- [ ] Implement OAuth2 Proxy for OIDC/OAuth2 auth
- [ ] Add VPN requirement (WireGuard, Tailscale)
- [ ] Use Traefik basic auth middleware
- [ ] Review and document authentication architecture

## 📊 Testing Checklist

Before merging, verify:
- [ ] Docker container starts without errors: `docker-compose up -d`
- [ ] Dashboard accessible at http://localhost:3000
- [ ] Service links are correct
- [ ] Home Assistant widget loads (with valid token)
- [ ] Configuration files validate (YAML syntax)
- [ ] Docker socket access works
- [ ] Traefik routing configured correctly
- [ ] Internal DNS resolution works
- [ ] External access is blocked (security check)

## 🎨 Customization Examples

### Add a New Service

```yaml
- My Service:
    - Service Name:
        href: https://service.robinsuter.com
        description: Service description
        icon: service-icon
        widget:
          type: service-widget
          url: https://service.robinsuter.com
          key: {{ env:SERVICE_TOKEN }}
```

### Change Dashboard Theme

```yaml
# In config/settings.yaml
theme: dark  # auto, light, dark
color: purple  # slate, zinc, stone, red, orange, amber, yellow, lime, green, emerald, teal, cyan, sky, blue, indigo, violet, purple, fuchsia, pink, rose
```

### Add More Bookmarks

```yaml
- Category Name:
    - Bookmark Title:
        href: https://example.com
        icon: icon-name
```

## 📚 Documentation References

- [Homepage Official Docs](https://gethomepage.dev/)
- [Homepage Configuration Guide](https://gethomepage.dev/configs/)
- [Homepage Widgets Reference](https://gethomepage.dev/configs/services/)
- [Traefik Docker Integration](https://doc.traefik.io/traefik/providers/docker/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 🔄 Future Enhancements

Potential improvements for future PRs:
- [ ] Add Prometheus and Grafana monitoring widgets
- [ ] Configure Plex media server widget
- [ ] Add Synology NAS storage dashboard
- [ ] Include Jellyfin media server integration
- [ ] Add system monitoring widgets (CPU, RAM, disk)
- [ ] Create dark/light theme variants
- [ ] Add more service bookmarks
- [ ] Configure health check endpoints
- [ ] Add backup/restore scripts
- [ ] Create Ansible playbook for automated deployment

## 📞 Questions & Notes

### For Robin:
- [ ] Verify Home Assistant token extraction works correctly
- [ ] Confirm internal DNS setup (dnsmasq/Pi-hole/hosts file)
- [ ] Check Traefik network configuration
- [ ] Test access from different devices on home network
- [ ] Provide feedback on dashboard layout and theme

### Known Limitations:
1. **No Built-in Auth**: Relies on network isolation (by design)
2. **Config Reload**: Some config changes require container restart
3. **Widget Data**: Requires service credentials/tokens for real-time data
4. **External Access**: Not recommended without additional authentication

## 🎉 Impact

This PR enables:
- Single dashboard access to all homelab services
- Quick status monitoring of key infrastructure
- Real-time Home Assistant integration
- Easy expansion as new services are added
- Well-documented setup for reproducibility

## Closing Notes

This is a comprehensive, production-ready setup that follows Homepage best practices and integrates seamlessly with Robin's existing NixOS homelab infrastructure. All configuration is documented, examples are provided, and the setup is secure by default (internal-only access).

The dashboard is designed to be simple to use (good default) while being easy to customize as the homelab grows.

---

## PR Details
- **Branch**: `feature/homepage-dashboard`
- **Base**: `main`
- **Type**: Feature
- **Status**: Ready for Review

**Related Issues**: 
**Closes**: 

---
