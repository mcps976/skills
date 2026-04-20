---
name: bitcoin-node
description: Load full context for Martin's Bitcoin node stack. Auto-invoke on anything touching Bitcoin Core, Electrs, Mempool.space, Alby Hub, Lightning Network, Sparrow Wallet, bitcoin-tui, publicpool, Antminer S9 mining, Braiins OS, Bitcoin protocol questions (OP_RETURN, BIPs, post-quantum), mempool analysis, fee estimation, transaction broadcast, or node operations on nodebox.
user-invocable: false
---

# Bitcoin Node Context

Node stack on **nodebox** — Beelink Mini S, Ubuntu Server, 16GB RAM, 2TB SSD.
- **IP:** 10.54.30.100 (BTC VLAN — 10.54.30.0/24)
- **SSH:** `ssh nodebox` (user: chef4brains)
- **VLAN isolation:** BTC VLAN is separate from LAN and SERV. OPNsense controls inter-VLAN routing

---

## Bitcoin Core v29.3

**Service:** systemd (`bitcoind`)
**Data dir:** `~/.bitcoin/`
**Shutdown:** `bitcoin-cli stop` — never `bitcoind -stop`

**Key bitcoin.conf settings:**

```ini
server=1
txindex=1          # required by Electrs and Mempool.space
daemon=1
rpcport=8332
proxy=127.0.0.1:9050          # all outbound through Tor SOCKS5
bind=127.0.0.1                # Tor-only: inbound only via .onion
listen=1
zmqpubrawblock=tcp://0.0.0.0:28332
zmqpubrawtx=tcp://0.0.0.0:28333
zmqpubhashblock=tcp://0.0.0.0:28334
asmap=/home/chef4brains/.bitcoin/asmap.dat
datacarriersize=83
```

**Networking model:** Tor-only. All outbound peers through Tor SOCKS5 proxy (`127.0.0.1:9050`). Inbound connections only via `.onion` hidden service. Full unpruned node (~822GB on disk as of Feb 2026).

**ASMap:** Peer diversity protection grouping outbound peers by Autonomous System rather than IP prefix — mitigates eclipse attacks. Map file: `~/.bitcoin/asmap.dat` (1.5MB, from `bitcoin-core/asmap-data` on GitHub). Update manually: download new `.dat`, replace, restart bitcoind. **On v31+** simplify to `asmap=1` (embedded map). **Do NOT use `asmap=1` on v29** — no embedded map in this version.

**Known open items:**
- `dbcache=1500` — discussed, not yet added (would improve IBD performance)
- `rpcallowip=192.0.0.0/8` — technically wrong (should be `192.168.0.0/16`), functionally harmless on isolated BTC VLAN
- Switch to `asmap=1` when upgrading to v31+

**Upgrade workflow (established pattern — 29.1 → 29.3):**
1. `wget` tarball + `SHA256SUMS` + `SHA256SUMS.asc`
2. `sha256sum --ignore-missing --check SHA256SUMS`
3. GPG verify against `fanquake.gpg` (from `bitcoin-core/guix.sigs`)
4. `bitcoin-cli stop && sleep 30`
5. `tar -xzf`, `sudo install` to `/usr/local/bin/`
6. `sudo systemctl start bitcoind`
7. Verify: `bitcoind --version` + `bitcoin-cli getblockchaininfo`
8. Clean up tarballs and GPG key

GPG note: "Not certified with a trusted signature" warning is expected — "Good signature from" is what matters.

---

## Electrs — Electrum Server

- **Install:** Built from source via Cargo → `~/electrs/target/release/electrs`
- **Service:** systemd
- **Port:** 50001 (TCP, no TLS)
- **Database:** `~/electrs/database/` — 56GB index
- **RPC connection:** `127.0.0.1:8332` with rpcauth credentials
- **Clients:** Sparrow Wallet (10.54.10.22) and Mempool.space (localhost)

**CRITICAL: Never delete `~/electrs/database/`** — re-indexing takes many hours.

No TLS on Electrs — connection from Sparrow travels over internal LAN via firewall-controlled path, not the public internet.

---

## Mempool.space — Block Explorer

- **Port:** 4080 — accessible at `mempool.internal` (HAProxy) and via Tailscale (100.110.156.42)
- **Compose:** `~/mempool/docker/docker-compose.yml`
- **Requires:** `txindex=1` + all three ZMQ endpoints (28332–28334) in bitcoin.conf
- **Watchtower:** Removed permanently — was crash-looping due to Docker SDK API incompatibility (Watchtower built against 1.25, Docker Engine required 1.44+). Manual image updates only.

---

## Alby Hub — Lightning Node

- **Port:** 8029 — accessible at `albyhub.internal` (HAProxy) and via Tailscale
- **Implementation:** LDK (Lightning Dev Kit) — embedded, lighter than standalone LND/CLN
- **Data dir:** `~/albyhub/`
- **Service:** `albyhub.service` (systemd)
- **Backend:** Bitcoin Core on the same machine

**CRITICAL: Never delete `~/albyhub/`** — contains Lightning channel state. Loss = loss of in-channel funds without backup recovery.

**NWC (Nostr Wallet Connect):** Alby Hub exposes an NWC connection string for programmatic access. Stored in `~/.secrets` on the MacBook (chmod 600). Rotate via: Alby Hub → Connections panel → revoke and recreate. **A previous NWC string was accidentally exposed in a Claude Code session — immediately revoked. Never log or display NWC strings.**

**Channel management:** Done via Alby Hub web UI at `albyhub.internal`.

