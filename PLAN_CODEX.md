# War With Hannibal — Interactive Map Reader (PLAN_CODEX.md)

## 1) Vision
Build a website that lets readers **see Livy’s narrative unfold**. When the text says “Consul A marched from X to Y with an army of size Z,” the interface should make that statement immediately legible: where X and Y are, what route is implied, how far it is, what else is happening nearby, and how confident we are about any reconstruction.

This is not a “total war simulator.” It is an **interactive companion to the book**, optimized for comprehension and trust: clear sourcing, visible uncertainty, and a reading-first layout that uses maps as explanatory diagrams rather than distractions.

## 2) Product Outcomes (What “Good” Looks Like)
- A reader can progress chapter-by-chapter with the map automatically presenting the relevant geography (“scenes”).
- Any place/person/force mention is clickable and reveals context (who, where, when, why it matters).
- Army movements are visually traceable (routes, start/end points, waypoints, and seasonality).
- Every interpretive choice (location, route, army size, identification of a person/place) can show **sources and confidence**.
- The system is designed for expansion from Book 21 to the full Second Punic War, and later to other works.

## 3) Scope
### MVP (v1)
- **Book 21** only (core arc: Saguntum → Pyrenees → Alps → Italy).
- Mediterranean and adjacent theaters needed for the narrative (Iberia, Gaul, Cisalpine Gaul, North Africa context).
- Public-domain English translation (swap-friendly text layer).
- Read-only experience: no user accounts, no comments, no editing in production UI.

### v1.5 / v2 (planned)
- Books 22–30, then cross-book search (“show me all mentions of Trasimene”).
- A lightweight editor interface for adding scenes/annotations with validation and review.
- Optional accounts for bookmarks, reading progress, personal notes, and sharing.

## 4) Primary Users And User Stories
### Readers (general audience)
1. “I’m confused where this happened” → click place → map pans, shows region context, shows other nearby scenes.
2. “Who is this commander?” → click person → card with role, faction, relationships, and appearances.
3. “How did the army get there?” → click route → highlights march line with distance estimate and confidence.

### Students / teachers
1. “Show the campaign in a lecture” → open a chapter, hit Present mode, step through scenes.
2. “Cite a claim” → copy a share link that encodes chapter + scene and displays sources.

### Editors (internal at first)
1. “Annotate places reliably” → select text span, link to gazetteer entry, attach citations, set confidence.
2. “Create a scene” → choose viewport, add force markers/routes/notes, preview in reader.

## 5) Information Model (Canonical First)
The core design decision: **canonical data is chapter-based**, stable across future text reformatting.

### Entities (global)
- **Place**: stable id, display name, alt names, geometry (point and optionally polygon), sources, confidence, editorial notes.
- **Person**: stable id, names, roles/titles, faction, lifespan (approx), sources.
- **Force**: stable id, faction, commander links (optional), iconography/color, notes, sources.

### Chapter (canonical)
Each chapter has:
- `blocks`: stable paragraph-ish units (never renumber once published).
- `annotations`: links from text anchors (quote/offset selectors) to entities.
- `scenes`: ordered map states tied to narrative moments.

### Presentation mapping
Keep “Book.Chapter.Section” as a **view layer**. A mapping file can define which blocks appear as sections like `21.5.3`, allowing later resectioning without breaking anchors.

### Identifiers, versions, and provenance
To keep links stable for years, ids must be predictable and never recycled:
- **Book/chapter ids**: `21.05` (zero-padded chapter for sorting).
- **Block ids**: `21.05.p01` (stable once published; edits happen inside the block text).
- **Annotation ids**: `21.05.a12` (stable references for discussion and review).
- **Scene ids**: `21.05.s03` (stable order; scenes can be inserted with explicit ids rather than renumbering).

Treat the dataset as a versioned artifact:
- Semantic version for the **data package** (not just the app): breaking changes only when ids or schemas change.
- A changelog entry whenever a place identification or route hypothesis changes.
- Provenance fields on entities and layers so the UI can show “why we think this is here” without hunting through git history.

## 6) Editorial Workflow (How Data Gets Made)
### Step 0: Editorial conventions (so work stays consistent)
- Adopt a short style guide: naming, capitalization, transliteration (e.g., “Carthago Nova” vs “New Carthage”), and how to present modern hints.
- Define what **counts as a “scene”**: a scene should answer a reader’s likely spatial question at that moment in the text, not try to encode every sentence.
- Define citation conventions (e.g., “Livy 21.5.3” plus optional modern secondary sources) and how to cite uncertain reconstructions.
- Define confidence rules: what qualifies as high/medium/low, and what UI treatment each must get.

### Step 1: Text ingestion
- Import public-domain translation into raw chapter files.
- Split into blocks (initially mechanical, later refined for readability).
- Generate deterministic ids (`21.05.p01`, etc.).

### Step 2: Gazetteer baseline
- Seed places from a curated list for Book 21: Saguntum, Carthago Nova, Emporiae, Rhone crossing, Alpine passes, Ticino, Trebia, etc.
- Prefer alignment to authority ids (Pleiades) when possible, but do not block MVP on perfect concordance.

