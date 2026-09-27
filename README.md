# gobrowse2

Systems and tooling work: sandbox infrastructure for AI agents, embedded
firmware, and MCP servers that make hardware design tools scriptable.

Currently [`@fedoragobrowse-design`](https://github.com/fedoragobrowse-design)
— an alt account of [@gobrowse](https://github.com/gobrowse).

---

## What I'm working on

### AgentForge — sandbox compute for autonomous agents

A serverless Firecracker control plane where agents get real isolation instead
of a `chroot` and a prayer. Independent open-source implementation, not
affiliated with any commercial sandbox platform.

The design commitment is that **the development runtime is not a security
boundary**. `bwrap-dev` and Docker modes are for local work and fail loudly as
unsafe for untrusted code; production is `firecracker` and fails closed when
KVM, a kernel, rootfs, or guest networking is missing.

The genuinely verified path is the Firecracker integration test: boot a VM, wait
for the guest agent, exec, write and read a guest file, snapshot memory and
devices plus a disk copy, destroy the source VM, restore the snapshot in a new
Firecracker process, and read the preserved file back. Restores across process
death are the part most sandbox demos skip.

```
agentforge-core      domain contracts + Platform composition
agentforge-runtime   Firecracker lifecycle, bubblewrap dev backend
agentforge-storage   PostgreSQL metadata/scheduling, S3 artifacts
agentforge-api       Axum API, worker RPC, orchestration, metrics
agentforge-client    Rust SDK + CLI
```

The README publishes no invented benchmark numbers — if it isn't measured by
`agentforge benchmark`, it isn't in the docs.

→ [github.com/fedoragobrowse-design/AIec](https://github.com/fedoragobrowse-design/AIec)

---

### 3-node LoRa mesh — Pico 2 W + SX1276

Three Raspberry Pi Pico 2 W boards on a private encrypted protocol at 915 MHz.
Rust + Embassy firmware on the radio side, Python host tools for the terminal.

- End-to-end encrypted per contact pair — X25519, ChaCha20-Poly1305 on hourly keys
- Relays forward packets without ever holding the keys
- Live-tested: 3 boards, all six directed links ACK, +2 dBm SF7/BW500

**What this deliberately is not**, and it's worth reading before you buy parts:

- No phone app, no groups, no GPS, no Bluetooth — paired messages only
- No store-and-forward; a message to an offline peer fails
- Short range on purpose — +2 dBm is the radio's minimum, tens of meters
- Through-hole soldering required

Install is a drag-and-drop UF2 from a web installer with a SHA-256 check, so
you don't need a Rust toolchain to try it. There's a plain-language tour at
`docs/how-it-works.html` for people without a crypto or embedded background.

Known limitations are documented rather than buried: flash secrets are plaintext
(physical possession is game over), hourly keys mean no forward secrecy, and
host message history sits in plaintext SQLite.

→ [firmware](https://github.com/fedoragobrowse-design/lora-mesh-radio) ·
[host tools](https://github.com/fedoragobrowse-design/lora-mesh-app)

---

### MCP servers for hardware design tools

KiCad, Fritzing, and the Pico are all GUI-first tools with excellent file
formats. These wrap them in MCP stdio servers so an agent can drive them
directly, with a real smoke test over raw stdio rather than a mocked unit test.

| Server | What it exposes |
|---|---|
| [pico-mcp](https://github.com/fedoragobrowse-design/pico-mcp) | Flash, program, and talk to a Pico over USB-CDC |
| [kicad-mcp](https://github.com/fedoragobrowse-design/kicad-mcp) | Schematics, PCBs, symbols, footprints, ERC/DRC, gerbers |
| [fritzing-mcp](https://github.com/fedoragobrowse-design/fritzing-mcp) | `.fzz` sketches, part search, headless render, `.fzp` generation |
| [chess-muserelf-mcp](https://github.com/fedoragobrowse-design/chess-muserelf-mcp) | Chess tools |

`kicad-mcp` and `fritzing-mcp` are MIT licensed. The smoke tests build a real
project end to end — create, place, wire, validate, run the checkers, export —
and print `SMOKE: N/N passed`.

Fritzing support also includes [`fz`](https://github.com/fedoragobrowse-design/fritzing-ai),
a stdlib-only Fritzing design toolkit, and
[robotics-technical-graphics](https://github.com/fedoragobrowse-design/robotics-technical-graphics),
a skill for producing classroom-ready deterministic SVG diagrams with exact
labels.

→ [pico-mcp](https://github.com/fedoragobrowse-design/pico-mcp)

---

## Also in the workshop

- [`agent-hub`](https://github.com/fedoragobrowse-design/agent-hub) — private agent hub with an
  [extension marketplace](https://github.com/fedoragobrowse-design/agent-hub-marketplace)
- [`harmonics-analysis`](https://github.com/fedoragobrowse-design/harmonics-analysis) — signal analysis,
  deployed to GitHub Pages
- [`connect4-ml`](https://github.com/fedoragobrowse-design/connect4-ml) — small ML experiments

## Stack

Rust · Embassy · Python · TypeScript · Axum · PostgreSQL · Firecracker ·
RP2040/RP2350 · SX1276 · X25519 / ChaCha20-Poly1305 · MCP

## Contact

Open to issues and PRs on any of the above. This is an alt account — the main
one is [@gobrowse](https://github.com/gobrowse).
