# AGENTS.md — War with Hannibal

> Guidelines for AI coding agents working in this interactive-reader project.

---

## RULE 0 — THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it — if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`.
- **Never reference `master` in code or docs.**

---

## Project Overview

War with Hannibal is an interactive reader for Livy's *Ab Urbe Condita* Book XXI (the Second Punic War). Text and synchronized map operate as one system so readers can immediately understand **who** is acting, **where** events happen, **what** moved, **how certain** reconstructions are, and **what** happens next — all without leaving the reading flow.

The canonical execution plan lives in **`PLAN_CODEX.md`**. Read it before starting any substantial work.

---

## Toolchain: Next.js + TypeScript

| Layer | Tool |
|-------|------|
| Framework | Next.js (TypeScript) |
| Map | MapLibre GL JS |
| State | Zustand (or equivalent minimal store) |
| Validation | JSON Schema or Zod |
| E2E tests | Playwright |
| Hosting | Static-friendly (Vercel / Netlify / equivalent) + CDN for PMTiles |

### Package Manager

Use **npm** exclusively. Never use yarn or pnpm unless explicitly told otherwise.

---

## Repository Layout

```text
/data
  /sources/livy          # Raw public-domain source texts (READ-ONLY after ingest)
  /books/21
    /chapters            # Canonical chapter JSON (deterministic output)
    /views/default       # Presentation view mappings (section labels)
    /indexes             # Cross-reference indexes (generated)
  /gazetteer             # places.geojson, people.json, polities.json, forces.json
/public
  /tiles                 # PMTiles for MapLibre
  /visual-assets         # Static imagery
/scripts
  /ingest                # Source ingestion pipeline
  /validate              # Schema and integrity validators
/src
  /app                   # Next.js app directory
  /components            # React components
  /lib                   # Shared utilities, types, stores
```

### Critical Paths

| Path | Purpose | Rules |
|------|---------|-------|
| `data/sources/livy/` | Public-domain Gutenberg source texts | **READ-ONLY.** Never modify. Hash-verified in CI. |
| `data/books/21/chapters/` | Canonical chapter JSON | Deterministic output from ingestion pipeline. IDs are immutable once published. |
| `data/gazetteer/` | Entity master data (places, people, polities, forces) | Schema-validated. Entity IDs must be globally unique. |
| `PLAN_CODEX.md` | Canonical execution plan | Always read before major work. |

---

## Source Lock and Integrity

The source text is from **Project Gutenberg eBook #10907** (Spillan & Edmonds translation, public domain).

**Integrity fingerprints (SHA-256):**

| File | Hash |
|------|------|
| `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt` | `ff3ede761458ec93d989a18384ad485715fae8b0f6c822cca0f44e874be9430e` |
| `data/sources/livy/book_21_pg10907.txt` | `33a23b3e36482a4cd23a0c51a730b366c1e0ec399a658acc5e8c3223b0282f3d` |
| `data/sources/livy/book_21_chapter_05_pg10907.txt` | `e8ee3451ffecf80cba074877eefa11a1f9049ab3570911530a1ab8da5733bd33` |

**Rules:**
- Source files are **never modified** after initial acquisition.
- Hash checks run in CI (`npm run validate:source`).
- If a hash mismatch is detected, **stop work and alert the user** — never silently proceed.

---

## Data Contracts (Summary)

Full specifications are in `PLAN_CODEX.md` sections 7.1–7.7.

### ID Formats (Immutable Once Published)

| Entity | Format | Example |
|--------|--------|---------|
| Chapter | `21.XX` | `21.05` |
| Block | `21.XX.bNN` | `21.05.b07` |
| Annotation | `21.XX.aNN` | `21.05.a19` |
| Event | `21.XX.eNN` | `21.05.e04` |
| Scene | `21.XX.sNN` | `21.05.s04` |

**Rules:**
- IDs are **never recycled**.
- New content appends new IDs; old IDs remain addressable.
- Every event references at least one block and one actor.
- Every block maps to at least one event.

### Four Entity Classes (V1)

1. **`place`** — Geographic locations (gazetteer with confidence + geometry candidates)
2. **`person`** — Historical actors (Roman, Carthaginian, allied, local)
3. **`polity`** — Political entities (Rome, Carthage, allied states)
4. **`force`** — Military units (with faction, commander, composition)

### Confidence System

