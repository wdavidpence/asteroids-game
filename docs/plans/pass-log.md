# Asteroids Pass Log

## Baseline freeze (pre Pass 0) — 2026-07-16

**File:** `/home/wdavi/asteroids-game/index.html`  
**Size:** 12158 bytes, 60 lines  
**Engine:** Canvas 2D single-file (not Phaser)

### CFG constants
```
rot: 4.4
thrust: 360
friction: 0.992
bulletSpeed: 650
bulletLife: 1.15
bulletCd: 0.16
shipR: 16
invuln: 1.8
maxBullets: 8
```

### Other gameplay notes (pre-pass)
- Lives model: **shields starting at 8** (not classic lives)
- Stars: 140 parallax particles
- Asteroid score tiers: r>32 → 30, r>20 → 50, else 100
- Split: if r>22 → two children at r*0.56
- Saucer: single type, hp=2, score 75 hit + 250 kill; spawn on even levels / random
- Touch: 4 buttons (L/R/thrust/fire), no hyperspace
- High score: `localStorage.asteroidsHighScore`

### Pass status
| Pass | Status | Notes |
|------|--------|-------|
| 0 | in progress | debug overlay |
| 1 | pending | vector look |
| 2–12 | pending | see master plan |

---

## Pass 0+1 complete — 2026-07-17 02:29 UTC

**Rescue path:** Ornith/Aider stalled ~9min on whole-file edit (connected, thinking, no write). Grok applied surgical Pass 0+1.

### Pass 0
- Baseline constants frozen in pass-log
- `?debug=1` enables FPS + entity counts overlay
- Runtime smoke: frames=70, listeners=5

### Pass 1
- Pure black background (CSS + canvas fill)
- `CFG.fxStars=false` (starfield disabled by default)
- Ship/asteroids/saucers stroke-only vector look
- Reduced glow (`vectorGlow:2`); bullets small points
- Monospace HUD; invuln ring only (no permanent shield halo)
- Gameplay/physics/shields unchanged for later passes

### Verify
- `node --check` on extracted JS: OK
- Node rAF smoke: OK
- Local HTTP 8765 serve: OK

### Next
- Pass 2 physics + Pass 3 lives

## Pass 2+3 complete — 2026-07-17 02:33 UTC

**Ornith:** skipped/rescue — model returns empty `content` (only `reasoning_content`); Aider whole-edit cannot finish.

### Pass 2 — physics / shots
- `friction: 1.0` (no drag)
- `thrust: 280`, soft `maxSpeed: 520`
- Bullets: fixed muzzle velocity (no ship-velocity inherit), `maxBullets:4`, life 1.05, cd 0.18

### Pass 3 — lives
- `startLives:3`, remove shields
- Hit → explode debris → lose life → `respawnDelay:2.0` center + invuln
- Extra life every 10_000 via `addScore`
- HUD mini-ship icons for remaining lives
- Game over on last life lost

### Verify
- node --check OK
- rAF smoke frames=70 OK

## Pass 4+5 complete — 2026-07-17 02:40 UTC

Ornith Aider: exited without edits (read-only answer). Grok surgical patches.

### Pass 4
- Discrete L/M/S: r 42/22/11, scores 20/50/100
- Spawn only L; L→2M, M→2S; speed tiers
- Wave count min(4+level,11)

### Pass 5
- Hyperspace: H/Shift + mobile HS button
- Vanish 0.35s, reappear random; ~1/6 fail → life loss
- Cooldown 0.85s; no collide/draw while hyper

### Verify
- node --check OK; rAF smoke OK

## Pass 6+7 complete — 2026-07-17 02:43 UTC

Grok surgical implementation (Ornith skipped for multi-system pass).

### Pass 6 saucers
- Large: slow, random fire, 200 pts, bigger silhouette
- Small: faster, aims at ship (accuracy rises with score), 1000 pts
- Only small after score ≥ 40_000
- Timer spawn 8–16s while rocks remain (lurking allowed)
- One-hit kill; wave clear waits for rocks AND saucers

### Pass 7 audio
- Thrust loop while thrusting
- Size-based bangs (L/M/S/ship)
- Saucer dual-tone warble L vs S
- Heartbeat beat speeds with danger
- M mute; hyperspace whoosh

### Verify
- node --check + rAF smoke OK

## Pass 8+9 complete — 2026-07-17 02:49 UTC

### Pass 8 pacing
- Wave rock count 4→11 via waveRockCount()
- rockSpeedMult soft-caps at 1.55
- WAVE CLEAR delay 0.85s before next level
- Saucer interval shortens with score/level, floor 5.5s
- Mobile: slower rocks (0.9×) + lower max ship speed (460)

### Pass 9 mobile
- L/R left cluster; thrust/fire right; HS above fire
- safe-area insets; landscape smaller corner layout
- Coarse: zone-tap disabled (buttons only)
- Active button feedback; no context menu on buttons

### Verify
- syntax + rAF smoke OK
