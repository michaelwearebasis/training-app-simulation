# Basis Trade app — training simulation

An interactive, single-file simulation of commissioning a Basis Smart Panel with the Trade app,
built for embedding in training content (Easygenerator or any LMS that accepts an iframe).

**Live page:** https://michaelwearebasis.github.io/training-app-simulation/

## Embed

```html
<iframe src="https://michaelwearebasis.github.io/training-app-simulation/"
        width="100%" height="1000" style="border:0" allowfullscreen></iframe>
```

Deep-link to a specific step with `?step=N` (0–41), e.g. `?step=27` opens at the
config-confirm moment.

## What it covers

Eight guided parts, each introduced by a what/why pop-up, driven entirely by tapping in the
simulated app (left) and pressing controls on the simulated switchboard (right):

1. Connect to the Smart Panel (Basis Button, hotspot join, authorise)
2. Update the software
3. Set the site details (address + ICP)
4. Configure a circuit — Label, MCB, RCD, Locations, AFDD, Meter load control, Standby lockout
5. Sync to the panel & test the RCD (TEST confirm, energise, RCD trip + reset)
6. Event logs
7. Offline mode
8. Report bugs & share feedback

## Fidelity notes

- App screens reproduce the real Trade app (copy sourced from the app codebase and screenshots).
- The Sub-Circuit render follows the firmware: LED solid red = live, solid yellow = electrical
  fault, blue 1 s/1 s flash = config/firmware pending (overrides state colour), breathing red =
  standby, off = safe; e-ink shows label/state/MCB/RCD/AFDD and only updates when config applies;
  TEST = confirm pending config or RCD self-test (injects ~4/3 × threshold, trips the breaker);
  reset is manual lever OFF→ON.
- System Manager LEDs follow the head-unit firmware: STATUS breathes white when idle, blinks blue
  when a Basis-Button press is expected, fast-blinks amber while firmware applies; COMMS blinks
  blue while the hotspot is up and goes solid blue once the phone joins; CONFIRM heartbeats.

Everything is inlined in `index.html` (no external requests) — images are embedded data URIs.

Demo uses sample data only; it is not connected to a real panel.
