# Directory and Module Map

Covers the top-level and `engine/` directory layout, the purpose of each module, and the key files to open first.
Read `01-overview.md` before this document for context on SimulationCraft's overall architecture.

---

## Repository layout

```
simc/
├── engine/          # C++ simulation engine (all game-logic code lives here)
├── qt/              # Qt graphical user interface
├── cli/             # CLI build target stub (qmake)
├── gui/             # Legacy GUI build target stub (qmake)
├── dbc_extract3/    # Python spell-data extraction toolchain
├── casc_extract/    # Python CASC archive extractor
├── profiles/        # Bundled .simc profile files for CI / pre-raid gear
├── ActionPriorityLists/  # Community APL files by spec
├── SpellDataDump/   # Reference spell-data snapshots
├── tests/           # Python regression/integration test runner
├── source_files/    # Build-system manifests (CMake / qmake / VS / Make)
├── cmake/           # CMake helper modules
├── doc/             # Doxygen configuration and this architecture guide
└── lib/             # (top-level) unused placeholder; vendored libs are in engine/lib/
```

---

## Engine sub-module table

| Path | Responsibility | Key entry-point file(s) |
|------|---------------|------------------------|
| `engine/sim` | Simulation core: the `sim_t` object drives the event loop, manages raid events, scale-factor sweeps, profilesets, plots, cooldowns, and progress reporting. | `engine/sim/sim.hpp:1` |
| `engine/player` | Player and actor model: `player_t` base class owns stat caches, action-priority list scheduling, gear/azerite data, assessors, and unique-gear dispatch. | `engine/player/player.hpp:1` |
| `engine/action` | Combat action hierarchy: `action_t`, `spell_t`, `attack_t`, `heal_t`, `absorb_t`; computes damage/healing state and drives DOT ticking. | `engine/action/action.hpp:1` |
| `engine/buff` | Buff and debuff system: `buff_t` base class plus typed sub-classes; manages stacks, durations, refresh/expire hooks, and stat-buff application. | `engine/buff/buff.hpp:1` |
| `engine/class_modules` | Per-class simulation logic: one `.cpp` per WoW class plus dedicated sub-modules for Monk, Paladin, Priest, and Warlock; all register via the `class_module.hpp` interface. | `engine/class_modules/class_module.hpp:1` |
| `engine/dbc` | Data-by-client (spell database): parsed WoW spell tables, hotfix overlays, azerite/embellishment data, talent trees, and spell-query expressions. | `engine/dbc/dbc.hpp:1` |
| `engine/item` | Item system: item loading, socket and equip effects, enchant application, and `special_effect_t` dispatch to unique-gear handlers. | `engine/item/item.hpp:1` |
| `engine/report` | Output report generation: HTML, text, and JSON formats; Highcharts chart data; gear-weight calculations and decorators. | `engine/report/reports.hpp:1` |
| `engine/interfaces` | External API clients: Battle.net/Armory (BCP API), HTTP layer with libcurl and WinINet backends, and a WoWHead connector. | `engine/interfaces/bcp_api.hpp:1` |
| `engine/util` | Shared utilities: RNG engine, `timespan_t`, I/O helpers, string/parse utilities, thread-concurrency primitives, and XML parsing. | `engine/util/util.hpp:1` |
| `engine/lib` | Vendored third-party libraries: `{fmt}` for formatting, `rapidjson`/`rapidxml` for JSON/XML, `utf8-cpp`, `tcb/span`. | `engine/lib/fmt/format.h:1` |

---

## Non-engine directory table

| Path | Responsibility | Key entry-point file(s) |
|------|---------------|------------------------|
| `dbc_extract3` | Python toolchain that reads WoW client data and generates C++ spell-data tables committed to `engine/dbc/`. Driven by `dbc_extract.py`. | `dbc_extract3/dbc_extract.py:1` |
| `casc_extract` | Python CASC (Content Addressable Storage Container) extractor; reads WoW's on-disk archive format to pull raw data files needed by `dbc_extract3`. | `casc_extract/casc_extract.py:1` |
| `cli` | qmake project stub for building the `simc` command-line executable. Actual source files are enumerated in `source_files/` and live in `engine/`. | `cli/cli.pro:1` |
| `gui` | qmake project stub for the legacy `SimulationCraft` GUI build target (largely unmaintained). | `gui/gui.pro:1` |
| `qt` | Qt-based graphical user interface: main window, import, simulation, and results tabs. Active GUI front-end for desktop releases. | `qt/MainWindow.hpp:1` |
| `profiles` | Bundled `.simc` profile files for CI regression runs, pre-raid sets, and mid-tier gear snapshots. `CI.simc` is the canonical smoke-test profile. | `profiles/CI.simc:1` |
| `ActionPriorityLists` | Community-maintained APL files organised by spec; serve as the reference input for theorycrafters and CI profile generators. | `ActionPriorityLists/README.md:1` |
| `SpellDataDump` | Reference text snapshots of exported spell data used for diffing changes across WoW patch cycles. | `SpellDataDump/README.md:1` |
| `tests` | Python-driven regression and integration test runner; `run.py` orchestrates simc binary invocations and diff-checks output. | `tests/run.py:1` |
| `source_files` | Build-system manifests listing every engine `.cpp`/`.hpp` for CMake, qmake, Visual Studio, and GNU Make. `synchronize.py` keeps them in sync. | `source_files/synchronize.py:1` |
| `cmake` | CMake helper modules: CPack packaging (`package.cmake`) and Windows Qt deployment (`windeployqt.cmake`). | `cmake/package.cmake:1` |

---

## Top-level files worth knowing

- `engine/sc_main.cpp:1` — program entry point; parses command-line options, constructs `sim_t`, and runs the simulation.
- `engine/simulationcraft.hpp:1` — umbrella include that pulls in all engine headers; useful for understanding the full public API surface.
- `CMakeLists.txt:1` — top-level CMake build definition; add new source files here (or in `source_files/`) and set compile features.
