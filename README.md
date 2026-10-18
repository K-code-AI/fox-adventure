# Fox-Adventure

A small 3D third-person game in Godot 4. You are a fox. A farmer patrols a farm clearing. You
steal the chickens and carry them back to your den without being noticed.

**Dash** gets you there fast, but every action makes noise and the farmer hears it.
**Sneak** is slow, but silent.

## Status: planning only — nothing is implemented

There is no playable build. No `project.godot`, no scenes, no scripts, no tests exist yet. The
repository currently contains documentation only:

| File | Contents |
|---|---|
| `PLAN.md` | Locked decisions, the I00–I20 increment ladder, verification strategy, risks, definition of done |
| `AGENT.md` | Work journal and session handoff |
| `ISSUES.md` | Bugs (none yet) and open design questions |

Everything below describes the agreed plan, not shipped functionality. Do not expect any of it to
run.

## Planned design

- **Camera**: third-person, orbit with follow damping and occlusion push-in.
- **Verbs**: dash and sneak.
- **Core mechanic**: every action emits a noise radius (sneak = 0, walk = small, run = medium,
  dash = large). The farmer hears noise and sees inside a vision cone.
- **Threat**: a farmer with four states — `PATROL`, `ALERT`, `SEARCH`, `CHASE`. Getting caught
  drops one chicken and returns you to the last checkpoint.
- **Goal**: collect all chickens, then reach the den.
- **Scope cap**: 1 biome, 2 verbs, 1 threat archetype, about 10 minutes of play.
- **Art**: greybox primitives first; a modelled fox and environment come last (increment I20).

## Requirements (not yet satisfied)

- Godot **4.7.2-stable** (standard build, not the .NET build), Linux x86_64.
  The engine is not installed in this environment; the user installs it before work begins.
- Renderer target: **Compatibility (OpenGL)** — chosen for low VRAM use, since the development
  machine's GPU is shared with a local LLM.

## How to build and run (once I00 exists)

Not yet available. Planned commands:

```sh
godot --headless --import          # build the import cache, no window, no GPU
godot --path .                     # run the game
godot --headless --path . -s res://tests/runner.gd   # run the test suite
```

Planned Linux export:

```sh
godot --headless --export-release "Linux/X11" build/fox-adventure.x86_64
```

These will be documented for real, with verified output, as each increment lands. Until then treat
them as intentions.

## Configuration

Planned: all movement and camera feel numbers live in a `FoxTuning` resource editable from the
Godot Inspector, rather than in code. Save files will live at `user://save.json`. Neither exists
yet.