### Step 3: Annotation pass
- Identify all place/person/force mentions.
- Link each mention to an entity, store confidence, and record citations (primary + modern reference when needed).
- Create “disambiguation notes” for ambiguous toponyms (e.g., rivers with contested identification).

### Step 4: Scene authoring
For each chapter, create a minimal set of scenes that match narrative pivots:
- “Setup geography” (where are we at chapter start).
- “Movement” (march route with start/end and key constraints like rivers or passes).
- “Engagement” (battle site view + unit positions only if the text supports it).
- “Aftermath” (retreat, winter quarters, political consequences if spatially relevant).

### Step 5: Validation and review
- JSON schema validation (structure, required fields, id formats).
- Geospatial validation (points in bounds, route geometries not wildly off).
- Editorial review checklist: “Does every scene cite its claims? Is uncertainty visible?”

### Optional Step 6 (v2): Editor tooling
MVP can be built with hand-edited JSON, but scaling to many books benefits from a simple internal editor:
- A chapter view that shows blocks and lets editors highlight spans to create annotations.
- An entity picker with fuzzy search (place/person/force) and “create new entity” flow.
- A scene builder that embeds the map and lets editors add layers (markers, routes, notes) with instant preview.
- A review mode that diffs changes, runs validation, and produces a “data PR checklist” automatically.

This can start as a “dev-only” page in the same Next.js app, protected by an environment flag, before becoming a standalone tool.

## 7) Map Strategy (Ancient-Friendly, Honest About Uncertainty)
### Basemap
Use an antique-styled basemap that emphasizes physical geography (coasts, rivers, mountains) and suppresses modern anachronisms. The guiding rule: **the map should explain Livy, not modern Europe**.

Recommended stack:
- Map rendering: **MapLibre GL JS**
- Tiles: **PMTiles** hosted as a static artifact (CDN-friendly)
- Data sources: Natural Earth physical layers + curated overlays for ancient places and routes

### Place labeling
All labels shown to the reader should come from the project gazetteer (ancient names and selected modern hints in parentheses when helpful). Avoid modern city clutter; allow a toggle for “modern reference” if desired later.

### Overlay vocabulary (what the map can render)
Define a small, composable set of layer types so scenes stay readable and tooling stays simple:
- `highlight_place`: pulse/halo a place point or polygon.
- `force_marker`: a faction-colored marker with optional commander + size label.
- `route`: a polyline with waypoints, distance estimate, and uncertainty styling.
- `battle_area`: a shaded polygon or circle for “in the vicinity of” when exact sites are debated.
- `label`: explicit scene labels that are not permanent map labels (e.g., “Rhone crossing”).
- `note`: explanatory text tied to a location on the map.

Keeping the layer list tight prevents the map from becoming a bespoke graphic system per chapter.

### Routes, distances, and “how long did that take?”
Readers often need two extra pieces of information Livy does not always give explicitly: distance and travel time.
- Store routes as LineStrings (or multi-segment routes) with named waypoints where the narrative demands it (river crossings, passes, camps).
- Precompute route length and show an **estimate range** for marching days (e.g., 20–30 km/day depending on terrain and baggage).
- Treat Alpine and river-crossing segments specially: present constraints and competing hypotheses instead of false precision.

In the UI, show these as optional details (collapsed by default) so the main reading flow stays clean.

### Uncertainty visualization
Every spatial claim is tagged:
- **High**: widely agreed location (e.g., Rome, Carthage).
- **Medium**: plausible identification with minor disputes.
- **Low**: speculative reconstruction (contested pass, approximate camp).

Render uncertainty through line styles, opacity, and explicit tooltips (“approximate; sources disagree”).

## 8) Reader Experience (UI/UX)
### Layout
- Split view: text left, map right (desktop). On mobile: stacked with a collapsible map.
- Scene controls: Prev/Next, plus a scene list per chapter.
- Optional scroll-sync: as the reader scrolls past anchored blocks, the active scene changes deterministically.

### Timeline ribbon (chronology without pretending certainty)
Add a thin timeline ribbon that anchors the reader in the war’s chronology:
- Default display: **consular year** and campaign season (spring/summer/autumn/winter) when known.
- For major events with widely accepted dates (e.g., famous battles), show a concrete date; for others, show “circa” ranges.
- Visually encode uncertainty (solid vs dashed) and allow a tooltip explaining why the date is approximate.

This ribbon should be informative but unobtrusive; it exists to prevent disorientation, not to force a strict calendar model onto the narrative.

### Deep links and sharing
Support links that reproduce the same reading/map state:
- `#block=21.05.p03` focuses the reader pane and highlights that block.
- `#scene=21.05.s02` loads the scene, pans the map, and opens the scene note.
- Optional `?ref=21.5.3` resolves via the presentation mapping (human-friendly citations).

These links make the site useful in classes, discussions, and citations.

### Present mode (teacher-friendly)
Provide a “Present” toggle that enlarges the map and gives big scene step controls. The goal is one-click usability in a classroom or on a projector: no tiny UI targets, no complex menus, and predictable keyboard controls.

