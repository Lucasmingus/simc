# SimC Onboarding Architecture Docs — Design

**Date:** 2026-06-29
**Status:** Approved (pending spec review)
**Repo:** Lucasmingus/simc fork (default branch `midnight`), work branch `docs/onboarding-architecture`

## Goal

Produce a set of hand-curated, Mermaid-in-Markdown documents that let a developer
navigate SimulationCraft without prior inherent knowledge. The output is an
**onboarding overview**: where things live, how the major pieces fit together, and
how a single simulation flows end-to-end.

This is a **documentation-only** effort. No engine code changes.

## Context

SimulationCraft is an event-driven C++ combat simulator (~343k LOC in `engine/`
alone), plus:

- `dbc_extract3/` — Python tooling that extracts WoW client data into generated C++.
- `engine/dbc/` — the generated game-data layer the sim consumes.
- `cli/` + `engine/sc_main.cpp` — command-line front-end and entry point.
- `gui/`, `qt/` — the (largely unmaintained) Qt GUI.
- `profiles/`, `ActionPriorityLists/` — input profiles and action priority lists (APLs).

The engine subsystems (under `engine/`):

| Folder | Role |
|---|---|
| `sim/` | Event-driven core: `sim_t`, event scheduler, options, raid events, profilesets |
| `player/` | Actor model: `player_t`, stats, resources, scaling, unique gear |
| `action/` | Combat action primitives (`action_t` and derived spell/attack bases) |
| `buff/` | Buff/debuff model |
| `class_modules/` | Per-spec implementations — ~60% of the engine, the bulk of real work |
| `dbc/` | Generated game-data access layer |
| `item/` | Items/gear |
| `report/` | HTML/JSON/text report generation |
| `interfaces/` | External data (Battle.net armory, JSON import, etc.) |
| `util/` | Shared utilities |

## Deliverables

All under a new `doc/architecture/` folder (alongside the existing `doc/` Doxygen
setup). An `index.md` links the set.

1. **`01-overview.md` — Top-level architecture map.**
   Mermaid component diagram of the major pieces (engine, dbc data layer,
   dbc_extract3 tooling, cli/gui front-ends, profiles/APL inputs) and how they
   connect. The 30,000-ft view.

2. **`02-engine-core.md` — Engine core model.**
   Mermaid `classDiagram` of the heart: `sim_t` ↔ `player_t` ↔ `action_t` / `buff_t`,
   plus the event scheduler (`event_t` / `event_manager_t`). Each class gets a
   one-paragraph "what it owns / its job" with clickable `file:line` pointers to the
   real headers.

3. **`03-simulation-flow.md` — "How a sim runs" data flow.**
   Mermaid `sequenceDiagram` + flowchart tracing one run end-to-end:
   `sc_main.cpp` → option/profile parsing → build raid/players → APL setup →
   event loop → N iterations → report generation. The control-flow spine.

4. **`04-directory-map.md` — Annotated "where things live" guide.**
   A table of every significant folder → responsibility + key entry-point file(s),
   covering both `engine/` subsystems and the surrounding tooling
   (`dbc_extract3`, `cli`, `gui`, `qt`, `profiles`).

5. **`05-class-module-pattern.md` — The class-module template.**
   All ~13 class modules follow one pattern (a `*_t` derived from `player_t`,
   actions derived from spell/attack bases, buffs, an APL). One doc that explains the
   pattern using a representative module so a reader can navigate *any* class file.

6. **`06-game-data-pipeline.md` — Game-data pipeline.**
   How `dbc_extract3` (Python) turns WoW client data into generated C++ in
   `engine/dbc/`, and how the sim consumes it. The most "alien" part for newcomers.

## Conventions

- **Diagrams:** Mermaid only — `flowchart`, `classDiagram`, `sequenceDiagram`.
  Every diagram is followed by prose explanation.
- **References:** real `path/to/file.ext:line` pointers (clickable), tied to the
  actual code, not invented names.
- **Accuracy:** read the actual code (entry points, key headers,
  `class_modules/class_module.hpp`, one representative class module, the
  `dbc_extract3` entry script) before writing each doc. Where something is uncertain,
  flag it rather than guess.
- **Each doc is self-contained** with a short "what this covers / what to read next"
  header, and `index.md` ties them together in reading order.

## Method / unit boundaries

Each document is an independent unit with one clear purpose; they reference each
other but can be read and verified on their own. To get breadth without reading all
343k lines:

- Map each subsystem via targeted reads of its headers and entry points (optionally
  using parallel read-only explore agents), then synthesize into the relevant doc.
- Depth is "enough to navigate confidently," not exhaustive call graphs.

## Verification

- Each Mermaid block is syntax-checked (renders without error).
- Every `file:line` reference is confirmed against the real file before inclusion.
- A final pass confirms the six docs + index are internally consistent (no
  contradicting names/roles) and that reading order in `index.md` is coherent.

## Out of scope

- Exhaustive per-function call graphs.
- Documenting every class module individually (we document the *pattern*, with one
  worked example).
- Any change to engine/tooling code.
- Auto-generated (Doxygen/Graphviz) output.

## Open questions

None outstanding. Add-ons 5 and 6 are confirmed in scope.
