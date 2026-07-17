# Asteroids Arcade Replica + Mobile Play Plan

> **For Hermes:** Grok orchestrates; implement each pass as a **small Ornith/Aider slice** via `~/.hermes/scripts/aider-coder.sh` (prefer `--auto-accept`). Verify after every pass before the next. Do **not** rewrite the whole file in one shot.

**Goal:** Evolve `/home/wdavi/asteroids-game/index.html` into a near-replica of the 1979 Atari standup *Asteroids* cabinet feel, while staying a single-file HTML5 game that is fully playable on mobile (portrait/landscape, coarse pointer, no keyboard required).

**Architecture:** Keep the current Canvas 2D + Web Audio single-file design. No Phaser. No external assets. Authentic vector look, physics, scoring, saucers, audio, and cabinet-style HUD; mobile controls layered on top without changing desktop 5-button fidelity.

**Tech stack:** Single `index.html`, Canvas 2D, Web Audio API, pointer/touch overlay, `localStorage` high score, GitHub Pages at `https://wdavidpence.github.io/asteroids-game/`.

**Live baseline:** `index.html` (~12 KB, ~59 lines minified-style compact script). Published + previously smoke-tested.

**Honest authenticity rating (current build vs original cabinet):** **4 / 10**

| Area | Current | Original 1979 Atari |
|------|---------|---------------------|
| Core loop | Playable rotate/thrust/fire/wrap | Same family |
| Look | Filled neon ship, filled asteroids, star parallax, glow | Pure **vector** white lines on black, no fill, no stars |
| Physics | Soft friction (`.992`), continuous thrust | Near-zero drag, heavy inertia (Spacewar!-derived) |
| Lives | **Shields 8** (absorb hits) | **3–5 lives**; ship explodes; brief respawn invuln |
| Asteroids | 3-ish sizes, scoring ~30/50/100 | Large **20**, med **50**, small **100**; 2 children per split |
| Saucers | One medium-ish, HP=2, mild aim error | **Large** (random fire, 200 pts) + **small** (aims at ship, 1000 pts); after ~40k only small |
| Hyperspace | **Missing** | 5th button; random reappear; ~1/6 self-destruct risk |
| Beat/pulse BGM | Missing | Iconic dual-tone “thump-thump” accelerating with tension |
| Score extras | No extra life rule | Extra life every **10,000** |
| Mobile | 4 on-screen buttons + zone taps | N/A (cabinet) — required for our deliverable |

---

## Non-negotiable constraints

1. **Single file** `index.html` remains the product (GitHub Pages).
2. **Mobile-first playable** every pass: coarse-pointer buttons remain; add hyperspace without crowding thumbs.
3. **Desktop keyboard** maps to classic 5-button set: rotate L/R, thrust, fire, hyperspace (+ pause/restart ok).
4. **No secrets / no large asset packs.** Synthesize audio; draw vectors.
5. **Slice size:** each Ornith task = one pass theme (avoid full-file rewrites; prior Aider timeout risk).
6. **Verify gate** before claiming a pass done:
   - `node --check` on extracted JS (or whole file if pure JS module extract)
   - local HTTP serve + runtime smoke (or Node vm stubs)
   - quick manual checklist for that pass’s acceptance criteria
7. Publish only after Grok review + Pages smoke.

---

## Target end-state (what “almost indistinguishable” means here)

### Cabinet fidelity (must-have for 8.5–9/10)
- Black field, **stroke-only** white (or near-white) vectors; optional subtle phosphor bloom, no filled polygons as primary look.
- Ship: classic wedge outline + rear thruster flame only while thrusting.
- Asteroids: irregular closed polylines, three sizes (L/M/S), correct split counts and speeds.
- Wrap-around for ship, bullets, rocks, saucers (no bounce walls).
- Near-frictionless drift; rotate-in-place; thrust along nose; bullets inherit little/no ship velocity feel (or match original short-life shots).
- **Hyperspace** with delay + fail chance.
- Large + small saucers with authentic scoring and aim behavior curve vs score.
- Scoring: 20 / 50 / 100 rocks; 200 large saucer; 1000 small saucer.
- Lives (not shields); extra ship every 10k; game over on last life lost.
- Audio: fire, thrust loop, bangs by size, saucer tones, **heartbeat beat** speeding up.
- Wave spawn: more large rocks over waves; difficulty plateaus late-game in original spirit.

