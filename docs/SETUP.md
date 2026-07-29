# Setup Guide

This guide covers every component of the QUANTUMSHIFT-XR system.

> ⚠️ **Order matters.** Start the Python AI server and the WebSocket relay **before** launching Unity. The Unity game connects to both at startup; if they are not yet running, it falls back gracefully but may not retry until the next scene load.

---

## Components Overview

| Component | Required? | Location |
|---|---|---|
| Unity project (Quest 3) | ✅ Yes | Repository root (`Edge-Cloud-AI-with-MRUK-main/`) |
| Python AI server | ❌ Optional | User-supplied (see Step 3) |
| WebSocket relay | ❌ Optional | `GameMasterServer/tactical_hub.py` |
| Phone dashboard | ❌ Optional | `GameMasterServer/index.html` |

---

## Step 1 · Unity Project

### First-time import

1. Open **Unity Hub**.
2. Click **Open → Add project from disk**.
3. Navigate to the repository root and select the **inner** `Edge-Cloud-AI-with-MRUK-main/` folder — the one that contains `Assets/`, `Packages/`, and `ProjectSettings/`.
4. Unity Hub will import the project. First import takes several minutes (shader compilation, package resolution via Package Manager).

### Targeting Meta Quest 3

1. **File → Build Settings** → Select **Android**.
2. Click **Switch Platform**.
3. In **Player Settings → XR Plug-in Management → Android**, ensure **OpenXR** is checked.
4. Under **OpenXR → Features**, enable **Meta Quest Support**.

### Testing in the Editor

1. Press **Play**. The game runs in a simulated XR environment.
2. `GameBridge` will attempt to connect to `http://127.0.0.1:8000` — if no AI server is running, it logs a warning and gameplay continues normally.
3. `NetworkClient` will attempt to connect to `ws://localhost:8765` — same graceful fallback.

### Building for Quest 3 (standalone)

1. Connect the Quest 3 via USB (Developer Mode must be enabled on the headset).
2. **File → Build Settings → Build and Run**.
3. The APK installs and launches automatically on the headset.

---

## Step 2 · Network Configuration

All devices must be on the **same Wi-Fi network** (or hotspot).

Find your PC's LAN IP address:
```powershell
# Windows
ipconfig
# Look for: IPv4 Address . . . : 192.168.x.x or 10.x.x.x
```

Set these values in the Unity Inspector (before building):

| Component | Inspector field | Value |
|---|---|---|
| `GameBridge` | `Server Base Url` | `http://<PC_IP>:8000` |
| `NetworkClient` | `Candidate Urls[0]` | `ws://<PC_IP>:8765` |

> The `GameBridge` component is created at runtime as `GameBridge_LocalAI`. To configure it persistently, find its `MonoBehaviour` in the scene or use the `[RuntimeInitializeOnLoadMethod]` auto-spawn and set fields via a config asset.

---

## Step 3 · Python AI Server (optional)

The AI server is **not included** in this repository. You must supply a compatible server.

`GameBridge.cs` expects these HTTP endpoints at the configured `Server Base Url`:

### Required endpoints

```
GET  /health
     → Return 200 OK (any body)

POST /game-state
     → Body: JSON (GameStatePacket — any JSON is accepted)
     → Return 200 OK

POST /commentary
     → Body: JSON { "eventType": "...", "details": "...", "state": {...} }
     → Return: {"text": "comment text", "audio_url": "/audio/file.wav"}
        OR: plain text (displayed as caption; no audio plays)
```

### Minimal test server (Python / FastAPI)

```python
# minimal_server.py — returns a static comment for all events
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/game-state")
async def game_state(req: Request):
    return {}

@app.post("/commentary")
async def commentary(req: Request):
    return JSONResponse({"text": "Nice shot!", "audio_url": ""})
```

```bash
pip install fastapi uvicorn
uvicorn minimal_server:app --host 0.0.0.0 --port 8000
```

### Full AI server (Ollama + Piper TTS)

```bash
# 1. Install Ollama
#    https://ollama.com/download

# 2. Pull the model (requires ~4 GB disk space)
ollama pull qwen:7b

# 3. Install Piper TTS
#    https://github.com/rhasspy/piper
pip install piper-tts

# 4. Build your FastAPI server that:
#    - Receives POST /commentary
#    - Sends the game context as a prompt to Ollama
#    - Pipes the response to Piper for synthesis
#    - Returns {"text": "...", "audio_url": "/audio/file.wav"}
#    - Serves the WAV file at the returned URL
```

---

## Step 4 · WebSocket Relay (optional)

Required only if using the phone Game Master dashboard.

```bash
cd GameMasterServer
pip install -r requirements.txt
python tactical_hub.py
```

Expected startup output:
```
============================================================
  TACTICAL HUB — Game Master WebSocket Relay
============================================================
  Listening on  ws://0.0.0.0:8765
  Started at    2026-07-26 10:00:00

  Waiting for connections from Phone UI and Unity...
============================================================

[10:00:00] 🟢 Server is LIVE on ws://0.0.0.0:8765
```

The server shuts down cleanly with **Ctrl+C**.

> **Windows note:** SIGTERM is not fully supported on Windows. Use Ctrl+C (`KeyboardInterrupt`) to stop the server. The relay handles this case and prints a shutdown message.

---

## Step 5 · Phone Game Master Dashboard (optional)

### Serving the dashboard

`index.html` is a self-contained single file with no build step. It needs to be served over HTTP (not opened as a local `file://` URL) so the WebSocket connection works correctly on most phones.

**Option A — Python quick server (same PC as the relay):**

```bash
cd GameMasterServer
python -m http.server 9000
```

Then open on your phone:
```
http://<PC_IP>:9000/index.html
```

**Option B — Any other static file server (nginx, Caddy, etc.) on the same LAN.**

### Using the dashboard

1. The status dot in the header turns **blue** when connected to the relay.
2. Select enemy type: **Normal** (alien) or **Boss**.
3. Tap anywhere on the radar circle to spawn at that world position.
4. Use the zoom slider to scale the radar range (1× = 20m radius, 4× = 80m radius).
5. The spawn history log shows the last 30 commands.

---

## Troubleshooting

### Unity Console: "GameBridge: Server not found"

- Verify the AI server is running: `curl http://<PC_IP>:8000/health` should return 200.
- Verify Quest and PC are on the same Wi-Fi network.
- Check that `Server Base Url` in `GameBridge` is the PC's LAN IP, not `127.0.0.1` (localhost refers to the Quest itself, not the PC).
- Check Windows Firewall: open inbound port 8000 TCP.

### Phone dashboard: status dot stays red

- Verify `tactical_hub.py` is running.
- Verify the phone is on the same Wi-Fi network as the PC.
- Check Windows Firewall: open inbound port 8765 TCP.
- Try navigating directly to `ws://<PC_IP>:8765` in the browser's developer console to verify connectivity.

### Unity: aliens don't spawn / all shots miss

- Verify the `Alien` and `Boss` physics layers exist: **Edit → Project Settings → Tags and Layers**.
- Both layers are defined in this project's `ProjectSettings/TagManager.asset` (layers 8 and 9).

### Quest: game builds but doesn't launch

- Ensure Developer Mode is enabled on the Quest (Meta Quest companion app → headset settings → Developer Mode).
- Ensure USB debugging is authorised (prompt appears inside the headset when connected via USB).
