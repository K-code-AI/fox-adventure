# ISSUES.md — Fox-Adventure

Read before starting work. Record bugs with enough detail to reproduce. Move resolved items to
`Fixed` only after verifying the fix.

## Bugs

None. No code exists yet, so nothing has been executed or verified.

## Open questions

1. **Do chickens get spooked?** (design, decide at **I13**)
   Proposal: running or dashing near a chicken spooks it; a spooked chicken squawks; the squawk is
   a noise event that alerts the farmer. This makes the collectible itself part of the detection
   system, so "sneak up on the chicken" becomes the real verb pairing instead of "sneak up on the
   farmer".
   Cost: one extra state on the chicken (`CALM → SPOOKED → SETTLED`) and one extra noise source.
   Benefit: the sneak verb stops being optional.
   Not decided yet — deliberately deferred until the noise system exists, so the decision is made
   against working code rather than in the abstract.

2. **How many chickens, and how many farmers in the slice?** (level design, decide at **I09**)
   Plan says 1–2 farmers. Chicken count not fixed. Affects clearing layout and how long the 10
   minutes feels.

3. **Can the fox carry a chicken, or must it herd/drive them to the den?** (decide at **I10**)
   Carrying is simpler to implement and reads clearly. Herding is more interesting but needs chicken
   pathfinding, which is a second AI system and a scope risk. Default assumption until decided:
   **the fox carries one chicken at a time**, so a trip is chicken → den.

## Constraints that behave like issues

- **GPU is shared.** The user runs a local LLM on the same card (measured 11.8/12.2 GiB used during
  planning). Launching the Godot editor while the LLM is loaded risks `GL_OUT_OF_MEMORY` and driver
  faults. Use `godot --headless` for everything the agent runs; ask the user before any windowed
  launch. Re-check VRAM with `nvidia-smi` — the LLM comes and goes.
- **Godot is not installed in this environment yet.** The user will install it before work starts.
  Do not download or install it without explicit permission.
- **Disk is tight** (93% used, 37 GB free at planning time). No bulk asset packs.

## Fixed

Nothing yet.
