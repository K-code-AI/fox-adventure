# PLAN.md — Fox-Adventure

Scope, decisions and the increment ladder. This file is the durable memory of *what we agreed to
build*. Session-to-session state and what was actually done live in `AGENT.md`.

## What the game is

A 3D third-person game. You are a fox. A farmer patrols a clearing. There are chickens. You steal
the chickens and carry them back to your den without being noticed. **Dash** gets you there fast
but is loud. **Sneak** is slow but silent. Collect all chickens, reach the den, win.

## Locked decisions

| Topic | Decision |
|---|---|
| Engine | Godot 4.7.2-stable, GDScript |
| Renderer | Compatibility (OpenGL) — lowest VRAM, GPU is shared with the user's LLM |
| Camera | Third-person: orbit, follow damping, occlusion push-in |
| Verbs | Dash + Sneak |
| Threat | A farmer (1 archetype, 1–2 in the slice) |
| Collectibles | Chickens |
| Win | Collect all chickens → reach the den |
| Input | Keyboard/mouse AND gamepad, from I02 |
| Platform | Linux desktop build. Web export not required |
| Art | Greybox now (CSG + capsule); Blender art deferred to I20 |
| Physics | `CharacterBody3D` + `move_and_slide` |
| Cadence | One increment per session, verified checkpoint, then stop |

**Scope cap (hard):** 1 biome · 2 verbs · 1 threat archetype · ~10 minutes of play.

## The core mechanic

Every action emits a **noise radius**: sneak = 0, walk = small, run = medium, dash = large.
The farmer hears noise and sees inside a vision cone. One detection system serves both verbs and
creates the decision: fast-and-loud vs slow-and-silent.

**Proposed extension (unapproved, confirm at I13):** chickens are skittish — running or dashing
near one spooks it, and a spooked chicken squawks, which makes noise that alerts the farmer. The
collectible becomes an intermediary in the detection system.

### AI scope cap

- 1 guard archetype (the farmer), 1–2 instances.
- 4 states: `PATROL → ALERT → SEARCH → CHASE`.
- Vision cone + noise radius, **debug-drawn** so detection is verifiable without art.
- Caught = drop one chicken, return to last checkpoint.
- Explicitly NOT in scope: combat, health bars, alert propagation between farmers, per-guard
  per-frame raycast storms, multiple threat types.

## Repo layout

```
Fox-Adventure/
  AGENT.md  README.md  ISSUES.md  PLAN.md
  project.godot  icon.svg
  scenes/
    main.tscn
    player/fox.tscn
    levels/clearing.tscn
    interaction/collectible.tscn  den.tscn  trigger.tscn
    ui/hud.tscn  pause_menu.tscn
  scripts/
    autoload/events.gd  game_state.gd  save_manager.gd
    player/fox.gd  fox_states.gd
    camera/third_person_camera.gd
    interaction/*.gd
    levels/clearing.gd
    tuning/fox_tuning.gd
  assets/
    models/  materials/  audio/  blender/     # blender/ stays empty until I20
  tests/
    runner.gd  assert_lib.gd  test_fox.gd  test_save.gd  test_goals.gd  test_noise.gd
    run_tests.sh
```

`FoxTuning` is a `Resource` with `@export` fields (run speed, accel, gravity, jump velocity,
coyote time, dash cooldown, camera damping, noise radii). **Every feel number lives there, never
hardcoded.**

## Architecture

- **`Events` autoload** — one signal bus (`collectible_picked`, `player_landed`, `goal_reached`,
  `fox_state_changed`, `noise_emitted`, `farmer_detected_player`). Systems never reference each
  other directly.
- **`GameState`** — title/playing/paused/won; owns chicken count and timer.
- **`SaveManager`** — `user://save.json`, versioned, explicit `save()`/`load()`.
- **Fox** = `CharacterBody3D` → `CollisionShape3D` + `MeshInstance3D` (capsule) + `SpringArm3D` +
  `Camera3D` + `AnimationPlayer` (stubbed until I20).
- **State machine**: `IDLE · RUN · SNEAK · JUMP · FALL · LAND · DASH` — one script, enum + table.
  Seven states do not earn a node-per-state tree.
- **Physics layers**: 1 world · 2 character · 3 trigger · 4 camera-occluder.

## Verification strategy

- `tests/run_tests.sh` → `godot --headless --path . -s res://tests/runner.gd`
- In-tree ~40-line assertion library. **Zero external test dependencies.**
- Movement math is a **pure function**; the node only applies the result. This is what makes
  locomotion testable headless with no window and no GPU.
- Import cache built with `godot --headless --import` (CPU only, no VRAM).
- **Hard rule: the agent never claims anything looks right.** Visual verdicts belong to the user.
  When eyes are needed, ask for a screenshot PNG.

