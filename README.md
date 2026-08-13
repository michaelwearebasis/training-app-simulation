# Basis app training simulations

Interactive, single-file simulations of the Basis apps, built for embedding in training
content (Easygenerator or any LMS that accepts an iframe). One repo hosts them all —
every `.html` file in the repo root gets its own GitHub Pages URL:

All URLs below are under `https://michaelwearebasis.github.io/training-app-simulation/`.

| Page | Path | What it teaches |
|---|---|---|
| `index.html` | `/` | **Trade app** — commissioning a Smart Panel end to end (app + interactive switchboard) |
| `leds.html` | `/leds.html` | **LED guide** — animated legend of every Sub-Circuit and System Manager light and what it means (real firmware colours + timings) |
| `home.html` | `/home.html` | **Home app** — four-phone showcase of what the customer gets (live power, routine, insights, plans) |
| `mission-control.html` | `/mission-control.html` | **Mission Control** — guided 3-part walkthrough of the fleet dashboard sparkies use to support a home (site → insights → savings), on real datalake data |
| `margin-calculator.html` | `/margin-calculator.html` | **Margin calculator** — interactive tool: a sparky's net margin on a Basis Board install (board size, rebate tier, labour, fully installed cost) |
| `supported-devices.html` | `/supported-devices.html` | **Supported devices** — which solar inverters and EV chargers integrate with Basis, with real brand logos and integration status |
| `case-studies.html` | `/case-studies.html` | **Case studies** — auto-scrolling wall of electrician testimonials/quote tiles |
| `consult.html` | `/consult.html` | **Pre-quote conversation** — the questions to ask a customer before quoting a Basis Panel |

To add another demo, drop `whatever.html` in the repo root and push — it's live at
`/whatever.html` in about a minute. No extra repos or config needed.

**Cache-busting embeds.** Easygenerator (and most LMSs) cache the iframe `src`. When you ship
an update, bump a `?v=N` query on the embed URL (`…/mission-control.html?v=8`) so learners get
the new build. The number is arbitrary — just change it. GitHub Pages itself serves the latest
within ~1 minute of a push.

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

- `?step=N` (0–76) deep-links into the flow and skips the start screen — e.g. `?step=11` opens
  at "Configure a circuit", `?step=57` at Sync all. Useful for one-part-per-course-page.
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

Ten parts, 77 steps (0–76):

| Part | Steps | What happens |
|---|---|---|
| 1 Connect | 0–3 | Auto-discovery → Connect → iOS Wi-Fi join → verify press on the Basis Button |
| 2 Firmware | 4–7 | Firmware banner → download → install → Basis Button press |
| 3 Site details | 8–10 | Address form → ICP selection |
| 4 Configure | 11–20 | Circuit 01: Label, MCB, RCD, Locations |
| 5 Sync & RCD test | 21–28 | Sync → TEST confirms → lever ON → Finish → RCD trip + reset |
| 6 More circuits | 29–56 | 02 HWC (AFDD + meter load control) → bottom next-button straight to 03 office Power (standby lockout) → back + scroll down to 19 Solar (bottom module, no RCD) → the "if you mess up" tour of the panel menu (set all to spare / discard all pending) |
| 7 Sync all | 57–66 | Multi-select sync → bulk apply via two Basis Button presses (2s charge ring between them) → energise circuits 02 and 03 → trip-test the new HWC circuit + reset |
| 8 Events | 67–69 | Event log → Earth Leakage Fault detail |
| 9 Offline | 70–73 | Barcode scan → download panel data → downloaded panels |
| 10 Feedback | 74–76 | Settings → Support → Report a bug / Share feedback (→ completion → free play) |

Guided label steps highlight and **enforce** the required choice (HWC / Power / Solar) without
preselecting it. App toggles (AFDD, load control, lockout) flip freely and never advance the
step themselves — the coach card grows a **Next** button that enables once the toggle is in
the required state, so the guidance text can't vanish mid-read. Mid-guided, any detail row
(Label / MCB / RCD / Locations) opens its sheet as an overlay so learners can go back and
change earlier choices without derailing the script; the MCB and RCD sheets carry a **Reset**
in the top bar, opposite the X. TEST never RCD-trips a circuit that has no RCD configured.

