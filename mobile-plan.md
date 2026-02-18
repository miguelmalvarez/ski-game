# Mobile Plan — Ski & Cycle Game

## Status: Planning complete, implementation pending

---

## 1. Diagnosis — What's Wrong on Mobile

The game runs on mobile but has a collection of real problems discovered through code analysis:

### 1.1 Critical Bugs

| # | Problem | File:Lines | Root Cause |
|---|---------|------------|-----------|
| C1 | Game is vertically clipped on iOS Safari | `index.html:14-15` | `height: 100vh` includes the Safari address bar, cutting off the bottom of the canvas |
| C2 | Stuck movement after notifications / app-switch | `index.html:124-127` | `touchcancel` event not handled; if the OS interrupts a touch (notification, call), `keys.left/right` stays `true` forever |
| C3 | No way to pause on mobile | — | Pause only mapped to `Escape` key (keyboard only); no touch alternative |
| C4 | Audio silenced on iOS | `index.html:154-158` | iOS requires `audioCtx.resume()` to be explicitly called after a user gesture; lazy init alone is not enough |

### 1.2 Layout & Safe-Area Issues

| # | Problem | File:Lines | Root Cause |
|---|---------|------------|-----------|
| L1 | Controls hidden behind iPhone home indicator / Android gesture bar | `index.html:33` | `bottom: 20px` hard-coded; doesn't use `env(safe-area-inset-bottom)` |
| L2 | Content hidden behind notch / Dynamic Island on some devices | `index.html:5` | `viewport` meta doesn't include `viewport-fit=cover` needed to extend into safe areas |
| L3 | Control buttons too small on large phones | `index.html:42-43` | Fixed `70×70 px` regardless of physical screen size |
| L4 | Controls visible on title and game-over screens | `index.html:62-65` | `#controls` is always rendered; movement buttons are confusing when there's nothing to move |

### 1.3 Performance Issues

| # | Problem | File:Lines | Root Cause |
|---|---------|------------|-----------|
| P1 | Unnecessary GPU load on 3× screens | `index.html:84` | `devicePixelRatio` used raw (can be 3× on modern iPhones); pixel-art rendering at 3× doesn't improve quality, only wastes GPU |

### 1.4 UX & Platform Gaps

| # | Problem | Root Cause |
|---|---------|-----------|
| U1 | No landscape orientation handling | Portrait-only canvas, no message or support for landscape |
| U2 | Game can't be added to home screen as PWA | Missing `apple-mobile-web-app-capable`, status bar meta, `<link rel="manifest">`, and touch icon |
| U3 | No haptic feedback on crash events | Vibration API not used |
| U4 | Double-tap to zoom still possible in some browsers | Only blocked via `user-scalable=no`; CSS `touch-action` approach is more robust |

---

## 2. Design Decisions

### Decision A — Viewport height strategy
**Use `100dvh` (dynamic viewport height) with `100vh` fallback.**
`dvh` is the viewport height excluding the browser UI (address bar, navigation). Supported on iOS 16+ and Android Chrome 108+. Fallback to `100vh` for older devices.

```css
height: 100vh;          /* fallback */
height: 100dvh;         /* modern browsers */
```

### Decision B — Safe area insets
**Extend viewport to cover the full screen and use `env()` insets.**
Add `viewport-fit=cover` to the viewport meta tag, then push content inward with `env(safe-area-inset-*)`. This correctly handles notch, Dynamic Island, and bottom home indicator on all Apple and Android devices.

### Decision C — Control button sizing
**Use `clamp()` to size buttons relative to viewport width.**
Buttons should be at least 64px (touch target recommendation) and scale up to ~90px on large phones. Formula: `clamp(64px, 14vw, 90px)`.

### Decision D — DevicePixelRatio cap
**Cap DPR at 2.**
Pixel art games don't benefit from 3× rendering. Cap it: `const dpr = Math.min(window.devicePixelRatio || 1, 2)`. This halves the GPU memory footprint on iPhone Pro devices.