Every interpretive geographic claim must include:
- `confidence`: `high` | `medium` | `low`
- `sources[]`: citation references
- `editorial_note`: required when confidence is `low`

---

## Ingestion Pipeline

The pipeline is **deterministic** — same input always produces same output. See `PLAN_CODEX.md` section 8 for the full 11-step specification.

Pipeline order:
1. **Acquire & verify** — hash check source file
2. **Strip Gutenberg wrapper** — extract between START/END markers
3. **Extract Book XXI** — between `BOOK XXI.` and `BOOK XXII.` headings
4. **Normalize text** — unwrap hard-broken lines, normalize spacing
5. **Split chapters** — exactly 63, strict sequence 1..63
6. **Extract footnotes** — move inline `[Footnote: ...]` to `chapter.notes[]`
7. **Segment blocks** — 80–220 words preferred, 280 hard max
8. **Annotate** — tag places, people, polities, forces
9. **Extract events** — narrative beats per chapter
10. **Build scenes** — map viewport + layers per event
11. **Format output** — stable key ordering, deterministic JSON

**Hard failures (pipeline must abort):**
- Missing or mismatched source hash
- Missing Gutenberg START/END markers
- Missing `BOOK XXI.` / `BOOK XXII.` boundaries
- Chapter count != 63
- Chapter sequence gaps or duplicates

---

## CI Commands

```bash
npm run validate:source     # Source hash and boundary checks
npm run ingest:book21       # Deterministic canonical chapter output
npm run validate:data       # Schema / reference / integrity checks
npm run build:indexes       # Cross-reference index generation
npm run test:unit           # Parser / validator / store logic
npm run test:integration    # Chapter render + sync behavior
npm run test:e2e:smoke      # Primary user journeys (Playwright)
npm run test:a11y           # Accessibility checks
```

**Pipeline order in CI:** validate:source → ingest:book21 → validate:data → build:indexes → test:unit → test:integration → test:e2e:smoke → test:a11y → build

---

## QA Gates (Release Blockers)

A chapter **fails release** if any of these checks fail:

1. Schema validity pass
2. No dangling entity/event/scene references
3. Book 21 integrity (63 chapters)
4. Source-lock hash checks pass
5. All interpretive layers include confidence + sources
6. Actor coverage complete for chapter events
7. Disputed geographies explicitly flagged
8. Keyboard support for annotations and stepper
9. Mobile smoke tests pass
10. Text-only fallback pass (map failure graceful)
11. Deep link restoration pass
12. Accessibility audit baseline pass
13. Performance budgets pass
14. Scene delta summaries match scene-layer diffs
15. Glossary hints resolve and are keyboard accessible

---

## Performance, Accessibility, and Reliability Budgets

### Performance

| Metric | Target |
|--------|--------|
| Initial chapter load (cached) | < 3s on mid-range mobile |
| First map interaction | < 200ms perceived latency |
| Scene switch animation | <= 1.2s default |

### Accessibility

- Full keyboard traversal for reader, annotations, stepper
- Semantic markup for chapter structure and headings
- ARIA labels for map controls and entity cards
- Adequate color contrast across faction/confidence styling
- Respect `prefers-reduced-motion`

### Reliability

- Graceful map fallback without content loss
- Deterministic deep-link restoration
- No hard crash on missing optional assets
- Text-only mode fully functional if WebGL unavailable

---

## Code Editing Discipline

### No Script-Based Bulk Changes

**NEVER** run a script that processes/changes code files in this repo with brittle regex transformations.

- Always make code changes manually
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.

**NEVER** create variations like `componentV2.tsx`, `main_improved.ts`, or `utils_enhanced.ts`.

New files are reserved for **genuinely new functionality** that makes zero sense to include in any existing file. The bar for creating new files is **incredibly high**.

### Over-Engineering

- Don't add features, refactor code, or make "improvements" beyond what was asked.
- Don't add error handling for scenarios that can't happen.
- Don't create abstractions for one-time operations.
- Don't design for hypothetical future requirements.
- Three similar lines of code is better than a premature abstraction.

---

## Backwards Compatibility

We do not care about backwards compatibility during early development. We want to do things the **RIGHT** way with **NO TECH DEBT**.

- Never create "compatibility shims"
- Never create wrapper functions for deprecated APIs
- Just fix the code directly

---

