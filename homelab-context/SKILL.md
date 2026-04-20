---
name: homelab-context
description: Load Martin's homelab infrastructure context. Auto-invoke when the conversation touches networking, VLANs, DNS, VPN, Docker, TrueNAS, OPNsense, HAProxy, SSH, services (Jellyfin, Sonarr, Radarr, etc.), Bitcoin node, Debian workstation, firewall rules, storage layout, or any homelab infrastructure topic.
user-invocable: false
---

# Homelab Infrastructure Context

## Network Topology

| VLAN | Subnet | Purpose |
|---|---|---|
| LAN | 10.54.10.0/24 | Client devices, workstations, Apple devices |
| SERV | 10.54.20.0/24 | TrueNAS and server workloads |
| BTC | 10.54.30.0/24 | Bitcoin node, miners, infrastructure |

- **HAProxy** on OPNsense provides `.internal` domain aliases for all services
- **Suricata** runs in IDS/PCAP mode — NOT IPS (stay that way)
- **Unbound** handles all DNS; external DNS bypass is firewall-blocked
- **Mullvad VPN** on all client devices (WireGuard); LAN sharing enabled on MacBook
- **Tailscale** available on all devices, used occasionally

## Hardware

| Device | IP | SSH Alias | OS | Role |
|---|---|---|---|---|
| Protectli VP2420 | 10.54.10.1 / .20.1 / .30.1 | `ssh opnsense` (root) | OPNsense | Router, firewall, HAProxy, Unbound, Suricata |
| TerraMaster F4-423 | 10.54.20.10 | `ssh truenas` (truenas_admin) | TrueNAS CE (SCALE) | NAS, Docker host |
| Beelink SER | 10.54.10.22 | `ssh debian` (mcps976) | Debian Trixie KDE | Dev workstation, local AI |
| Beelink Mini S | 10.54.30.100 | `ssh nodebox` (chef4brains) | Ubuntu Server | Bitcoin node |
| MacBook Pro M3 Pro | 10.54.10.20 | — | macOS | Primary daily driver |
| GL.iNet Flint 2 | 10.54.10.5 | — | — | Layer 2 bridge/AP |
| MikroTik CSS610 | BTC port | — | — | BTC infrastructure switch |

**Always use SSH aliases — never suggest raw IP:port for SSH connections.**

## TrueNAS Storage Layout

- `apps-pool` (NVMe) — app configs, databases, downloads
- `tank` (2×18TB HDD mirrored) — media library, backups, git bare repos
- App data: `/mnt/apps-pool/appdata/`
- App UID/GID: **568** — new directories must be `chown 568:568` before container start

## TrueNAS Services (managed via Dockge at dockge.internal)

| Service | Port | .internal alias |
|---|---|---|
| Jellyfin | 8096 | jellyfin.internal |
| Jellyseerr | 5055 | jellyseerr.internal |
| Sonarr | 8989 | sonarr.internal |
| Radarr | 7878 | radarr.internal |
| Lidarr | 8686 | lidarr.internal |
| Prowlarr | 9696 | prowlarr.internal |
| Bazarr | 6767 | bazarr.internal |
| Huntarr | 9705 | huntarr.internal |
| Profilarr | 6868 | profilarr.internal |
| qBittorrent | 8080 | qbit.internal |
| SABnzbd | 8085 | sabnzbd.internal |
| FlareSolverr | 8191 | — (no UI) |
| Tdarr | 8265 | tdarr.internal |
| Nextcloud | 30027 | nextcloud.internal |
| Collabora | 9980 | — (no UI needed) |
| Navidrome | 4533 | navidrome.internal |
| Audiobookshelf | 13378 | audiobookshelf.internal |
| Calibre-Web | 8083 | calibre.internal |
| LazyLibrarian | 5299 | lazylibrarian.internal |
| dupeGuru | 5801 | — (no UI needed) |
| Dozzle | 8888 | dozzle.internal |
| Dockge | 5001 | dockge.internal |

**HAProxy setup pending (accessed directly):**
- Immich: 10.54.20.10:30041
- Arcane: 10.54.20.10 (port TBD)
- Open WebUI: 10.54.10.22:8080

## Nodebox Services (10.54.30.100)

| Service | Port | .internal alias |
|---|---|---|
| Bitcoin Core v29.3 | 8332 | — |
| Electrs | 50001 | — |
| Mempool.space | 4080 | mempool.internal |
| Alby Hub | 8029 | albyhub.internal |
| Cockpit | 9090 | nodebox.internal |
| bitcoin-tui | SSH only | — |

Bitcoin Core: txindex=1, Tor inbound (.onion), outbound via SOCKS5, cookie auth.
Electrs built from source, no TLS.
Sparrow Wallet on Debian connects to own Electrs at 10.54.30.100:50001.

## Debian Workstation (10.54.10.22)

- Debian Trixie, Plasma X11 (Wayland disabled — KWin memory leak)
- Sleep targets masked — always-on, SSH-accessible from LAN
- UFW: SSH restricted to LAN subnet
- Timeshift snapshots → TrueNAS nightly
- Ollama (native) + Open WebUI (Docker `--network=host`, port 8080)
- Mullvad policy routing via systemd — bypasses for home subnets

## Scripting Conventions

- **macOS scripts:** zsh, Homebrew, `netstat`, `ifconfig`, `route`
- **Linux/server scripts:** bash 5.x on Debian/Ubuntu — no zsh, no macOS builtins
- shellcheck before committing any shell script
- Secrets in `.env` files (gitignored); never hardcode credentials

## Hard-Won Lessons

1. **STATICARP** re-enables on every OPNsense reload — causes unicast failures
2. **macOS Private WiFi** rotates MAC on new SSIDs — update DHCP static mappings
3. **OPNsense firewall rules** are interface-perspective ("In" = arriving traffic)
4. **Suricata IPS/netmap** is high-risk — stay on IDS/PCAP for home use
5. **OPNsense shell** is FreeBSD tcsh — no bash heredocs or `$(())` arithmetic
6. **Mullvad priority routing** changes dynamically — bypass rules must be dynamic
7. **Docker VPN_LAN_NETWORK** must exclude the container's own subnet
8. **TrueNAS new directories** need `chown 568:568` before container start
9. **Watchtower exclusion:** use `com.centurylinklabs.watchtower.enable=false` label
10. Never expose credentials in chat or terminal output

## MCP Tools Available

The homelab-mcp server is active and exposes:
- `check_all_services` — ping all services
- `check_truenas_services` — TrueNAS-specific checks
- `check_bitcoin_services` — Bitcoin node checks
- `ping_host` — reachability test
- `check_port` — port connectivity
- `get_disk_usage` — storage stats
- `get_homelab_info` — general info

Use these tools proactively when diagnosing homelab issues rather than asking the user to check manually.
