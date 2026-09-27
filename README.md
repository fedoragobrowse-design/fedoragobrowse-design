# gobrowse2

I build things that teach other people to build things — and the systems
underneath them. A live after-school STEM program, an offline-first class hub,
sandbox compute for AI agents, embedded firmware, and MCP servers that make
hardware design tools scriptable.

Currently [`@fedoragobrowse-design`](https://github.com/fedoragobrowse-design)
— an alt account of [@gobrowse](https://github.com/gobrowse).

---

## What I'm working on

### Learn by Make — a Python + electronics program for ages 8–14

**[learnbymake.com](https://learnbymake.com)** · ~16 in-person classes at
Brookwood Library, Hillsboro, Oregon · starting October 2026

Students aged 8–14 write real Python and wire real circuits, then advance to
MicroPython on Raspberry Pi Pico 2 hardware. No block coding, no drag-and-drop,
no pre-soldered shields — real syntax and real components from the first class.

I wrote the 16-class curriculum and the tooling that runs it. Each class pairs
one Python topic with one physical build, so the code and the circuit are always
talking about the same thing: students wire a resistor by hand, then write the
Ohm's Law calculator that models it. Python and electronics are taught as one
discipline rather than two parallel tracks — the breadboard-column mental model
from Class 1 becomes GPIO pin grouping in Class 12.

The tooling side is the part I'm proudest of:

- **Deterministic SVG diagrams, not AI raster art** — every circuit diagram is
  generated from source, so labels are exact and output is diffable in git. A
  resistor that reads `220Ω` in one class and `22OΩ` in another is a bug.
- **An offline-first class hub** — Pyodide is bundled locally, so a classroom
  with bad wifi still gets working Python execution. The whole site runs from
  `file://`.
- **A lab checker that actually runs the code** — it executes student
  submissions in Pyodide and grades them four ways: static source patterns,
  scripted stdin cases, function probes, and files the lab should write. Not a
  regex match against a textarea.

Curriculum source is private while it's being finished.

→ [learnbymake.com](https://learnbymake.com) · [program & curriculum](https://learnbymake.com/program)

---

### AIec — AI elastic compute for autonomous agents

Isolated, disposable computers for agents: each sandbox gets its own kernel,
filesystem and network, and is destroyed the moment the work is done.
Independent open-source implementation, not affiliated with any commercial
sandbox platform.

The design follows **DeepSeek Elastic Compute (DSec)** — *"A Sandbox
Infrastructure for Effective Agentic Training at Scale"*, DeepSeek-AI &
Tsinghua University, 2026 ([arXiv:2609.22978](https://arxiv.org/abs/2609.22978)).
DSec's argument is that agent workloads arrive in bursts, need heterogeneous
isolation, and keep state across long interactions — which calls for an elastic
execution platform rather than one sandbox runtime. AIec implements the parts of
that which survive contact with a small team: one runtime contract with several
backends, capability-aware placement, and fenced, recoverable ownership.

The security commitment is that **the development runtime is not a security
boundary**. Docker and the dev backend are for local work and fail loudly as
unsafe for untrusted code; public workloads run on Firecracker microVMs and the
control plane refuses to hand a tenant a weaker runtime.

What is actually exercised, rather than asserted: a real Firecracker guest
clones a repository over HTTPS, edits a tracked file, runs validation and
returns a `git diff`; a worker is killed mid-flight and its sandbox is reassigned
to another worker at a higher fencing generation, which reconstructs the
workspace — and the old worker, when it returns, is rejected. Three consecutive
recovery iterations, cross-tenant access attempts against known resource IDs,
and a backup drilled by restoring the database and starting the control plane
against the restored copy.

```
aiec-core            domain contracts + Platform composition
aiec-runtime         Firecracker, hosted provider and Docker backends
aiec-storage         PostgreSQL metadata/scheduling, S3 artifacts
aiec-api             Axum API, worker RPC, orchestration, metrics
aiec-client          Rust SDK + CLI
aiec-network-linux   TAP and nftables isolation
aiec-mcp             local-only MCP server
```

The Python module is still `agentforge`, so the documented import stays
`from agentforge import AIec` — only the client class carries the product name.

It also ships a **local MCP server**, so any MCP-capable agent can ask for a
disposable machine on a cluster you run yourself. It is local-only by
construction rather than by convention: a remote AIec base URL is refused at
startup and cannot be enabled, the `hosted` and `e2b` runtimes are unavailable,
and there is no code path from a tool call to a process on the host serving MCP.
A full local worker reports `LOCAL_CAPACITY_UNAVAILABLE` rather than quietly
sending the workload to a cloud provider.

Exercised rather than asserted: an MCP client over Streamable HTTP created a
Firecracker microVM, ran `uname` and `/etc/os-release` inside it, wrote and read
a file, cloned a repository over HTTPS, edited a tracked file and returned the
`git diff`, then destroyed it. OMP installs and runs in a sandbox, and a real
model drove it to write a file into a cloned repository, touching nothing else.

No invented benchmark numbers — if it isn't measured, it isn't in the docs.

→ [github.com/fedoragobrowse-design/AIec](https://github.com/fedoragobrowse-design/AIec) ·
[aiec.gobrowse.dev](https://aiec.gobrowse.dev)

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
  extension marketplace
- [`harmonics-analysis`](https://github.com/fedoragobrowse-design/harmonics-analysis) — signal analysis,
  deployed to GitHub Pages
- `opensystemclass` — the Learn by Make curriculum source and class hub (private while in progress)

## Stack

**Teaching** — Python · Pyodide · deterministic SVG · Astro · offline-first

**Systems** — Rust · Embassy · Axum · PostgreSQL · Firecracker · RP2040/RP2350 ·
SX1276 · X25519 / ChaCha20-Poly1305 · MCP

## Contact

Enquiries about the program: [learnbymake.com/contact](https://learnbymake.com/contact).

Open to issues and PRs on any of the repos above. This is an alt account — the
main one is [@gobrowse](https://github.com/gobrowse).
