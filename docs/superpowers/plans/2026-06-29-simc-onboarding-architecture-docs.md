# SimC Onboarding Architecture Docs — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce six hand-curated Mermaid-in-Markdown onboarding documents (plus an index) under `doc/architecture/` that let a developer navigate SimulationCraft without prior knowledge.

**Architecture:** Each document is an independent unit synthesized from a fixed reading list of real source files. A reusable shell verifier checks two things per doc — every `path:line` reference resolves to a real file/line, and every Mermaid block is well-formed — so accuracy is enforced mechanically, not by eye. Docs only; no engine/tooling code changes.

**Tech Stack:** Markdown + Mermaid (`flowchart`, `classDiagram`, `sequenceDiagram`). Verifier is a Bash script run via Git Bash. Source under inspection is C++ (`engine/`) and Python (`dbc_extract3/`).

## Global Constraints

- **Repo root:** `C:\Users\noah\Projects\simc` (Git Bash path `/c/Users/noah/Projects/simc`). Run all `git`/verifier commands from there.
- **Branch:** `docs/onboarding-architecture` (already created off default branch `midnight`).
- **Output folder:** `doc/architecture/` (singular `doc`, matching the repo's existing Doxygen folder). Do **not** create `docs/architecture`.
- **Verifier location:** scratchpad — `C:\Users\noah\AppData\Local\Temp\claude\C--Users-noah-Projects\90a664cb-8802-47f9-b676-cf0bf77ce02b\scratchpad\verify_doc.sh`. Keep it out of the repo (docs-only deliverable).
- **Every `file:line` reference must be confirmed against the real file** (the verifier enforces this). Reference real symbol/class names only — never invented ones.
- **Each diagram is followed by prose.** No bare diagrams.
- **Each doc starts** with a 2-line header: what it covers / what to read next.
- **Commit per task**, message prefix `docs:`.

---

### Task 1: Verifier + Directory map (`04-directory-map.md`)

Builds the verification harness (the "test") first, proves it catches a bad reference, then writes the most factual doc — the directory map — and makes it pass.

**Files:**
- Create: `<scratchpad>/verify_doc.sh` (verification harness; not committed)
- Create: `doc/architecture/04-directory-map.md`

**Interfaces:**
- Produces: `verify_doc.sh <repo_root> <markdown_file>` — exit 0 if all `path:line` refs resolve and all mermaid blocks are well-formed; non-zero + diagnostics otherwise. Every later task consumes this.

- [ ] **Step 1: Write the verifier (the test harness)**

Write `<scratchpad>/verify_doc.sh`:

```bash
#!/usr/bin/env bash
# verify_doc.sh <repo_root> <markdown_file>
# 1) every path:line(-range) reference resolves to a real file + in-range line
# 2) every ```mermaid block starts with a known diagram keyword; fences balanced
set -u
root="$1"; md="$2"; status=0

# --- reference check ---
tmp="$(mktemp)"
grep -oE '[A-Za-z0-9_./-]+\.[A-Za-z0-9_]+:[0-9]+(-[0-9]+)?' "$md" | sort -u > "$tmp"
while IFS= read -r ref; do
  [ -z "$ref" ] && continue
  path="${ref%%:*}"; start="${ref#*:}"; start="${start%%-*}"
  file="$root/$path"
  if [ ! -f "$file" ]; then echo "MISSING FILE: $ref"; status=1; continue; fi
  total=$(wc -l < "$file")
  if [ "$start" -gt "$total" ]; then echo "LINE OUT OF RANGE: $ref (file has $total lines)"; status=1; fi
done < "$tmp"; rm -f "$tmp"

# --- mermaid block check ---
fences=$(grep -cE '^```' "$md")
if [ $(( fences % 2 )) -ne 0 ]; then echo "UNBALANCED CODE FENCES"; status=1; fi
awk '
  /^```mermaid/ {inblock=1; next}
  inblock && /^```/ {inblock=0; next}
  inblock {
    if ($0 ~ /^[[:space:]]*$/) next
    if ($0 !~ /^[[:space:]]*(flowchart|graph|classDiagram|sequenceDiagram|stateDiagram|erDiagram)/) {
      print "BAD MERMAID HEADER: " $0; bad=1
    }
    inblock=0
  }
  END {exit bad?1:0}
' "$md" || status=1

if [ "$status" -eq 0 ]; then echo "OK: $md (refs + mermaid valid)"; fi
exit $status
```

- [ ] **Step 2: Prove the verifier fails on a bad reference**

Run:
```bash
printf '`engine/sim/sim.hpp:99999999`\n\n```mermaid\nflowchart TD\n A-->B\n```\n' > /tmp/_vbad.md
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc /tmp/_vbad.md; echo "exit=$?"
```
Expected: prints `LINE OUT OF RANGE: engine/sim/sim.hpp:99999999 ...` and `exit=1`.

- [ ] **Step 3: Read the source to map directories**

Read/confirm structure (already partly known):
```bash
cd /c/Users/noah/Projects/simc
ls -A | grep -v '^.git$'
ls engine/
for d in engine/*/; do echo "== $d =="; ls "$d" | head -8; done
ls dbc_extract3/ casc_extract/ cli/ gui/ qt/ | head -60
ls profiles/ | head; ls ActionPriorityLists/ | head
```
Also read the first 30 lines of `engine/sc_main.cpp` and `README.md` for framing.

- [ ] **Step 4: Write `doc/architecture/04-directory-map.md`**

Required contents:
- 2-line header (what it covers / read `01-overview.md` first).
- A table with columns **Path | Responsibility | Key entry-point file(s)** covering at minimum: `engine/sim`, `engine/player`, `engine/action`, `engine/buff`, `engine/class_modules`, `engine/dbc`, `engine/item`, `engine/report`, `engine/interfaces`, `engine/util`, `engine/lib`, `dbc_extract3`, `casc_extract`, `cli`, `gui`, `qt`, `profiles`, `ActionPriorityLists`, `SpellDataDump`, `tests`, `source_files`, `cmake`.
- Each "key entry-point file" cited as a real `path/file.ext:1` reference (line 1 is fine for "this file exists / start here").
- A short "Top-level files worth knowing" list: `engine/sc_main.cpp:1`, `engine/simulationcraft.hpp:1`, `CMakeLists.txt:1`.

- [ ] **Step 5: Verify the doc**

Run:
```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/04-directory-map.md; echo "exit=$?"
```
Expected: `OK: ... (refs + mermaid valid)` and `exit=0`. Fix any flagged ref and re-run until clean.

- [ ] **Step 6: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/04-directory-map.md
git commit -m "docs: add directory/module map for onboarding"
```

---

### Task 2: Top-level architecture map (`01-overview.md`)

**Files:**
- Create: `doc/architecture/01-overview.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1).

- [ ] **Step 1: Read the source for component relationships**

```bash
cd /c/Users/noah/Projects/simc
sed -n '1,40p' README.md
sed -n '1,60p' engine/sc_main.cpp        # includes show which subsystems main drives
sed -n '1,40p' CMakeLists.txt
ls source_files/ 2>/dev/null              # how the build aggregates engine sources
sed -n '1,40p' dbc_extract3/dbc_extract.py
```
Confirm: `dbc_extract3` generates data consumed by `engine/dbc`; `cli` + `gui`/`qt` drive `engine`; `profiles`/`ActionPriorityLists` are inputs; `report` is output.

- [ ] **Step 2: Write `doc/architecture/01-overview.md`**

Required contents:
- 2-line header (30,000-ft view / read `04-directory-map.md` for folder detail, `02-engine-core.md` next).
- One Mermaid `flowchart` with these nodes and edges (label edges with the relationship):
  - `dbc_extract3 (Python)` → generates → `engine/dbc (generated data)`
  - `WoW client / casc_extract` → feeds → `dbc_extract3`
  - `profiles/ + ActionPriorityLists/` → input → `engine`
  - `cli (simc)` and `gui/qt (SimulationCraft)` → drive → `engine`
  - `engine` (subgraph: `sim`, `player`, `action`/`buff`, `class_modules`, `report`) consumes `engine/dbc`
  - `engine/report` → output → `HTML/JSON/text`
- Prose paragraph per component (engine, dbc data layer, dbc_extract3, front-ends, profiles/APL, report) with a real entry-point `file:line` reference each (e.g. `engine/sc_main.cpp:1`, `engine/dbc/dbc.hpp:1`, `dbc_extract3/dbc_extract.py:1`).

- [ ] **Step 3: Verify**

```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/01-overview.md; echo "exit=$?"
```
Expected: `OK`, `exit=0`.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/01-overview.md
git commit -m "docs: add top-level architecture overview"
```

---

### Task 3: Engine core model (`02-engine-core.md`)

**Files:**
- Create: `doc/architecture/02-engine-core.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1).

- [ ] **Step 1: Read the core headers**

Read enough of each to state its responsibility and find a class-declaration line:
```bash
cd /c/Users/noah/Projects/simc
grep -n 'struct sim_t' engine/sim/sim.hpp | head
grep -n 'struct player_t' engine/player/player.hpp | head
grep -n 'struct action_t' engine/action/action.hpp | head
grep -n 'struct buff_t' engine/buff/buff.hpp | head
grep -n 'struct event_t' engine/sim/event.hpp | head
grep -n 'struct event_manager_t\|class event_manager_t' engine/sim/event_manager.hpp | head
```
Then read ~80 lines around each declaration to capture the key owned members (e.g. `sim_t` holds `player_list`/`target_list` and the `event_mgr`; `player_t` holds `action_list`/`buff_list`/resources; `action_t`/`buff_t` hold back-pointers to `player`/`sim`).

- [ ] **Step 2: Write `doc/architecture/02-engine-core.md`**

Required contents:
- 2-line header (the core object model / read `03-simulation-flow.md` next).
- One Mermaid `classDiagram` showing: `sim_t "1" o-- "*" player_t`, `player_t "1" o-- "*" action_t`, `player_t "1" o-- "*" buff_t`, `sim_t "1" *-- "1" event_manager_t`, `event_manager_t "1" o-- "*" event_t`, and back-references `action_t --> player_t`, `buff_t --> player_t`. Include 3-5 key members/methods per class (real names from Step 1).
- A "responsibilities" subsection: one paragraph per class (`sim_t`, `player_t`, `action_t`, `buff_t`, `event_t`/`event_manager_t`) each with a real `file:line` reference to its declaration (from `grep -n` in Step 1).

- [ ] **Step 3: Verify**

```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/02-engine-core.md; echo "exit=$?"
```
Expected: `OK`, `exit=0`.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/02-engine-core.md
git commit -m "docs: add engine core object model"
```

---

### Task 4: Simulation flow (`03-simulation-flow.md`)

**Files:**
- Create: `doc/architecture/03-simulation-flow.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1); references classes documented in `02-engine-core.md`.

- [ ] **Step 1: Trace the run path in source**

```bash
cd /c/Users/noah/Projects/simc
grep -n 'int main' engine/sc_main.cpp
grep -n 'sim_signal_handler\|parse\|::execute\|->execute\|combat\|analyze\|print_\|report::' engine/sc_main.cpp | head -40
grep -n '::execute\|::iterate\|::combat\|::analyze\|combat_begin\|combat_end' engine/sim/sim.cpp | head -40
grep -n 'loop\|next_event\|execute\|while' engine/sim/event_manager.cpp | head -30
grep -n 'parse\|sim_control_t' engine/sim/sim_control.hpp engine/sim/option.cpp | head -20
grep -n '' engine/report/reports.hpp | head -5
```
Capture real line numbers for: `main`, option/sim_control parse, sim init/build players, `sim_t::execute`, the iteration/event loop, analyze, report entry.

- [ ] **Step 2: Write `doc/architecture/03-simulation-flow.md`**

Required contents:
- 2-line header (end-to-end control flow / assumes `02-engine-core.md`).
- One Mermaid `sequenceDiagram` with participants `main`, `sim_control_t`, `sim_t`, `player_t`, `event_manager_t`, `report`, showing: parse options/profile → build raid & players → init APL → `execute()` → loop over N iterations (`combat_begin` → drain events → `combat_end` → reset) → analyze → generate report.
- One Mermaid `flowchart` summarizing the phases (Parse → Build → Iterate×N → Analyze → Report).
- Prose walking each phase with real `file:line` references captured in Step 1 (`engine/sc_main.cpp:<main line>`, `engine/sim/sim.cpp:<execute line>`, `engine/sim/event_manager.cpp:<loop line>`, etc.).

- [ ] **Step 3: Verify**

```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/03-simulation-flow.md; echo "exit=$?"
```
Expected: `OK`, `exit=0`.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/03-simulation-flow.md
git commit -m "docs: add end-to-end simulation flow"
```

---

### Task 5: Class-module pattern (`05-class-module-pattern.md`)

Uses `sc_mage.cpp` as the representative single-file module (well-known, moderate size, not split into a subfolder).

**Files:**
- Create: `doc/architecture/05-class-module-pattern.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1); references `action_t`/`player_t`/`buff_t` from `02-engine-core.md`.

- [ ] **Step 1: Read the registration interface and the example module**

```bash
cd /c/Users/noah/Projects/simc
sed -n '1,120p' engine/class_modules/class_module.hpp
grep -n 'namespace mage\|struct mage_t\|struct mage_spell_t\|struct .*_buff_t\|create_action\|init_spells\|init_action_list\|module_t\|create_player' engine/class_modules/sc_mage.cpp | head -50
```
Identify, with real line numbers in `sc_mage.cpp`: the spec class `mage_t : public player_t`, an action base like `mage_spell_t`, a representative buff, `create_action()`, `init_spells()`, `init_action_list()`/APL hookup, and the module registration struct. In `class_module.hpp`, find the `module_t`/`player_module_t` interface line.

- [ ] **Step 2: Write `doc/architecture/05-class-module-pattern.md`**

Required contents:
- 2-line header (the template every class module follows / assumes `02-engine-core.md`).
- A Mermaid `classDiagram` (or annotated `flowchart`) showing the pattern with `mage_t` as the example: `player_t <|-- mage_t`, `action_t <|-- spell_t <|-- mage_spell_t`, `buff_t <|-- <example>_buff_t`, and `module_t` registering `mage_t`.
- A "the recipe" numbered list mapping each pattern element to a real `engine/class_modules/sc_mage.cpp:<line>` reference: spec struct, action base, buffs, `create_action`, `init_spells`, `init_action_list`/APL, registration. Plus the registration interface `engine/class_modules/class_module.hpp:<line>`.
- A closing "how to navigate any other class file" paragraph (same struct names, find `namespace <class>` / `struct <class>_t`).

- [ ] **Step 3: Verify**

```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/05-class-module-pattern.md; echo "exit=$?"
```
Expected: `OK`, `exit=0`.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/05-class-module-pattern.md
git commit -m "docs: add class-module pattern guide"
```

---

### Task 6: Game-data pipeline (`06-game-data-pipeline.md`)

**Files:**
- Create: `doc/architecture/06-game-data-pipeline.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1).

- [ ] **Step 1: Read the extraction tooling and the consumption layer**

```bash
cd /c/Users/noah/Projects/simc
sed -n '1,60p' dbc_extract3/dbc_extract.py
ls dbc_extract3/ | head -40
ls casc_extract/ | head
sed -n '1,60p' engine/dbc/dbc.hpp
grep -n 'struct spell_data_t\|class spell_data_t' engine/dbc/*.hpp | head
ls engine/dbc/ | grep -iE 'generated|\.inc' | head
```
Capture: `dbc_extract.py` entry, what it outputs into `engine/dbc/` (generated `.inc`/`.hpp`), and the `dbc.hpp` access layer / `spell_data_t` the sim uses.

- [ ] **Step 2: Write `doc/architecture/06-game-data-pipeline.md`**

Required contents:
- 2-line header (how WoW data becomes sim data / standalone, optional read).
- One Mermaid `flowchart`: `WoW client (CASC)` → `casc_extract` → raw DBC/DB2 → `dbc_extract3 (Python)` → generated C++ in `engine/dbc/` → `engine/dbc/dbc.hpp` access layer (`spell_data_t` etc.) → consumed by `engine` (sim/player/class_modules).
- Prose: what `dbc_extract3` does, what it emits, and how the engine reads it at runtime, each with a real `file:line` reference (`dbc_extract3/dbc_extract.py:1`, `engine/dbc/dbc.hpp:<spell_data_t or top line>`).
- A short "regenerating data" note pointing at `dbc_extract3/` as the tool (high level — do not document every flag).

- [ ] **Step 3: Verify**

```bash
bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc doc/architecture/06-game-data-pipeline.md; echo "exit=$?"
```
Expected: `OK`, `exit=0`.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/06-game-data-pipeline.md
git commit -m "docs: add game-data pipeline overview"
```

---

### Task 7: Index + cross-doc consistency pass

**Files:**
- Create: `doc/architecture/index.md`

**Interfaces:**
- Consumes: `verify_doc.sh` (Task 1); all six docs from Tasks 1-6.

- [ ] **Step 1: Write `doc/architecture/index.md`**

Required contents:
- Title + 2-3 sentence intro ("Onboarding map of SimulationCraft. Read in order.").
- An ordered list linking each doc with a one-line description, in reading order:
  1. `01-overview.md` — the 30,000-ft component map
  2. `04-directory-map.md` — where everything lives
  3. `02-engine-core.md` — the core object model
  4. `03-simulation-flow.md` — how one sim runs end-to-end
  5. `05-class-module-pattern.md` — the template behind every class
  6. `06-game-data-pipeline.md` — how game data is produced and consumed
- A note: "Diagrams are Mermaid; they render on GitHub. `file:line` references point into the source tree."

- [ ] **Step 2: Verify all docs (refs + mermaid)**

```bash
cd /c/Users/noah/Projects/simc
for f in doc/architecture/*.md; do bash "<scratchpad>/verify_doc.sh" /c/Users/noah/Projects/simc "$f"; done
```
Expected: every file prints `OK`. Fix any failure and re-run.

- [ ] **Step 3: Cross-doc consistency check**

Manually confirm names are consistent across docs (no class called `sim_t` in one and `simulation_t` in another; reading-order links in `index.md` resolve to real files):
```bash
cd /c/Users/noah/Projects/simc
ls doc/architecture/                       # index + 01..06 all present
grep -oE '0[1-6]-[a-z-]+\.md' doc/architecture/index.md | sort -u   # links match real filenames
```
Expected: the six links in `index.md` correspond to the six files on disk.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/noah/Projects/simc
git add doc/architecture/index.md
git commit -m "docs: add architecture docs index and reading order"
```

---

## Self-Review

**Spec coverage:**
- Top-level architecture map → Task 2 ✓
- Engine core model → Task 3 ✓
- "How a sim runs" data flow → Task 4 ✓
- Directory/module map → Task 1 ✓
- Class-module pattern (add-on 5) → Task 5 ✓
- Game-data pipeline (add-on 6) → Task 6 ✓
- Index, conventions, accuracy method, verification → Task 1 (verifier) + Task 7 ✓
- Out of scope (no code changes, no Doxygen, no exhaustive call graphs) → respected; all tasks create only Markdown ✓

**Placeholder scan:** Reading lists and required-contents are concrete; the verifier script is given in full. Doc *prose* is synthesized at execution from the fixed reading lists (inherent to a documentation-synthesis task), but each task specifies exactly which files to read, the required diagram type + nodes, and the exact verification command — no "TBD"/"add details later".

**Type/name consistency:** Class names used across tasks (`sim_t`, `player_t`, `action_t`, `buff_t`, `event_t`, `event_manager_t`, `spell_t`, `mage_t`, `mage_spell_t`, `module_t`, `spell_data_t`) are confirmed to exist or to be located via the `grep -n` step in their owning task before use. The verifier enforces every `file:line` reference.

**Note on `<scratchpad>`:** literal placeholder for `C:\Users\noah\AppData\Local\Temp\claude\C--Users-noah-Projects\90a664cb-8802-47f9-b676-cf0bf77ce02b\scratchpad` — substitute the real path when running.