## Increment ladder

One row = one working session. Nothing starts until the previous row is green. Tick the box only
when the checkpoint actually passes.

### Phase A — skeleton

- [ ] **I00** Scaffold: `project.godot` (Compatibility, 1 unit = 1 m, physics layers 1–4, main
      scene), folder tree, **complete input map** (move, jump, dash, sneak, pause, restart — KB +
      gamepad). No gameplay code.
      *Checkpoint: `--headless --import` then headless boot, exit 0, zero script errors.*
- [ ] **I01** Greybox world: floor + CSG blocks, fox capsule **at real fox proportions**, camera
      parented, no movement.
      *Checkpoint: scene loads headless, fox exists at origin.*
- [ ] **I02** Move: KB + gamepad, accel/friction, `FoxTuning` resource with every number
      `@export`'d.
      *Checkpoint: headless sim asserts the fox reaches a target position in N seconds.*
- [ ] **I03** Jump: gravity, coyote time, jump buffering, variable height.
      *Checkpoint: asserts apex height and air time match tuning.*
- [ ] **I04** Dash: cooldown, dash duration, no mid-air spam.
      *Checkpoint: asserts dash distance and cooldown enforcement.*
- [ ] **I05** Sneak: hold/toggle posture, capsule height change, speed tiers (sneak/walk/run).
      *Checkpoint: asserts speed per posture and capsule height.*

### Phase B — camera & feel

- [ ] **I06** Third-person camera: orbit, damping, sensitivity + invert-Y.
      *Checkpoint: user confirms; headless asserts no NaN/jitter over a scripted walk.*
- [ ] **I07** Camera occlusion push-in (layer 4).
      *Checkpoint: scripted walk past a wall, assert arm length shortens.*
- [ ] **I08** Squash/stretch, landing dust, camera shake.
      *Checkpoint: user confirms.*

### Phase C — the loop

- [ ] **I09** Level blockout: clearing layout, checkpoints, respawn.
      *Checkpoint: fall off → respawn at checkpoint.*
- [ ] **I10** Chickens as collectibles: count in `GameState`, HUD.
      *Checkpoint: headless test collects all of them.*
- [ ] **I11** Den + win condition + win screen.
      *Checkpoint: headless test drives fox to den → `goal_reached`.*
- [ ] **I12** Save/load + pause menu + restart.
      *Checkpoint: save round-trip test.*

### Phase D — sneak / farmer AI

- [ ] **I13** Noise system: noise events with radius, emitted by posture and dash.
      *Checkpoint: unit tests on noise emission.*
- [ ] **I14** Farmer skeleton: patrol path, `PATROL` state, debug-drawn vision cone.
      *Checkpoint: farmer loops the path; cone visible in debug.*
- [ ] **I15** Detection: vision cone + hearing, `ALERT`/`SEARCH`/`CHASE`.
      *Checkpoint: scripted scenarios — sneak past = undetected, dash past = detected.*
- [ ] **I16** Caught consequence: drop a chicken, return to checkpoint.
      *Checkpoint: headless test asserts both.*

### Phase E — finish

- [ ] **I17** Game loop UI: title → play → win → restart; settings.
      *Checkpoint: state-transition test.*
- [ ] **I18** Juice: day-night tint, wind, placeholder audio, cover/grass that visualises sneak.
      *Checkpoint: user confirms; 60 fps and draw-call count logged.*
- [ ] **I19** Linux export — standalone build runs.
      *Checkpoint: the user runs the binary.*
- [ ] **I20** Art phase (Blender decision point): fox model, rig, anims, environment kit, swapped
      into the capsule's slot.
      *Checkpoint: zero controller re-tuning; user confirms.*

## Risks

| Risk | Mitigation |
|---|---|
| VRAM contention with the user's LLM → editor OOM / GL faults | Compatibility renderer; user launches the editor with the LLM unloaded; agent stays headless |
| `.godot/` import cache needs one real import | `godot --headless --import`, no window |
| "Feel" is subjective and the agent cannot see it | `FoxTuning` resource; user gates I02–I06; screenshots as evidence |
| Capsule → fox swap re-tunes everything | capsule built to fox proportions from I01 |
| AI scope creep | the cap above, written into `AGENT.md` |
| Context exhaustion | one increment per session; `PLAN.md` + `AGENT.md` carry all state |
| Disk at 93% | no bulk asset packs; `.godot/` and `.blend` backups gitignored |

## Definition of done (the slice)

A player presses Play, drives a fox through a farm clearing, steals all the chickens without being
noticed by the farmer (or gets caught and pays for it), carries them to the den, sees a win screen,
and can quit and resume from a save — at 60 fps, with movement and camera that feel good, in under
10 minutes.