**LDK vs alternatives:**
| Implementation | Type | Notes |
|---|---|---|
| LDK | Library (embedded in Alby) | Lighter, no separate daemon — right choice for personal node |
| LND | Standalone daemon | Most widely deployed, RTL/ThunderHub UIs |
| CLN | Standalone daemon | Blockstream, plugin architecture |

**BTCPay Server (planned):** Hetzner VPS deployment for Blackberry & Thistle payment processing, connecting to this Lightning node for instant settlement.

---

## Wallet Stack (Sparrow → Electrs → Bitcoin Core)

```
Sparrow Wallet (Debian, 10.54.10.22)
    ↓ TCP port 50001
Electrs (nodebox, 10.54.30.100)
    ↓ RPC auth
Bitcoin Core v29.3 (nodebox)
    ↓ P2P network
Bitcoin network (Tor + clearnet via proxy)
```

Sparrow runs on the Debian workstation, connected to own Electrs. No third-party servers involved in any balance check, address lookup, or transaction broadcast.

**UTXO hygiene:** Fresh addresses for every payment. Never reuse. Previously-spent addresses have the public key on-chain — higher quantum risk.

**PSBT workflow:** Sparrow constructs tx → exports as PSBT → signs on air-gapped device (SeedSigner / Krux / Jade) via QR or SD card → broadcasts through own node.

**Quantum triage (BIP-360 context):**
| Address type | Public key exposure | Action |
|---|---|---|
| Previously-spent | On-chain — exposed | Sweep to fresh address when possible |
| Receive-only, unspent | Hash-protected | Leave — spending exposes the key |
| During spend | 10-min mempool window | Negligible risk today |

---

## Mining — Antminer S9s

Two units on BTC VLAN, running Braiins OS+ (BOSminer 0.9.0):

| Unit | Hostname | IP | Hash Rate | Notes |
|---|---|---|---|---|
| S9i | s9i | 10.54.30.80 | ~9.8 TH/s | Chains 6+7 active (63 chips each) |
| S9 | s9 | 10.54.30.81 | ~4.9 TH/s | Chain 8 only (63 chips) |

**Total: ~14.7 TH/s**

**Pool:** Braiins Pool (`stratum+tcp://stratum.braiins.com:3333`). User format: `plebminer21.miner-s9i` / `plebminer21.miner-s9`.

**publicpool (solo mining):** Self-hosted on nodebox via Docker (`sethforprivacy/public-pool`). Stratum port 3333, UI at `http://10.54.30.100:8081`. Config: `~/pool/pool.env`. **Payout address not yet configured** — needs Sparrow wallet address.

**Key Braiins OS+ gotchas:**
- **Dead temp sensors + autotuning:** Autotuner requires temp feedback to progress past Stage 0. Dead sensors → stuck at 476 MHz. Fix: `[temp_control] mode = 'disabled'` in bosminer.toml (Chain 8 tuned to ~684 MHz this way).
- **Fan tach failure:** Fans spin but RPM not reported — dead tach wires, not control board. Fix: `min_fans = 0`.
- **Braiins dev-fee DNS:** `us-west.a830bcc3.bos.braiins.com` failing to resolve on Unbound/OPNsense. Not critical to mining. OPNsense alias `Bitcoin_miners` covers both IPs; `Mining_Pools_Ports` alias: 3333, 3334, 3336.

**Why mine at home:** Not economically competitive (S9 draws ~1.3kW each; net cost at Perth electricity rates). Value is network participation, decentralisation, and learning Stratum protocol. Stratum V2 home block template construction is the longer-term interest.

---

## bitcoin-tui

- Built from source via Cargo, on `$PATH`
- Terminal dashboard: blocks, mempool, peers
- Access via `ssh nodebox` or Tailscale

---

## Weekly Maintenance Cron

Script: `~/scripts/node-cleanup.sh` — runs Sundays 3AM. Logs to `~/logs/cleanup.log`.

Covers: rotate debug.log (>7 days), Docker builder cache prune, Docker dangling image prune, systemd journal vacuum (2 weeks), `apt clean` + `apt autoremove`, `/tmp` files >7 days.

**Deliberately excluded:** `mempool.dat` deletion (requires clean bitcoind stop), Docker image updates, Electrs database, Alby Hub data.

---

## Active Protocol Debates (relevant to this node)

- **OP_RETURN / Bitcoin Core 30:** Relay policy change lifted 80-byte limit to ~4MB. Debate: sound money infrastructure vs neutral network. Node currently runs Bitcoin Core v29.3, predating this change.
- **BIP-110:** Soft fork proposal to reinstate strict data limits. First signalling block mined by Ocean (March 2026). Consensus-level change — different in kind from relay policy.
- **BIP-360/361 (Post-quantum):** New P2MR output type with `bc1z` addresses, ML-DSA/SLH-DSA/XMSS signatures. Working implementation on testnet v0.3.0 (March 2026). Qubit threshold revised down to <500,000 (from 9M in 2023). Wallet UTXO triage above is directly relevant.
- **Stratum V2:** Miners constructing own block templates — relevant to publicpool long-term.

---

## MCP Tools Available

The Bitcoin MCP server points to `10.54.30.100:8332`. Use proactively for:
- `get_blockchain_info` / `get_block_count` — sync status
- `get_mempool_info` / `analyze_mempool` — mempool state
- `get_fee_estimates` / `get_fee_recommendation` — fee advice
- `get_network_info` / `get_peer_info` — connectivity
- `analyze_transaction` / `decode_raw_transaction` — tx inspection
- `get_mining_info` — difficulty, hash rate
- `get_node_status` / `get_indexer_status` — health checks

Homelab MCP: `check_bitcoin_services` and `ping_host` for quick service reachability.
