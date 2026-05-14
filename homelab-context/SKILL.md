---
name: homelab-context
description: Load Martin's homelab infrastructure context. Auto-invoke when the conversation touches networking, VLANs, DNS, VPN, Docker, TrueNAS, OPNsense, HAProxy, SSH, services (Jellyfin, Sonarr, Radarr, Forgejo, etc.), Bitcoin node, NixOS workstation, firewall rules, storage layout, or any homelab infrastructure topic.
user-invocable: false
---

# Homelab Infrastructure Context

**Last verified:** 2026-05-14 (live SSH audit across all machines)

## Network Topology

| Segment | Subnet | Purpose |
|---|---|---|
| LAN | 10.54.10.0/24 | Client devices, workstations, Apple devices |
| SERV | 10.54.20.0/24 | TrueNAS and server workloads |
| BTC | 10.54.30.0/24 | Bitcoin node, miners, infrastructure |

**Note:** Physically separate interfaces on Protectli VP2420 — NOT VLANs.

- **HAProxy** on OPNsense provides `.internal` domain aliases for all services
- **Suricata** runs in IDS/PCAP mode — NOT IPS (stay that way)
- **Unbound** handles all DNS; external DNS bypass is firewall-blocked
- **Dnsmasq** handles DHCP only (migrated from ISC DHCPv4 in 2026-05)
- **Mullvad VPN** on all client devices (WireGuard); LAN sharing enabled on MacBook
- **Tailscale** subnet routed via OPNsense — active on router, NOT installed on individual machines

## Hardware

| Device | IP | SSH Alias | OS | Role |
|---|---|---|---|---|
| Protectli VP2420 | 10.54.10.1 / .20.1 / .30.1 | `ssh opnsense` (root) | OPNsense 26.1.7 (FreeBSD 14.3) | Router, firewall, HAProxy, Unbound, Suricata |
| TerraMaster F4-423 | 10.54.20.10 | `ssh truenas` (truenas_admin) | TrueNAS CE 25.10.3 (SCALE) | NAS, Docker/native apps, Forgejo |
| Beelink SER | 10.54.10.22 | `ssh nix` (mcps976) | NixOS 25.11 GNOME Wayland | Dev workstation, local AI (Ollama + Open WebUI) |
| Beelink Mini S | 10.54.30.100 | `ssh nodebox` (chef4brains) | Ubuntu Server 24.04 | Bitcoin node, Electrs |
| MacBook Pro M3 Pro | 10.54.10.20 | — | macOS | Primary daily driver |
| MacBook Air M2 (Ruth) | 10.54.10.21 | — | macOS | General use |
| GL.iNet Flint 2 | 10.54.10.5 | — | — | Layer 2 bridge/AP |
| MikroTik CSS610 | BTC port | — | — | BTC infrastructure switch |

**Always use SSH aliases — never suggest raw IP:port for SSH connections.**

## TrueNAS Storage Layout

- `apps-pool` (NVMe) — app configs, databases, downloads, compose files
- `tank` (2×18TB HDD mirrored) — media, backups, git repos, Time Machine, music share
- App data: `/mnt/apps-pool/appdata/<app>/`
- Compose files: `/mnt/apps-pool/stacks/<stack>/compose.yaml` (never data here)
- App UID/GID: **568** — new directories must be `chown 568:568` before container start

**ZFS tank datasets (verified 2026-05-14):**
- `tank/git-repos` — Forgejo backup repos
- `tank/immich/userdata` — Immich photo library
- `tank/nextcloud/userdata` — Nextcloud files
- `tank/timemachine/{mart,ruth}` — Time Machine backups
- `tank/musicshare` — Music share

**Docker management:**
- **Arcane (port 3552, arcane.internal)** — PRIMARY. Use for new services and Compose projects.
- **Dockge (port 5001, dockge.internal)** — Legacy/secondary. Manages established stacks: arr-stack, dispatcharr-gluetun, homarr, nextcloud.

**Forgejo:** Runs as a TrueNAS native app (ix-apps, not Dockge-managed). Port 3000, SSH port 2222. All 21 repos mirrored to GitHub (origin) + Forgejo (forgejo remote) via `gpush`.

## TrueNAS Services (all via HAProxy, verified 2026-05-14)

**Arcane-managed:**

| Service | Port | .internal alias |
|---|---|---|
| Arcane | 3552 | arcane.internal |
| Backrest | 9898 | backrest.internal |
| Beszel hub | 8090 | beszel.internal |
| Bookstack | 6875 | bookstack.internal |
| ConvertX | 3001 | convertx.internal |
| Khoj | 42110 | khoj.internal |
| SearXNG | 4040 | searxng.internal |

**Dockge arr-stack:**

| Service | Port | .internal alias |
|---|---|---|
| Audiobookshelf | 13378 | audiobookshelf.internal |
| Bazarr | 6767 | bazarr.internal |
| Calibre-Web | 8083 | calibre.internal |
| Cleanuparr | 11011 | cleanuparr.internal |
| Dozzle | 8888 | dozzle.internal |
| FlareSolverr | 8191 | — (no UI) |
| Jellyfin | 8096 | jellyfin.internal |
| Jellyseerr | 5055 | jellyseerr.internal |
| LazyLibrarian | 5299 | lazylibrarian.internal |
| Lidarr | 8686 | lidarr.internal |
| Navidrome | 4533 | navidrome.internal |
| Neutarr | 9705 | neutarr.internal |
| Profilarr | 6868 | profilarr.internal |
| Prowlarr | 9696 | prowlarr.internal |
| qBittorrent | 8080 | qbit.internal |
| Radarr | 7878 | radarr.internal |
| SABnzbd | 8085 | sab.internal |
| Sonarr | 8989 | sonarr.internal |
| Tdarr | 8265 | tdarr.internal |

