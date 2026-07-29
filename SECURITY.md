# Security Policy

## Supported Versions

This is a portfolio project built at a hackathon. It is not a production service. There is no versioned release history with security patch commitments.

| Version | Supported |
|---|---|
| `main` branch | ✅ Best-effort |
| Any tagged release | ⚠️ Best-effort |
| Forks / derivatives | ❌ Not supported |

---

## Scope

### In scope

- **`GameMasterServer/tactical_hub.py`** — WebSocket relay server. It binds to `0.0.0.0:8765` and broadcasts all received messages to all connected clients. This is intentional for the LAN-demo use-case but would be a significant risk if exposed to the internet.
- **`GameMasterServer/index.html`** — Browser dashboard. Connects to the relay via WebSocket; no server-side state, no authentication.
- **`Assets/Scripts/GameBridge.cs`** — Unity-side HTTP client. Connects to a local Python AI server (`127.0.0.1:8000` or a LAN IP). Sends game-state JSON and receives AI commentary text + audio URLs.

### Out of scope

- Issues with third-party packages (`Ollama`, `Sarvam AI`, `AR Foundation`, Unity packages) — report those upstream.
- Issues arising from running the server on a public network — this project is designed for a closed LAN/hotspot.

---

## Known Security Considerations

> These are by-design limitations for a hackathon demo, not vulnerabilities. Document them here so contributors are informed.

1. **No authentication on the WebSocket relay.** Any device on the same network can connect to `tactical_hub.py` and send spawn commands to Unity. Do not run this server on an untrusted network.

2. **No input validation on the relay.** `tactical_hub.py` re-broadcasts all received messages to all other clients without sanitising the payload. A malicious client could send arbitrary JSON to Unity.

3. **No HTTPS.** Both the relay (`ws://`) and the AI server (`http://`) use plaintext protocols. This is acceptable for a closed LAN demo.

4. **`GameBridge.cs` uses reflection.** The bridge reads fields from arbitrary `MonoBehaviour` components by name. This is an internal code-quality consideration, not a remote attack surface.

---

## Reporting a Vulnerability

If you discover a genuine security issue (e.g., the WebSocket relay can be exploited in a way that was not anticipated), please **do not open a public GitHub Issue**.

Instead, report it privately via:

- **GitHub private vulnerability reporting**: use the "Report a vulnerability" button on the Security tab of this repository.

You will receive a response within **7 business days**. Given the scope of this project, the most likely response will be a public advisory + documentation update rather than a patched release.

---

## Recommendations for Deployers

If you adapt this project for a real deployment (beyond a hackathon demo):

- Run `tactical_hub.py` on a firewalled machine, not exposed to the internet.
- Add token-based authentication to the WebSocket relay before exposing it outside a trusted LAN.
- Replace plain `http://` with `https://` for the AI server connection in `GameBridge.cs`.
- Replace Unity's deprecated `FindObjectsOfType` calls in `GameBridge.cs` with cached/event-driven component references.
