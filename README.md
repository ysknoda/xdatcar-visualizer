# XDATCAR Visualizer

> **Beta release.** This project is feature-complete for its intended
> scope but still being validated; bugs may still be found and fixed
> before a stable release. Use in production with caution, and pin the
> exact version you tested against.

A single self-contained HTML file that loads a VASP `XDATCAR` trajectory and
renders atomic motion and cell deformation as an interactive 3D animation,
alongside quantitative analysis panels (cell parameters, fractional
coordinates, MSD, radial distribution function, bond-length and bond-angle
distributions). There is no build step, no server, and no installation: open
the file in a browser and drop an `XDATCAR` onto it.

## Features

- **3D trajectory animation** — atoms and the unit cell, with playback
  controls (play/pause, step, jump to first/last, frame slider, 1-1000 FPS).
- **Periodic bonds and replica atoms** — bonds are detected across periodic
  boundaries; a bonded neighbour outside the cell is drawn as a translucent
  ghost image, with a configurable replica margin.
- **Coordination polyhedra** — per-element central atom and ligand selection,
  with adjustable colour and opacity.
- **Picking and measurement** — click atoms to measure distance, angle, and
  dihedral (up to 4 atoms), plus coordination-number-based highlight rules.
- **Display controls** — per-atom and per-element colours, atom labels, atom
  and bond scaling, an opaque-region filter, three axis-display modes
  (off / overlay / corner inset), orthographic and perspective projection,
  and seven background themes.
- **Analysis tabs** — cell volume and lattice constants, per-atom fractional
  coordinates, mean squared displacement, radial distribution function g(r),
  and bond-length / bond-angle histograms.
- **Export** — PNG screenshot, CSV for every analysis panel, and video
  recording of the animation (MP4 where supported, WebM otherwise).

## Use it online

Nothing to install and nothing to set up — the app runs entirely in your
browser:

### **https://ysknoda.github.io/xdatcar-visualizer/**

Open that URL and drop an `XDATCAR` file onto it. The trajectory is parsed
locally by your own browser and is never uploaded anywhere; there is no server
side to this tool at all.

**An internet connection is required.** React 18, Three.js r128, Recharts,
Babel Standalone, and prop-types are fetched from a CDN rather than bundled,
which is what keeps this a single readable file.

### Running a pinned version

The hosted page always tracks the latest commit. When you need a fixed version
instead -- citing the exact build used for a figure, or archiving it alongside
a dataset -- download a tagged release and open it locally:

```bash
curl -LO https://github.com/ysknoda/xdatcar-visualizer/releases/download/v0.1.0b1/xdatcar_visualizer.html
```

A CDN connection is still required when the downloaded file is opened. Note
that this URL pins the tag explicitly: GitHub's `releases/latest/download/...`
shortcut deliberately skips pre-releases, so it will not resolve until a
stable (non-beta) release exists.

## Quick start

1. Open <https://ysknoda.github.io/xdatcar-visualizer/> in a browser.
2. Drag an `XDATCAR` file onto the drop zone (or click to browse).
3. Use the **Structure 3D** tab to animate the trajectory:
   drag to rotate, right-drag or Ctrl+drag to pan, scroll to zoom.
4. Enable **Bonds** and pick element pairs under "Bond rules" — bonds are off
   until at least one pair is enabled.
5. Switch tabs for quantitative analysis, and use **Export CSV** on any panel.

## Input format

Standard VASP `XDATCAR`:

- **Fixed-cell (NVT)** — one global header, then `Direct configuration=` blocks.
- **Variable-cell (NPT)** — a repeated header before every frame; the per-frame
  lattice is picked up automatically and used for volume, MSD, and g(r).
- **VASP 5/6 headers** (with an element-symbol line) and **VASP 4 headers**
  (counts only) are both accepted; VASP 4 files get placeholder element labels
  `A, B, C, ...` and a warning banner, since the symbols are not in the file.
- A **negative scaling factor** is interpreted the way VASP defines it: as the
  target cell volume, not as a multiplier.

A truncated final frame is dropped with a warning rather than silently, and a
malformed header produces a specific error message instead of a blank screen.

## Analysis notes

- **MSD** is computed from a minimum-image-unwrapped fractional trajectory,
  converting each step's displacement with that step's own lattice, so it is
  correct for variable-cell (NPT) runs and immune to periodic-boundary jumps.
- **g(r)** is normalised so that it tends to 1 at large r for a structureless
  system, including for the small unit cells this tool is usually pointed at
  (the finite-size self-exclusion term matters at the 10% level below ~10
  atoms). Partial g(r) is available for every element pair.
- **Neighbour search** uses a spatial cell list in fractional space, so it
  works unchanged for triclinic cells and covers as many periodic images as
  the cutoff actually requires.

### Known limitations

- MSD does not remove centre-of-mass drift; a drifting COM inflates it.
- g(r) beyond half the shortest cell width includes periodic self-correlation,
  as it does for any single-cell radial distribution function.
- The video export container is browser-dependent — the file extension follows
  whatever `MediaRecorder` actually selected, so it may be `.webm`, not `.mp4`.

## Testing

There is no automated test suite. Correctness has been verified interactively
in a browser against known references:

- **Rocksalt NaCl (a = 5.64 Å)** — neighbour shells recovered exactly at
  a/2, a/√2, a√3/2 and a (2.820 Å ×6, 3.988 Å ×12, 4.884 Å ×8, 5.640 Å ×6),
  with coordination numbers back-integrated from g(r) giving 6, 12, and 12.
- **Ideal-gas limit** — g(r) → 1 at large r for the total and every pair.
- **Parser** — malformed, truncated, VASP 4, and negative-scaling-factor inputs.

## License

MIT -- see [LICENSE](LICENSE).