**Dockge dedicated stacks:**

| Service | Port | .internal alias |
|---|---|---|
| Dispatcharr | 9191 | dispatcharr.internal |
| Homarr | 7575 | homarr.internal |
| Nextcloud (+ built-in CODE) | 30027 | nextcloud.internal |

**TrueNAS native apps:**

| Service | Port | .internal alias |
|---|---|---|
| Dockge | 5001 | dockge.internal |
| Forgejo | 3000 (HTTP) / 2222 (SSH) | forgejo.internal |
| Immich | 30041 | immich.internal |

**DNS alias corrections:**
- SABnzbd alias is `sab.internal` (not `sabnzbd.internal`)
- Open WebUI alias is `ollama.internal` (not `openwebui.internal`)
- Huntarr has been replaced by Neutarr; `huntarr.internal` no longer exists

## Nodebox Services (10.54.30.100 — Ubuntu Server 24.04)

| Service | Port | .internal alias |
|---|---|---|
| Bitcoin Core v31.0.0 | 8332 | — |
| Bitcoin ZMQ | 28332 | — |
| Electrs 0.10.9 | 50001 | — |
| Mempool.space | 4080 | mempool.internal |
| Alby Hub | 8029 | albyhub.internal |
| UTXOracle | 8090 | utxoracle.internal |
| Cockpit | 9090 | nodebox.internal |
| Public Pool | 8081 | publicpool.internal |

**Bitcoin Core:** txindex=1, Tor inbound (.onion), outbound via SOCKS5, cookie auth. Systemd service: `bitcoind.service` (not `bitcoin.service`). Fully synced (mainnet, 949,337 blocks as of 2026-05-14).

**Electrs:** Built from source, no TLS. Port 50001 plain TCP. Sparrow Wallet on beelink connects here.

**UTXOracle:** Exchange-free BTC/USD price oracle. Derives price solely from on-chain UTXO patterns. Python http.server on port 8090, `utxoracle-web` systemd service.

## Beelink SER (10.54.10.22 — NixOS 25.11 GNOME Wayland)

- **Kernel:** 6.12.85 #1-NixOS SMP PREEMPT_DYNAMIC
- **Network:** wlo1 (WiFi, 10.54.10.22/24), enp1s0 (UP, no IP — BTC VLAN interface, verify if intentional)
- Always-on — sleep targets masked in systemd
- SSH: YubiKey only, firewall allows LAN + OPNsense only
- NixOS config repo: mcps976/nixos-beelink
- Mullvad VPN — `mullvad lan set allow` configured for home subnets
- Docker with nftables (no iptables workarounds needed)
- SSH_AUTH_SOCK overridden to gpg-agent socket (GNOME Keyring SSH disabled)

**Ollama (0.21.1, native systemd, active):**
- Listens on 0.0.0.0:11434 (accessible from SERV subnet for Khoj)
- 7 models: moondream, phi4-mini, deepseek-r1:7b, nomic-embed-text, qwen2.5-coder:1.5b, llama3.1:8b, qwen3.5:4b
- OPNsense firewall rule: SERV → beelink:11434

**Open WebUI (Docker, port 8080):**
- DNS alias: `ollama.internal` (via HAProxy)
- Image: `ghcr.io/open-webui/open-webui:main`

**Beszel agent:** Running (`henrygd/beszel-agent` Docker container) — reports to Beszel hub on TrueNAS (beszel.internal:8090).

## Git Setup

All repos use two remotes:
- `origin` → GitHub (`git@github.com:mcps976/<repo>.git`)
- `forgejo` → Forgejo on TrueNAS (`git@forgejo:mcps976/<repo>.git`, SSH port 2222)

`gpush` = `git push origin main && git push forgejo main`. Always use `gpush` instead of `git push`.

Repos in ~/Git/ with NO git remotes (plain directories): homelab, dotfiles, projects, resources.

## Scripting Conventions

- **macOS scripts:** zsh, Homebrew, `netstat`, `ifconfig`, `route`
- **Linux/server scripts:** bash 5.x on Ubuntu (nodebox) — no zsh, no macOS builtins
- **NixOS scripts:** `#!/usr/bin/env bash` — never hardcode `/bin/bash` or absolute tool paths (Nix store paths). Avoid imperative state changes; prefer declarative NixOS config.
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
11. **New .internal service** requires two steps: (1) add hostname to OPNsense Internal Wildcard cert SAN + regenerate, (2) add Unbound DNS override. After cert change: flush macOS DNS cache and clear brave://net-internals/#dns.
12. **HAProxy + Cockpit (Nodebox):** Cockpit is HTTPS-only — HAProxy backend must use SSL with cert verification disabled (self-signed cert).
13. **Forgejo SSH alias:** `git@forgejo` resolves to 10.54.20.10 port 2222. Remotes use `git@forgejo:mcps976/<repo>.git`, NOT `truenas:/mnt/tank/git-repos/` paths (those are retired bare repos).
14. **Arcane is primary Docker UI** — use for new services. Dockge retained for legacy stacks only.
15. **VSCodium on NixOS** requires `vscodium.fhs` (not `vscodium`) — FHS chroot for extensions with pre-compiled binaries.
16. **NixOS networking.nftables.enable = true** is REQUIRED for extraInputRules to work.
17. **bitcoind.service** is the correct systemd unit name on nodebox — not `bitcoin.service` or `bitcoin-core.service`.

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
