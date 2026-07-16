# Basis app training simulations

Interactive, single-file simulations of the Basis apps, built for embedding in training
content (Easygenerator or any LMS that accepts an iframe). One repo hosts them all —
every `.html` file in the repo root gets its own GitHub Pages URL:

| Page | URL | What it teaches |
|---|---|---|
| `index.html` | https://michaelwearebasis.github.io/training-app-simulation/ | **Trade app** — commissioning a Smart Panel end to end (app + interactive switchboard) |
| `home.html` | https://michaelwearebasis.github.io/training-app-simulation/home.html | **Home app** — the customer-value pitch: why the panel pays for itself |

To add another demo, drop `whatever.html` in the repo root and push — it's live at
`/whatever.html` in about a minute. No extra repos or config needed.

---

# Trade app simulation (`index.html`)

An interactive simulation of commissioning a Basis Smart Panel with the Trade app.

**Live page:** https://michaelwearebasis.github.io/training-app-simulation/

Learners work through ten guided parts by tapping in a simulated phone app (left) while a
simulated switchboard (right) responds like the real product — then get a free-play mode to
experiment. Everything is one self-contained `index.html`: no build step, no dependencies, no
network requests (all images are inlined data URIs).

---

## Embedding

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/"
        width="100%" height="1050" style="border:0" allowfullscreen
        title="Basis Trade app — commissioning simulation"></iframe>
