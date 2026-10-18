# AGENT.md — Fox-Adventure work journal

Read this FIRST, every session, before touching anything. Update it after EVERY file change.
Rule of this project: **if it is not written here, it did not happen.** Never carry state in your
head across a session boundary.

## Project Overview

Fox-Adventure is a small 3D third-person game in Godot 4. The player is a fox in a farm clearing.
The fox must steal chickens without being noticed by the farmer. Two verbs: **dash** (fast, loud)
and **sneak** (slow, silent). Scope is capped hard: 1 biome, 2 verbs, 1 threat archetype, ~10
minutes of play.

## Current Status

- **Nothing is implemented.** No `project.godot`, no scenes, no scripts, no tests. Documentation
  only.
- Godot is **not installed in this environment** as of this session (verified: no `godot` on PATH).
  The user has said they will install it before work starts. Do not download or install anything
  without explicit permission — the user objected strongly to that in the planning session.
- No build, no test run, no verification of any kind has been performed on game code, because no
  game code exists.
- Environment facts that matter (from `/home/kaida/Project/ENVIRONMENT.md`, re-verify before
  relying on them):
  - GPU is shared and mostly occupied by the user's local LLM (measured 11.8/12.2 GiB used during
    planning). Godot editor launch may OOM. Prefer headless.
  - Renderer chosen: **Compatibility (OpenGL)** — lowest VRAM appetite.
  - The user verifies anything visual themselves. Never claim something looks good. Ask for a
    screenshot (niri `screenshot-window` writes a PNG that can be read and analysed).
  - Disk was at 93% used / 37 GB free during planning.

## Current Task

Planning documentation was written. **Next action: increment I00** (see `PLAN.md`) — project
scaffold, `project.godot`, folder tree, complete input map. No gameplay code in I00.

Before I01, confirm the Godot binary path with the user (they said it will be installed before we
start; it is not on PATH yet as of this session).

## Locked Decisions

Do not re-litigate these without the user asking.

| Topic | Decision | Reason |
|---|---|---|
| Engine | Godot 4.7.2-stable, GDScript | latest stable; no compile step, fastest iteration |
| Renderer | Compatibility (OpenGL) | lowest VRAM; user's GPU is shared with a local LLM |
| Camera | Third-person, orbit + follow damping + occlusion push-in | user's choice |
| Verbs | Dash **and** Sneak | user's choice; unified by the noise system (see below) |
| Threat | **A farmer** | user's choice |
| Collectibles | **Chickens** | user's choice |
| Win condition | Collect all chickens, then reach the den | user's choice |
| Gamepad | Supported from I02, input map defined in I00 | retrofitting gamepad later is painful |
| Platform | Linux desktop build; web export NOT required | user's choice |
| Art | Greybox first (CSG + capsule), Blender art deferred to I20 | user chose "art later" |
| Physics | `CharacterBody3D` + `move_and_slide` | standard, predictable |
| Cadence | Small increments, one verified checkpoint each, then STOP | user's explicit preference; context budget |

### The core design idea (why dash and sneak are one system)

Every action emits a **noise radius**: sneak = 0, walk = small, run = medium, dash = large.
The farmer hears noise and sees inside a vision cone. This makes one detection system serve both
verbs and creates the real decision: *dash gets you there fast but announces you; sneak is slow
but silent.*

**Proposed extension (NOT yet approved — confirm at I13):** chickens are skittish. A fox that runs
or dashes near a chicken spooks it; a spooked chicken squawks, and the squawk makes noise that
alerts the farmer. The collectible itself becomes an intermediary in the detection system, so
"sneak up on the chicken" is the actual verb pairing rather than "sneak up on the farmer".

### AI scope cap (this is where small games die — enforce it)

- **1 guard archetype**: the farmer.
- **4 states**: `PATROL → ALERT → SEARCH → CHASE`.
- Vision cone + noise radius, both **debug-drawn** so detection is verifiable without art.
- On caught: **drop one chicken, return to last checkpoint**. No combat, no health bar, no alert
  propagation between farmers, no per-guard per-frame raycast storms.
- **1–2 farmers in the slice.**

## Key Decisions & Pitfalls

- **Greybox capsule is built to the real fox's proportions** (height, body length, eye height,
  centre of mass) so swapping in a model at I20 does not re-tune the controller. Cheap now,
  expensive to retrofit.
