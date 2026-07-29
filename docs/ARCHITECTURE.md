# Architecture Overview

This document describes the software architecture of QUANTUMSHIFT-XR as verified from source code.

---

## High-Level System

QUANTUMSHIFT-XR has three independently deployable components connected over a shared LAN:

```
┌─────────────────────────────────────────────────────────────┐
│                      Meta Quest 3                           │
│                                                             │
│   Unity 6 Game ──► GameBridge ──► HTTP :8000               │
│        │                                                    │
│        └────────► NetworkClient ──► WS :8765               │
└─────────────────────────────────────────────────────────────┘
              │                         │
              │ HTTP                    │ WebSocket
              ▼                         ▼
   ┌─────────────────┐       ┌─────────────────────┐
   │  Python AI      │       │  tactical_hub.py    │
   │  Server :8000   │       │  WebSocket Relay    │
   │  (user-supplied)│       │  :8765              │
   └─────────────────┘       └──────────┬──────────┘
                                        │ WebSocket
                                        ▼
                              ┌─────────────────────┐
                              │  Phone Browser      │
                              │  index.html         │
                              │  Radar Dashboard    │
                              └─────────────────────┘
```

Each component is optional. The Unity game runs without the AI server and without the relay — both subsystems gracefully degrade to no-ops.

---

## Unity Internal Architecture

### State Machine (GameManager)

```
WaitingToStart
    │ player pulls trigger
    ▼
Playing
    │ all aliens dead
    ▼
WaveComplete  ──► StartNextWave ──► Playing
    │                 │
    │                 └── every 5th wave ──► StartBossWave ──► Playing
    │
    │ player health reaches 0
    ▼
GameOver
    │ player pulls grip
    ▼
[RestartGame → scene cleanup → WaitingToStart]
```

### Core Script Responsibilities

| Script | Lines | Single Responsibility |
|---|---|---|
| `GameManager.cs` | ~350 | State machine, wave loop, scoring |
| `GameBridge.cs` | ~2052 | AI telemetry, commentary, HTTP, audio, caption UI |
| `PlayerHealth.cs` | ~124 | HP, regen, death, damage feedback |
| `PlayerShooter.cs` | ~857 | XR input, raycast, melee, ADS, reload, VFX |
| `AlienWaveManager.cs` | ~269 | AR-aware spawning, wave tracking |
| `AlienAI.cs` | ~453 | Enemy state machine (Spawning→Chasing→Attacking→Dying) |
| `NetworkClient.cs` | ~337 | WebSocket client (tactical hub) |
| `GemHuntDirector.cs` | ~295 | Gem Hunt mode orchestration |
| `GemManager.cs` | ~172 | Gem placement (AR-aware) + collection events |
| `GemBossController.cs` | ~318 | Gem Hunt final boss movement + health |
| `BossEnemy.cs` | ~723 | Classic wave boss (rigged GLB, dialogue) |
| `HorrorAtmosphere.cs` | ~435 | Boss-phase atmosphere (fog, embers, lightning) |

---

## Gem Hunt Mode Flow

```
StartRun()
  └── SpawnWave (3 aliens) ──────────────────────────────────┐
      └── [after wave clear] spawn next wave ◄───────────────┘
                                                │ 4th gem collected
  GemManager.onAllGemsCollected ────────────────┘
      └── HorrorAtmosphere.Activate()
      └── GemBossController spawns at nearest real wall
              └── BossHealth reaches 0
                      └── GameManager.OnGemHuntVictory()
                              └── Victory screen (time + shots)
```

---

## AR Spawn Strategy

`AlienWaveManager.GetSpawnPosition()` evaluates three tiers in order:

```
Priority 1: ARBoundingBoxManager trackables (furniture surfaces)
  → requires: ARBoundingBoxManager present + ≥1 trackable
  → chooses random bounding box, offset upward by its height
  → min-distance enforcement from player

Priority 2: ARPlaneManager trackables (floor/horizontal planes)
  → requires: ARPlaneManager present + ≥1 plane
  → random point on random plane
  → min-distance enforcement from player

Priority 3: Radial fallback (always available)
  → random angle 0–360°, distance 4–8m, player Y + offset
  → guarantees a spawn position even with no AR data
```