### Mobile playability (must-have)
- Always-visible translucent controls on coarse pointers:
  - Left cluster: rotate L / rotate R  
  - Right cluster: thrust / fire  
  - Hyperspace: smaller secondary (e.g. top-right or dual-tap thrust hold → separate button)
- No reliance on keyboard for a complete run on phone.
- Safe viewport: `viewport-fit=cover`, no iOS rubber-band scroll, `touch-action: none`.
- Readable HUD at phone widths; optional “cabinet bezel” frame in landscape without blocking controls.
- Performance: stable 60fps on mid phones (cap DPR at 2, limit particles).

### Explicit non-goals (for this plan)
- Cycle-accurate 6502 / Vector Generator hardware emulation.
- Exact ROM samples (we approximate with Web Audio).
- Multiplayer cocktail cabinet mode.
- Account/cloud leaderboards (local high score is enough).

---

## Strengths to preserve (do not regress)

- ✅ Working single-file loop (`rAF`, resize, wrap helpers)
- ✅ Touch buttons + coarse-pointer CSS already present
- ✅ Basic saucer + split asteroids + particles + high score
- ✅ AudioContext resume-on-gesture pattern
- ✅ Published Pages path under `wdavidpence/asteroids-game`

---

## Multi-pass plan (implementation order)

Each pass is independently shippable and re-playable on mobile.

### Pass 0 — Baseline freeze & measurement `[x]`
**Objective:** Snapshot behavior and add a tiny debug/metrics hook for later A/B feel checks.  
**Files:** `index.html`; optional `docs/plans/pass-log.md`  
**Work:**
- Record current constants: rot, thrust, friction, bullet speed/life/cd, scores, lives model.
- Add non-intrusive `?debug=1` overlay: FPS, asteroid count, saucer state (hidden in normal play).
**Acceptance:** Game still loads on Pages; debug only with query flag.  
**Owner:** Ornith (tiny edit) → Grok verify.

### Pass 1 — Vector presentation (look) `[x]`
**Objective:** Replace filled/glow “neon arcade” with cabinet vector look.  
**Work:**
- Background pure black (optional very faint phosphor vignette only).
- Ship, asteroids, saucers, bullets as **strokes**; remove solid fills (or keep 0-alpha fill).
- Kill starfield parallax (not original) or gate behind `CFG.fxStars=false` default off.
- HUD: simple vector-ish monospace score top-left; lives as mini-ships (not “SHIELDS N”).
**Acceptance:** Still playable; screenshot/compare checklist: black + white lines.  
**Owner:** Ornith → Grok visual review.

### Pass 2 — Authentic ship physics & controls `[x]`
**Objective:** Match standup control feel.  
**Work:**
- Friction ≈ 1.0 (or 0.999+ only for numerical safety); no soft “ice with brakes.”
- Rotation rate tuned to original “crisp hold-to-spin.”
- Thrust acceleration along nose; visible flame only while thrust held.
- Bullet: short life, limited simultaneous shots (~4 like original), fire from nose, fixed shot speed (minimal inherited velocity).
- Keyboard: Left/Right/Up or W/Space + **Hyperspace key** (Shift or H) stub if hyperspace comes next pass.
**Acceptance:** Ship coasts forever after thrust; stopping requires reverse thrust feel.  
**Owner:** Ornith.

### Pass 3 — Lives, death, respawn (remove shields) `[x]`
**Objective:** Original life model.  
**Work:**
- Replace `shields:8` with `lives:3` (config 3–5).
- On rock/saucer/bullet hit: explode ship (line debris), lose life, pause briefly, respawn center with invulnerability blink.
- Extra life every 10_000 points (ding sound).
- Game over only when lives exhausted mid-explosion sequence completes.
**Acceptance:** No shield ring; three ships in HUD; death is lethal.  
**Owner:** Ornith.