- **All feel numbers live in `FoxTuning`, a `Resource` with `@export` fields** — never hardcoded.
  Lets the user drag sliders in the Inspector and lets us diff a `.tres` instead of a script.
- **Movement math is a pure function**, the node only applies it. This is what makes locomotion
  testable headless, with no window and no GPU.
- **Test assertions are an in-tree ~40-line lib.** No gdUnit4/gut download; dependency list stays
  at zero.
- Physics layers: 1 world · 2 character · 3 trigger · 4 camera-occluder. Camera ignores triggers;
  character ignores camera-occluders.
- `.godot/` import cache: generate with `godot --headless --import` (CPU only, no VRAM) rather
  than launching the editor.
- Art pipeline contract, fixed in advance so I20 is mechanical: `assets/blender/*.blend` + the
  generating `.py` → export `.glb` → `assets/models/`. Transforms applied, 1 unit = 1 m, no
  lights/cameras in the glb, **animation clip names equal `AnimationTree` names**
  (`fox_idle`, `fox_run`, `fox_sneak`, `fox_jump`, `fox_dash`). Fox budget ≤ 2k tris, one atlas
  material.
- Known weak spot for the art phase: scripted rigging/weight painting. Plan for Rigify or a simple
  6-bone rig plus scripted secondary motion (tail/ear sway). Expect 2–3 ugly iterations on the run
  cycle.

## Progress Journal

### 2026-10-18 — Session 1: planning only

- Read `Instruction.md` (project rules) and `ENVIRONMENT.md` (host facts). Followed the greeting
  rule.
- Verified tooling without installing anything: no `godot`, no `blender` on PATH; `python3 3.14.7`,
  `uv` present, `git` present; network reachable; disk 93% used / 37 GB free; GPU 11.8/12.2 GiB
  already used by the user's LLM. Checked latest Godot release via GitHub API: **4.7.2-stable**.
- User explicitly said: **do not download or install stuff.** No downloads were performed. Only
  read-only HTTP HEAD/GET version checks.
- Brainstormed scope, then locked the decisions table above. User chose: 3rd person camera, dash +
  sneak, collectibles as win condition, gamepad from the start, Linux desktop target, small
  increments.
- User chose the threat (**a farmer**) and the collectibles (**chickens**). The "Godot install path"
  question was closed by the user: Godot will be installed before work starts, so it is not tracked
  as an open question.
- Created directory `Fox-Adventure/` and wrote `AGENT.md` (this file).
- Wrote `PLAN.md` (decision table, I00–I20 ladder with checkboxes, verification strategy, risks,
  definition of done). Verified: file exists, 1 increment ladder present.
- Wrote `ISSUES.md` (no bugs; 3 open design questions: are chickens spooked, how many
  chickens/farmers, carry vs herd). Verified: file exists.
- Wrote `README.md` (states plainly that nothing is implemented; documents no unbuilt features as
  if they existed). Verified: file exists.
- Did NOT create `scenes/ scripts/ assets/ tests/` — they will be created by I00 when real files
  go into them. **No `.tscn`, no `.gd`, no `project.godot` exist yet.**
- Nothing was downloaded or installed. No build or test was run, because there is nothing to build.

## Next Steps

1. **I00** — project scaffold: `project.godot` (Compatibility renderer, 1 unit = 1 m, physics
   layers 1–4, main scene), folder tree, and the **complete input map** (move, jump, dash, sneak,
   pause, restart — keyboard AND gamepad). No gameplay code.
2. Ask the user for the Godot binary path, then run the I00 checkpoint:
   `godot --headless --import` and a headless boot, expecting exit 0 and zero script errors.
3. Then I01 (greybox world + fox capsule at real fox proportions). Stop after each increment.

## Project Files

- `AGENT.md` — this journal. Read first, update after every change.
- `PLAN.md` — locked decisions, the I00–I20 increment ladder with checkboxes, verification
  strategy, risks, definition of done. The durable memory of scope.
- `README.md` — user-facing documentation. Currently documents only the planning state.
- `ISSUES.md` — known bugs and open questions. Currently no bugs.
- `scenes/ scripts/ assets/ tests/` — do not exist yet; I00 creates them.
