# DLSCMS — Defence Logistics and Supply Chain Management System

Official implementation of **"Design and Evaluation of an Object-Oriented Defence Logistics and Supply Chain Management System"**, an Object Oriented Programming course project at SVKM's NMIMS MPSTME, Mumbai.

![Results](paper/figures/fig_baseline_comparison.png)

## Abstract

Defence and humanitarian logistics networks suffer from the same four problems: some bases run short while others hold excess stock, deliveries are late, and nobody has a single, trustworthy view of what is where. *DLSCMS* is a Java 17 application that manages bases, supplies, inventories, vehicles, supply requests and deliveries, and automatically plans transfers of surplus stock between bases. The design is deliberately object-oriented: an abstract `Vehicle` hierarchy, interfaces for tracking and persistence, registries (generic) and repositories, a checked exception hierarchy, and two Strategy interfaces that make donor selection and vehicle selection swappable at run time. Because real defence data was unobtainable, seeded datasets (synthetic) were generated at three scales (40, 200 and 1,000 requests) and replayed in an hour-by-hour simulator under **12** strategy combinations, including random baselines, with **30** seeds each (1,080 runs). 
Coordinated strategies cut mean lead time from **84.5 h** to **34.8 h** and transport distance by **42%** against an uncoordinated random baseline on the largest dataset. About 2% of requests remain unfulfilled; this is traced to donors' reorder floors, a small fuel-truck fleet and the finite simulation horizon. The system is covered by 119 automated tests and replays the largest scenario in about **one second**.

**Authors:** Vedant Pathak, Dhairya Shah, Kanishak Thakran

**Note:** All data (bases, supplies, inventory, vehicles, scenario_requests, requests, deliveries, meta) is synthetic and fully reproducible, with base names taken from Indian city names. It comes from a seeded generator, so anyone can regenerate it, and no real defence information was involved. The generator guarantees that shortages can in principle be met, so the results describe the model, not real operations. The datasets are included in `data/`, and `scripts/gen_data.py` can regenerate them.

## Key Results (Mean ± Std over seeds 1–30)

Vehicle strategy FASTEST for the coordinated rows. The baseline picks both donor and vehicle at random. Distance is the total of all delivered trips in thousand km.

| Scale | Strategy | Lead time (h) | Fulfilled (%) | Distance (thousand km) |
|-------|----------|---------------|---------------|------------------------|
| Small (40 requests) | Nearest | 24.3 ± 6.9 | 97.5 ± 3.2 | 36 ± 6 |
| | **Balanced** | **22.9 ± 6.9** | 98.2 ± 3.0 | 38 ± 6 |
| | Surplus | 26.5 ± 6.3 | 98.2 ± 3.0 | 46 ± 6 |
| | Random baseline | 46.2 ± 7.1 | 97.1 ± 3.4 | 56 ± 7 |
| Medium (200 requests) | Nearest | 32.2 ± 7.2 | 98.7 ± 2.0 | 238 ± 30 |
| | **Balanced** | **29.2 ± 6.9** | 99.1 ± 1.3 | 248 ± 29 |
| | Surplus | 36.3 ± 6.2 | 99.0 ± 1.2 | 315 ± 23 |
| | Random baseline | 73.9 ± 6.6 | 96.6 ± 2.7 | 382 ± 22 |
| Large (1,000 requests) | Nearest | 34.8 ± 8.2 | 98.3 ± 1.6 | 1,188 ± 99 |
| | **Balanced** | **28.4 ± 7.7** | **99.3 ± 1.4** | 1,219 ± 107 |
| | Surplus | 40.0 ± 8.5 | 98.8 ± 1.7 | 1,687 ± 110 |
| | Random baseline | 84.5 ± 8.2 | 95.3 ± 1.3 | 2,043 ± 163 |

All values are computed from `results/raw/runs.csv` (1,080 runs) and agree with `results/summary.csv`. Significance uses paired Wilcoxon signed-rank tests with Holm-Bonferroni correction over 93 tests (74 remain significant). 
**Details:** `results/stats.txt` and `docs/findings.md`.

## Experiment Design

| Factor | Levels |
|--------|--------|
| Dataset scale | small (40 requests), medium (200), large (1,000) |
| Dataset seed | 1 to 30 (each seed regenerates bases, stock, fleet and request stream) |
| Donor strategy | Nearest, Surplus, Balanced, Random baseline |
| Vehicle strategy | Smallest, Fastest, Random baseline |
| **Total runs** | 3 × 30 × 4 × 3 = **1,080** |

## Unfulfilled Requests

About 2% of requests stay open at the end of a run (2.2% small, 1.7% medium, 2.1% large, over all strategies). Each open request is classified by the simulator as stock-limited (donors keep their reorder level), waiting for a vehicle, or still in flight when the 168-hour horizon ends. On medium and large, almost all requests waiting for a vehicle or in flight are FUEL requests (fuel trucks are about 15% of the fleet and the slowest vehicle). 
**Details:** `docs/findings.md` and `results/unfulfilled_summary.txt`.

## Repository Structure

