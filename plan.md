# SKI & CYCLE GAME — Project Plan

## Status

| Item            | Detail                                              |
|-----------------|-----------------------------------------------------|
| Implementation  | Complete — all 14 milestones shipped                |
| File            | `index.html` — 47 KB, 1 364 lines, zero dependencies |
| Repository      | https://github.com/miguelmalvarez/ski-game          |
| Deployment      | Vercel (connect repo at vercel.com → New Project)   |

---

## Overview

A retro 2D browser-based game with two selectable modes — **Winter** (skiing) and **Summer**
(cycling). On launch the player picks a theme from a mode-select screen; the core loop,
scoring, and ambulance crash scene are shared across both modes.

The player controls either a skier on a snow slope (Winter) or a cyclist on a summer road
(Summer), dodging obstacles that increase in density over time. Speed ramps up continuously.
A crash triggers a comedic ambulance scene. The game ships as a single self-contained HTML
file — runnable offline or hosted at a URL.

---

## Game Modes

### Winter Mode
The player is a skier racing down a snow-covered mountain slope.
- Background: scrolling snow bands (3 white/blue tones), pine trees on margins, ski tracks
- Obstacles: NPC skiers (up to 12) with lateral drift AI
- Jump objects: ski ramps (white wedge, 32×16 px) → airborne state, +500 pts
- Decorations: snow-covered pine trees, red/blue slalom flags
- Player: red jacket (#CC2222), blue trousers (#2244AA), white skis, brown poles

### Summer Mode
The player is a cyclist riding down a country road.
- Background: asphalt road (centre 60%), green grass verges (outer 20% each side),
  scrolling yellow centre-line dashes, bright blue sky
- Obstacles: animals crossing the road (up to 12 active) — see table below
- Static obstacles: rocks (small and large) — up to 6 active, no lateral movement
- Jump objects: speed bumps (yellow/black striped hump, 32×14 px) → airborne state, +500 pts
- Decorations: leafy summer trees and bushes on grass verges
- Player: orange jersey (#FF6600), dark shorts (#222244), animated bicycle with spinning wheels

#### Animal obstacle types

| Animal   | Speed factor | Lateral behaviour                              |
|----------|--------------|------------------------------------------------|
| Cat      | 0.9–1.1×     | Darts across; pauses 0.3–0.8 s mid-road       |
| Dog      | 0.7–1.0×     | Steady cross with slight wobble                |
| Squirrel | 1.2–1.5×     | Erratic — reverses lateral direction 2–4× per crossing |
| Turtle   | 0.3–0.5×     | Slow and steady; stays on road                 |

#### Rock variants (Summer only)

| Variant | Size      | Visual                                         |
|---------|-----------|------------------------------------------------|
| Small   | 16×12 px  | Single rounded grey boulder with shadow        |
| Large   | 28×18 px  | Two overlapping boulders with shadow ellipse   |

Rocks spawn ~every 6–10 s; a minimum X-gap is enforced between rocks to prevent walls.
Rocks do **not** trigger a jump — collision only.

#### Cyclist sprite states

| State   | Description                                              |
|---------|----------------------------------------------------------|
| forward | Upright; pedalling legs animate (sin wave offset)        |
| left    | Lean left; handlebar turned                              |
| right   | Lean right; handlebar turned                             |
| air     | Airborne over speed bump; sparkles emitted               |
| crash   | Rider and bike separate and splay out                    |

---

## Mode Selection Screen

Shown immediately after the title splash, before any gameplay:

```
┌────────────────────────────────────────┐
│           CHOOSE YOUR MODE             │
│                                        │
│  ┌──────────────┐  ┌──────────────┐   │
│  │   WINTER     │  │   SUMMER     │   │
│  │  [live ski   │  │  [live cycle │   │
│  │   preview]   │  │   preview]   │   │
│  │ Snow slope   │  │ Country road │   │
│  │ dodge skiers │  │ dodge animals│   │
│  └──────────────┘  └──────────────┘   │
│                                        │
│  Key 1/W = Winter  ·  Key 2/S = Summer │
└────────────────────────────────────────┘
```

- Each panel renders a live clipped scene (actual sprites, not static images)
- Hover glows the panel border; click/tap selects
- Keyboard: `1` or `W` = Winter, `2` or `S` = Summer
- High scores stored separately per mode:
  - Winter: `localStorage` key `ski_highscore`
  - Summer: `localStorage` key `cycle_highscore`

---

## Tech Stack

| Layer       | Choice                      | Reason                                           |
|-------------|-----------------------------|--------------------------------------------------|
| Runtime     | Browser (HTML5)             | Zero install, works on Windows/Mac/Linux/iOS     |
| Rendering   | HTML5 Canvas (2D API)       | Full pixel-level control, no dependencies        |
| Language    | Vanilla JavaScript (ES2020) | No build step, single file                       |
| Art style   | Pixel art via Canvas rects  | No image files needed; fully self-contained      |
| Audio       | Web Audio API (procedural)  | No audio files needed; mode-specific sounds      |
| Persistence | localStorage                | High score survives browser restarts, no backend |
| Packaging   | Single `.html` file         | Shareable by link or attachment                  |

No npm, no bundler, no server required. The file runs offline.

---

## Sprites (all drawn programmatically)

### Winter sprites

| Asset              | States / variants                                | Approx size   |
|--------------------|--------------------------------------------------|---------------|
| Player skier       | down, left, right, air, crash                    | 24×32 px      |
| NPC skier          | down, left, right (green jacket)                 | 24×32 px      |
| Ski ramp           | Static wedge                                     | 32×16 px      |
| Snow pine tree     | Single variant, size parameterised               | 16–44×24–44px |
| Slalom flag        | Red and blue variants                            | 8×24 px       |

### Summer sprites

| Asset              | States / variants                                | Approx size   |
|--------------------|--------------------------------------------------|---------------|
| Player cyclist     | forward, left, right, air, crash                 | 26×34 px      |
| Cat                | Facing left/right (flipped via ctx.scale)        | 18×18 px      |
| Dog                | Facing left/right                                | 22×22 px      |
| Squirrel           | Facing left/right                                | 12×20 px      |
| Turtle             | Facing left/right                                | 28×14 px      |
| Speed bump         | Striped ellipse hump                             | 32×14 px      |
| Summer leafy tree  | Size parameterised                               | 18–44×24–44px |
| Bush               | Size parameterised                               | 8–24px wide   |
| Rock (small)       | Single boulder + shadow                          | 16×12 px      |
| Rock (large)       | Two boulders + shadow                            | 28×18 px      |

### Shared sprites

| Asset       | Description                                          |
|-------------|------------------------------------------------------|
| Ambulance   | White body, red cross, flashing siren, rolling wheels |
| Hospital    | Building with red cross, windows                     |

### Color palettes

**Winter**
- Snow bands: #F0F4FF, #E8EEFF, #F8FBFF
- Player: jacket #CC2222, trousers #2244AA, skis #EEEEEE
- NPC: jacket #22AA44
- Trees: canopy #1A6E1A–#2DAA2D, trunk #8B4513
- Flags: #CC2222 (red), #2244BB (blue)

**Summer**
- Road: #555566, centre dashes #FFD700
- Grass verge: #5AB552 / #4FA847
- Sky: #87CEEB
- Cyclist: jersey #FF6600, shorts #222244, helmet #FF3300
- Cat: #888888, Dog: #CC8844, Squirrel: #BB6622, Turtle: #447744 / #336633
- Rocks: #888888 / #777777

**Shared**
- Ambulance: body #FFFFFF, cross #CC2222, siren #FF2222 / #880000

---

## Game Mechanics

### State machine

```
TITLE → MODE_SELECT → PLAYING ⇄ PAUSED
                          ↓
                      AMBULANCE → GAMEOVER → MODE_SELECT
```

### Core loop

1. World scrolls downward continuously at `gameSpeed` px/frame.
2. Player steers left/right to avoid obstacles and aim for jump objects.
3. `gameSpeed` starts at 3, increases by 0.15 every 10 seconds of play.
4. Score accumulates as `gameSpeed × dt × 10` per frame.
5. Crash → 0.5 s freeze → ambulance scene (~3.5 s) → game over screen.

### Player controls

| Input          | Action                                  |
|----------------|-----------------------------------------|
| ← / A          | Move and lean left                      |
| → / D          | Move and lean right                     |
| (no key)       | Lean returns to centre over 200 ms      |
| 1 / W          | Select Winter on mode-select screen     |
| 2 / S          | Select Summer on mode-select screen     |
| Escape         | Pause / unpause                         |
| Mobile buttons | On-screen ◀ ▶ buttons below canvas     |

### Collision

- AABB at 80% of sprite dimensions for both player and obstacles.
- Airborne player has no collision with moving obstacles.
- Rock collision is active even while airborne (rocks cannot be jumped).

### Scoring

| Event                            | Points              |
|----------------------------------|---------------------|
| Surviving                        | gameSpeed × dt × 10 |
| Jump (ramp or speed bump)        | +500                |
| Style bonus (obstacle under arc) | +200 per obstacle   |
| New high score                   | "NEW RECORD!" banner with bounce animation |

### Difficulty curve

| Time elapsed | gameSpeed  | Obstacle spawn interval |
|--------------|------------|------------------------|
| 0 s          | 3.0        | 3.0 s                  |
| 10 s         | 3.15       | ~2.8 s                 |
| 60 s         | 3.9        | ~2.0 s                 |
| 120 s        | 4.8        | ~1.5 s                 |
| 300 s+       | 7.5+       | 1.2 s (floor)          |

---

## Screen / UI Layout

```
┌──────────────────────────────────────────┐
│  SCORE: 00000  ❄ WINTER  BEST: 00000    │  ← HUD (36px, semi-transparent)
│                                          │
│         [scrolling world]                │
│                                          │
│         player sprite (y = 65%)          │
│                                          │
└──────────────────────────────────────────┘
         [◀ LEFT]        [RIGHT ▶]          ← mobile touch buttons
```

Canvas: **480×640 px** logical (3:4 portrait). CSS-scaled to fill window, `devicePixelRatio`
applied for sharp rendering on retina screens.

---

## Audio (Web Audio API, procedural — no files)

| Event           | Winter                        | Summer                        |
|-----------------|-------------------------------|-------------------------------|
| Jump            | Sine sweep 200→600 Hz, 0.3 s  | Bike bell: two short pings    |
| Land            | Sine sweep 500→300 Hz, 0.2 s  | Higher ping 700→500 Hz        |
| Crash           | Sawtooth 80 Hz, 0.6 s         | Sawtooth 120 Hz, 0.4 s        |
| Ambulance siren | Square 800/600 Hz alternating, 3 bursts (shared) |

---

## File Structure

```
ski_game/
├── plan.md       ← this file
├── index.html    ← entire game (HTML + CSS + JS, 47 KB)
└── vercel.json   ← { "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

---

## Deployment

### GitHub repository
`https://github.com/miguelmalvarez/ski-game`

Two commits on `main`:
1. `Add project plan and Vercel config` — plan.nd, vercel.json
2. `feat: implement full game — Winter ski + Summer cycle modes (M1-M14)` — index.html

### Vercel (chosen hosting)
Connect the GitHub repo at **vercel.com → New Project → Import `ski-game`**.
The `vercel.json` is already present; no additional config needed.
Every push to `main` auto-redeploys.

### Alternative: direct file share
Send `index.html` — recipient double-clicks, plays immediately. No server needed.

---

## Milestones

### M1 — Canvas engine & scrolling background ✅
- 480×640 canvas with `devicePixelRatio` scaling, CSS aspect-ratio preservation
- Alternating snow bands with parallax ski tracks (Winter)
- `requestAnimationFrame` loop with 50 ms delta cap; `update(dt)` / `render()` split

### M2 — Player sprite & movement ✅
- `drawSkier(cx, cy, state, jacket)` — ~25 filled rectangles per frame
- Left/right movement ±4 px/frame; 200 ms lean return timer
- 20 px edge margin; mobile touch buttons

### M3 — Slope decoration ✅
- Snow pine trees on left/right 17% margins (parallax 0.6×)
- Red/blue slalom flags in centre corridor (1.0×)
- Recycling pool — items reappear at top when scrolled off bottom

### M4 — NPC skiers ✅
- `drawNPCSkier` — same as skier with green jacket
- Pool up to 12; speed 0.8–1.3× gameSpeed; drift ±1.5 px/frame flipping every 1–3 s
- Spawn interval `max(1.2, 3 - elapsed/60)` seconds

### M5 — Jumps ✅
- Ski ramp wedge sprite; AABB overlap triggers 0.9 s arc (sin wave, 30 px height)
- No moving-obstacle collision while airborne
- +500 pts on land; +200 per obstacle under arc path; animated popup text

### M6 — Collision & ambulance scene ✅
- AABB at 80% sprite size; checked every frame during PLAYING
- 0.5 s freeze on hit, then AMBULANCE state machine:
  - t=0.0 — bluish tint fades in
  - t=0.5 — ambulance enters from right
  - t=0.8 — siren plays (3 bursts)
  - t=2.5 — hospital icon appears
  - t=3.5 — transition to GAMEOVER

### M7 — Scoring & persistence ✅
- Score displayed live in HUD; localStorage save on game over
- "NEW RECORD!" banner with vertical bounce animation
- Separate keys `ski_highscore` / `cycle_highscore`

### M8 — Polish (Winter) ✅
- Web Audio procedural sounds (jump whoosh, crash thud, ambulance siren)
- Title screen with animated snowflakes and blinking "PRESS ANY KEY" prompt
- ESC pause/unpause; responsive canvas resize handler

### M9 — Mode selection screen ✅
- `MODE_SELECT` state between TITLE and PLAYING
- Two 190×300 px rounded panels with live clipped sprite previews
- Mouse hover glow; click / touch / keyboard to select
- `gameMode` variable gates all mode-specific logic

### M10 — Summer background & decorations ✅
- Road: #555566 centre 60%, grass verges #5AB552 outer 20% each side
- Scrolling yellow centre-line dashes (lineDashOffset animation)
- Grass stripe texture (0.4× parallax); road patch texture (0.9×)
- Leafy summer trees + bushes on verges (parallax 0.6×)

### M11 — Cyclist sprite ✅
- `drawCyclist(cx, cy, state)` — bike frame, two wheels with animated spokes,
  handlebars, pedalling legs (sin-wave offset), helmet, jersey
- All 5 states: forward, left, right, air (sparkles), crash (rider + bike separate)
- Player clamped to road width (W×0.22 – W×0.78) in Summer mode

### M12 — Animal obstacles ✅
- `drawAnimal(cx, cy, type, facingLeft)` — ctx.scale(-1,1) flip for direction
- Cat: `pauseTimer` + `pauseX` — drifts to pauseX then holds for 0.3–0.8 s
- Squirrel: `reversals` counter (2–4) with short drift timer on each reversal
- Turtle: speed factor 0.3–0.5×; visually distinct shell dome
- Speed bump replaces ski ramp; same +500 / style bonus mechanic

### M13 — Static rock obstacles ✅
- `drawRock(x, y, large)` — quadratic-curve boulders with shadow ellipse
- Pool of 6; spawn every 6–10 s; min 40 px X-gap between active rocks
- AABB collision active even mid-air (rocks cannot be jumped)

### M14 — Summer sounds & HUD polish ✅
- Bike bell: two-tone sine ping (1800 Hz + 1600 Hz, 90 ms apart)
- Summer crash: sawtooth 120 Hz, shorter than Winter version
- HUD centre label: "❄ WINTER" (#AADDFF) or "☀ SUMMER" (#FFDD88)
- Game over and ambulance scenes use correct mode background
- Title screen shows skier + cyclist side by side

---

## Definition of Done

**Shared / infrastructure**
- [x] Game runs by opening `index.html` in Chrome / Firefox / Edge / Safari
- [x] Mode selection screen appears before gameplay; both modes selectable by click or key
- [x] Player can steer left/right; speed increases over time in both modes
- [x] Jump objects award +500 pts and trigger airborne state in both modes
- [x] Crash triggers ambulance scene (same scene for both modes)
- [x] High score persists across sessions, separately per mode
- [x] Game scales correctly on phone and desktop
- [ ] Accessible via a live Vercel URL (pending: connect repo at vercel.com)

**Winter mode**
- [x] Snow slope background scrolls with parallax pine trees and flags
- [x] NPC skiers spawn, drift, and can be crashed into
- [x] Ski ramp triggers jump; style bonus when NPC is under arc

**Summer mode**
- [x] Road background with asphalt, grass verges, leafy trees scrolls correctly
- [x] Cyclist sprite renders in all 5 states (forward, left, right, air, crash)
- [x] Cat, dog, squirrel, and turtle obstacles appear with correct behaviours
  - [x] Cat pauses mid-road
  - [x] Squirrel reverses direction mid-crossing
  - [x] Turtle is noticeably slower than other animals
- [x] Speed bump triggers jump; style bonus when animal is under arc
- [x] Small and large rocks appear as static obstacles; collision ends run
- [x] Bike bell sound on jump; distinct crash sound

---

## Open Questions

- [ ] Should there be a difficulty selector (Easy / Normal / Hard)?
- [ ] Should Summer rocks also cluster in rare "rock slide" events (group of 3)?
- [ ] Should the squirrel's small hitbox allow jumping over it for a style bonus?
- [ ] Should mode selection be remembered across sessions (localStorage)?
- [ ] Should trees on the margins act as obstacles, or remain decorative?
- [ ] Time-of-day visual effect after 3+ minutes (sunset palette shift)?
- [ ] DeviceOrientation API for phone-tilt steering on mobile?
