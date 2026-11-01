# Fractal Darkroom 🌀

[![CI](https://github.com/Prekzursil/fractal-darkroom/actions/workflows/ci.yml/badge.svg)](https://github.com/Prekzursil/fractal-darkroom/actions/workflows/ci.yml)
[![Pages](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://prekzursil.github.io/fractal-darkroom/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Coverage](https://img.shields.io/badge/python%20coverage-100%25%20of%20measured%20modules-brightgreen)](https://github.com/Prekzursil/fractal-darkroom/actions/workflows/ci.yml)

**Long-exposure photographs of mathematical escape — developed in a darkroom made of counting.** A Buddhabrot family renderer:
instead of asking *"does this point escape?"*, it asks *"where does it travel on the way out?"*
— and plots millions of escaping orbits as a density image. Plus 17 sibling fractals grown
from the same idea: Antibuddhabrot, Nebulabrot, power mandalas (z³/z⁴/z⁵), Burning Ship,
Tricorn, Celtic, Buffalo, sin/cos entire-function brots, Julia-brot, and a family of
custom cosine/tan/sinh/exp mutations.

![Buddhabrot](docs/assets/hero-buddha.jpg)

*Every image on this page is a 2-D histogram of orbit visitation counts — no raytracing,
no painting, just `z ↦ z² + c`, iterated over and over.*

## Try it in the browser

The [**Playground**](https://prekzursil.github.io/fractal-darkroom/playground.html) runs live in
your browser (no install, no server): animate single orbits Alan-Becker-style, watch a
Buddhabrot develop like a photograph in real time, zoom a Mandelbrot explorer, and write
your own iteration formulas in a sandbox. The [Math guide](https://prekzursil.github.io/fractal-darkroom/math.html)
and [Code walkthrough](https://prekzursil.github.io/fractal-darkroom/code.html) explain everything
in depth, with interactive diagrams.

## Quickstart (Python)

```bash
pip install -r requirements.txt

# the classic 2048² Buddhabrot (74M samples)
python buddhabrot.py render

# your first mutation: x' = cos(x²) − y, y' = 2xy  (38M samples)
python cosbrot.py render

# every farm map (the 17 stock maps plus any in usermaps/), sharded across 4 processes
python brot_farm.py --shard 0 --shards 4 &
python brot_farm.py --shard 1 --shards 4 &
python brot_farm.py --shard 2 --shards 4 &
python brot_farm.py --shard 3 --shards 4 &

# add samples drawn from a cropped window to a cached histogram
# (the result mixes two sampling windows; see topup.py)
python topup.py z3 1728 12000000 8000000 5000000
```

Outputs land in `out/` as PNG + typography poster. Histograms are cached as `.npy`, so
re-toning (`python buddhabrot.py tone --gamma 1.2`) is instant.

## The idea in one paragraph

Sample a random point `c`. Iterate `zₙ₊₁ = zₙ² + c` starting at `z₀ = 0`. If the orbit
stays bounded forever, `c` is a *prisoner* — it belongs to the Mandelbrot set and its
orbit never draws anything. If the orbit escapes past radius 2, it has traced a path
through the plane first — and *that path* is the picture. Plot every point every escaping
orbit visits into a 2-D histogram, repeat for millions of samples, and the Buddha appears.
Split the work into three channels by iteration depth (80 / 400 / 1500) and shallow
orbits glow blue while deep ones burn red.

## Maps included

| Family | Maps | Signature |
|---|---|---|
| Classic | Buddhabrot, Antibuddhabrot (prisoner orbits), Nebulabrot (caps 50/500/5000), Julia-brot (fixed c = −0.7+0.27i) | the figure |
| Powers | z³+c, z⁴+c, z⁵+c | (n−1)-fold mandalas |
| Folds | Burning Ship, Tricorn, Celtic, Buffalo | the \|2xy\| fold breaks mirror symmetry (Burning Ship, Buffalo); Tricorn and Celtic keep it |
| Entire functions | sin z + c, cos z + c | prisoners confined to a horizontal band |
| Mutations | cos(x²)−y, cos(x²−y²), cos(2xy), tan, sinh, exp variants | asymmetric, chaotic, occasionally pathological |

## Architecture (short version)

Two-phase vectorized Monte Carlo, pure numpy, no dependencies beyond numpy/Pillow:

1. **Phase 1 — classify**: batch 3M samples, iterate with periodic array compaction, mark who escapes within the channel's cap. Analytic pre-culling of the main cardioid + period-2 bulb drops provably-never-escaping samples before iteration: about 11.9% of uniform draws in the classic window (the two regions cover 7π/16 ≈ 1.374 of its 3.4 × 3.4 area).
2. **Phase 2 — replay**: re-iterate only the escapees, scatter every pre-escape orbit point into a `bincount`-fed histogram.
3. **Tone map**: log-compress, anchor black/white points at histogram percentiles, gamma, per-channel gains, vibrance.

Full reasoning — why two phases, why `bincount` over `np.add.at`, why float64, why
percentile tone anchors, sample budgets, and the alternatives not taken — lives in the
[Code walkthrough](https://prekzursil.github.io/fractal-darkroom/code.html).

## Tests & CI

```bash
python -m pytest --cov       # the whole suite, with 100% line+branch coverage of the
                             # Python modules listed in .coveragerc (enforced by CI;
                             # usermaps/, scripts/verify_site.py and the browser
                             # JavaScript are not measured): step rules vs complex
                             # arithmetic, escape classification vs scalar reference,
                             # tone monotonicity, histogram bounds, culling soundness,
                             # seed determinism, mirror-symmetry law, CLI entry points,
                             # caching paths, the topup tool, the S0/S1/S2 samplers,
                             # the build suite and its Python<->JavaScript map-name
                             # parity, the collection index and APRS validator, and
                             # a tiny end-to-end render
bash scripts/preflight.sh    # the lint, type and coverage lanes, locally
```

CI (`.github/workflows/ci.yml`) runs the suite on Python 3.11–3.14 on Ubuntu and Windows,
plus the Playwright site suite; `ci-green` aggregates them. `.github/workflows/quality.yml`
runs the lean quality gate (lint/format, types, coverage, SAST, secrets, dependencies) and
CodeQL scans pushes and pull requests; `ci-green`, `quality` and CodeQL are required to
merge into `main`. Pages deploys `docs/` on every push to `main` that touches `docs/`
(`.github/workflows/pages.yml`).

## License

MIT — see [LICENSE](LICENSE).