The engine is multi-circuit: `CIRCS[n]` is config staged in the app, `DEV[n]` is what's applied on
the device (what the e-ink shows), `CST[n]` is runtime (live / devPending / fault). "Ready to sync"
means CIRCS differs from DEV. Free play uses the same rules: edit any circuit → it goes app-pending →
sync (one, several, or SYNC ALL) → the device flashes blue → TEST **or switching the breaker on**
confirms. The kebab menu on a circuit offers "Set Circuit to spare" and "Discard pending
updates"; the kebab on the Circuits screen opens the panel-wide menu (set **all** circuits to
spare / discard **all** pending changes). The whole stage scales to fit
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
   blocked until it completes) → second press applies → lever circuits 02 and 03 ON → TEST
   on 02 must trip (lever throws, yellow LED) → lever resets it. Free toggles and trips are
   allowed here; flicking a spare's lever pops up a "needs configuring" explainer, and TEST
   on a circuit with no RCD must toast instead of tripping.
5. Toggle steps (AFDD / load control / lockout): flipping the toggle must NOT advance — the
   coach card's Next button enables instead. While on those steps, opening another sheet
   (say MCB) as an overlay, editing, and confirming must land you back on the same step.
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

# Home app — four-phone showcase (`home.html`)

One page, four phones, each on its part of the customer story — with the **real app's look
and flows** inside every phone, built from **a real Basis home's data**. Lime highlights with
little action bubbles point at what to press ("Open the cylinder", "Start here", "Open the
cheapest plan"), and values update live.

| Phone | Real flow inside it |
|---|---|
| Live, circuit by circuit | Control grid (20 circuits, wattages ticking, sticky whole-home total) → circuit detail (hero icon, green live line, Tags, Activity cards, protection rows) → black Standby / green Turn-on button; flicking HWC jumps the total ~450 W → 3.4 kW |
| Set-and-forget routines | The real creation flow: Energy-routines promo → Select Circuit (radio list) → Custom routine editor (repeat day chips recalc standby hours, time window) → mint Create → "Routine created ✓" → active card with Stop routine |
| The month, in dollars | Cost screen: D/W/M/Y segments, donut, all 15 circuits priced ($209.94 real month) → tap a circuit for its daily chart + monthly average |
| The right plan, proven | Real comparison list vs ecoSOLAR $429.71 (Freedom $330.65 ↓23% "$99 saved" → plans costing $116 MORE) → plan detail with rate windows (the current plan shows the real TOU peak bands) → "Switch to this plan" → Ready-to-switch steps |

`?pair=1` shows just the live + routines phones, `?pair=2` the insights + plans phones —
for one two-phone block per course page. No parameter shows all four.

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/home.html?pair=1"
        width="100%" height="860" style="border:0" allowfullscreen
        title="Basis Home app — live power and routines"></iframe>

<iframe src="https://michaelwearebasis.github.io/training-app-simulation/home.html?pair=2"
        width="100%" height="860" style="border:0" allowfullscreen
        title="Basis Home app — insights and energy plans"></iframe>
```

The phones wrap responsively: 4-up above ~1420px (height ≈ 760), 2×2 on a typical course
column (height ≈ 1400), single column on phones. Tab bars are rendered per the real app;
tapping a tab that belongs to another phone toasts a pointer so each phone stays on topic.
Single self-contained file, ~60 KB, no network requests.

Earlier formats (guided walkthrough, query-param mini-sims, simplified wall) are in git
history if ever wanted.

---

# Mission Control — guided fleet-dashboard walkthrough (`mission-control.html`)

A guided tour of **Mission Control**, the internal fleet dashboard sparkies use to support
every home they've installed a Basis panel in. One self-contained `mission-control.html`,
light Basis/stone theme, Oracle font — visually matched to the Trade and Home sims so the
three sit together in one course.

**Live page:** https://michaelwearebasis.github.io/training-app-simulation/mission-control.html

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/mission-control.html?v=8"
        width="100%" height="860" style="border:0" allowfullscreen
        title="Basis Mission Control — guided walkthrough"></iframe>
```

## The storyline

A `Get started` intro pop-up, then **three darkened-backdrop section pop-ups** break the tour
into parts. Within a part, the guidance is **small floating pills that point at the thing to
focus on** (no docked card, no full-screen dimming). A floating **Restart** button re-runs it.

1. **Site overview** — the **Sites** list (search + Address / Panels / ICP) → open **1 B
   Northumberland Avenue** → its **Overview** (site-info + at-a-glance pills) → **Property**
   (Trade Users / Home Users, Energy-retailer plan) → **Panel** (20-circuit configuration;
   Connectivity with realistic Wi-Fi telemetry in plain language).
2. **Insights** — **Faults** (open the critical **MCB trip on C14 Power** → **waveform capture**:
   primary current cuts to 0 at the trip line, residual current, phase voltage, with a trip
   line + cause annotations) → **Peak load** in **current (A)** — 57 A vs the 63 A supply,
   `7d / 30d / 12m` ranges, and a "what drew the peak" circuit breakdown → **Power quality**
   (voltage touched 253.5 V) → **Temperature**.
3. **Savings** — **Retail comparison** with real NZ retailer logos (Mercury is both current
   and cheapest), a plan "lollipop" list, plus **Savings** overview and **Projections** sub-views.

## How it's built

- **Real data.** Site config, connectivity, peak load, power quality and temperature were
  pulled from the AWS Athena datalake (`datalake_silver_prd`) for panel serial
  `FALUBX3KR9RSQCNQ0G5E` (ICP `1002057214UN96F`). User names are generic; waveforms and a few
  app-domain figures (faults, retail plans) are synthetic but consistent with the panel.
- **App shape** mirrors `wearebasis/platform-web` → `apps/mission-control` (the retailer /
  savings / routines components in particular). The left sidebar is intentionally dropped —
  the tour is about the middle content.
- **Vanilla-JS SPA**: a `S` state object + `render()` (innerHTML) + `wire()`. Guidance lives in
  `STEPS[]` (each: `section`, `target`/`anchor`, `place`, `msg`, `advanceOn`) and `STORY{}` (the
  section pop-ups). Coach pills/rings are positioned in **viewport coords (`position:fixed`) and
  re-tracked on scroll**, and the **tab bar is sticky under the banner**, so a highlighted tab is
  never hidden and the ring never drifts. The waveform modal locks page scroll behind it.
- **Deep-link hash** for debugging: `#screen=site&tab=insights&insTab=peak&coach=17&intro=none`
  (`intro=none` suppresses the section pop-up; `coach=99` hides the coach; `wave=f1` opens a
  waveform). Keys map straight onto `S`.

Headless verification uses Chrome (`--headless=old … --screenshot`) since the built-in browser
pane renders external files as static snapshots (no JS).

---

# Margin calculator (`margin-calculator.html`)

An interactive tool showing a sparky the **net margin on a Basis Board install** — board size
(12 / 16 / 20 circuit), quarterly **rebate tier** (0–2 / 3–9 / 10–15 / 16+ panels → $0–$400),
labour hours + rate, travel, sundries, optional inspection, and a **Fully installed cost**
slider (starts at the ideal **$3,999**). It shows the Basis Board trade price (with rebate),
input costs, and the net margin **per job and per hour** live. Inputs persist in `localStorage`
(except the fully installed cost, which resets to $3,999 each load); the page hugs its content
and posts its height to the parent for auto-resize.

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/margin-calculator.html?v=N"
        width="100%" height="980" style="border:0" title="Basis margin calculator"></iframe>
```

---

# Supported devices (`supported-devices.html`)

Which **solar inverters** (Tesla, Sigenergy supported; Fronius, SolaX, Sungrow, GoodWe coming)
and **EV chargers** integrate with Basis, each with its real brand logo and integration status,
plus a short line on what "integrates with Basis" actually gets the customer. Two-column cards,
full-width intro. Logo assets live in `logos/`. Built from `wearebasis/SolarDigitalGuideline`.

# Case studies (`case-studies.html`)

An auto-scrolling (slow ping-pong, pauses on hover) wall of electrician **testimonials and quote
tiles**, with Basis-green pills matching the brand cue.

# Pre-quote conversation (`consult.html`)

The **questions to ask a customer before quoting** a Basis Panel — a text/reference page for the
consultation step of the sales process.

---

## Retailer logos (`logos/retailers/`)

Mission Control's savings comparison shows real NZ energy-retailer marks (Mercury, Meridian,
Genesis, Octopus, Contact, Powershop, Electric Kiwi). They're the retailers' own favicons,
fetched at 128 px and committed as `fav_<name>.png`; the HTML references them by relative path
(`logos/retailers/fav_mercury.png`) so they resolve on GitHub Pages. To refresh one, re-fetch
its favicon and overwrite the file.
