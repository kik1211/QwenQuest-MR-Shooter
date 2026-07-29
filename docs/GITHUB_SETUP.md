# GitHub Repository Configuration Guide

Everything in this file is ready to copy and paste directly into GitHub.
Do this after pushing the repository.

---

## 1 · Repository Description

Paste this into **Settings → General → Description**:

```
Mixed-reality alien shooter for Meta Quest 3 — local LLM commentary, AR-aware spawning, phone radar dashboard. Built in 24h at Snapdragon Multiverse Hackathon 2026.
```

*(160 characters — within GitHub's limit)*

---

## 2 · Repository Website / URL

If you record a demo video and upload it to YouTube, paste the URL here.
Otherwise, leave blank.

---

## 3 · Repository Topics

Add these in **About → ⚙ gear icon → Topics**:

```
unity
unity3d
meta-quest
mixed-reality
augmented-reality
xr
vr-game
ar-foundation
csharp
game-development
llm
local-ai
ai-commentary
python
websocket
hackathon
snapdragon
quest3
```

**How to add:**
1. Go to your repository on GitHub
2. Click the **⚙ gear icon** next to "About" (top-right of repo home page)
3. Type each topic and press Enter after each
4. Click **Save changes**

---

## 4 · Social Preview Image

GitHub shows a card when your repo link is shared on Twitter, LinkedIn, Slack, etc.

1. Go to **Settings → General → Social preview**
2. Click **Edit** and upload `docs/screenshots/hero.png`
   (a screenshot from inside the Quest showing the player and an alien in the real room)

If `hero.png` is not yet captured, see [`docs/screenshots/README.md`](screenshots/README.md).

---

## 5 · GitHub Badges (for the README)

These are already included in the new `README.md`. For reference, here are the raw badge URLs:

```markdown
![Unity](https://img.shields.io/badge/Unity-6000.3.19f1-black?logo=unity&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Meta%20Quest%203-blue?logo=meta&logoColor=white)
![AR Foundation](https://img.shields.io/badge/AR%20Foundation-6.4.1-7a5cff)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
```

**Optional additional badges** (add once you have them):

```markdown
<!-- If you add a GitHub Actions workflow: -->
![Build](https://github.com/<YOUR_USERNAME>/<YOUR_REPO>/actions/workflows/<WORKFLOW>.yml/badge.svg)

<!-- If you deploy a demo video to YouTube: -->
[![Demo Video](https://img.shields.io/badge/Demo-YouTube-red?logo=youtube)](https://youtu.be/YOUR_VIDEO_ID)
```

---

## 6 · First Release

### Release Title

```
v1.0.0 — Hackathon Submission
```

### Release Tag

```
v1.0.0
```

### Release Notes (ready to paste)

```markdown
## QUANTUMSHIFT-XR v1.0.0

Built in 24 hours at the **Snapdragon Multiverse Hackathon Finale**,
Qualcomm Bangalore Campus, July 2026.

### What's included

**Unity project (Meta Quest 3)**
- Mixed-reality alien shooter — real-room passthrough, AR-aware spawning
- Gem Hunt mode: collect 4 gems → darkness → boss fight → victory screen
- Classic wave mode: endless waves, boss every 5 waves
- Raycast combat with location damage (headshots 3×, body shots 1×)
- Melee punch via controller velocity
- Aim Down Sights (ADS) system
- Procedural audio and VFX — no asset files required
- Weapon pickup system (classic mode)

**AI Commentary**
- `GameBridge.cs` bridges Unity → local Python AI server
- Qwen (via Ollama) + Piper TTS; zero cloud LLM dependency
- Typewriter caption overlay, audio playback, auto event triggers

**Game Master Dashboard**
- `GameMasterServer/tactical_hub.py` — WebSocket relay server
- `GameMasterServer/index.html` — radar dashboard for phone browsers
- Tap the radar to spawn enemies at real-world coordinates

**Extras**
- 3D-printable Quest 3 gun attachment (2× STL files)

### Known Limitations

See `README.md → Current Limitations` for the full list.

### Setup

See `README.md → Setup & Installation`.
```

---

## 7 · Recommended Repository Settings

| Setting | Recommended | Reason |
|---|---|---|
| **Issues** | ✅ Enable | Bug reports and feature proposals |
| **Discussions** | Optional | Enable if you want community Q&A |
| **Wiki** | ❌ Disable | README + docs/ is sufficient |
| **Projects** | Optional | Useful if tracking post-hackathon work |
| **Packages** | ❌ Disable | Not applicable |
| **Allow forking** | ✅ Yes | Portfolio project — forks are fine |
| **Default branch** | `main` | |
| **Squash merging** | ✅ Preferred | Keeps history clean for a portfolio |
| **Auto-delete head branch on merge** | ✅ Yes | Keeps branch list clean |
| **Require PR before merging** | Optional | Enforce if you add collaborators |

---

## 8 · Pinned Repository (GitHub Profile)

If you pin this repository to your GitHub profile:

**Suggested description for the pinned card:**

```
MR alien shooter built in 24h — local LLM commentary, AR spawning, phone radar dashboard · Meta Quest 3 · Unity 6
```

---

## 9 · LinkedIn / Resume Project Description

Suggested one-liner for your CV or LinkedIn:

> **QUANTUMSHIFT-XR** (2026) — Mixed-reality shooter for Meta Quest 3 built in 24h at Qualcomm's Snapdragon Multiverse Hackathon. Unity 6, C#, AR Foundation, local LLM (Qwen via Ollama), Piper TTS, WebSocket relay, phone companion radar dashboard.

---

## 10 · Screenshots to Capture Before Publishing

| Priority | File | What to show |
|---|---|---|
| 🔴 Must-have | `docs/screenshots/hero.png` | Passthrough view: player's real room + alien |
| 🔴 Must-have | `docs/screenshots/architecture.png` | Block diagram of the 3-component system |
| 🟡 Recommended | `docs/screenshots/gameplay.gif` | 15–30s clip: shoot alien → pick up gem → boss appears |
| 🟡 Recommended | `docs/screenshots/dashboard.png` | Phone browser showing radar + connected status |
| 🟢 Optional | `docs/screenshots/terminal.png` | `tactical_hub.py` running, showing `🟢 Server is LIVE` |

See [`screenshots/README.md`](screenshots/README.md) for capture instructions.