### Interaction rules (to reduce cognitive load)
- Clicking a mention highlights it, opens an entity card, and briefly emphasizes the relevant map feature.
- The map never “thrashes” during normal scrolling: scene changes should be smooth and limited.
- A “Reset view” button returns to the current scene’s canonical viewport.

### Cards and overlays
- **Place card**: names, brief description, coordinates, confidence, sources, chapter mentions.
- **Person card**: roles, faction, relationships (commander of force), appearances.
- **Force card**: size estimates (with ranges), composition notes, commander, track across scenes.

## 9) Technical Architecture (MVP-Friendly, Expandable)
### Principles
- Static-first where possible (fast, cheap, resilient).
- Data-driven UI (chapters/scenes/entities are content, not code).
- Build-time validation to prevent shipping broken data.

### Performance budgets (so the map doesn’t swallow the reader)
- First meaningful text render should not wait on the map.
- Chapter JSON should be split so the initial payload is small (load scenes/entities on demand).
- Avoid expensive re-renders: treat map state as a small state machine (active scene → derived layers).
- Prefer precomputed indexes over runtime scanning for “find all mentions.”

### Proposed components
1. **Web app (Next.js)**
   - Routes: `/book/21`, `/book/21/chapter/5`, `/place/:id`, `/person/:id`, `/force/:id`
   - Shared state: active chapter, block, scene, active entity.
2. **Content package (data/)**
   - JSON/GeoJSON for chapters and gazetteer.
   - A build step compiles and indexes data for fast client use.
3. **Build tooling**
   - Schemas + validators.
   - Scripts to generate ids, check references, and build a search index.

### Storage and hosting
- Host the app on a static-capable platform (Vercel/Netlify/Cloudflare Pages).
- Host `pmtiles` and large assets via CDN with range requests enabled.

## 10) Data Pipeline (From Editorial Data To Runtime)
### Source-of-truth format
Keep human-edited files readable and reviewable in git (JSON with stable ordering or YAML if preferred later). Enforce:
- Stable ids.
- Strict schemas.
- No “magic” derived fields committed manually.

### Runtime compilation
At build time:
- Validate schemas and cross-references.
- Produce compact runtime bundles per book/chapter.
- Generate an inverted index (entity → mentions; chapter → scenes; place → all scene layers).
- Precompute route lengths (approx) and bounding boxes for fast map framing.

### Testing and quality gates
Treat content like code: it should be hard to publish broken chapters.
- Unit tests for validators and index builders.
- Snapshot-style tests for “chapter renders without missing entities” (no dangling references).
- A small set of Playwright E2E tests: load chapter, step scenes, click an entity, verify map updates.
- Continuous checks on every PR so editorial work can be safely merged.

## 11) Milestones (Beads-Friendly Breakdown)
Organize work into small, reviewable tasks (beads) that produce visible progress.

### Phase 0: Foundations (1–2 weeks)
- Choose stack, initialize Next.js, add MapLibre map pane, set up data folder layout.
- Implement minimal chapter reader with mock data.
- Implement gazetteer loading and map highlighting for a clicked place.

### Phase 1: Book 21 MVP (3–6 weeks)
- Ingest Book 21 text, block split, TOC and chapter pages.
- Create gazetteer entries for all Book 21 places; add people and forces baseline.
- Annotate all chapters with place mentions (people/forces for main figures).
- Author scenes for each chapter with viewports and key movements.
- Add citations UI and confidence styling.

### Phase 2: Polish + Reliability (2–4 weeks)
- Search (by place/person) and cross-chapter mention lists.
- Performance pass: pmtiles delivery, lazy-load chapter data, smooth scene transitions.
- Accessibility: keyboard navigation for scene stepping and entity cards.
- QA checklist and regression tests for schemas and rendering.

### Phase 3: Expansion (ongoing)
- Repeatable workflow for Book 22+: ingestion → gazetteer updates → annotations → scenes → review.
- Add “compare scenes” for multi-front war context (optional side-by-side mini-map).
- Add contribution pipeline (issue templates, editorial guidelines, and a review rubric).

## 12) Risks And Mitigations
- **Text licensing**: mitigate by using a public-domain translation and keeping text swappable.
- **Geographic disputes**: mitigate by explicit uncertainty, citations, and “multiple hypotheses” support later.
- **Scope creep**: mitigate by Book 21-only MVP and strict definition of “scene” as narrative aid.
- **Map performance**: mitigate by vector tiles + PMTiles, and keeping overlays light.

## 13) Success Metrics
- Readers can finish a chapter without losing their place (low UI friction).
- High engagement with entity clicks (signals comprehension support).
- Fast load times (<2s to first readable text on good connections; map loads progressively).
- Editorial throughput: a new chapter can be annotated + scened in a repeatable process.

## 14) “Next Actions” Checklist
1. Decide the MVP translation source and ingestion format.
2. Finalize the chapter/annotation/scene schemas.
3. Build the skeleton reader + map and prove one fully-annotated chapter end-to-end.
4. Scale the workflow to all of Book 21 with consistent quality and citations.