### Pass 4 — Asteroid sizes, split rules, scoring `[x]`
**Objective:** Classic rock economy.  
**Work:**
- Three radii (e.g. 40 / 20 / 10 scaled to canvas min-dimension).
- Large → 2 medium; medium → 2 small; small → gone.
- Scores: 20 / 50 / 100.
- Initial wave count scales with level (start ~4 large; increase toward a cap).
- Child rocks get higher speed; slight random direction change on split.
**Acceptance:** Clearing a large yields correct cascade and points.  
**Owner:** Ornith.

### Pass 5 — Hyperspace `[x]`
**Objective:** Fifth cabinet control.  
**Work:**
- Button + key; ship vanishes N ms, reappears random safe-ish position.
- ~1/6 chance of re-entry explosion (configurable).
- Mobile: dedicated small control (label HS / ↔) not colliding with fire/thrust.
**Acceptance:** Usable under panic on phone; risk is real.  
**Owner:** Ornith.

### Pass 6 — Dual saucers + AI/scoring curve `[ ]`
**Objective:** Authentic UFOs.  
**Work:**
- Large saucer: slower, dumb random shots, 200 pts.
- Small saucer: faster, aims at ship (accuracy tightens with score), 1000 pts.
- After score ≥ 40_000, only small saucers.
- Spawn timing: periodic while rocks remain; “lurking” possible if one rock left (original meta — do not hard-ban).
- Saucer bullets kill player; player bullets destroy saucer in one hit.
**Acceptance:** Distinct threat profiles; correct points.  
**Owner:** Ornith (logic-heavy — keep patch local).

### Pass 7 — Authentic audio suite `[ ]`
**Objective:** Cabinet sound identity without ROM dumps.  
**Work:**
- Thrust: looping low saw/noise while thrust key down.
- Fire: short high blip.
- Bangs: three decay lengths for L/M/S rocks + ship death.
- Saucer: continuous two-tone warble differing large vs small.
- **Beat:** alternating low thumps; interval decreases as remaining rock “danger” or level rises.
- Extra-life jingle; silence-safe resume on first gesture.
**Acceptance:** Mute-able via optional `M` key later; no audio exceptions crash game.  
**Owner:** Ornith.

### Pass 8 — Wave / difficulty pacing `[ ]`
**Objective:** Original pressure curve.  
**Work:**
- Next wave only when all rocks **and** saucers cleared (or match original: rocks clear, saucer rules separate — document chosen rule in pass-log).
- Increase large-rock count per wave with soft cap.
- Optional small saucer earlier at high scores.
**Acceptance:** Early waves teach; late waves stressful but fair.  
**Owner:** Ornith.

### Pass 9 — Mobile control redesign (cabinet-on-glass) `[ ]`
**Objective:** Phone play without losing 5-button mapping.  
**Layout proposal:**
```
[ L ] [ R ]              [ HS ]
                         [ THRUST ]
                         [ FIRE ]
```
- Larger hit targets (≥44px, prefer 64–76px).
- Haptic optional (`navigator.vibrate` short) on fire/death if available.
- Zone-tap fallback only if buttons fail; prefer explicit buttons for precision rotate.
- Landscape: controls in lower corners; HUD top safe-area insets.
**Acceptance:** Full clear of wave 1–2 on phone using only touch.  
**Owner:** Ornith → Grok device-check if available.

### Pass 10 — Cabinet presentation polish `[ ]`
**Objective:** Standup “feel” without breaking mobile.  
**Work:**
- Optional letterboxed playfield with max aspect (simulate monitor) on desktop wide screens.
- Attract mode: ship autopilot / demo rocks when idle on title.
- Insert-coin style “PRESS FIRE” title; high score initials optional (3 letters) if time.
- CRT-ish slight bloom via shadowBlur low + thin lines (keep tasteful).
**Acceptance:** Title → play → game over → restart loop feels arcade.  
**Owner:** Ornith.

### Pass 11 — Hardening, a11y, perf, docs `[ ]`
**Objective:** Ship quality.  
**Work:**
- Cap particles; pool bullets/rocks if needed.
- Pause when `document.hidden`.
- Reduce motion respect optional.
- Update `package.json` description (Canvas not Phaser).
- Update `docs/debug-polish-plan.md` status or supersede with this plan.
- Pass-log + handoff file for multi-session work.
**Acceptance:** 10-minute soak without leak/crash; Pages 200 + smoke.  
**Owner:** Ornith + Grok publish judgment.

