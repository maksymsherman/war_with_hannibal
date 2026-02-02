# War With Hannibal Interactive Reader (Livy) — Build Plan

## Goal
Create a website where readers can follow Livy’s *War with Hannibal* chapter-by-chapter with an interactive, zoomable map that stays in sync with the narrative. When a passage mentions a place/person/army movement, the map highlights the relevant locations and shows (approximate) positions and routes of forces at that point in the story.

Initial scope is **Book 21** and the **Mediterranean basin**.

## Licensing And Text Strategy
- Livy’s Latin text is public domain, but most modern English translations are not.
- For v1, publish the full text using a **public-domain English translation** (older is fine).
- Design the app so the “text layer” is swappable later:
  - Replace translation text without rewriting map/scene annotations.
  - Support a “companion mode” later (for readers using a copyrighted print edition) without reproducing that text.

## Product Requirements (MVP)
- Table of contents for Book 21 (chapters).
- Chapter page with split layout:
  - Reader pane (left) with stable anchors for deep links.
  - Map pane (right) that responds to clicks/scroll.
- Clickable references in text:
  - Places highlight and pan/zoom the map.
  - People/forces open an info card and (if applicable) highlight associated markers/routes.
- “Scenes” tied to the narrative:
  - Prev/Next scene controls.
  - Optional scroll-sync: as the user scrolls, the active scene updates deterministically.
- Forces + routes are part of v1:
  - Roman/Carthaginian force markers.
  - March routes as polylines (with clear uncertainty signaling).

## Key Design Principle: Canonical Data Is Chapter-Level
We want **Book.Chapter.Section** in the UI, but we do *not* want the underlying data model to become dependent on the final “sectioning” scheme.

Plan:
- Store canonical information per **chapter** as stable paragraph blocks.
- Store anchors as “block + selector” (quote/offset-based).
- Treat `Book.Chapter.Section` as a **view** derived from a mapping file that can change later.

This keeps the implementation fluid: you can re-split text into sections without breaking annotations/scenes.

## Data Model

### 1) Chapter (Canonical)
One file per chapter, e.g. `data/books/21/chapters/05.json`.

Fields:
- `chapter_id`: `"21.05"`
- `source`: translation metadata (title, editor/translator, publication year, where the source text came from)
- `blocks`: stable paragraph-ish units
  - `block_id`: `"21.05.p01"` (once published, never renumber)
  - `text`
- `annotations`: links from a span of text to an entity
  - `annotation_id`
  - `anchor`: `{ block_id, selector }`
  - `selector`: `{ quote, prefix?, suffix?, startOffset?, endOffset? }`
  - `entity`: `{ type: "place"|"person"|"force", id: "..." }`
  - `confidence`: `"high"|"medium"|"low"`
  - `sources`: bibliography/citations for the identification or placement
  - `note`: optional editorial note
- `scenes`: map states tied to narrative moments
  - `scene_id`: `"21.05.s03"`
  - `anchor`: `{ block_id, selector? }`
  - `viewport`: `{ lng, lat, zoom }`
  - `layers`: render instructions

Layer types (minimum set):
- `highlight_place`: `{ place_id, confidence, sources }`
- `force_marker`: `{ force_id, position: { place_id } | { lng, lat }, label?, confidence, sources }`
- `route`: `{ force_id, geometry: LineString, style?, confidence, sources }`
- `note`: `{ text, sources }`

### 2) View Mapping (Presentation)
One file per chapter, e.g. `data/books/21/views/default/05.json`.

Purpose:
- Defines how `blocks` are grouped into displayed sections like `21.5.1`, `21.5.2`, etc.
- Allows changing section boundaries later without invalidating canonical anchors.

Fields:
- `chapter_id`: `"21.05"`
- `sections`: array of `{ label: "21.5.1", block_ids: ["21.05.p01", "21.05.p02"] }`

### 3) Gazetteer (Global Entities)
Files:
- `data/gazetteer/places.geojson`
- `data/gazetteer/people.json`
- `data/gazetteer/forces.json`

Places (GeoJSON `Feature` properties):
- `place_id`: stable identifier (consider aligning with Pleiades ids later, but not required for v1)
- `display_name`, `alt_names`
- `confidence`, `sources`, `note`