---

## AI Commentary Pipeline

### Server Endpoints (GameBridge expects)

```
GET  /health
     Response: any 200 OK → marks server online

POST /game-state
     Body: GameStatePacket (JSON)
     Response: 200 OK (body ignored)

POST /commentary
     Body: CommentaryRequest {
       eventType: string,
       details:   string,
       state:     GameStatePacket
     }
     Response: {
       text:      string,     ← caption to display
       audio_url: string      ← relative path to WAV/MP3/OGG
     }
     OR: plain text body (used as caption only, no audio)
```

### Commentary Trigger Events (from GameBridge source)

| Event | Trigger condition |
|---|---|
| `shot` | `ReportShot()` — any trigger pull |
| `hit` | `ReportHit()` — raycast connected |
| `headshot` | `ReportHeadshot()` — hit on head HitZone |
| `kill` | `ReportKill()` — enemy HP reached 0 |
| `multikill` | `ReportMultiKill()` — multiple kills in rapid succession |
| `boss_appeared` | `ReportBossAppeared()` |
| `boss_defeated` | `ReportBossDefeated()` |
| `player_damaged` | `ReportPlayerDamaged()` |
| `player_death` | `ReportPlayerDeath()` |
| `player_healed` | `ReportPlayerHealed()` |
| `wave_completed` | `ReportWaveCompleted()` |
| `low_health` | Polled: player HP < 25% |
| `boss_low_health` | Polled: boss HP < 25% |

> **Note (verified):** As of the hackathon submission, `PlayerShooter.cs` and `PlayerHealth.cs` do not call `GameBridge.Instance.Report*()` directly. The trigger for all events is the reflection-based state sampler (0.25s interval), not event push from the source scripts.

---

## NetworkClient → Tactical Hub Protocol

### Message Format (phone → hub → Unity)

```json
{
  "device": "phone",
  "action": "spawn",
  "intent": "none",
  "type":   "tiny_normal_tank",   // or "boss"
  "x":      7.4,                  // metres, Unity world X
  "y":      -3.1,                 // metres, mapped to Unity Z (depth)
  "scale":  1.0                   // radar zoom multiplier (informational)
}
```

The hub re-broadcasts this to all connected clients without modification.

`NetworkClient.cs` reads `type` and calls `AlienWaveManager.SpawnEnemyAt()` with the `x`/`y` coordinates at the player's floor Y plus `spawnHeightOffset`.

### Connection

`NetworkClient` tries each URL in `candidateUrls` in rotation on failure:

```
ws://10.79.7.153:8765   ← replace with your PC's LAN IP
ws://192.168.137.1:8765 ← PC hotspot (Windows ICS default gateway)
ws://localhost:8765     ← Editor / same-machine testing
```

> **Note:** The first URL (`10.79.7.153`) is the hardcoded IP from the hackathon demo network. Update `NetworkClient → Candidate Urls[0]` in the Unity Inspector before building for Quest.

Reconnect interval: 3 seconds (configurable in Inspector).

---


## Physics Layers (verified from TagManager.asset)

| Layer | Index | Used by |
|---|---|---|
| `Default` | 0 | General colliders |
| `TransparentFX` | 1 | |
| `Ignore Raycast` | 2 | |
| `Water` | 4 | |
| `UI` | 5 | |
| `Spawner Contact` | 6 | AR surface detection |
| `Placeable Surface` | 7 | AR surface detection |
| `Alien` | 8 | `PlayerShooter` hit detection |
| `Boss` | 9 | `PlayerShooter` hit detection |
| `XR Simulation` | 29 | Editor XR simulation |

`PlayerShooter` initialises `hitLayers = LayerMask.GetMask("Alien", "Boss")` in `Start()`. Both layers exist in the project.