### Decision E — Pause button placement
**Add a floating pause button (⏸) in the top-right corner of the canvas wrapper.**
Positioned inside `#wrapper` so it scales with the canvas. Only visible during `PLAYING` and `PAUSED` states. Same styling as control buttons. On tap: calls the existing `togglePause()` function.

### Decision F — Control visibility
**Show `#controls` and `#btnPause` only during `PLAYING` and `PAUSED` states.**
Handle this in the JS game loop: show/hide `#controls` and the pause button whenever `state` transitions.

### Decision G — iOS AudioContext unlock
**Call `audioCtx.resume()` in the first `touchstart` event on `document`.**
AudioContext created lazily (good), but iOS also requires an explicit `.resume()` call. Add a one-time `document.addEventListener('touchstart', unlockAudio, { once: true })`.

### Decision H — Landscape orientation
**Show a "Rotate your device" overlay when in landscape mode.**
The game is portrait-only and doesn't benefit from landscape. Add a full-screen overlay that appears when `window.innerWidth > window.innerHeight`. No game logic changes needed.

### Decision I — PWA meta tags
**Add iOS and Android PWA meta tags.**
When a user adds the game to their home screen it should launch full-screen, without the browser chrome. Requires `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style`, a theme color, and a `manifest.json`.

### Decision J — Haptic feedback
**Add `navigator.vibrate()` on crash.**
Android Chrome and some iOS versions support the Vibration API. Pattern: a short `[50, 30, 80]` (buzz–gap–buzz) on crash. Wrapped in a try/catch since iOS Safari doesn't support it.

### Decision K — `touchcancel` handling
**Mirror `touchend` listeners to `touchcancel` on control buttons.**
Same handler: set `keys.left = false` and `keys.right = false`. This prevents stuck movement when the OS interrupts a touch.

---

## 3. To-Do List

Tasks are grouped by phase. Execute in order — earlier phases fix blocking bugs; later phases add polish.

---

### Phase 1 — Critical Bug Fixes (do first)

- [x] **T01** — Fix iOS viewport height
  - Change `body { height: 100vh }` to use `100dvh` with `100vh` fallback
  - File: `index.html` CSS block (~line 15)

- [x] **T02** — Add `viewport-fit=cover` to meta viewport
  - Change: `content="width=device-width, initial-scale=1.0, user-scalable=no"`
  - To: `content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover"`
  - File: `index.html` line 5

- [x] **T03** — Fix stuck keys: add `touchcancel` to control buttons
  - Duplicate the `touchend` listeners for `touchcancel` on `#btnLeft` and `#btnRight`
  - File: `index.html` ~lines 124-127

- [x] **T04** — Unlock AudioContext on iOS
  - Add a one-time `touchstart` listener on `document` that calls `getAudio()` then `audioCtx.resume()`
  - File: `index.html` after audio setup block (~line 199)

---

### Phase 2 — Layout & Safe Areas

- [x] **T05** — Apply safe-area insets to bottom controls
  - Change `#controls { bottom: 20px }` to `bottom: max(20px, env(safe-area-inset-bottom))`
  - File: `index.html` ~line 33

- [x] **T06** — Add safe-area padding to body (top/sides for notch)
  - Add `padding: env(safe-area-inset-top) env(safe-area-inset-right) 0 env(safe-area-inset-left)` to `body`
  - File: `index.html` CSS block

- [x] **T07** — Scale control buttons with viewport
  - Replace fixed `70px` width/height with `clamp(64px, 14vw, 90px)`
  - Adjust `font-size` to scale proportionally: `clamp(22px, 5vw, 32px)`
  - File: `index.html` `.ctrl-btn` CSS rule (~lines 41-55)

- [x] **T08** — Add landscape orientation overlay
  - Add CSS: a `#landscape-msg` overlay div that shows only when `orientation: landscape`
  - Add HTML: a styled overlay with "Please rotate your device" message
  - Handle resize event in JS to also toggle it (for split-screen on tablets)
  - File: `index.html` CSS + HTML + JS

---

### Phase 3 — Controls & UX