### Pass 12 — Publish & verification gate `[ ]`
**Objective:** Ship to `wdavidpence.github.io/asteroids-game/`.  
**Work:**
- Commit per logical pass groups (not one mega-commit if avoidable).
- Push `master` → `origin/main`.
- Re-enable/verify GitHub Pages.
- Runtime smoke against live HTML; list remaining known deltas vs cabinet.
**Acceptance:** Live URL playable on mobile Safari/Chrome; rating self-score ≥ **8.5/10** on checklist below.  
**Owner:** Grok orchestrates; Ornith only if code fix needed.

---

## Authenticity checklist (endgame scorecard)

- [ ] Vector-only primary art
- [ ] Near-frictionless inertia
- [ ] 5 controls including hyperspace risk
- [ ] Lives + extra every 10k (no shield pool)
- [ ] Rock scores 20/50/100 + correct splits
- [ ] Large + small saucers, 200 / 1000, aim curve
- [ ] Beat + thrust + bangs + saucer tones
- [ ] Wrap all entities
- [ ] Mobile complete runs without keyboard
- [ ] Attract/title arcade loop

**Scoring guide:**  
3–4 = generic clone (today) → 6 = physics+lives+rocks → 8 = saucers+hyperspace+audio → 9 = mobile cabinet polish + pacing.

---

## Orchestration playbook (this repo)

| Role | Responsibility |
|------|----------------|
| **Grok (this session)** | Analysis, plan, pass ordering, acceptance review, rescue on Ornith stall, publish |
| **Ornith via Aider** | Mechanical edits to `index.html` **one pass at a time** |
| **Human** | Playtest feel on real phone; veto control layouts |

**Ornith prompt pattern (copy per pass):**
```text
Project: /home/wdavi/asteroids-game
Edit only index.html for Pass N from docs/plans/2026-07-16-arcade-replica-mobile-plan.md.
Do not reformat the whole file. Keep single-file Canvas 2D.
Preserve mobile .btn controls unless the pass explicitly redesigns them.
After edits: ensure no syntax errors.
Summarize diff and remaining risks.
```

**If Aider times out:** split pass into 2 sub-edits (e.g. Pass 6a large saucer only, 6b small saucer aim).

---

## Suggested first execution batch (after plan approval)

1. Pass 0 + Pass 1 (look) — high visual payoff, low risk  
2. Pass 2 + Pass 3 (physics + lives) — biggest “feels like Asteroids” jump  
3. Pass 4 + Pass 5 (rocks + hyperspace)  
4. Pass 6 + Pass 7 (saucers + audio)  
5. Pass 9 early enough that mobile never breaks — **re-test mobile after every batch**  
6. Pass 8, 10, 11, 12 to finish

---

## Risks & mitigations

| Risk | Mitigation |
|------|------------|
| Ornith full-file rewrite / timeout | Tiny pass prompts; refuse drive-by refactors |
| Mobile controls crowded after hyperspace | Pass 9 layout mock first; thumb-zone test |
| Frictionless physics feels “too hard” on phone | Slightly lower rock max speed on coarse pointer only |
| Audio autoplay policies | Resume on first pointer/key; never throw |
| Authenticity vs fun | Prefer original rules; only soft-cap difficulty for mobile FPS |

---

## Session handoff

- **Plan file:** `docs/plans/2026-07-16-arcade-replica-mobile-plan.md` (this file)
- **Code:** `/home/wdavi/asteroids-game/index.html`
- **Repo/Pages:** `https://github.com/wdavidpence/asteroids-game` · `https://wdavidpence.github.io/asteroids-game/`
- **Next action when approved:** Start Pass 0+1 with Ornith; Grok verifies before Pass 2.

---

## Status

- [x] Analyze current build vs original standup arcade  
- [x] Write multi-pass plan (this document)  
- [ ] User approval / adjust priorities  
- [ ] Execute passes with Ornith under Grok orchestration  