## Compiler / Lint Checks (CRITICAL)

**After any substantive code changes, you MUST verify no errors were introduced:**

```bash
npx tsc --noEmit                    # TypeScript type checking
npx next lint                       # Next.js / ESLint linting
npm run validate:data               # Data integrity (if data changed)
npm test                            # Run test suite
```

If you see errors, **carefully understand and resolve each issue**. Read sufficient context to fix them the RIGHT way.

---

## Map and Cartography Notes

### Stack

- **Renderer:** MapLibre GL JS
- **Tile format:** PMTiles (static hosting + HTTP range requests)
- **Generation:** Planetiler + Tippecanoe pipeline
- **Bounds:** Mediterranean basin (`-10,20,45,50`)

### Zoom Levels

| Zoom | Scale |
|------|-------|
| 0–3 | Mediterranean strategic overview |
| 4–6 | Regional campaign theaters |
| 7–10 | Battle and pass-level operational detail |

### Visual Style

- Readable historical tone (digital vellum/parchment), not novelty skin
- No modern labels by default; labels come from project gazetteer and scenes
- Faction colors: Roman = restrained red, Carthaginian = amber/gold
- Uncertainty: high = solid, medium = dashed/reduced opacity, low = dotted/fuzzy + editorial note

### Uncertainty Visualization

Every uncertain map layer must answer:
- **What** is uncertain
- **Why** it's uncertain
- **Source basis** for the claim

---

## Reader-Map Sync

### Sync Modes (V1)

| Mode | Behavior |
|------|----------|
| `auto` (default) | Scroll-driven scene switching with hysteresis |
| `manual` | Event stepper controls map; scroll does not |

### Conflict Resolution

1. Direct user interaction beats auto-sync
2. Active animation can be interrupted by explicit action
3. Back/forward navigation restores consistent reader + map state
4. Manual mode suppresses scroll-driven scene changes

### Failure Mode (Must Work)

If the map fails to load:
- Text still renders completely
- Event summaries still navigable
- Place/person cards show textual context
- Deep links still resolve to block/event

---

## Pilot Chapter: 21.5

Chapter 21.5 is the canonical template because it contains multi-actor operations, movement, combat, and contested geography.

**Minimum event set:**
1. Hannibal commits to war strategy toward Saguntum
2. Olcades campaign and capture of Carteia
3. Winter at New Carthage and spring campaign against Vaccaei
4. Hermandica and Arbocala actions
5. Carpetani coalition engagement near the Tagus
6. River-crossing setup and cavalry strike
7. Carthaginian victory and regional submission beyond Iberus

**Pilot completion criteria:**
- All events mapped with scenes
- Actor completeness validated
- Confidence/source metadata complete
- Reader can narrate sequence without external atlas

---

## Deep Linking

Supported URL patterns:

```
/book/21/chapter/5
/book/21/chapter/5#block=21.05.b04
/book/21/chapter/5#event=21.05.e03
/book/21/chapter/5#scene=21.05.s03
/book/21/chapter/5?ref=21.5.3
```

Priority order: `scene` > `event` > `block` > `ref`. Invalid locators fail gracefully to chapter start.

---

## Beads (br) — Dependency-Aware Issue Tracking

Beads provides a lightweight, dependency-aware issue database and CLI (`br` - beads_rust) for selecting "ready work," setting priorities, and tracking status.

**Important:** `br` is non-invasive — it NEVER runs git commands automatically. You must manually commit changes after `br sync --flush-only`.

### Essential Commands

```bash
br ready                  # Show issues ready to work (no blockers)
br list --status=open     # All open issues
br show <id>              # Full issue details with dependencies
br create --title="..." --type=task --priority=2
br update <id> --status=in_progress
br close <id> --reason "Completed"
br sync --flush-only      # Export to JSONL (NO git operations)
```

### Key Concepts

- **Dependencies:** Issues can block other issues. `br ready` shows only unblocked work.
- **Priority:** P0=critical, P1=high, P2=medium, P3=low, P4=backlog (use numbers, not words)
- **Types:** task, bug, feature, epic, question, docs
- **Blocking:** `br dep add <issue> <depends-on>` to add dependencies

### Workflow Pattern

1. **Start:** Run `br ready` to find actionable work
2. **Claim:** Use `br update <id> --status=in_progress`
3. **Work:** Implement the task
4. **Complete:** Use `br close <id>`
5. **Sync:** Run `br sync --flush-only` then manually commit

