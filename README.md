# Universe Simulator

An interactive universe sandbox: a full-screen living universe on black where you play god.
Tap to birth a star, drag to tear open a black hole, detonate a supernova, or spin up a whole
spiral galaxy. Presets: **Big Bang, Milky Way, Black Hole, Nebula**.

- **Author site:** https://sparkainlpx.xyz

Version 0.1.0. Licence: AGPL-3.0-only. No DOI yet (no Zenodo record).

## How to play

1. Open `index.html` in a browser (see "Run locally" below). It starts as a Milky Way.
2. **Tap anywhere** to birth a star. **Drag** to tear open a black hole — it pulls nearby stars
   into orbit and swallows the ones that fall in (watch the gold accretion disk feed).
3. **Supernova — tap a star:** arm it, then tap a star to explode it into a nebula, with new
   stars born in the shock front.
4. **Create galaxy:** spins up a fresh spiral of 210 stars around a new galactic heart.
   **Black hole — tap to drop:** drops a black hole where you tap.
5. Sliders: **Time** (0–3×), **Gravity** (0–2×), **Stars** (20–800), **Scale** (zoom from
   quantum foam out to galaxy clusters). Pause/play and Reset in the dock.
6. **Quantum field** toggle: reveals flowing wave lines beneath everything — a nod to Faggin's
   quantum-field view of the universe (see "Honest limits": it is a visual motif, not a
   measurement).
7. The live panel tracks star count, black holes, universe age in billions of years, and entropy
   (rising as stars spread and burn out).

## Run locally

No build step, no backend, no install. Either:

- Double-click `index.html`, or
- Serve the folder: `python3 -m http.server 8000` and open http://localhost:8000/

The page works offline except for the Google Fonts (Outfit + JetBrains Mono), which fall back
to system fonts if unreachable. No state is saved; a reload restarts with a fresh Milky Way.

## Architecture

Single self-contained `index.html` — inline CSS + one inline `<script>` (an IIFE, `"use strict"`).
No JavaScript libraries, no network calls except the font stylesheet, no localStorage. The page
is a fixed full-screen `<canvas id="sky">` with fixed overlay UI: stats panel (top-left),
field toggle + pause (top-right), a hint toast, and a bottom control dock.

- **State:** arrays `stars, particles, nebulae, holes, cores, motes, foam, ripples`; scalars
  `age, burnedOut, swallowed, entropyFloor, paused, fieldOn, supernovaArmed, holeArmed, time, flash`;
  parameters `speed, gravity, scaleVal`. Caps: 900 stars, 700 particles, 6 black holes, 26 nebulae.
- **Physics (`step(dt)`):** gravity wells are black holes + galaxy cores. Force is
  `G·m / max(d², 400) × dt × 0.055` with `G = 60 × gravity` — a deliberately simplified
  inverse-square law. A star inside a hole's radius + 2 is swallowed (the hole's mass grows and
  its radius becomes `10 + √mass × 0.55`). Stars get a gentle drag (×0.9995) so orbits do not fly
  apart forever, burn out slowly (`life` decays; burnout spawns a small burst), and wrap softly
  at the edges (±80 px). At high time speed the step is substepped twice for stable orbits.
  Age advances by `dt × 0.045 × speed`.
- **Entities:** `makeStar` (colour from a 6-entry palette, twinkle phase, decay rate),
  `makeNebula` (radius, colour, drift, breathing seed), `makeHole` (mass, radius, 46 accretion-disk
  particles), `burst` (explosion particles), `spiralGalaxy` (2–3 arms, radius distribution
  `t^0.62`, orbital velocity `√(900·g/dist) × 26 × 0.016`), `supernova` (removes the star, three
  bursts, a ripple, a nebula, +4 newborn stars, `entropyFloor += 2.5`).
- **Render loop:** `frame(now)` on `requestAnimationFrame` — advances `time`, steps physics
  (skipped while paused), draws, updates the HUD every 6th frame. `draw()` applies a view
  transform (scale 0.45–2.0 from the scale slider, centred on screen), then draws, in order:
  the quantum-field wave lines (9 rows, two sine components) when the field is on; a distant
  galaxy-cluster backdrop at scale ≥ 70; nebulae (three breathing radial-gradient layers each,
  composited `lighter`); galaxy cores; ripples; particles; stars (radial-gradient halos +
  twinkling cores, composited `lighter`); black holes (disk glow, 46 disk particles on a
  `scale(1, 0.38)` tilted ellipse, gold photon ring, black event horizon); a dashed drag preview
  while dragging. Quantum foam (240 flickering points) is drawn in screen space after the view
  transform is reset, with alpha fading in below scale 34; drifting light motes and the
  supernova flash overlay finish the frame.
- **Entropy (HUD):** `spreadScore` (mean star distance from the centroid, ≤ 52) + `burnScore`
  (burned-out + 1.4× swallowed, ≤ 30) + `entropyFloor × 0.35`, clamped 0–100. It is a
  game quantity, not a thermodynamic measurement.
- **Controls mutate state directly:** pointer events on the canvas (`pointerdown` / `pointermove` /
  `pointerup` with `setPointerCapture`; a tap spawns a star, a drag over 26 px opens a black hole
  with mass `500 + dragDistance × 9`, armed modes redirect the tap); dock buttons (presets,
  reset, supernova arm, galaxy, hole arm); sliders for time, gravity, star count and scale;
  the field checkbox and pause button. `toWorld` converts screen coordinates to world
  coordinates through the view scale.

## Honest limits

- **A playful simulation, not real astrophysics.** Gravity and orbits are simplified for fun:
  the force law is softened (`max(d², 400)`), drag is invented, stars wrap at the edges, and
  the "entropy" is a game quantity built from spread + burnouts, not a thermodynamic measurement.
- **The Quantum field toggle is a visual motif**, not a measurement: flowing sine-wave lines
  under the stars, evoking Faggin's view of the universe as waves in a field — not things in
  space. Nothing here tests that view.
- Star counts, ages and masses are arbitrary game numbers.
