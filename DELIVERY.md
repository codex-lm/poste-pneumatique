# Delivery note — Pneumatic Post v1.0.0 (2026-10-31/11-01)

Honest account of what was built, tested, and how the specification was
interpreted. Tracking file for this mission: `MISSION.json`.

## Specification → implementation

| Spec | Status |
|---|---|
| 6×6 board, entrance/obstacles/destinations | ✔ entrances are brass ports *outside* the grid aimed at a first cell; houses are cells (accept from any side) |
| Limited inventory: straight / elbow / crossing / switch | ✔ per-level inventory; switch carries a fixed category (colour+symbol badge) |
| Capsule colour + symbol | ✔ 3 categories (coral heart, sky-blue star, yellow sun) — colour-blind safe (redundant symbol) |
| Switch: one category to the side branch, others straight | ✔ also defined: entering from the branch merges forward; against the flow passes through |
| Prepare, then simulate | ✔ edit mode / run mode; STOP returns to edit |
| Capsules spaced, never block each other | ✔ constant speed + fixed spawn spacing (2.6 s at 1.75 cells/s); no collision logic needed |
| Lost / wrong address / endless loop = failed run | ✔ three distinct failure messages and markers |
| Network stays in place after a run | ✔ win and fail overlays close back to edit with the network untouched |
| 10 hand-made levels, progressive | ✔ one house → elbows → crossing → switch → branches → 3 categories → economy → two entrances → knot → grand sorting |
| Economic bonus star | ✔ ★★ for solving, ★★★ at or under the par target (par = author solution, +1 piece of slack on introduction levels 4 and 8, per Kimi review) |
| Drag & drop, click to rotate, Launch/Stop, touch | ✔ pointer events everywhere; tap = rotate; drag back to tray = remove |
| Joyful illustrated city, coral/sky/yellow houses, transparent tubes, soft bounce | ✔ new palette family (not reused from previous games); capsules bounce-scaled on delivery with confetti puff |
| EN/FR/ES | ✔ full parity, EN default, persisted |
| Recorded + tested solution per level | ✔ `solution` field per level; Node suite replays them all |
| Exit-less circuit detection | ✔ loop detection by (cell, entry) state; capsule is shown circling one extra lap, then flagged — no infinite simulation (hard step guard too) |
| Routing tests, browser trial, public repo, LOB7 publication | ✔ see below |

## Tests (games/poste-pneumatique-dev/, outside this repo)

`node --test tests/pp.test.mjs` → **18/18 pass** against the shipped
`index.html` (loaded in a VM, no DOM):

- structure of all 10 levels (bounds, no overlap, inventory covers solution,
  every spawned category has a house, par within [solution, solution+2]);
- every recorded solution wins: all capsules delivered, 0 failures;
- level 7 par = Manhattan lower bound (pure economy level);
- engine units: straight (2 orientations), elbow (4 rotations), crossing
  (independent axes), switch (branch/straight/reverse/merge, all 4 orientations);
- failures: gap → lost, mismatched house → wrong, off-board exit → lost,
  crafted ring → **loop** detected with bounded path;
- determinism (identical double simulation), shared-crossing levels, i18n
  key parity, self-contained static page (no external src/href, no network calls).

## Real browser play (local HTTP, IAB panel)

Level 1 won by real drag & drop (5 pipes) → ★★★; level 2 failed on a
deliberate gap (“A capsule was lost”), “Fix the network” kept the pieces,
tap-rotation checked, then won with the recorded solution; level 4 (first
switch) and level 10 (finale) won ★★★; STOP mid-run returns to edit with the
network intact; FR/EN switching complete (title, rules, hints, buttons);
mobile 390×780 layout verified. Evidence screenshots: `evidence/poste-pneumatique/`.

**Environment limitation (honest):** the in-app browser panel currently
routes pointer *moves* but no button presses (known local panel fault, seen by
other sessions today). Interactions were therefore replayed through the game's
own PointerEvent handlers (same code path: hit-testing, drag state machine,
inventory), and the run clock through the game's watchdog/`step()` hook —
both are part of the shipped game. A plain browser needs no workaround.

## Known limits

- Switch orientation semantics (branch merge / against-the-flow) are a design
  choice — the rules screen documents the main behaviour only.
- Best-stars persistence is local (guarded localStorage); no cloud save.
- Audio is synthesized (Web Audio); no separate volume slider, toggle only.