### Mapping Cheat Sheet

| Concept | Value |
|---------|-------|
| Mail `thread_id` | `br-###` |
| Mail subject | `[br-###] ...` |
| File reservation `reason` | `br-###` |
| Commit messages | Include `br-###` for traceability |

---

## bv — Graph-Aware Triage Engine

bv is a graph-aware triage engine for Beads projects. It computes PageRank, betweenness, critical path, cycles, and other metrics.

**CRITICAL: Use ONLY `--robot-*` flags. Bare `bv` launches an interactive TUI that blocks your session.**

```bash
bv --robot-triage           # THE MEGA-COMMAND: start here
bv --robot-next             # Minimal: just the single top pick
bv --robot-plan             # Parallel execution tracks
bv --robot-insights         # Full graph metrics
bv --robot-priority         # Priority misalignment detection
```

### jq Quick Reference

```bash
bv --robot-triage | jq '.quick_ref'                    # At-a-glance summary
bv --robot-triage | jq '.recommendations[0]'           # Top recommendation
bv --robot-plan | jq '.plan.summary.highest_impact'    # Best unblock target
```

---

## MCP Agent Mail — Multi-Agent Coordination

For multi-agent workflows, use MCP Agent Mail for asynchronous coordination.

### Same Repository Workflow

1. **Register identity:** `ensure_project` + `register_agent`
2. **Reserve files before editing:** `file_reservation_paths` with exclusive lock
3. **Communicate with threads:** `send_message` with `thread_id="br-###"`
4. **Quick reads:** `resource://inbox/{Agent}?project=<abs-path>`

### Common Pitfalls

- `"from_agent not registered"`: Always `register_agent` first
- `"FILE_RESERVATION_CONFLICT"`: Adjust patterns, wait for expiry, or use non-exclusive
- Always use Beads issue ID as thread_id for traceability

---

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** — Create beads issues for anything that needs follow-up
2. **Run quality gates** (if code changed) — Types, lint, tests, data validation
3. **Update issue status** — Close finished work, update in-progress items
4. **PUSH TO REMOTE** — This is MANDATORY:
   ```bash
   git pull --rebase
   br sync --flush-only    # Export beads to JSONL (no git ops)
   git add .beads/         # Stage beads changes
   git add <other files>   # Stage code changes
   git commit -m "..."     # Commit everything
   git push
   git status              # MUST show "up to date with origin"
   ```
5. **Clean up** — Clear stashes, prune remote branches
6. **Verify** — All changes committed AND pushed
7. **Hand off** — Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing — that leaves work stranded locally
- NEVER say "ready to push when you are" — YOU must push
- If push fails, resolve and retry until it succeeds

---

## Note on Other Agents' Changes

If you see uncommitted changes in the working tree that you don't recognize, those are changes created by other agents working on the project concurrently. This is normal and happens constantly.

**Rules:**
- NEVER stash, revert, overwrite, or otherwise disturb other agents' work
- Treat those changes identically to changes you yourself made
- Just proceed with your own work alongside them

---

## Note on Built-in TODO Functionality

If explicitly asked to use built-in TODO functionality, comply. Don't insist on using beads when direct instructions say otherwise. Always comply with explicit orders.

---

## Third-Party Library Usage

If you aren't 100% sure how to use a third-party library, **SEARCH ONLINE** to find the latest documentation and current best practices. Do not guess at APIs.

---

## Scope Model Quick Reference

### MVP (V1) — Book 21 Only

- 63 chapters, dual-view (reader + synchronized map)
- Four entity classes: place, person, polity, force
- Event timeline, scene stepper, scroll sync
- Scene delta summaries, clickable annotations
- Force markers, route polylines, battle areas
- Contextual military glossary (inline definitions)
- Confidence + citation display
- Deep links, mobile layout, text-only fallback

### Launch+1

- Entity index/detail pages
- Cross-chapter references
- Reader progress + bookmarks (local storage)
- Additional sync modes + animation controls

### Launch+2

- Character portraits
- Army composition visualizations
- Battle sequence diagrams
- Narrative timeline views
- Thematic reading paths

### Explicitly Out of Scope for V1

- Accounts, comments, social features
- Public editing console
- Multi-book expansion beyond Book 21
- Cloud sync of personal data
