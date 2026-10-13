# Homelab

> A two-server homelab for media, home automation, storage, development, and monitoring.

The setup is split between **super potato**, the primary media and applications server, and
**dietpotato**, the secondary server for IoT, automation, and redundancy. Container image
references (`repository:tag`) are listed alongside each service in the inventories below.

## Contents

- [Overview](#overview)
- [Super potato](#device-1-super-potato)
- [Diet potato](#device-2-dietpotato)
- [Network architecture](#network-architecture)
- [Data flow and service relationships](#data-flow-and-service-relationships)
- [Backup and maintenance](#backup-and-maintenance)
- [Current status](#current-status)
- [Notes and unknowns](#notes--unknowns)
- [Service dependencies](#service-dependencies)

## Overview

The two servers run Docker containers for media management, automation, development tools,
and monitoring. Together, they provide service redundancy and functionality for personal
and family use.

### Network at a glance

```text
Internet
   |
   +--- Tailscale (VPN)
   |
   +--- Public IP
   |   +--- 0.0.0.0:8282 (Tautulli, Plex monitoring)
   |   +--- 0.0.0.0:7094 (Scrutiny, HDD monitoring)
   |   +--- 0.0.0.0:5216 (MySpeed, speed test)
   |   +--- 0.0.0.0:5055 (Caddy, jobboard reverse proxy)
   |   +--- 0.0.0.0:2222 (Gitea, SSH + HTTP)
   |   +--- 0.0.0.0:5678 (n8n, workflow automation)
   |   +--- 0.0.0.0:5006 (Actual Budget)
   |   +--- 0.0.0.0:5860 (DHT11 dashboard)
   |   +--- 0.0.0.0:1883 (Mosquitto, MQTT)
   |   +--- 0.0.0.0:5055 (Jellyseerr)
   |   +--- 0.0.0.0:7878 (Radarr)
   |   +--- 0.0.0.0:1970 (SwingMusic)
   |   +--- 0.0.0.0:3030 (Vert)
   |   +--- 0.0.0.0:10443 (Nextcloud)
   |   +--- 0.0.0.0:3004 (OpenSpeedTest)
   |   +--- 0.0.0.0:8582 (Downtify)
   |   +--- 0.0.0.0:4533 (Navidrome)
   |   +--- 0.0.0.0:9180 (Feishin)
   |   +--- 19132-19132/udp (Crafty)
   |   +--- 25500-25600 (Crafty)
   |   +--- 8112-8123 (Crafty)
   |   +--- 8111 (Crafty HTTPS)
   |   +--- 8686 (Lidarr)
   |   +--- 9666-9666, 8001-8000 (PyLoad)
   |   +--- 8000 (Portainer)
   |   +--- 8888 (Dozzle)
   |   +--- 8989 (Sonarr)
   |   +--- 2283 (Immich server)
   |   +--- 5432 (Immich Postgres)
   |   +--- 9696 (Prowlarr)
   |   +--- 8090 (Beszel)
   |   +--- 8510 (Luan)
   |   +--- 8008 (Credential Broker)
   |   +--- 8090 (Peanut)
   |   +--- 8181 (QBittorrent)
   |   +--- 6379 (Immich Redis)
   |
   +--- Tailscale (VPN) 2222→22, 3008→3000
   |
   +--- File network ports etc.
```

## Device 1: super potato

<p align="center">
  <img src="./potato.png" alt="Super potato tower server" width="300">
</p>

| System | Specification |
| --- | --- |
| **OS** | Debian |
| **CPU** | Intel i5-6500 |
| **RAM** | 16 GB |
| **Storage** | 15 TB total |
| **Role** | Primary server: media and applications |

### Container Services

#### Media & Entertainment
- **Tautulli** (`linuxserver/tautulli:2.17.1`)
  - Port: 8282 → 8181
  - Description: Plex monitoring & statistics
  - Status: Up 15 hours

- **Plex** (`linuxserver/plex:1.43.4`)
  - Description: Media server
  - Status: Up 18 hours

- **Jellyfin** (`linuxserver/jellyfin:10.11.10`)
  - Description: Alternative media server
  - Status: Up 18 hours

- **Jellyseerr** (`fallenbagel/jellyseerr:2.7.3`)
  - Port: 5055
  - Description: Media request manager
  - Status: Up 18 hours

- **SwingMusic** (`ghcr.io/swingmx/swingmusic:latest`)
  - Port: 1970
  - Description: Music streaming service
  - Status: Up 18 hours

- **Navidrome** (`deluan/navidrome:0.58.5`)
  - Port: 4533
  - Description: Music streaming (Subsonic)
  - Status: Up 18 hours

- **Feishin** (`ghcr.io/jeffvli/feishin:latest`)
  - Port: 9180
  - Description: Music client (desktop)
  - Status: Up 18 hours

- **Radarr** (`linuxserver/radarr:5.26.2`)
  - Port: 7878
  - Description: Movie manager
  - Status: Up 18 hours

- **Lidarr** (`linuxserver/lidarr:3.1.0`)
  - Port: 8686
  - Description: Music collection manager
  - Status: Up 18 hours

- **Sonarr** (`linuxserver/sonarr:4.0.15`)
  - Port: 8989
  - Description: TV series manager
  - Status: Up 18 hours

- **Prowlarr** (`linuxserver/prowlarr:1.37.0`)
  - Port: 9696
  - Description: Torrent indexer manager
  - Status: Up 18 hours

#### Storage & Backup
- **Scrutiny** (`ghcr.io/analogj/scrutiny:v0.9.5-omnibus`)
  - Port: 7094 → 8080
  - Description: HDD health / SMART monitoring
  - Status: Up 16 hours

- **Nextcloud** (`linuxserver/nextcloud:34.0.1`)
  - Port: 10443 → 443
  - Description: File sharing / cloud storage
  - Status: Up 18 hours

- **Actual Budget** (`actualbudget/actual-server:26.5.2`)
  - Port: 15006 → 5006
  - Description: Budget tracking
  - Status: Up 18 hours

- **OpenSpeedTest** (`openspeedtest/latest:v2.0.6`)
  - Port: 3004 → 3000
  - Description: Network speed testing
  - Status: Up 18 hours

- **Immich** (Photo Management)
  - Server (`ghcr.io/immich-app/immich-server:v2.4.1`) - Port 2283
  - Redis (`redis:6.2.20-alpine`) - Port 6379
  - Postgres (`ghcr.io/immich-app/postgres:14-vectorchord0.3.0-pgvectors0.2.0`) - Port 5432
  - Description: Photo management system
  - Status: All services Up 18 hours

#### Monitoring & Automation
- **MySpeed** (`germannewsmaker/myspeed:development`)
  - Port: 5216
  - Description: Network speed testing tool
  - Status: Up 18 hours

- **DHT11 Dashboard** (`uno-dashboard`)
  - Port: 5860
  - Description: DHT11 sensor dashboard
  - Status: Up 18 hours

- **Mosquitto** (`eclipse-mosquitto:2`)
  - Port: 1883
  - Description: MQTT broker
  - Status: Up 18 hours

- **QBittorrent** (`ghcr.io/hotio/qbittorrent:release-5.0.4`)
  - Port: 8181 → 8080
  - Description: Torrent client
  - Status: Up 18 hours

- **PyLoad** (`linuxserver/pyload-ng:0.5.0`)
  - Ports: 9666, 8001 → 8000
  - Description: Download manager
  - Status: Up 18 hours

- **EspHome** (`ghcr.io/esphome/esphome:2025.11.0`)
  - Description: ESP32/ESP8266 firmware management
  - Status: Up 18 hours

#### Container Orchestration & Tools
- **Portainer** (`portainer/portainer-ce:2.37.0-alpine`)
  - Ports: 8000, 9000, 9443
  - Description: Container management UI
  - Status: Up 18 hours

- **Dozzle** (`amir20/dozzle:v10.1.2`)
  - Port: 8888 → 8080
  - Description: Log viewer
  - Status: Up 18 hours

- **Docker Socket Proxy** (`tecnativa/docker-socket-proxy:latest`)
  - Port: 2375
  - Description: Docker socket proxy
  - Status: Up 18 hours

- **Beszel** (`henrygd/beszel:0.17.0`)
  - Port: 8090
  - Description: System monitoring dashboard
  - Status: Up 18 hours

- **Beszel Agent** (`c92f5efdd656`)
  - Description: System monitoring agent
  - Status: Up 18 hours

- **Music Assistant** (`ghcr.io/music-assistant/server:2.7.11`)
  - Description: Music orchestration service
  - Status: Up 18 hours

- **Playit** (`f9652ae6d6a3`)
  - Description: Tunnel agent
  - Status: Up 18 hours

#### Development & CI/CD
- **Gitea** (`gitea/gitea:1.27.3`)
  - Ports: 2222 → 22 (SSH), 3008 → 3000 (HTTP)
  - Description: Git server
  - Status: Up 18 hours

- **Gitea Act Runner** (`gitea/act_runner:latest`)
  - Description: Gitea Actions CI/CD runner
  - Status: Up 18 hours

- **n8n** (`n8nio/n8n:2.34.4`)
  - Port: 5678
  - Description: Workflow automation
  - Status: Up 18 hours

- **db-n8n** (`postgres:14.2`)
  - Port: 5432
  - Description: n8n database
  - Status: Up 18 hours

#### Third-party Applications
- **MediaHub** (`mediahub:local`)
  - Port: 8594 → 5055
  - Description: Media hub application
  - Status: Up 18 hours

- **HomeBox** (`ghcr.io/sysadminsmedia/homebox:latest`)
  - Port: 3100 → 7745
  - Description: IT asset inventory
  - Status: Up 18 hours

- **Crafty** (`registry.gitlab.com/crafty-controller/crafty-4:4.10.4`)
  - Ports: 5520-5550, 8000, 19132, 25500-25600, 8112-8123, 8111
  - Description: Minecraft server manager
  - Status: Up 18 hours

- **Jellyseerr** (`fallenbagel/jellyseerr:2.7.3`)
  - Port: 5055
  - Description: Media request manager
  - Status: Up 18 hours

#### LARP Agents (Multi-container Setup)
- **Njuguna** (`larp-agents:njuguna`)
  - Port: 8010
  - Description: LARP agent (njuguna)
  - Status: Up 18 hours

- **Luan** (`larp-agents:luan`)
  - Port: 8510
  - Description: LARP agent (luan)
  - Status: Up 18 hours

- **Credential Broker** (`larp-agents:broker`)
  - Port: 8000 (localhost)
  - Description: LARP credential broker
  - Status: Up 18 hours

#### Monitoring
- **Peanut** (`brandawg93/peanut:5.19.1`)
  - Port: 8084 → 8080
  - Description: UPS monitoring UI
  - Status: Up 18 hours

- **ESPHOME** (`ghcr.io/esphome/esphome:2025.11.0`)
  - Description: ESP32/ESP8266 firmware
  - Status: Up 18 hours

#### Utility & Integration
- **jobboard-app** (`jobboard-app`)
  - Port: 9032 → 3000
  - Description: Job board frontend
  - Status: Up 18 hours

- **jobboard-worker** (`jobboard-worker`)
  - Port: 3000
  - Description: Job board background worker
  - Status: Up 18 hours

- **jobboard-caddy-1** (`caddy:2-alpine`)
  - Port: 9092 → 80
  - Description: Job board reverse proxy
  - Status: Up 18 hours

- **jobboard-db-1** (`postgres:16-alpine`)
  - Port: 5432
  - Description: Job board database
  - Status: Up 18 hours

#### Other
- **POURLARR** (`linuxserver/prowlarr:1.37.0`)
  - Port: 9696
  - Description: Torrent indexer manager
  - Status: Up 18 hours

## Device 2: dietpotato

<p align="center">
  <img src="./small-potato.png" alt="Diet potato laptop server" width="360">
</p>

| System | Specification |
| --- | --- |
| **CPU** | Intel i5-3320M |
| **RAM** | 6 GB |
| **Storage** | 500 GB |
| **Role** | Secondary server: IoT and redundancy |

### Container Services

#### Home Automation & IoT
- **Home Assistant** (`ghcr.io/home-assistant/home-assistant:2026.5.1`)
  - Description: Home automation hub
  - Status: Up 7 minutes

- **ESPHome** (`ghcr.io/esphome/esphome:2025.11.0`)
  - Description: ESP32/ESP8266 firmware
  - Status: Up 18 hours

- **Peanut** (`brandawg93/peanut:5.19.1`)
  - Port: 9191 → 9191
  - Description: UPS monitoring UI
  - Status: Up 7 minutes

#### File Management
- **File Browser** (`filebrowser/filebrowser:v2.23.0`)
  - Port: 10180 → 80
  - Description: File manager UI
  - Status: Up 7 minutes

#### Media Services (Duplicate/Subset of super potato)
- **Jellyfin** (`linuxserver/jellyfin:latest`)
  - Ports: 7359, 8096, 1901→1900, 8921→8920
  - Description: Media server
  - Status: Up 7 minutes

- **Navidrome** (`deluan/navidrome:0.55.1`)
  - Description: Music streaming
  - Status: Up 7 minutes

- **QBittorrent** (`ghcr.io/hotio/qbittorrent:release-5.0.4`)
  - Port: 8181 → 8080
  - Description: Torrent client
  - Status: Up 7 minutes

- **Sonarr** (`linuxserver/sonarr:4.0.15`)
  - Port: 8989
  - Description: TV series manager
  - Status: Up 7 minutes

- **Prowlarr** (`linuxserver/prowlarr:1.37.0`)
  - Port: 9696
  - Description: Torrent indexer manager
  - Status: Up 7 minutes

- **Plex** (`linuxserver/plex:1.43.1`)
  - Description: Media server
  - Status: Up 7 minutes

#### Infrastructure
- **Portainer** (`portainer/portainer-ce:2.37.0-alpine`)
  - Ports: 8000, 9000, 9443
  - Description: Container management UI
  - Status: Up 7 minutes

- **Portainer Agent** (`portainer/agent:2.37.0`)
  - Port: 9001
  - Description: Portainer agent
  - Status: Up 7 minutes

- **Dozzle** (`amir20/dozzle:v10.1.2`)
  - Port: 8888 → 8080
  - Description: Log viewer
  - Status: Up 7 minutes

#### Automation Tools
- **Job Scout** (`job-scout-job-scout`)
  - Port: 8494
  - Description: Job search automation
  - Status: Up 7 minutes

- **Tech Scout** (`tech-scout-tech-scout`)
  - Port: 8493
  - Description: Tech news aggregation
  - Status: Up 7 minutes

#### VPN & Network
- **Tailscale** (`tailscale/tailscale:v1.84.3`)
  - Description: VPN mesh networking
  - Status: Up 42 seconds

#### Utility & Home Services
- **HomeBox** (`ghcr.io/sysadminsmedia/homebox:latest`)
  - Port: 3100 → 7745
  - Description: IT asset inventory
  - Status: Up 7 minutes

- **Plex** (`linuxserver/plex:1.43.1`)
  - Description: Media server
  - Status: Up 7 minutes

#### Monitoring
- **Beszel Agent** (`c7ad60c6e461`)
  - Description: System monitoring agent
  - Status: Up 7 minutes

#### Software Versions (Optional)
- **Plex**: 1.43.1 (vs 1.43.4 on super potato - newer on primary)
- **Jellyfin**: latest (vs 10.11.10 on super potato - latest on secondary)
- **Peanut**: 5.19.1 (vs 5.19.1 on super potato - same)
- **QBittorrent**: release-5.0.4 (vs release-5.0.4 on super potato - same)
- **Sonarr**: 4.0.15 (vs 4.0.15 on super potato - same)
- **Prowlarr**: 1.37.0 (vs 1.37.0 on super potato - same)

## Network architecture

### DNS Names
- **super potato**: The main server
- **dietpotato**: The secondary/IoT server

### Ports and Services

#### Public/Firewall

**Exposed Services (super potato):**
- **Tautulli**: 0.0.0.0:8282:8181/TCP (Plex monitoring)
- **Scrutiny**: 0.0.0.0:7094:8080/TCP (HDD monitoring)
- **MySpeed**: 0.0.0.0:5216:5216/TCP (Speed testing)
- **Caddy (jobboard)**: 0.0.0.0:9092:80/TCP (Reverse proxy)
- **Gitea**: 0.0.0.0:2222:22/TCP (SSH), 0.0.0.0:3008:3000/TCP (HTTP)
- **n8n**: 0.0.0.0:5678:5678/TCP (Workflow automation)
- **Actual Budget**: 0.0.0.0:15006:5006/TCP (Budget tracking)
- **DHT11 Dashboard**: 0.0.0.0:5860:5860/TCP (Sensor dashboard)
- **Mosquitto**: 0.0.0.0:1883:1883/TCP (MQTT broker)
- **Jellyseerr**: 0.0.0.0:5055:5055/TCP (Media requests)
- **Radarr**: 0.0.0.0:7878:7878/TCP (Movie management)
- **SwingMusic**: 0.0.0.0:1970:1970/TCP (Music streaming)
- **Vert**: 0.0.0.0:3030:80/TCP (Web analytics)
- **Nextcloud**: 0.0.0.0:10443:443/TCP (File sharing)
- **OpenSpeedTest**: 0.0.0.0:3004:3000/TCP (Network speed test)
- **Downtify**: 0.0.0.0:8582:8000/TCP (Spotify backup)
- **Navidrome**: 0.0.0.0:4533:4533/TCP (Music streaming)
- **Feishin**: 0.0.0.0:9180:9180/TCP (Music client)
- **Crafty**: Multiple UDP/TCP ports (Minecraft server)
- **Lidarr**: 0.0.0.0:8686:8686/TCP (Music management)
- **PyLoad**: 0.0.0.0:9666:9666/TCP, 0.0.0.0:8001:8000/TCP (Download manager)
- **Portainer**: 0.0.0.0:8000:8000/TCP, 0.0.0.0:9000:9000/TCP, 0.0.0.0:9443:9443/TCP
- **Dozzle**: 0.0.0.0:8888:8080/TCP (Log viewer)
- **Sonarr**: 0.0.0.0:8989:8989/TCP (TV management)
- **Immich Server**: 0.0.0.0:2283:2283/TCP (Photo management)
- **Immich Redis**: 0.0.0.0:6379:6379/TCP (Immich cache)
- **Immich Postgres**: 0.0.0.0:5432:5432/TCP (Immich database)
- **Prowlarr**: 0.0.0.0:9696:9696/TCP (Indexer manager)
- **Beszel**: 0.0.0.0:8090:8090/TCP (Monitoring)
- **QBittorrent**: 0.0.0.0:8181:8080/TCP (Torrent client)
- **Peanut**: 0.0.0.0:8084:8080/TCP (UPS monitoring)

**Exposed Services (dietpotato):**
- **File Browser**: 0.0.0.0:10180:80/TCP (File management)
- **Jellyfin**: 0.0.0.0:7359:7359/TCP, 0.0.0.0:8096:8096/TCP, 0.0.0.0:1901:1900/TCP, 0.0.0.0:8921:8920/TCP
- **Peanut**: 0.0.0.0:9191:9191/TCP (UPS monitoring)
- **QBittorrent**: 0.0.0.0:8181:8080/TCP (Torrent client)
- **Sonarr**: 0.0.0.0:8989:8989/TCP (TV management)
- **Prowlarr**: 0.0.0.0:9696:9696/TCP (Indexer manager)
- **Portainer**: 0.0.0.0:8000:8000/TCP, 0.0.0.0:9000:9000/TCP, 0.0.0.0:9443:9443/TCP
- **Portainer Agent**: 0.0.0.0:9001:9001/TCP
- **Dozzle**: 0.0.0.0:8888:8080/TCP (Log viewer)
- **Job Scout**: 0.0.0.0:8494:8494/TCP (Job automation)
- **Tech Scout**: 0.0.0.0:8493:8493/TCP (Tech aggregation)

### Internal Networking

**super potato internal:**
- **Docker Network**: Default bridge
- **Services communicate via container names**
- **Immich stack**: immich-server ↔ immich-redis ↔ immich-postgres
- **Jobboard stack**: jobboard-app ↔ jobboard-worker ↔ jobboard-db ↔ jobboard-caddy
- **LARP agents**: njuguna, luan, credential-broker

**dietpotato internal:**
- **Docker Network**: Default bridge
- **Services communicate via container names**

### VPN/Mesh Networking

- **Tailscale**: Both servers connected via Tailscale VPN mesh
  - super potato: Tailscale interface available
  - dietpotato: Tailscale interface available
  - Allows secure internal communication between servers and from internet to services

## Data flow and service relationships

### Primary Services (super potato)

#### Media Stack
1. **Plex** → **Tautulli** (monitoring data)
2. **Jellyfin** → **Jellyseerr** (request management)
3. **All media services** → **Crafty** (potential backup/storage)
4. **Music services** (SwingMusic, Navidrome, Feishin) → **PyLoad** (content acquisition)

#### Monitoring & Infrastructure
1. **Scrutiny** → **Beszel** (system health integration)
2. **QBittorrent** ↔ **Peanut** (UPS monitoring for downloads)
3. **EspHome** ↔ **Home Assistant** (dietpotato) (sensor data flow)
4. **Immich** (super potato) ↔ **FileBrowser** (dietpotato) (photo access)

#### Automation & Integration
1. **Home Assistant** (dietpotato) → **Portainer** (container management)
2. **Tailscale** enables cross-server communication
3. **Gitea** (super potato) ↔ **Gitea Act Runner** (CI/CD)
4. **n8n** workflows can trigger services on both servers

### Redundancy & Backup

#### Service Duplication
- **Plex**: Both servers (different versions)
- **Jellyfin**: Both servers (different versions)
- **Navidrome**: Both servers (different versions)
- **QBittorrent**: Both servers (same version)
- **Sonarr**: Both servers (same version)
- **Prowlarr**: Both servers (same version)

#### Purpose of Duplication
- Primary/backup for critical services
- Different content libraries on each server
- Load distribution
- Failover capability

## Backup and maintenance

### Backup Strategies
1. **Docker Volumes**: Most containers use named volumes for persistence
2. **Database Backups**:
   - PostgreSQL instances (jobboard-db, db-n8n, immich-postgres)
   - Redis (Immich cache)
3. **File Storage**:
   - Nextcloud volumes (cloud file storage)
   - Plex/Jellyfin libraries (local NAS storage)
4. **Configuration Files**: Most services store configs in volumes

### Maintenance Tools
1. **Portainer**: Container management interface
2. **Dozzle**: Log monitoring
3. **Tautulli**: Plex monitoring
4. **Scrutiny**: HDD health monitoring
5. **Home Assistant**: Home automation & service monitoring

### Recommendations

#### Regular Maintenance
1. **Weekly**: Check service health via Portainer
2. **Monthly**: Review and backup database dumps
3. **Quarterly**: Update container images
4. **As needed**: Clean up unused Docker volumes

#### Monitoring Setup
1. **Immich**: Photo management with Redis cache
2. **Beszel**: System monitoring dashboard
3. **EspHome**: IoT device monitoring
4. **Home Assistant**: Comprehensive home automation

#### Security Considerations
1. **Tailscale**: VPN mesh networking
2. **Docker Socket Proxy**: Secure Docker access
3. **Firewalled exposed ports**: Limited public access
4. **Internal network segmentation**: Services communicate via Docker network

## Current Status

### All Services Running (as of last check)
- **super potato**: 49 containers running
- **dietpotato**: 17 containers running
- **Total**: 66 containers

### Health Status
- **Immich stack**: All healthy
- **Gitea**: All healthy
- **jobboard stack**: All healthy
- **LARP agents**: All healthy
- **Crafty**: Healthy
- **Home Assistant**: Up 7 minutes

## Notes and unknowns

### Pi-hole and NUT
- **Pi-hole**: Mentioned as running on dietpotato for DNS ad-blocking, but no container details provided
- **NUT**: Mentioned as running on dietpotato for UPS monitoring, but no container details provided

### Lan Cache
- **Lan Cache**: Mentioned as running on super potato for game downloads, but no container details provided

### Version Inconsistencies
- **Plex**: super potato (1.43.4) vs dietpotato (1.43.1) - super potato is newer
- **Jellyfin**: super potato (10.11.10) vs dietpotato (latest) - dietpotato is newer
- **Navidrome**: super potato (0.58.5) vs dietpotato (0.55.1) - super potato is newer

## Service Dependencies

### Service Groups
1. **Media Group**: Plex, Jellyfin, Jellyseerr, Radarr, Sonarr, Lidarr
2. **Music Group**: SwingMusic, Navidrome, Feishin, Music Assistant
3. **Download Group**: QBittorrent, PyLoad, Home Assistant (autodiscover)
4. **Monitoring Group**: Scrutiny, Beszel, EspHome, Peanut
5. **Infrastructure Group**: Portainer, Dozzle, Docker Socket Proxy, Tailscale
6. **Development Group**: Gitea, n8n, Gitea Act Runner
7. **Asset Management Group**: Nextcloud, Immich, FileBrowser
8. **Automation Group**: Home Assistant, n8n, jobboard services

### Cross-Server Dependencies
1. **Media redundancy**: Both servers can serve media
2. **Service backup**: Secondary server can take over if primary fails
3. **Network**: Tailscale enables communication between servers
4. **Monitoring**: Beszel monitoring on both servers
5. **File access**: Cross-server file sharing via Nextcloud/Immich

This document provides a comprehensive overview of the homelab infrastructure, including hardware specifications, running containers, network architecture, data flows, and maintenance strategies. The setup appears to be well-organized with redundancy and comprehensive monitoring capabilities.
