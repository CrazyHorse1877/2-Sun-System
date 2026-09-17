# Two suns

An interactive 2D simulation of planets in a binary star system, showing where "Tatooine-style" planets can survive and where they can't. It is packaged as a single HTML file with no build step and no dependencies.

Planets can orbit a binary in two stable ways: close to one star (S-type orbits) or far outside both (P-type, or circumbinary orbits). In between lies a chaotic zone, where the two stars take turns tugging on a planet until it is thrown out of the system or falls into a star. The simulation draws the predicted zones, lets you drop planets anywhere, and can test the predictions by brute force.

<img width="780" height="793" alt="image" src="https://github.com/user-attachments/assets/92ac93a8-3d57-460a-a178-d00e6116d496" />

## Quick start

Open `binary-star.html` in any modern browser (Chrome, Firefox, Safari, Edge).

The page works offline. It loads two typefaces (Cormorant Garamond and IBM Plex Sans) from Google Fonts when a connection is available and falls back to system fonts otherwise.

## Using the simulation

### Scenarios

| Scenario | What it shows |
| --- | --- |
| One planet per zone | Stars with a 35/65 mass split and eccentricity 0.2. One planet orbits each star inside its safe zone, one orbits both stars safely far out, and two start in the chaotic zone and are soon lost. |
| Kepler-16 | The first confirmed circumbinary planet (Doyle et al., 2011). Mass share 0.23, eccentricity 0.16, planet at 3.14 star separations, just outside the predicted edge of 2.89. |
| Kepler-34 | Two nearly equal stars on a very eccentric orbit (0.52), with the planet at 4.78, beyond the predicted edge of 3.66. |
| Ring of planets | 18 planets on circular orbits from 1.2 to 4.6 separations, spanning the chaotic zone and the stable region beyond it. |

### Controls

- **Pause / Play** stops and resumes time.
- **Restart** relaunches every planet from its original orbit.
- **Remove planets** clears all planets.
- **Star B's share of the mass** (0.1–0.5) sets the mass ratio. At 0.5 the stars are twins.
- **Orbit eccentricity** (0–0.7) sets how elongated the stars' orbit around each other is.
- **Speed** is measured in orbits of the stars per real second.
- **Trail length** is measured in orbits of the stars.
- **Show predicted zones** toggles the shaded zones and dashed boundaries.
- **Show scan results** toggles the rings drawn by the stability scan.

Changing either star slider restarts every planet from its original orbit and clears old scan results, since the zones have moved.

### Mouse and touch

- **Tap** anywhere to add a planet on a circular orbit. If you tap close to a star (within 45% of the stars' current separation), the planet orbits that star; otherwise it orbits both. Up to 60 planets can be added.
- **Drag** to pan.
- **Scroll** or **pinch** to zoom.

### Reading the view

- Green shaded discs around each star are the zones where a planet can safely orbit that star alone.
- The red shaded region is the chaotic zone.
- The red dashed circle is the inner edge of the stable region for planets orbiting both stars. Everything outside it is safe.
- The faint ellipses are the paths of the two stars.
- A lost planet is marked with a cross and its trail fades. The **What happened** panel lists each loss, its starting orbit, and how long it lasted.

## The stability scan

**Run stability scan** checks the predicted zones by brute force. It launches test planets on circular orbits at:

- 41 distances around both stars, from 1.0 to 5.0 separations
- 29 distances around each star, from 0.04 to 0.60 separations

Each distance is tried at four different starting angles and binary phases, for about 400 test planets in total, and each runs for 100 orbits of the stars. A distance counts as stable only if all four planets survive with their orbits intact. A planet fails if it hits a star, becomes unbound from what it was orbiting, or its distance from what it was orbiting halves or doubles.

Results appear as green (stable) and red (unstable) rings, and the panel compares the measured edges with the predicted ones. The scan runs in small slices between animation frames, so the simulation stays responsive while it works.

Sample results:

| System | Around both stars (scan / predicted) | Around star A | Around star B |
| --- | --- | --- | --- |
| One planet per zone | 2.80 / 3.07 | 0.26 / 0.25 | 0.18 / 0.17 |
| Kepler-16 | 2.70 / 2.89 | 0.30 / 0.30 | 0.16 / 0.14 |
| Kepler-34 | 3.30 / 3.66 | 0.08 / 0.11 | 0.08 / 0.11 |

The zones around single stars agree closely. The edge for circumbinary planets comes out a little closer in than predicted because the scan runs for 100 binary orbits, while the formula was fitted to runs of 10,000. Some orbits near the edge take longer than 100 orbits to break down.

## How it works

### Units

Distances are measured in units of the binary's semi-major axis (the stars' average separation), masses as fractions of the combined star mass, and time in orbits of the stars. With those choices the gravitational constant is G = 4π².

To convert to a real system, multiply distances by its actual separation. Kepler-16's stars are 0.224 AU apart, so its planet's 3.14 corresponds to 0.705 AU.

### The stars

The stars follow the exact two-body Kepler solution, so their orbit never drifts. Their positions and velocities are precomputed at 2000 points over one orbit by solving Kepler's equation with Newton's method, then looked up during integration. Star A is at −μ·**r** and star B at (1−μ)·**r**, where **r** is their separation vector and μ is B's share of the mass, which keeps the centre of mass fixed at the origin.

### The planets

Planets are massless test particles: they feel both stars but don't pull on them or on each other. That is a very good approximation for real planets, which are thousands of times lighter than stars.

Planets are integrated with a leapfrog (kick–drift–kick) scheme at 2000 steps per binary orbit, using the tabulated star positions. A planet is removed if it comes within 0.02 separations of a star or moves beyond 40 separations with enough energy to escape.

### Predicted zones

The zones come from the empirical fits of Holman & Wiegert (1999), which are based on long numerical integrations. With binary eccentricity *e*:

Stable beyond this distance when orbiting both stars, using μ = the smaller star's mass share:

```
a_P = 1.60 + 5.10e − 2.22e² + 4.12μ − 4.27eμ − 5.09μ² + 4.61e²μ²
```

Stable within this distance when orbiting one star, using μ = the other star's mass share:

```
a_S = 0.464 − 0.380μ − 0.631e + 0.586μe + 0.150e² − 0.198μe²
```

The fits are valid for mass shares of 0.1–0.5 and eccentricities of 0–0.7, which is exactly the range the sliders allow.

## Project structure

```
binary-star.html          The entire application: markup, styles and script
binary-star-README.md     This file
```

## Customising

All the tunable pieces live near the top of the `<script>` block:

- `scenarios` holds the presets. Each lists the mass share, eccentricity, and planets as `{kind, r, theta}`, where `kind` is `'A'` or `'B'` for a planet orbiting one star and `'P'` for one orbiting both.
- `N` is the number of integration steps per binary orbit.
- `RCOL` is the collision distance and `ESC` the escape distance.
- `PALETTE` sets the planet colours.
- The scan's distance ranges and its length (100 orbits) are set in `startScan()`.

## Limitations

- Motion is confined to a plane, and all planets orbit in the same direction as the stars.
- Planets are massless and don't interact with each other.
- Stars are point masses; the collision radius is the same for both.
- The scan's 100-orbit runs make its circumbinary edge slightly optimistic compared with long-term stability.

## Further reading

- M. J. Holman and P. A. Wiegert, "Long-term stability of planets in binary systems," *The Astronomical Journal*, 1999.
- L. R. Doyle et al., "Kepler-16: A transiting circumbinary planet," *Science*, 2011.
- W. F. Welsh et al., "Transiting circumbinary planets Kepler-34 b and Kepler-35 b," *Nature*, 2012.
