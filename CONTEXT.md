# Gaming Server

A friends-first, always-on host for playing classic (2001–2003-era) multiplayer
shooters on desktop and mobile. This glossary fixes the vocabulary so "server"
never means three different things at once.

## Language

**Game server**:
An authoritative host that players' game clients connect to for multiplayer.
_Distinct from a **game-streaming server**, which renders a game and sends
video to a device._ We only mean the former.
_Avoid_: gaming server

**Dedicated server**:
A headless authoritative host that plays no game itself: no window, no sound,
no local player. In this project it means the modern `chupathingyce-server`
binary, not the original engine.
_Avoid_: host, game host

**Classic dedicated server**:
The original 2001-era Halo CE / Halo PC `haloceded.exe` (run under Wine on
Linux) or CoD1's `cod_lnxded`. Not reachable by modern mobile clients.
_Avoid_: real server, original server

**Client build**:
A specific downloadable game executable for one platform (Windows, Mac, Linux,
Android). Every player must have a version-compatible build.

**Network version**:
The integer that decides who can join whom. Clients and servers of the same
network version interoperate; a mismatch is refused with a message naming both.
_Avoid_: version (ambiguous with release version)

**Delta**:
The ChupathingyCE network family and its services on `halo.milenko.org`:
public game list, stats, profiles, and dedicated-server admin. Optional; games
work without it.
_Avoid_: the network, the platform
