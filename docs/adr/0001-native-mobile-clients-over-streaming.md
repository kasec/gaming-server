# Native mobile clients, not game streaming

## Status

accepted

## Context

The goal is to play 2001–2003-era shooters (Halo CE, CoD1) from phones as well
as desktops, on a small VPS. There are two ways to put a desktop game on a
phone: run a **native mobile client** on the phone, or **stream** the game from
a machine that renders it and send the video.

## Decision

Target **native mobile clients**. Anchor on **ChupathingyCE**, whose decompilation
builds run natively on Android (and Windows/Mac/Linux) and whose
`chupathingyce-server` is joinable from all of them. We do **not** build or buy a
game-streaming host, and we do **not** run the classic `haloceded.exe` under Wine.

## Consequences

- The **VPS needs no GPU**, so the current 4 GB / 2 vCPU box suffices.
- Only games with a native mobile build are phone-playable. Halo CE qualifies;
  **CoD1 does not**, so it stays desktop-only (phase 2).
- Every player must run a **network-version-compatible** client build and supply
  their **own Xbox disc image**; we ship no game content.
- Choosing streaming later is a **separate, GPU-backed project**, not a tweak to
  this one.

## Considered options

- **Streaming (Sunshine/Moonlight)** — plays *any* game incl. CoD1 on a phone,
  but requires a GPU host and a low-latency link; the VPS cannot do it.
- **Classic `haloceded` under Wine** — authentic, but unreachable by modern
  mobile clients and an ongoing update/security liability.
