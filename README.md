# J1GChaos - Chaos Mod for Jak 1 

A lightweight "chaos" system for Jak 1 that periodically applies fun/random gameplay and visual effects (big heads, teleports, weather changes, orb/cell changes, fake crashes, and more).

---

## Quick Start 

1. Start the game and use a controller to toggle the mod UI:
   - Open the *retail popout menu*: **L1 + R1 + L2** (also shown in-game as: "PRESS L1+R1+L2 TO OPEN MENU").
   - Legacy quick-select (hold): **L1 + R1 + L3**.
2. In the CHAOS menu:
   - Use D-pad / Left-stick to navigate, **X** to activate/toggle the selected row.
   - Toggle **"Chaos: ON"** to enable periodic random effects.
   - Use the **Effects** submenu to pick or trigger specific effects immediately.

Tip: You can also control the system from the REPL for testing. See "REPL & API" below.

---

## What it does - Features 

- Periodic, stackable effects triggered at a configurable interval (default 8s).
- Visual effects: big/small/huge heads, reflections, mirror world, PS1-style textures, sky/time-of-day changes, weather.
- Gameplay modifications: invincibility, harder rats, hero mode, teleportation (random and directional), spawn orbs, add/subtract/double orbs and cells, change life.
- Special effects: fake crash (freeze position), instant crash, kill-delay, collision rendering toggle.
- In-game UI showing: watermark, timer until next effect, active effects list with remaining time.

---

## Controls & UI 

- Open popout menu: **L1 + R1 + L2**
- Quick-select (legacy): hold **L1 + R1 + L3**
- Navigation: D-pad / left stick
- Activate/toggle: **X** (or corresponding keyboard mapping)
- In Effects submenu: select an effect, press **X** to trigger immediately
- In Main menu: toggle watermark, interval, crash effects, clear all, and hide UI

---

## REPL & API (developer/testing) 

All primary modulation logic lives in:
- `goal_src/jak1/engine/mods/mod-custom-code.gc` (core chaos logic)
- `goal_src/jak1/engine/mods/mod-common-functions.gc` (helper drawing functions & display names)

Useful functions & variables (callable from REPL when a running game is attached):

- Toggle timer (programmatic):
  - (set! *chaos-enabled?* #t) - enable
  - (set! *chaos-enabled?* #f) - disable
  - (chaos-toggle-timer (the int (-> *display* base-frame-counter))) - toggle and schedule next

- Start/stop effects:
  - (chaos-start-effect 'big-head (the int (-> *display* base-frame-counter))) - start an effect now
  - (chaos-start-effect-permanent 'big-head (the int (-> *display* base-frame-counter))) - permanent (startup-persistent) effect
  - (chaos-clear-all) - clear all active/pending effects and restore default state

- Test helpers:
  - (chaos-add-orbs 50.0) - add 50 orbs
  - (chaos-set-orbs 0.0) - set orb count
  - (chaos-add-cells 1.0) - add powercell
  - (chaos-set-cells 0.0) - set powercell count

- Timer and configuration:
  - (set! *chaos-interval-sec* 10) - change interval between automatic effects
  - *chaos-enabled?* - boolean (#t/#f)
  - *chaos-show-watermark?* - boolean (show watermark text)

REPL example: start an effect immediately

```scheme
gc> (chaos-start-effect 'big-head (the int (-> *display* base-frame-counter)))
```

Note: most functions take a current tick/frame int (usually obtained via `(the int (-> *display* base-frame-counter))`).

---

## Effects (overview) 

Effects are defined as symbols in the code (see `*chaos-effect-list*`). A few notable examples:

- Visual: `big-head`, `small-head`, `huge-head`, `mirror`, `no-tex` (PS1 look), `sky`, `weather-thunder` / `weather-rain` / `weather-snow`.
- Gameplay: `invinc` (invincibility), `hard-rats` (increase enemy difficulty), `hero-mode`.
- Teleports: `tp-n-05`, `tp-s-15`, `tp-rand-05-10`, `tp-rand-120-200` (various distances and directions).
- Economy: `orbs-add-25`, `orbs-add-100`, `orbs-sub-50`, `orbs-set-0`, `orbs-double`.
- Cells: `cells-add-1`, `cells-add-5`, `cells-set-0`, `cells-double`, `cells-half`.
- Spawns: `spawn-orbs-10`, `spawn-orbs-wide-20`.
- Dangerous: `kill-delay`, `crash-delay`, `fake-crash`, `instant-crash`.

For human-readable names and messages the mod uses `chaos-effect-display-name` (see `mod-common-functions.gc`).

For the full list of effects and their behavior, inspect `goal_src/jak1/engine/mods/mod-custom-code.gc` (search for `set! (-> *chaos-effect-list*`).

---

## Developer Notes / How to extend

- Add a new effect:
  1. Add a symbol into `*chaos-effect-list*` (in `mod-custom-code.gc`).
  2. Implement the behavior in `chaos-apply-effect` within `mod-custom-code.gc` (look for the case statement). Use helper functions (e.g. `chaos-apply-cheat`, `chaos-add-orbs`, etc.).
  3. Add a display name in `chaos-effect-display-name` (in `mod-common-functions.gc`) so the UI shows friendly text.
  4. Optionally add constants to control duration or behavior.

- UI strings and drawing helpers are in `mod-common-functions.gc` (functions like `draw-timer`, `draw-active-effects`, `draw-watermark`).

- Persistent vs one-shot effects: the code differentiates persistent cheat-like effects vs one-shot visuals.

---

## Files of interest 

- `goal_src/jak1/engine/mods/mod-custom-code.gc` - Chaos logic and effect implementations
- `goal_src/jak1/engine/mods/mod-common-functions.gc` - Display, helper functions, and display-name mapping
- `goal_src/jak1/engine/mods/mod-settings.gc` - (project settings/flags)

---

## Contributing & License 

If you want to contribute improvements or new effects, please open a PR and include brief notes about the effect and testing steps. This project follows the repository's main LICENSE file in the project root.