Forces:
- `force_id`, `label`, `faction` (`"Rome"|"Carthage"|etc.)
- `commander_person_id` (optional)
- optional metadata (icon, color, default label)

## Map Stack (Self-Hosted, Zoomable, Antique Look)

### Basemap Strategy (v1)
- Use **MapLibre GL** in the browser.
- Serve a self-hosted **`.pmtiles`** file from the app itself (static asset).
- Tile extent: **Mediterranean basin** only (smaller tileset, faster iteration).
- Label strategy: **no modern basemap labels in v1**.
  - Rationale: avoids anachronistic label conflicts and avoids hosting glyphs/fonts early.
  - All labels visible to users come from the project gazetteer (ancient context).

### Basemap Styling (Antique)
- Parchment background (self-hosted texture image).
- Muted palette, thicker coastlines, emphasized rivers, minimal modern roads.
- Relief/hillshade as a later enhancement (optional).

### Tile Build Pipeline
- Use a tile builder (e.g., Planetiler) to produce the `.pmtiles`.
- Store tile build config/scripts in `scripts/tiles/` so rebuilds are deterministic.
- Keep zoom levels moderate at first (optimize for performance, then increase if needed).

## Frontend Architecture
- Framework: Next.js (static-friendly pages, clean routing).
- State: small shared store for “active scene”, “active entity”, “active block”.
- Pages:
  - `/book/21` (TOC)
  - `/book/21/chapter/5` (reader + map)
  - `/place/:place_id` (place detail + list of mentions/scenes)
- Deep links:
  - Stable ids: `#scene=21.05.s03`, `#block=21.05.p04`
  - Friendly refs: `?ref=21.5.3` resolved via the view mapping.

## Reader ↔ Map Sync Behavior
- Clicking a `place` annotation:
  - sets active entity
  - pans/zooms to the place geometry
  - highlights the place (halo + label)
- Clicking a `force` annotation:
  - opens force card
  - highlights the force marker (if present in active scene)
- Activating a scene:
  - sets viewport (with animation)
  - renders that scene’s layers (forces/routes/highlights)
- Scroll-sync (optional in MVP but recommended):
  - determine active scene based on the nearest scene anchor above the viewport
  - throttle updates to avoid jitter

## Editorial Workflow (How Content Gets Made)

### Phase 1: Minimal Manual Authoring
- Create a starter gazetteer for Book 21:
  - ~30–80 key places (cities, rivers, passes, regions).
- Define 10–30 scenes for the major beats of Book 21:
  - each scene includes force positions and routes where relevant
  - every route/position has `confidence` + `sources`
- Add annotations incrementally:
  - start with place mentions
  - then add force mentions and key people

### Phase 2: Build A Dev-Only Editor (High Leverage)
Add internal tooling inside the app (behind a dev flag):
- Annotation editor:
  - click text to create a reference
  - auto-capture quote selector (plus offsets) to minimize brittle anchors
- Scene editor:
  - choose a chapter + scene anchor
  - click map to place force markers
  - draw/edit route polylines
  - set viewport quickly (“capture current view”)
- Save changes back to chapter JSON files.

## Validation, Quality, And Trust
- Add schemas (Zod or JSON Schema) for:
  - chapters, view mappings, gazetteer entities, scenes/layers
- Add `scripts/validate` that checks:
  - referenced ids exist
  - GeoJSON validity
  - anchors refer to real blocks
  - citations present for non-trivial claims
- Display uncertainty in the UI:
  - low-confidence positions/routes are visually distinct (e.g., dashed lines, lighter color)
  - always show “why” (note + sources) in the info card

## Testing
- Unit tests for validators (fixtures for valid/invalid data).
- E2E tests (Playwright) for core flows:
  - open chapter page
  - click a place reference and assert map pans/opens card
  - step scenes and assert force markers/routes update
  - deep link to `#scene=...` loads correct state

## Deployment
- Keep data and tiles as static assets so the site can be hosted simply.
- Ensure `.pmtiles` is served with correct headers and supports HTTP range requests (important for efficient tile access).
- Use a single build pipeline:
  - validate data
  - build web app
  - deploy static assets + server

## Milestones (Suggested Order)

### 0) Setup
- Choose the PD translation source for Book 21.
- Add a repeatable ingestion script to create chapter JSON with paragraph blocks.

### 1) Reader Skeleton
- TOC + chapter page
- render blocks with stable ids and anchors
- implement the view mapping layer for `Book.Chapter.Section` labels

### 2) Map Skeleton (Antique)
- MapLibre map loads with self-hosted tiles
- antique style applied
- place overlay from `places.geojson`

### 3) Clickable References
- add annotations for 1 chapter
- click-to-pan + highlight + info card

### 4) Scenes With Forces + Routes
- implement scene model + scene stepper
- render force markers and route polylines per scene
- add scroll-sync

### 5) Editor + Validation
- dev-only editor UI for annotations/scenes
- `scripts/validate` wired into CI

### 6) Book 21 Completion
- expand gazetteer coverage and scenes across all chapters in Book 21
- add place pages (mentions, scenes list)

## Defaults And Opinionated Choices For v1
- Text blocks are paragraphs.
- Basemap has no modern labels; all labels come from the project gazetteer and scenes.
- Everything “map-assertive” includes confidence + citations.

