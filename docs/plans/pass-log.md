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