```
DLSCMS_using_ObjectOrientedProgramming/
├── src/
│   ├── main/java/com/dlscms/   # Source: model, model/vehicle, service, strategy,
│   │                           #   persistence, ui, util, exception, sim
│   └── test/java/com/dlscms/   # 21 JUnit 4 test classes (119 tests)
├── data/
│   ├── small/                  # Synthetic CSV datasets (seed 42): 6 bases, 40 requests
│   ├── medium/                 # 10 bases, 200 requests
│   ├── large/                  # 12 bases, 1,000 requests
│   ├── template/               # Empty files for loading your own data
│   └── example_sme/            # Tiny small/medium-business example
├── scripts/
│   ├── build.sh / build.bat    # Build jar, optionally run tests
│   ├── gen_data.py             # Seeded dataset generator
│   ├── validate_data.py        # Dataset validator
│   ├── run_experiments.sh      # Full experiment pipeline (30 seeds)
│   ├── analyze.py              # Statistics and figures
│   ├── check_repro.sh          # Determinism check
│   └── ...                     # Smoke test, manual verifier, diagnosis helpers
├── results/
│   ├── raw/runs.csv            # All 1,080 simulation runs
│   ├── summary.csv             # Means and confidence intervals
│   ├── stats.txt               # Paired tests with Holm correction
│   └── ...                     # Dataset summary, test matrix, unfulfilled diagnosis
├── paper/
│   ├── paper.md / .docx / .pdf # Research paper
│   └── figures/                # Result figures
├── slides/                     # Presentation (.pptx and .pdf)
├── docs/                       # Requirements, design, UML (docs/img), dataset notes,
│                               #   findings, user manual
├── lib/                        # JUnit and Hamcrest jars (tests only)
├── run.bat / run.sh            # Launchers (build the jar on first run)
```

## Environment Setup

The application needs **JDK 17 or newer**

```bash
java -version      # must report 17 or newer
```

Only the optional analysis step (`scripts/analyze.py`) needs Python 3 with `numpy`, `scipy` and `matplotlib` libraries.

## Quick Start

### Run the application

| Goal | Windows | Linux / macOS |
|---|---|---|
| Run the terminal | `run.bat` | `./run.sh` |
| No colours (old terminals) | `run.bat --plain` | `./run.sh --plain` |
| Scripted demo, then exit | `run.bat --demo` | `./run.sh --demo` |
| Other dataset | `run.bat --data data\medium` | `./run.sh --data data/medium` |
| Build and run all tests | `scripts\build.bat test` | `scripts/build.sh test` |

The launchers build `dist/dlscms.jar` automatically on first run. Type `HELP` inside the program for help.
**Details:** [docs/user_manual.md](docs/user_manual.md).

### Reproduce all results (30 seeds)

```bash
scripts/run_experiments.sh
```

This generates one dataset per seed, replays all 1,080 runs, and writes `results/raw/runs.csv`, `results/summary.csv`, `results/stats.txt` and the figures in `paper/figures/`. 
The full grid took about **6.5 minutes** in our run.

### Run individual steps

```bash
python3 scripts/gen_data.py --scale all --seed 42 --out data   # regenerate datasets
python3 scripts/validate_data.py                               # validate datasets
scripts/check_repro.sh                                         # check determinism
```

### If you want to try your own data

Fill in the CSV files in `data/template/` and follow [docs/bring_your_own_data.md](docs/bring_your_own_data.md).

## Design

Domain classes (`Base`, `Supply`, `InventoryItem`, `SupplyRequest`, `Delivery`, and a `Vehicle` hierarchy of `Truck`, `FuelTruck`, `MedicalVan`, `Helicopter`) are coordinated by a `LogisticsManager`. Donor-base and vehicle choice use the Strategy pattern and can be switched at run time. Persistence is generic (`CsvRepository<T extends Persistable>`). Errors are checked exceptions from a custom hierarchy and are shown as a red `ERR` line, never a stack trace.
**Details:** [docs/design.md](docs/design.md) and the UML diagrams in [docs/class_diagram.md](docs/class_diagram.md).

## Reproducibility

- Datasets are produced by a seeded generator: seed 42 for the bundled data, seeds 1 to 30 for the experiments. The same seed always gives byte-identical files.
- The random baselines use their own derived seeds, so every run is deterministic. `scripts/check_repro.sh` re-runs the grid twice and compares every column except timing.
- Only `wall_ms` and `used_mb` in `results/raw/runs.csv` vary between runs.
**Details:** [docs/dataset.md](docs/dataset.md).

## Citation

If you use this code or results, please cite:

```
@misc{pathak2026dlscms,
  title  = {Design and Evaluation of an Object-Oriented Defence Logistics and Supply Chain Management System},
  author = {Vedant},
  year   = {2026},
  url    = {https://github.com/withvedantpathak/DLSCMS_using_ObjectOrientedProgramming}
}
```

## Contact

Vedant Pathak — [@withvedantpathak](https://github.com/withvedantpathak) [with.vedantpathak@gmail.com]

SVKM's NMIMS MPSTME, Mumbai, India