- [x] **T09** — Add mobile pause button
  - Add `<div id="btnPause">⏸</div>` inside `#wrapper`, positioned top-right
  - Style it like `.ctrl-btn` but smaller (50px) and positioned `top: max(10px, env(safe-area-inset-top))`
  - Add `touchstart` listener that calls `togglePause()`
  - File: `index.html` HTML + CSS + JS

- [x] **T10** — Control visibility tied to game state
  - In the state transition functions (`startGame`, `triggerCrash`, `showModeSelect`, `showTitle`, `gameOver`): toggle `display` on `#controls` and `#btnPause`
  - Controls visible: `PLAYING`, `PAUSED`
  - Controls hidden: `TITLE`, `MODE_SELECT`, `AMBULANCE`, `GAMEOVER`
  - File: `index.html` JS state functions

- [x] **T11** — Improve touch feedback on control buttons
  - The `:active` CSS state works but has a small delay on iOS. Add `touchstart` visual feedback via JS: toggle a CSS class `.pressed` on `touchstart`/`touchend`/`touchcancel`
  - File: `index.html` CSS + JS

---

### Phase 4 — Performance

- [x] **T12** — Cap devicePixelRatio at 2
  - Change `const dpr = window.devicePixelRatio || 1`
  - To: `const dpr = Math.min(window.devicePixelRatio || 1, 2)`
  - File: `index.html` line 84

---

### Phase 5 — PWA & Platform Polish

- [ ] **T13** — Add iOS PWA meta tags
  - Add to `<head>`:
    - `<meta name="apple-mobile-web-app-capable" content="yes">`
    - `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">`
    - `<meta name="theme-color" content="#111111">`
  - File: `index.html` head block

- [ ] **T14** — Add Web App Manifest
  - Create `manifest.json` with name, icons, display: standalone, orientation: portrait
  - Add `<link rel="manifest" href="manifest.json">` to `<head>`
  - Files: new `manifest.json` + `index.html` head

- [ ] **T15** — Add haptic feedback on crash
  - In `triggerCrash()`: call `navigator.vibrate?.([50, 30, 80])`
  - File: `index.html` ~crash trigger function

---

### Phase 6 — Testing Checklist

- [ ] **T16** — Test on iOS Safari (iPhone SE, iPhone 15 Pro Max)
  - Verify no clipping at top/bottom
  - Verify audio plays on first touch
  - Verify controls respond and don't stick
  - Verify pause button works

- [ ] **T17** — Test on Android Chrome (small phone, large phone)
  - Verify safe areas don't conflict with gesture navigation
  - Verify haptic feedback fires on crash
  - Verify landscape overlay appears correctly

- [ ] **T18** — Test PWA install flow
  - Add to home screen on iOS (via Share > Add to Home Screen)
  - Add to home screen on Android (via Chrome install prompt or menu)
  - Verify game launches full-screen without browser chrome

- [ ] **T19** — Test on tablet (iPad, Android tablet)
  - Landscape mode: overlay should appear
  - Portrait mode: game should scale correctly with large buttons

---

## 4. Out of Scope (Intentional Non-Changes)

These ideas were considered but rejected to keep the game simple and the file self-contained:

- **Swipe/accelerometer controls** — The existing left/right button layout is cleaner and more precise than swipe gestures for this type of game. Adds complexity with no clear benefit.
- **Gamepad API support** — Nice to have but not a mobile issue.
- **Full landscape mode** — Would require redesigning the canvas dimensions and all sprite layouts. Too much change for a minor use case.
- **ARIA/accessibility** — Out of scope for this pass; canvas games require a separate accessibility overhaul.
- **Service worker / offline caching** — The game already works offline (single HTML file). A service worker adds complexity without user benefit here.

---

## 5. Execution Order Summary

```
Phase 1 (Critical bugs)   → T01, T02, T03, T04
Phase 2 (Layout)          → T05, T06, T07, T08
Phase 3 (Controls/UX)     → T09, T10, T11
Phase 4 (Performance)     → T12
Phase 5 (PWA)             → T13, T14, T15
Phase 6 (Testing)         → T16, T17, T18, T19
```

Total tasks: **19**
Critical path (must-do before shipping): **T01–T12**
