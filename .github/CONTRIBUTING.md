# Contributing to QUANTUMSHIFT-XR

Thank you for your interest in this project. It was originally built in 24 hours at the **Snapdragon Multiverse Hackathon Finale** (Qualcomm Bangalore Campus, July 2026) and is now maintained as an open portfolio project.

---

## Ground Rules

- **Preserve existing gameplay behaviour.** Do not change game mechanics, networking protocols, or the AI commentary pipeline without a detailed proposal first.
- **One concern per pull request.** Small, focused PRs are reviewed faster and are easier to revert if a problem is found.
- **No invented benchmarks.** Any performance claims must be reproducible. If you measured something, document exactly how.
- **Keep technical claims accurate.** Do not describe the project as using technology it does not use.

---

## Development Environment

| Tool | Version | Notes |
|---|---|---|
| Unity | 6000.3.19f1 | Open `Edge-Cloud-AI-with-MRUK-main/` in Unity Hub |
| Python | 3.10 + | For `GameMasterServer/tactical_hub.py` |
| Meta XR SDK (MRUK) | 203.0.0 | Installed via `Packages/manifest.json` |
| AR Foundation | 6.4.1 | Same manifest |
| XR Interaction Toolkit | 3.4.1 | Same manifest |

> **Important:** The Unity project root is the inner folder `Edge-Cloud-AI-with-MRUK-main/`, not the outer zip-extracted parent.

---

## Project Layout

```
Edge-Cloud-AI-with-MRUK-main/
├── Assets/
│   └── Scripts/          ← All C# game scripts (Unity)
├── GameMasterServer/
│   ├── tactical_hub.py   ← WebSocket relay server (Python)
│   ├── index.html        ← Phone companion dashboard (browser-only)
│   └── requirements.txt  ← Python dependencies
├── Packages/
│   └── manifest.json     ← Unity package dependencies
├── ProjectSettings/      ← Unity project settings (do not hand-edit)
├── docs/                 ← Architecture, setup, and GitHub configuration guides
├── QuestGunAttachment*.stl  ← 3D-printable controller grip (2 parts)
├── .github/              ← GitHub-specific files (this folder)
├── CHANGELOG.md
├── LICENSE
├── SECURITY.md
└── README.md
```

---

## Making Changes

### Unity / C#

1. Open the project in Unity Hub → Unity 6000.3.19f1.
2. Make your changes in `Assets/Scripts/`.
3. Enter Play Mode in the Editor to verify the change compiles and runs without errors.
4. If you change a public API on any of the five core scripts (`GameManager`, `GameBridge`, `PlayerHealth`, `PlayerShooter`, `AlienWaveManager`), update the relevant XML doc-comments.

### Python Server

1. Install dependencies:
   ```bash
   cd GameMasterServer
   pip install -r requirements.txt
   ```
2. Run the relay:
   ```bash
   python tactical_hub.py
   ```
3. Verify the banner prints and the server accepts a WebSocket connection from the phone dashboard.

### Phone Dashboard

The dashboard (`GameMasterServer/index.html`) is a single self-contained HTML file with no build step. Open it directly in a browser for testing. It connects via WebSocket to `ws://<hostname>:8765` (the relay server).

---

## Pull Request Checklist

Before opening a PR, confirm:

- [ ] The Unity project opens without import errors
- [ ] All modified scripts compile with zero errors in the Unity Console
- [ ] Gameplay behaviour (shooting, waves, gem hunt) is unchanged
- [ ] Any new public methods have XML doc-comments
- [ ] The PR description explains **why** the change was made, not just what

---

## Reporting Bugs

Open a GitHub Issue with:

1. A description of the observed behaviour
2. Steps to reproduce (device, Unity version, server running?)
3. The relevant Unity Console log lines (filter: Errors only)
4. Whether the bug occurs in the Editor, on-device (Quest), or both

---

## Scope Limitations

This project does not currently have:

- Automated tests (Unity Test Framework is in the manifest but no tests are written)
- A CI pipeline
- A staging or deployment environment

PRs that add any of these are welcome, provided they do not alter gameplay.

---

*This project was built at a hackathon and is maintained as a portfolio piece. Reviews may take time. Thank you for your patience.*
