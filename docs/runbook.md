# Runbook: Halo CE dedicated server on the VPS

Follow in order. Steps 1–7 stand the server up; 8–9 open it to your friends.

## 0. Prerequisites

- The VPS, reachable over SSH (`x86_64` → the **x64** server binary).
- Docker installed on the VPS.
- Your own **Xbox Halo: Combat Evolved disc image** (`.iso`/`.xiso`, **NTSC**)
  for the maps. This is per-person game data; the server ships none.

## 1. Get the game maps

Copy the game's `maps` folder out of the disc image. You need `ui.map` and
**all** the multiplayer maps (~300 MB). Put them in the data folder's `maps/`.

```
data/
  maps/            ui.map, bloodgulch.map, sidewinder.map, ...   (Xbox, NTSC)
  maps_ce/         (optional) Custom Edition maps + bitmaps/mounds/loc
  playlists/
```

## 2. Get the server binary

Download `chupathingyce-server-linux-x64` from the ChupathingyCE releases page
and unpack it. You get `chupathingyce-server` and a `playlists/` folder.

```sh
uname -m            # expect x86_64
```

## 3. Build the image (on the VPS)

```sh
mkdir -p image/bin/amd64
cp server/deploy/Dockerfile image/
cp chupathingyce-server-linux-x64/chupathingyce-server image/bin/amd64/
docker build --build-arg TARGETARCH=amd64 -t chupathingyce-server image
```

The image runs the server as uid 1000, with `/data` as the data folder.

## 4. Pick a playlist

`data/playlists/friends.txt`:

```
# map            game type
bloodgulch       slayer
sidewinder       {team_slayer}
hangemhigh       ctf
damnation        team_slayer
chillout         slayer
```

Team games need two players; a lone player skips to the next non-team entry.
Built-in types: `slayer`, `team_slayer`, `ctf`, `king`, `oddball`, `race`, …

## 5. Data folder

The data folder must be readable by uid 1000 and contain `maps/ui.map`.

```sh
chown -R 1000:1000 /path/to/data
```

## 6. Run it

`docker-compose.yml`:

```yaml
services:
  halo:
    image: chupathingyce-server
    container_name: halo
    restart: unless-stopped
    network_mode: host
    user: "1000:1000"
    cap_drop: [ALL]
    security_opt: [no-new-privileges]
    mem_limit: 1g
    pids_limit: 256
    volumes:
      - /path/to/data:/data
    environment:
      HALO_DATA_ROOT: /data
      HALO_SAVE_ROOT: /data/saves
      HALO_DEDICATED: playlists/friends.txt
      HALO_DEDICATED_NAME: "Friends Halo"      # 15 chars max
      HALO_DEDICATED_PUBLIC: "false"           # D6: unlisted, invite-only
      HALO_DEDICATED_MAXIMUM_PLAYERS: "16"     # D9
      HALO_DEDICATED_MINIMUM_PLAYERS: "1"
      HALO_NET_TUNNEL_PORT: "2302"             # the one UDP port
      HALO_DEDICATED_CONTROL: "8080"           # loopback only
```

```sh
docker compose up -d
docker attach halo        # optional console; Ctrl-P Ctrl-Q to detach
```

`--network host` is required: internet play's hole punching does not survive a
bridge's NAT.

## 7. Open the one UDP port

Three layers, or players see "could not connect":

1. **Provider firewall / security group** — ingress, source `0.0.0.0/0`, UDP,
   dest port `2302`.
2. **Host firewall** — `sudo ufw allow 2302/udp` (or `iptables -I INPUT -p udp
   --dport 2302 -j ACCEPT` + `sudo netfilter-persistent save`).
3. **Docker** — covered by `network_mode: host`.

The log's `this machine's public address is …:2302` line confirms the port.
No ports are needed for the control panel (next step).

## 8. Control panel, owner, moderators

1. The first start prints a one-time **owner setup link** on the console (not in
   `debug.txt`). Reach the panel over a tunnel:
   `ssh -N -L 8080:127.0.0.1:8080 you@vps`, then open the link.
2. Set the owner name/password and a **second factor**.
3. Add moderators: **People → Make an invitation**; send links privately.
   In-game moderation is bound by their moderator key, never by name.

Back up the `control_*` files and `moderators.txt` privately — they hold
password hashes and second factors.

## 9. Give it to your friends

Each friend:

1. Installs the **matching ChupathingyCE build** (same **network version** as
   the server — check with `docker exec halo chupathingyce-server --version`).
2. Supplies their **own Xbox Halo CE ISO** when the client first asks.
3. On a phone: Android 9+, 64-bit. On desktop: Windows 10+/macOS 13+/Linux.
4. Joins via the **invite link** you share, or **Multiplayer → Online Games**.

A "version mismatch" message names both network versions — that is a friend on
the wrong build, not a server fault.

## 10. Keep it healthy

- **Logs:** `/path/to/data/debug.txt`; container logs `docker logs halo`.
- **Backups:** the data folder, especially `control_*`, `bans.txt`, `admin/`.
- **Upgrades:** replace the binary, rebuild the image, `docker compose up -d`.
  Keep server and clients on the same network version.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `no maps` on start | `maps/ui.map` missing | Step 1; check `HALO_DATA_ROOT` |
| Listed, nobody can join | UDP 2302 blocked | Step 7, all three layers |
| Friend sees version mismatch | Wrong client build | Match to the server's network version |
| `cannot commit the Xbox heap` | 64-bit on ≤1 GB RAM | Use the x86 binary or add swap (not our case) |
| Nobody shows in browser | `HALO_DEDICATED_PUBLIC=false` | Expected (D6); join by invite link |