```

- `?step=N` (0–72) deep-links into the flow and skips the start screen — e.g. `?step=11` opens
  at "Configure a circuit", `?step=55` at Sync all. Useful for one-part-per-course-page.
- The whole stage transform-scales to the viewport width (min 0.3), so phone + panel stay side
  by side even in narrow embeds and on phones.
- GitHub Pages sends no frame-blocking headers; if an LMS strips iframes, link out instead.

## Deploying changes

Push to `main`. GitHub Pages serves the repo root, so `index.html` goes live in ~1 minute.
The embed URL never changes, so courses pick updates up automatically.

There is no build or minification — edit `index.html` directly and refresh. For local work,
any static server does (`python -m http.server`), or just open the file.

---

## The guided flow

Ten parts, 73 steps (0–72):

| Part | Steps | What happens |
|---|---|---|
| 1 Connect | 0–3 | Auto-discovery → Connect → iOS Wi-Fi join → verify press on the Basis Button |
| 2 Firmware | 4–7 | Firmware banner → download → install → Basis Button press |
| 3 Site details | 8–10 | Address form → ICP selection |
| 4 Configure | 11–20 | Circuit 01: Label, MCB, RCD, Locations |
| 5 Sync & RCD test | 21–28 | Sync → TEST confirms → lever ON → Finish → RCD trip + reset |
| 6 More circuits | 29–54 | 02 HWC (AFDD + meter load control) → bottom next-button straight to 03 office Power (standby lockout) → back + scroll down to 19 Solar (bottom module, no RCD) |
| 7 Sync all | 55–62 | Multi-select sync → bulk apply via two Basis Button presses (2s charge ring between them) → energise circuits 02 and 03 |
| 8 Events | 63–65 | Event log → Earth Leakage Fault detail |
| 9 Offline | 66–69 | Barcode scan → download panel data → downloaded panels |
| 10 Feedback | 70–72 | Settings → Support → Report a bug / Share feedback (→ completion → free play) |

Guided label steps highlight and **enforce** the required choice (HWC / Power / Solar) without
preselecting it, and app toggles (AFDD, load control, lockout) flip freely but the step only
advances once they're in the required state.

The engine is multi-circuit: `CIRCS[n]` is config staged in the app, `DEV[n]` is what's applied on
the device (what the e-ink shows), `CST[n]` is runtime (live / devPending / fault). "Ready to sync"
means CIRCS differs from DEV. Free play uses the same rules: edit any circuit → it goes app-pending →
sync (one, several, or SYNC ALL) → the device flashes blue → TEST **or switching the breaker on**
confirms. The kebab menu on a circuit offers "Set Circuit to spare". The whole stage scales to fit
narrow viewports so phone + panel stay side by side even on a mobile screen, and a lime connector
line draws from the coach card to whatever the current step wants pressed — but only when the
card is showing new text; if the target is below the fold, a bouncing "Scroll down" pill shows
instead until it's in view.

## How the code is organised

Everything lives in `index.html`. CSS first, then markup, then one `<script>`. The script is
divided by banner comments — search for these:

- `/* ============ assets ============ */` — the inlined images (see below) and SVG icons.
- `/* ============ demo constants ============ */` — serial, address, box serial.
- `/* ============ screens ============ */` — `SCREENS.*`: each app screen is a function
  returning `{html, tab, black, noBar}`. Screens read state from `fx` and render fresh each time.
- `/* ============ parts & steps ============ */` — the heart of it:
  - `PARTS[]` — the 10 parts with step ranges and sub-chips (drives the chip stepper).
  - `steps[]` — one entry per step: `screen`, `hotKey` (what gets the lime highlight),
    `advanceOn` (which tap advances), `sim` (a physical panel action: `'button'`, `'sctest'`,
    `'sclever'`), `auto` (ms until auto-advance).
  - `SHORT{}` / `POP{}` — per-step guidance copy for the docked bubble (SHORT is the condensed
    version that must fit the bubble slot; POP holds kicker + title).
  - `stateFor(i)` — **pure derivation of demo state from a step index.** Jumping to any step
    (chips, deep links) rebuilds `fx` from here. If you add or move steps, update the index
    thresholds in this function and the `PARTS` ranges together.
- `/* ============ engine ============ */` — `render()` (redraws phone + stepper + bubble +
  panel), `onEnter()` (per-step timers/animations), `goto()`, click routing on the phone
  (`screenEl` listener), and `freeAct()` — the free-play navigation router.
- `/* ---- physical panel (RHS) ---- */` — the switchboard sim: `panelNeeds()` (which physical
  control is expected), `simPress()` (Basis Button), `scTestPress()`/`scLeverClick()`
  (Circuit Module), `renderSC()` (e-ink/LED/lever state), `renderPanel()` (zoom visibility,
  blur, System Manager LEDs, serial callout), `renderBubble()`.

### State model

- `fx` — everything the current render needs; rebuilt by `stateFor()` on every `goto()`.
  Local interactions (sheet picks, toggles) mutate it between steps.
- `CIRCS[n]` — config staged in the app, per circuit. `userCfgs[n]` holds the learner's own
  choices and survives step jumps (`stateFor()` overlays it on the scripted defaults).
- `DEV[n]` — what's actually applied on the Sub-Circuit. Only updated when a sync lands and
  TEST (or switching the breaker on) confirms. The e-ink renders `DEV`, the app renders
  `CIRCS` — that split is deliberate and mirrors the real product (the display doesn't preview
  pending config). `CST[n]` (via `cst(n)`) is runtime state: live / devPending / fault.
- `freePlay` — after completion: no step gating, `freeAct()` routes navigation, config changes
  go app-pending → sync makes them device-pending → TEST/lever behave per firmware.

## Firmware fidelity (what the panel does and why)

Behaviour was extracted from the real firmware — keep these rules if you touch the panel sim:

**Sub-Circuit** (from `wearebasis/subcircuit`):
- LED: **solid red** = live · **solid yellow** = any electrical fault (RCD/MCB/AFDD) ·
  **blue 1s on / 1s off** = config or firmware pending (overrides every other colour) ·
  **breathing red (3.5s)** = standby · **off** = safe/OFF.
- E-ink: black background, white mono text; rows = label+number, state word (`ON`/`OFF`/
  `ELEC FAULT` + detail like `RCD`), `CONFIG PENDING`, `MCB C16`, `RCD A30mA`, `AFDD ACTIVE`.
  Unconfigured = `NONE` on all three. It refreshes with an inversion flash and **only updates
  when config is applied**, never while editing in the app.
- TEST: short press confirms a pending config; on a live circuit it injects ~4/3 × the RCD
  threshold for ≥400ms → the breaker must trip (lever throws, solid yellow, `ELEC FAULT/RCD`).
- Reset after any trip is manual: lever OFF→ON. There is no powered re-close.

**System Manager** (from `wearebasis/system_manager`):
- STATUS: white breathing = idle · **blue slow-blink = a Basis Button press is expected**
  (app authorisation, apply window) · amber fast-blink (250ms) = firmware applying.
- COMMS: off = no network · blue slow-blink = hotspot up, no client · solid blue = phone
  joined · solid white = cloud connected.
- The panel only animates these two LEDs; there is no separate CONFIRM indicator.
- Hotspot SSID is `BasisBoard-<serial>`; discoverability (BLE) is on from boot.

## Assets

All images are base64 data URIs inside `index.html`:

| Asset | Source |
|---|---|
| Basis logos (light/dark) | Brand kit SVGs (Google Drive → Assets → Logo Suite) |
| Panel photo (app cards/dashboard) | Cropped from a real app screenshot |
| Full unit render (RHS panel) | `BP_005_0010_v008_Unit.png` marketing render → 520px webp |
| Press-button illustration | Frame from app screen recording |
| Apply-instructions illustration | Frame from app screen recording |
| Barcode scan camera view | Real app screenshot |
| Panel avatar (offline download) | Real app screenshot crop |

To swap one: produce the new image (keep it small — webp/jpeg, ≤50KB), base64 it, and replace
the corresponding `data:image/...` string (they're assigned to constants like `IMG_PANEL` at
the top of the script, except the unit render and logos which sit in the markup/`src`).

## Making common changes

- **Copy**: edit `SHORT{}` (bubble text — keep it to ~3 sentences so it fits) and `POP{}`
  (titles/kickers). App screen copy lives inline in each `SCREENS.*` function and comes from
  the real app — check the Trade app before "fixing" odd casing (e.g. the lowercase `events`
  title is genuine).
- **Values** (serial, address, circuit label/ratings): constants at the top of the script +
  defaults in `stateFor()`/`userCfg` fallbacks (`'Lights'`, `'C 16A'`, `'A 30mA'`).
- **Add a step**: insert into `steps[]`, then renumber everything downstream of it:
  `PARTS` ranges, `stateFor()` thresholds, `SHORT`/`POP` keys, and any hardcoded indices in
  the panel-visibility rules in `renderPanel()` (`pzShow`/`scShow`) and `renderSC()`.
  Grep for the old index numbers before shipping.
- **Timings**: `auto` per step, animation durations in `onEnter()` (firmware download 3.4s,
  install 2.6s, RCD inject 450ms, download ring 2.2s).

## Testing checklist before pushing

1. Serve locally, open with a clean URL (start screen should appear; `?step=N` should skip it).
2. Click through all 10 parts end to end — every advance is a tap in the app or on the panel;
   there is no "next" button. The lime highlight, connector line or "Scroll down" pill marks
   the next action (the line only draws when the coach card is showing new text).
3. Part 5 specifically: TEST confirm → lever ON → Finish → RCD test (lever must throw + app
   tile shows FAULT) → lever reset.
4. Part 7 specifically: first Basis press starts a 2s lime charge ring (a second press is
   blocked until it completes) → second press applies → lever circuits 02 and 03 ON. Free
   toggles and trips are allowed here; flicking a spare's lever pops up a "needs configuring"
   explainer.
5. Completion dialog → **Explore freely**: change a setting → it goes app-pending → sync →
   device flashes blue → TEST or breaker-on confirms; RCD test + reset; tab/back navigation
   everywhere.
6. Jump to every part via the chips (state must be seeded correctly — e.g. jumping to Sync
   gives a fully configured circuit 01).
7. Check a ~860px and a ~375px viewport: phone + panel stay side by side (the stage scales
   down), no horizontal scroll.
8. No console errors.

## Known simplifications

- Timings are compressed (the real 30-minute apply/revert windows are shown but don't expire).
- Phone status bar is decorative; firmware-update screens are code-derived (no product
  screenshots existed for them).
- E-ink uses a generic mono font stack (DM Mono isn't bundled), sizes are approximate.
- The guided flow scripts circuits 01, 02, 03 and 19; the rest start as spares — but every
  circuit is fully simulated and configurable in free play.
- Sample data only — nothing talks to a real panel or backend.

## Source references

Fidelity came from these repos (Basis GitHub org access required): `wearebasis/ios` (Trade app
screens and copy — `Trade/Trade/Views/**`, `Localizable.xcstrings`), `wearebasis/subcircuit`
(LED patterns `src/images/faraday/subcircuit_consumer/status_led_patterns.hpp`, display
`src/application/ui/display.cpp`, config FSM `src/application/config/fsm.hpp`, RCD test
`src/application/rcd_self_test/`), and `wearebasis/system_manager` (`app/ui/ui.go`,
`app/ui/ui_output.go`, `app/bluetoothcomms/`).

---

# Home app simulation (`home.html`)

A sales-pitch walkthrough of the **Home app** for the trade audience: what the customer gets,
and why the panel pays for itself. Phone on the left, coach card plus a running
**"Basis pays for itself, three ways"** tracker on the right — the three pillars (see the
waste / shift the big loads / the right plan, proven) light up with dollar values as the
learner works through the flow, ending in an ROI screen: **$39 + $44 ≈ $83/mo, about $1,000
a year** for the demo home.

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/home.html"
        width="100%" height="1050" style="border:0" allowfullscreen
        title="Basis Home app — customer value demo"></iframe>
```

Seven parts, 39 steps (0–38), `?step=N` deep-links work the same as the Trade demo:

| Part | Steps | What happens |
|---|---|---|
| 1 Connect | 0–2 | Unlinked home → Connect → pairing progress |
| 2 Energy plan | 3–9 | Add retailer (Meridian) → pick plan (Anytime) → confirm rates → everything turns into dollars |
| 3 Live | 10–12 | Live usage on Home → Control tab circuit grid → Hot water 3 kW at peak |
| 4 Costs | 13–20 | Cost insight → $284 month → towel rails ($30, 24/7) → garage fridge ($17) → pillar 1: $39/mo |
| 5 Activity | 21–24 | Overview (safety events + **Your electrician** card) → event log → overheating HWC detail ("Get help" → installer) |
| 6 Routines | 25–33 | Suggested hot-water routine → EV → towel rails (Standby-scheduling, as in the real app) → pillar 2: 10.8 kWh/day |
| 7 Right plan | 34–38 | My energy plan → Compare plans re-prices 30 days of real data → MoveMaster saves $44/mo → pillar 3 → ROI |

The layout matches the real Home app (mined from `wearebasis/ios` → `Home/`): 4 tabs
(Home / Control / Routines / Activity), light theme, `#39E581` accent, live power formatted
`999 W` / `1.500 kW`, importing red / exporting green, routines put a circuit on **Standby**
during scheduled windows, Activity has Overview|Events segments and a linked-electrician
section. The demo chrome (coach card, stepper, value tracker) uses the Basis brand palette
(Coal/Lime/Sand) to stay visually distinct from the app under demonstration.

Numbers are indicative NZ prices (flat 28.7c/kWh vs a TOU night rate), chosen so the three
pillars never double-count: insights savings work on any plan; the routines + plan-switch
saving is quoted as a combination (switching without shifting would cost more — that's the
"only Basis does both" line in part 7). A footnote on the page marks it as sample data.

After the ROI screen, **Explore freely** unlocks the whole app: tabs, routine toggles,
the plan-setup flow and the comparison all work.
