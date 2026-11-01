# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Added
- `.github/CONTRIBUTING.md` — contributor guide with environment table, layout overview, and PR checklist
- `.github/CODE_OF_CONDUCT.md` — Contributor Covenant v2.1
- `.github/ISSUE_TEMPLATE/bug_report.md` — structured bug report template
- `.github/ISSUE_TEMPLATE/feature_proposal.md` — scoped feature proposal template
- `.github/PULL_REQUEST_TEMPLATE.md` — PR checklist template
- `SECURITY.md` — security policy and known design-level considerations
- `CHANGELOG.md` — this file
- `LICENSE` — MIT License
- `GameMasterServer/requirements.txt` — Python dependency declaration
- `docs/ARCHITECTURE.md` — detailed architecture, state machine, protocol specifications
- `docs/SETUP.md` — full multi-component setup guide with troubleshooting

### Changed
- `README.md` — complete rewrite: accurate project structure, Mermaid diagrams, controls table, verified limitations, future improvements
- `.gitignore` — added macOS, Windows, Python venv, npm, and Vosk model patterns
- `GameMasterServer/tactical_hub.py` — module docstring updated with correct file paths and security note

### Fixed
- README setup instructions: phone dashboard URL corrected (relay port 8765 is WebSocket-only, not HTTP)

---

## [1.0.0] — 2026-07-26 · Hackathon Submission

*Built in 24 hours at the Snapdragon Multiverse Hackathon Finale, Qualcomm Bangalore Campus.*

### Gameplay

- Mixed-reality alien shooter on Meta Quest 3 — real-room passthrough, raycast combat
- Gem Hunt mode: collect 4 gems → darkness event → boss fight → victory
- Classic wave mode: endless waves with difficulty scaling, boss every 5 waves
- Location-based damage: headshots (3 HP) vs body shots (1 HP)
- Melee punch: velocity-based controller swing registers a hit
- ADS (Aim Down Sights): controller-to-camera distance boosts damage and tightens spread
- Semi-auto and full-auto modes
- Magazine + reload system (A button / primary button to reload)
- Weapon pickup system: special weapons drop after waves
- Procedural audio: all sound effects generated at runtime, no AudioClip assets required
- Procedural VFX: muzzle flash, hit particles, aim laser — self-constructed, no prefabs required

### AI Commentary

- `GameBridge.cs`: Local AI bridge to a Python FastAPI server
- Qwen model via Ollama for LLM commentary
- Piper TTS for spoken audio response
- Typewriter caption UI built procedurally at runtime
- Commentary queue with cooldown (configurable, default 2s)
- Automatic low-health and boss-low-health events
- Candidate server IP probe: works in Editor (`127.0.0.1`) and on Quest over Wi-Fi (LAN IP)

### AR Spawning

- 3-tier spawn strategy: AR bounding boxes (furniture) → AR floor planes → radial fallback
- Minimum spawn distance enforcement
- `RoomCrackVFX` rift effect spawns 1 second before each alien

### Phone Companion Dashboard

- `GameMasterServer/index.html`: radar-style top-down view, tap to spawn enemy at world position
- Normal / Boss enemy type selector
- Zoom controls (1×–4×, 20m–80m range)
- Spawn history log
- WebSocket reconnect with exponential back-off

### WebSocket Relay

- `GameMasterServer/tactical_hub.py`: lightweight relay server
- Broadcasts all messages from any client to all other clients
- Graceful shutdown on SIGINT/SIGTERM (Windows-compatible)

### Unity Stack

- Unity 6000.3.19f1
- Universal Render Pipeline (URP) 17.3.0
- AR Foundation 6.4.1
- XR Interaction Toolkit 3.4.1
- Meta XR SDK (MRUK) 203.0.0
- Unity Input System 1.19.0
- GLTFast 6.19.0 (runtime 3D model loading)

---

*This project is maintained as a portfolio piece. It is not a production service.*
