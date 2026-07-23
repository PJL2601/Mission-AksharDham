# Mission Akshardham — Buzzer Edition 🔔

A zero-dependency, LAN-only quiz buzzer system built with vanilla Node.js and Server-Sent Events. One laptop acts as the referee, players buzz in from their own devices, and the game is displayed on a projector — no internet, no build step, no npm install.

![Node](https://img.shields.io/badge/node-%3E%3D14-339933?logo=node.js&logoColor=white)
![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

## How it works

```
┌─────────────┐   HTTP + SSE    ┌───────────────┐   HTTP + SSE   ┌─────────────┐
│  buzzer.html │ ◄─────────────► │   server.js   │ ◄─────────────► │ stage.html  │
│  (iPads /    │                 │  (Node http,  │                │ (projector) │
│   phones)    │                 │  no deps)     │                │             │
└─────────────┘                 └───────────────┘                └─────────────┘
```

- **`server.js`** is a plain Node `http` server (no Express, no framework). It serves the two HTML clients, timestamps every buzz with `perf_hooks.performance.now()`, and pushes state to all connected clients over **Server-Sent Events** (`/events`).
- **`buzzer.html`** is the client each player device opens — a big buzzer button that's disabled until the host arms it.
- **`stage.html`** is the host/projector view — team setup, scoring, question flow, and a self-contained fallback game if the server/network isn't available.
- **`music.mp3`** is background audio streamed by the server and controlled from the stage.

All game state lives in memory on the server (`state.teams`, `state.phase`, `state.buzzes`, `state.scores`, ...) and is broadcast to every client on change — no database, no persistence between restarts.

## Features

- **Sub-millisecond buzz timestamping** with a configurable near-tie window (`TIE_MS`) so the host can see and override close calls
- **Countdown trap** — a randomized arm sequence that penalizes false starts
- **Live scoring** with manual host adjustment (±) and a running event log
- **Steal/lockout logic** — wrong answers reopen buzzers for everyone except the team that missed
- **Offline fallback** — `stage.html` runs as a complete standalone game (keyboard input) if opened without the server
- **Zero dependencies** — runs anywhere Node.js runs, no `npm install`

## Requirements

- [Node.js](https://nodejs.org) 14+ (LTS recommended)
- All devices (host laptop, player devices, projector) on the same local network (Wi-Fi hotspot, router, or laptop-hosted hotspot) — **no internet connection required**

## Getting started

```bash
git clone <this-repo-url>
cd mission-akshardham-v2
node server.js
```

The server prints the LAN URLs to open:

```
📱 Buzzers open:    http://192.168.1.5:8080
🖥️  Projector open:  http://192.168.1.5:8080/stage
```

- Open the projector URL on the host machine and put it in fullscreen for the display.
- Have each player device open the buzzer URL in a browser.
- Build teams from the stage view — connected buzzers are shown live.

See [`GAME-GUIDE.md`](./GAME-GUIDE.md) for full host instructions, network setup options, and a venue-day checklist.

## Project structure

```
.
├── server.js         # Referee server — routing, SSE, buzz timing, game state
├── stage.html         # Host/projector UI (also works standalone, no server)
├── buzzer.html         # Player buzzer UI
├── music.mp3          # Background music, served by the server
└── GAME-GUIDE.md       # Host-facing setup and run instructions
```

## API overview

The server exposes a small REST + SSE surface, all consumed by the bundled clients:

| Endpoint | Method | Purpose |
|---|---|---|
| `/events?kind=stage\|buzzer` | GET | Subscribe to live game state (SSE) |
| `/api/info` | GET | LAN URLs + connected buzzer count |
| `/api/teams` | POST | Set/reset teams and scores |
| `/api/arm` | POST | Open buzzers for a question (optionally as a "trap") |
| `/api/close` | POST | Close buzzers |
| `/api/buzz` | POST | Register a buzz from a player device |
| `/api/scores` | POST | Update scores |
| `/api/swap` | POST | Reassign a locked buzz to a different team (near-tie override) |

## Contributing

Issues and pull requests are welcome. This project intentionally has no build tooling or dependencies — please keep it that way when contributing.

## License

MIT
