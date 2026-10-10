# Architecture

A friends-first, always-on game server for classic (2001–2003-era) shooters,
playable from phones as well as desktops, hosted on a small VPS.

## Goal

- Friends can drop into a persistent Halo: Combat Evolved game from **Android,
  Windows, Mac or Linux**.
- The host is always on and needs no one's PC.
- It runs on a **4 GB / 2 vCPU** VPS (see [Resource budget](#resource-budget)).

## Decisions

These are the settled outcomes of the design tree. Each is a real fork with
alternatives noted.

| # | Decision | Chosen | Rejected because |
| --- | --- | --- | --- |
| D1 | What "playable on mobile" means | **Native mobile clients** | Streaming needs a GPU, which the VPS lacks |
| D2 | Audience | **Friends-first, private** | Public listing is a one-variable change later (D6) |
| D3 | Game scope | **Halo CE now; CoD1 later (desktop-only)** | CoD1 has no mobile client at all |
| D4 | Hosting | **VPS now** | Home is viable later, but CGNAT/upstream is a risk |
| D5 | Ecosystem | **ChupathingyCE**, not the classic `haloceded` | Only ChupathingyCE's dedicated server is joinable from phones (see CONTEXT.md) |
| D6 | Access | **Unlisted, invite-only** (`HALO_DEDICATED_PUBLIC=false`) | Reversible later by flipping one variable |
| D7 | Deployment | **Docker Compose**, control panel on **loopback + SSH tunnel** | Never expose admin to the network |
| D8 | Stats | **Delta reporting on** (default) | Harmless; players can hide their profile |
| D9 | Capacity | **16 players**, classic playlist | Fits the upload budget; matches server design |
| D10 | Game data | **Players bring their own Xbox disc image** | ChupathingyCE ships no game content |

## Components

```
 player's phone / PC (ChupathingyCE client, own Halo Xbox ISO)
        │  invite link  halo://join/…
        ▼
 ┌───────────────────────────── VPS (4 GB / 2 vCPU) ─────────────────────────────┐
 │  chupathingyce-server  (Docker, restart: unless-stopped)                       │
 │    • headless host, no player of its own                                       │
 │    • playlist loop: Slayer / CTF on classic maps                               │
 │    • internet play: UDP 2302 (one port, opened in the provider firewall)       │
 │    • control panel on 127.0.0.1:8080  ── reached only via SSH tunnel           │
 └────────────────────────────────────────────────────────────────────────────────┘
        │  HTTPS (outbound only)
        ▼
   halo.milenko.org  (Delta: game list, stats, profiles)
```

- **Dedicated server** — the `chupathingyce-server` binary. Static, one file, no
  libraries. The container runs it as an unprivileged user with no capabilities,
  its memory and process count capped.
- **Client** — each friend installs the matching ChupathingyCE build (same
  **network version**) and supplies their own Halo CE Xbox disc image. This is
  what makes the phone work.
- **Delta** — optional network services on `halo.milenko.org`. Games work
  without it; it adds the game list and stats.

## Network

- **One UDP port** (`HALO_NET_TUNNEL_PORT=2302`) carries internet play. Opened
  in three places: the provider firewall/security group, the host firewall
  (`ufw allow 2302/udp`), and the Docker host network.
- **Host networking** (`--network host`): the server's hole punching does not
  survive a bridge's NAT.
- **Invite links** are the join path (D6). P2P hole punching means most homes
  need no port forwarding for *incoming* player connections.
- **Control panel** binds `127.0.0.1:8080`; reach it with
  `ssh -N -L 8080:127.0.0.1:8080 you@vps`.

## Resource budget

| Resource | Available | Used (Halo CE) | Notes |
| --- | --- | --- | --- |
| CPU | 2 vCPU | ~2% of one core idle | Headroom for several instances |
| RAM | 4 GB | ~215 MB (x64) | Plenty of room |
| Disk | (VPS disk) | ~300 MB maps + ~33 MB saves | Plus a little image overhead |
| Upload | provider's | ~2–3 Mbit/s at 16 players | The real ceiling, not CPU/RAM |

A second, 32-bit instance (`halo-dedicated-ce`) can run beside this one for
Halo PC Custom Edition maps.

## Non-goals

- **Game streaming** (rendering CoD1 on the server and sending video). Needs a
  GPU host; a different project.
- **Public matchmaking / anti-cheat at scale.** Friends-first, invite-only.
- **Classic `haloceded` under Wine.** Update/security liability with no mobile
  reach.

## Phases

1. **Halo CE on the VPS** — dedicated server, unlisted, invite links, playlist,
   control panel over SSH. (This spec's runbook.)
2. **CoD 1 (desktop-only)** — `cod_lnxded` instance on the same VPS for PC
   nights. ~64 MB, no mobile.
3. *(Optional, later)* **Home server** — a Pi 4/mini-PC on the LAN, once
   upstream and public-IP availability are confirmed.

## Open questions

- Confirm the VPS provider's **published upload bandwidth** (drives D9; at
  16 players it is ~2–3 Mbit/s).
- Which provider it is decides the exact firewall UI for the UDP port.
