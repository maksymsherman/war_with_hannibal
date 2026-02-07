# War With Hannibal - Ultimate Hybrid Execution Plan

## 1) Mission and Outcome
Build the definitive interactive reader for Livy's Book 21 where text and map operate as one system.

For any passage, the user should get immediate, evidence-backed answers:
- Who is acting?
- Where is this happening?
- What moved from where to where?
- How certain is this reconstruction?
- What happens next in narrative sequence?

The product succeeds when readers can understand strategy, geography, and actor relationships without leaving the reading flow.

## 2) Non-Negotiable Product Rules
1. Reading-first: text remains fully usable even if map or JS fails.
2. Source-locked: canonical text pipeline is deterministic and reproducible.
3. Event-first map logic: map follows events, not raw paragraph order alone.
4. Actor-complete: include major Roman, Carthaginian, allied, and local actors.
5. Explicit uncertainty: all interpretive geography displays confidence + sources.
6. Canonical/presentation decoupling: chapter data is stable; section labels are a view.
7. Accessibility-first interactions: keyboard and screen reader support are release blockers.
8. Mobile viability: reader remains primary on phones; map is contextual, not broken.
9. Performance discipline: map interactions and scroll sync must feel stable on mid-range devices.
10. Scope control: advanced visual extras are included via phased rollout, not by destabilizing MVP.

## 3) Scope Model (What Ships When)

### 3.1 MVP (V1 Release)
In scope:
- Book 21 only (63 chapters).
- Dual-view chapter page (reader + synchronized map).
- Four entity classes: `place`, `person`, `polity`, `force`.
- Event timeline per chapter with deterministic event IDs.
- Scene stepper (Prev/Next) + optional auto-sync by scroll.
- Clickable text annotations that pan/highlight map and open entity cards.
- Force markers, route polylines, battle areas, and place highlights.
- Confidence and citation display for interpretive claims.
- Stable deep links for chapter/block/event/scene.
- Mobile reader-first layout with map drawer.
- Text-only fallback if map initialization fails.

### 3.2 Launch+1 (Post-V1, still near-term)
- Entity index/detail pages (`/place/:id`, `/person/:id`, `/force/:id`) with chapter mentions.
- Cross-chapter "see also" references and precomputed relationship indexes.
- Reader progress, resume position, and user bookmarks stored locally.
- Additional sync modes (`annotation-only`, `disabled`) and animation speed controls.
- Expanded integration tests and telemetry dashboards.

### 3.3 Launch+2 (Advanced Storytelling)
- Character portraits with attribution metadata.
- Army composition visualizations.
- Battle sequence diagrams for major engagements.
- Narrative timeline views (war-scale and chapter-scale).
- Thematic reading paths and curated journeys.
- Community contribution workflow for corrections and alternative interpretations.

### 3.4 Explicitly Out of Scope for V1
- Accounts, comments, and social collaboration features.
- Public editing console for arbitrary users.
- Full multi-book expansion beyond Book 21.
- Mandatory use of portraits/diagrams/timeline in core reading flow.
- Cloud sync of personal progress data.

## 4) Target Users and Core Use Cases

### 4.1 Primary: History Enthusiasts
Needs:
- Fast geographic orientation while reading.
- Better understanding of strategic movement and campaign logic.

### 4.2 Secondary: Students and Educators
Needs:
- Reliable citations and confidence signaling.
- Clear visual aids for classroom and study use.

### 4.3 Tertiary: Digital Humanists and Advanced Readers
Needs:
- Transparent data model.
- Exportable geospatial and entity link data.

### 4.4 Key User Flows
1. Follow-along reading: scroll text, map updates at event anchors.
2. Deep investigation: lock map, explore terrain, inspect uncertain locations and evidence.
3. Revision/study: navigate by event stepper, entity links, and chapter deep links.
4. Mobile orientation check: open map drawer, verify current theater, return to text without losing place.

## 5) Source Lock, Licensing, and Provenance

### 5.1 Primary Text Source (V1)
- Project Gutenberg eBook #10907
- URL: `https://www.gutenberg.org/ebooks/10907`
- Local file: `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`

Observed source facts:
- UTF-8 BOM present in raw file.
- Gutenberg start/end markers present.
- `BOOK XXI.` and `BOOK XXII.` headings present.
- Book XXI chapter markers sequential `1.` through `63.`.
- Nine inline `[Footnote: ...]` notes in Book XXI.
- Hard wrapping near ~70 chars.

Integrity fingerprints (current baseline):
- `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`
  - `ff3ede761458ec93d989a18384ad485715fae8b0f6c822cca0f44e874be9430e`
- `data/sources/livy/book_21_pg10907.txt`
  - `33a23b3e36482a4cd23a0c51a730b366c1e0ec399a658acc5e8c3223b0282f3d`
- `data/sources/livy/book_21_chapter_05_pg10907.txt`
  - `e8ee3451ffecf80cba074877eefa11a1f9049ab3570911530a1ab8da5733bd33`

### 5.2 Text Licensing Policy
- Publish only public-domain translation text in V1.
- Keep translation metadata in each chapter source block.
- Preserve architecture for swappable text layer in future.
- Support companion-mode workflow later for copyrighted editions (without reproducing protected text).

### 5.3 Geo/Map Licensing Policy
- Natural Earth: public domain.
- OpenStreetMap-derived data: ODbL attribution and compliance.
- Pleiades and other authority datasets: preserve citation and license metadata in source records.

## 6) Technical Architecture

### 6.1 Stack
- Frontend: Next.js + TypeScript.
- Map: MapLibre GL JS.
- Shared state: Zustand (or equivalent minimal store).
- Data validation: JSON Schema or Zod.
- E2E tests: Playwright.
- Hosting: static-friendly deployment (`Vercel`, `Netlify`, or equivalent) with CDN for assets and PMTiles.

### 6.2 High-Level System Components
1. Source pipeline: ingest and normalize raw text into canonical chapter JSON.
2. Content data layer: canonical chapter files + global gazetteer + view mapping files.
3. Reader client: text rendering, event stepper, annotation interaction.
4. Map client: scene rendering, layer toggling, uncertainty visualization.
5. Validation/CI layer: schema, reference integrity, quality gates, tests.

### 6.3 Suggested Repository Layout
```text
/data
  /sources/livy
  /books/21
    /chapters
    /views/default
    /indexes
  /gazetteer
/public
  /tiles
  /visual-assets
/scripts
  ingest
  validate
/src
  /app
  /components
  /lib
```

## 7) Canonical Data Contracts

### 7.1 Chapter JSON (Canonical)
Required top-level fields:
- `chapter_id`
- `source`
- `blocks`
- `annotations`
- `notes`
- `events`
- `scenes`

ID formats:
- Chapter: `21.05`
- Block: `21.05.b07`
- Annotation: `21.05.a19`
- Event: `21.05.e04`
- Scene: `21.05.s04`

Rules:
- IDs are immutable once published.
- IDs are never recycled.
- New content appends new IDs; old IDs remain addressable.

### 7.2 Anchor Model
Anchors must survive editorial refinement.

Use:
- `block_id`
- `selector.quote`
- `selector.prefix` and `selector.suffix`
- `selector.startOffset` and `selector.endOffset`
- optional `selector.block_checksum`

This enables robust re-anchoring when punctuation/spacing changes.

### 7.3 Event Model
Required fields:
- `event_id`
- `block_ids[]`
- `title`
- `summary`
- `event_type`
- `actors[]`
- `places[]`
- `sources[]`

Event types:
- `movement`
- `battle`
- `siege`
- `diplomacy`
- `logistics`
- `political`
- `other`

Rules:
- Every event references at least one block.
- Every event lists at least one actor.
- Every block maps to at least one event.

### 7.4 Scene Model
Each event has one primary scene, with optional supporting scenes.

Required fields:
- `scene_id`
- `event_id`
- `anchor_block_id`
- `viewport` (`lng`, `lat`, `zoom`, optional `bearing`, `pitch`)
- `layers[]`

Layer types (V1):
- `highlight_place`
- `route`
- `force_marker`
- `battle_area`
- `label`
- `note`

For `route`, `force_marker`, `battle_area`, `label`:
- `confidence` is required.
- `sources` is required.
- `editorial_note` is recommended when confidence is `low`.

### 7.5 Presentation View Mapping (Decoupled)
Canonical data does not depend on section numbering style.

Per chapter mapping file:
- `data/books/21/views/default/05.json`
- defines display sections such as `21.5.1`, `21.5.2`, etc.
- maps section labels to canonical `block_ids[]`

Result:
- sectioning can change without invalidating canonical anchors, events, or scenes.

### 7.6 Gazetteer Contracts
Files:
- `data/gazetteer/places.geojson`
- `data/gazetteer/people.json`
- `data/gazetteer/polities.json`
- `data/gazetteer/forces.json`

Place schema essentials:
- `place_id`
- `display_name`
- `alt_names[]`
- `authority_refs[]`
- `geometry_candidates[]`
- `preferred_geometry_index`
- `confidence`
- `sources[]`
- `editorial_note`

Force schema essentials:
- `force_id`
- `label`
- `faction`
- `commander_person_id` (optional)
- `composition` (optional counts/ratios)
- `sources[]`

### 7.7 Cross-Reference Indexes
Generated indexes (initial in V1, expanded in Launch+1):
- place -> mention list (`chapter_id`, `block_id`, `event_id`, `scene_id`)
- person -> mention list
- force -> mention list
- chapter event sequence index for fast navigation

## 8) Deterministic Ingestion Pipeline (Must Be Implemented Exactly)

### Step A - Acquire and Verify
Input path (fixed):
- `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`

Checks:
1. file exists
2. hash matches expected baseline unless intentionally rotated
3. encoding valid UTF-8
4. BOM removed in normalized output artifact

### Step B - Strip Gutenberg Wrapper
Extract only content between START and END markers.
Hard fail if either marker is absent.

### Step C - Extract Book XXI Slice
Boundaries:
- start: `^BOOK XXI\.$`
- end: `^BOOK XXII\.$` (exclusive)

Fail if boundaries missing or inverted.

### Step D - Normalize Wrapped Text
Rules:
1. unwrap hard-broken lines into paragraph strings
2. preserve blank-line paragraph boundaries
3. normalize punctuation spacing
4. preserve chapter numeric headers `1.` to `63.`
5. retain footnote payload for provenance (detached in Step F)

### Step E - Split Into Chapters
Anchor regex:
- `^(\d+)\.\s`

Validation:
1. exactly 63 chapters
2. strict sequence 1..63
3. no duplicates, no gaps

### Step F - Footnote Policy
- Remove inline `[Footnote: ...]` from primary reading blocks.
- Persist in `chapter.notes[]` with anchor (`block_id` + quote snippet).
- Keep notes available in UI provenance panel (collapsed by default).

### Step G - Block Segmentation Policy
Do not rely on raw paragraph granularity.

Targets:
- preferred block size: 80-220 words
- hard max: 280 words

Split heuristics:
- sentence boundaries
- discourse pivots (actor/action/location changes)
- transition cues (`then`, `after`, `meanwhile`, `whereupon`, etc.)

Validation:
- no empty blocks
- strict monotonic order
- >95% of blocks in preferred range
- oversize block requires `oversize_justification`

### Step H - Annotation Pass
Create baseline annotations for:
- places
- major people
- polities
- forces

Rules:
- anchors must resolve to live block text
- entity IDs must exist in gazetteer files
- uncertainty/citation required for disputed identifications

### Step I - Event Extraction
Generate event beats per chapter.

Validation:
- every event has `block_ids[]`
- every event has at least one actor
- every block is included by at least one event
- event order follows block order (unless explicit retrospective note)

### Step J - Scene Build
At minimum one primary scene per event.

Validation:
- `scene.event_id` exists
- scene anchor block exists
- layer references are valid entities/geometries
- interpretive layers include confidence + sources

### Step K - Deterministic Output Formatting
- stable key ordering in emitted JSON
- deterministic array sort where order is semantic
- no timestamp-based non-determinism inside canonical artifacts

## 9) Historical Cartography and Map Data Strategy

### 9.1 Core Approach
Use modern physical data as a foundation, then apply historically motivated filtering and corrections.

Foundation:
- Natural Earth 1:10m physical layers (coastlines, major rivers, relief context).

Corrections and curation priorities:
- Po delta/coastline treatment for antiquity context.
- North African coastal nuances around Carthaginian core zones.
- Alpine passes and campaign-relevant corridors.
- Remove/de-emphasize modern political boundaries and contemporary place clutter.

### 9.2 Tile and Rendering Stack
- renderer: MapLibre GL JS
- format: PMTiles for static hosting and range requests
- generation: Planetiler + Tippecanoe pipeline
- delivery requirement: HTTP range requests enabled in production hosting
- budget target: keep basin archive reasonably compact for first-use load

Conceptual pipeline:
```bash
planetiler --osm-path=mediterranean-extract.osm.pbf \
  --natural-earth=ne_10m_physical.zip \
  --bounds="-10,20,45,50" \
  --output=base.mbtiles

tippecanoe --force \
  --output=war-with-hannibal.pmtiles \
  --maximum-zoom=10 \
  --base-zoom=5 \
  processed-features.geojson
```

### 9.3 Zoom-Level Information Design
- Zoom 0-3: Mediterranean strategic overview.
- Zoom 4-6: regional campaign theaters.
- Zoom 7-10: battle and pass-level operational detail.

### 9.4 Visual Style Direction
- Readable historical tone (digital vellum / parchment direction), not novelty skin.
- No modern labels by default; labels come from project gazetteer and scenes.
- Distinct faction color system and confidence-based styling.
- Suggested palette baseline:
  - land: warm parchment tone
  - water: muted blue-green
  - Roman emphasis: restrained red family
  - Carthaginian emphasis: amber/gold family
  - uncertain layers: desaturated with patterned strokes
- Typography direction:
  - body text: high-legibility serif
  - controls and metadata: neutral sans-serif

### 9.5 Uncertainty Visualization Rules
- high confidence: solid line/marker
- medium confidence: dashed or reduced opacity
- low confidence: dotted/fuzzy shape + explicit note in card

Every uncertain layer must answer:
- what is uncertain
- why
- source basis

### 9.6 Fallback and Network Strategy
- If WebGL is unavailable, provide static fallback map imagery for key scenes.
- If bandwidth is constrained, reduce detail layers and animation complexity automatically.
- Prefetch likely next-scene tiles to smooth stepping on slower connections.

## 10) Reader and Map UX Specification

### 10.1 Primary Layouts
Desktop:
- split view (reader + map)

Tablet:
- balanced split with adjustable divider

Mobile:
- reader-first full screen
- map in bottom sheet/drawer
- map can be expanded for investigation and collapsed to continue reading

### 10.2 Reader Behavior
- stable block anchors for deep links
- inline annotation highlights
- event stepper visible and keyboard operable
- optional distraction-free reading mode

### 10.3 Map Behavior
- map reflects active event in auto mode
- reset control restores canonical viewport for active scene
- map lock prevents scroll-driven camera changes
- entity card includes: name, type, role in current event, confidence, sources

### 10.4 Sync Modes
- `auto` (default): anchor threshold switching with hysteresis
- `manual`: event stepper controls map scene
- `annotation-only` (Launch+1): clicks trigger map changes, scroll does not
- `disabled` (Launch+1): no automatic synchronization

### 10.5 Failure Modes (Must Work)
If map fails to load:
- text still renders completely
- event summaries still navigable
- place/person cards show textual context
- deep links still resolve to block/event

## 11) Reader-Map Sync Algorithm and State Management

### 11.1 Active Scene Detection
Use viewport anchor strategy:
- activation line at top one-third of viewport
- consider nearest passed scene anchor
- apply hysteresis buffer to avoid oscillation
- debounce rapid scroll transitions

Pseudo-logic:
```javascript
const activationY = scrollTop + window.innerHeight * 0.33;
const scene = nearestSceneAtOrAbove(activationY);
if (sceneChanged(scene) && movedBeyondHysteresis(scrollTop)) {
  setActiveScene(scene);
}
```

### 11.2 Conflict Resolution Rules
1. Direct user interaction beats auto-sync.
2. Active animation can be interrupted by new explicit action.
3. Back/forward navigation restores consistent reader + map state.
4. Manual mode suppresses scroll-driven scene changes.

### 11.3 Shared App State (Conceptual)
```json
{
  "reading": {
    "chapter_id": "21.05",
    "active_block_id": "21.05.b04",
    "scroll_y": 1234
  },
  "map": {
    "active_scene_id": "21.05.s03",
    "viewport": { "lng": 12.4, "lat": 41.9, "zoom": 6 },
    "animating": false,
    "locked": false
  },
  "sync": {
    "mode": "auto",
    "paused": false,
    "hysteresis_px": 150
  }
}
```

### 11.4 Motion and Timing
- scene transition: ~1000-1200ms default
- user-initiated annotation jump: ~700-900ms
- respect `prefers-reduced-motion`
- expose animation speed controls in Launch+1

## 12) Deep Linking and Navigation Contracts

Supported URL patterns:
- `/book/21/chapter/5`
- `/book/21/chapter/5#block=21.05.b04`
- `/book/21/chapter/5#event=21.05.e03`
- `/book/21/chapter/5#scene=21.05.s03`
- `/book/21/chapter/5?ref=21.5.3` (resolved via view mapping)

Rules:
- first valid locator in priority order initializes view (`scene` > `event` > `block` > `ref`).
- invalid locators fail gracefully to chapter start with warning telemetry.

## 13) Editorial Workflow

### 13.1 Phase 1 - Manual High-Quality Authoring
- build starter gazetteer focused on Book 21 key entities
- author pilot chapter events/scenes manually for quality baseline
- establish style guide for summaries, uncertainty notes, and citation format
- maintain reviewer notes for disputed geographies and editorial rationale

### 13.2 Phase 2 - Dev-Only Internal Editor
Behind dev flag:
- annotation editor from text selection
- scene editor (capture viewport, place force markers, draw routes)
- confidence and source editing controls
- save back to chapter JSON with schema validation before write

### 13.3 Review Pipeline per Chapter
1. factual pass (source alignment)
2. geo pass (place and route plausibility)
3. UX pass (scene pacing and readability)
4. technical pass (schema and reference integrity)

### 13.4 Chapter Sequencing Strategy
- Internal pipeline pilot: chapter 21.5 for data model stress (multi-actor, movement, combat).
- Public demo candidates: chapter 21.4 (Alps) and chapter 21.10 (major battle narrative).
- Scale-up sequence after pilot: complete first 10 chapters, then finish full Book 21 chronologically.

### 13.5 Chapter Completion Criteria
- Geographic annotation coverage target: >=90% place mentions linked.
- Significant person coverage target: all major actors annotated.
- Event-scene coverage target: every major narrative beat represented.
- Citation coverage target: all interpretive claims source-backed.
- Mobile and accessibility checks pass before chapter is marked releasable.

## 14) Advanced Visual Storytelling Roadmap

These are included in the plan but gated to avoid MVP destabilization.

### 14.1 Portrait System (Launch+2)
- per-character image set with attribution metadata
- inline avatar on first mention
- larger portrait in entity cards

### 14.2 Army Composition Views (Launch+2)
- Roman and Carthaginian structure diagrams
- force card integration for major events

### 14.3 Battle Sequence Diagrams (Launch+2)
- phase-by-phase tactical overlays for major battles
- synchronized with event stepping

### 14.4 Narrative Timeline (Launch+2)
- war-wide and chapter-local temporal context
- confidence signaling for uncertain dates

## 15) Cross-Referencing, Progress, and Search

### 15.1 V1 Baseline
- per-entity mention list (chapter/event links)
- chapter TOC with completion state optional locally
- place/person/force pages can show first/last mention and direct jump links

### 15.2 Launch+1 Enhancements
- "see also" references (temporal, spatial, causal)
- local progress persistence (resume location)
- user bookmarks for blocks/events/scenes
- fast entity search with context snippets

### 15.3 Relationship Taxonomy
- temporal: earlier and later mentions of the same entity
- spatial: related events in the same location/theater
- causal: events that set up or result from current event
- thematic: similar tactical or diplomatic patterns

### 15.4 Progress Data Policy
- Keep progress local by default in browser/device storage.
- Capture resume position as chapter/block/scene tuple.
- Allow explicit user bookmarks with optional short notes.
- Defer cloud sync until privacy, auth, and support burden are justified.

## 16) QA Gates and Release Blockers
A chapter fails release if any rule fails:
1. schema validity pass
2. no dangling entity/event/scene references
3. Book 21 integrity pass (63 chapters)
4. source-lock checks pass
5. all interpretive layers include confidence + sources
6. actor coverage complete for chapter events
7. disputed geographies explicitly flagged
8. keyboard support for annotations and stepper
9. mobile smoke tests pass
10. text-only fallback pass
11. deep link restoration pass
12. accessibility audit baseline pass
13. performance budgets pass

## 17) CI Commands and Pipeline

Commands:
- `npm run validate:source` - source hash and boundary checks
- `npm run ingest:book21` - deterministic canonical chapter output
- `npm run validate:data` - schema/reference/integrity checks
- `npm run build:indexes` - cross-reference index generation
- `npm run test:unit` - parser/validator/store logic
- `npm run test:integration` - chapter render + sync behavior
- `npm run test:e2e:smoke` - primary user journeys
- `npm run test:a11y` - accessibility checks

Pipeline order:
1. `validate:source`
2. `ingest:book21`
3. `validate:data`
4. `build:indexes`
5. `test:unit`
6. `test:integration`
7. `test:e2e:smoke`
8. `test:a11y`
9. build and publish

## 18) Performance, Accessibility, and Reliability Budgets

### 18.1 Performance Targets
- initial chapter load (cached static assets): <3s on mid-range mobile
- first map interaction responsiveness: <200ms perceived latency
- scene switch animation completion: <=1.2s default
- map tile archive target size: keep practical and CDN-friendly

### 18.2 Accessibility Targets
- full keyboard traversal for reader, annotations, and stepper
- semantic markup for chapter structure and headings
- ARIA labels for map controls and entity cards
- adequate color contrast across faction/confidence styling

### 18.3 Reliability Targets
- graceful map fallback without content loss
- deterministic deep-link restoration
- no hard crash on missing optional assets (portraits/diagrams later)
- degraded-mode support for low-end devices and non-WebGL browsers

## 19) Pilot Chapter Blueprint (21.5)
Use 21.5 as canonical template because it contains multi-actor operations, movement, combat, and contested geography.

Minimum event set:
1. Hannibal commits to war strategy toward Saguntum.
2. Olcades campaign and capture of Carteia.
3. Winter at New Carthage and spring campaign against Vaccaei.
4. Hermandica and Arbocala actions.
5. Carpetani coalition engagement near the Tagus.
6. River-crossing setup and cavalry strike.
7. Carthaginian victory and regional submission beyond Iberus.

Pilot completion criteria:
- all events mapped with scenes
- actor completeness validated
- confidence/source metadata complete
- reader can narrate sequence without external atlas

## 20) Delivery Timeline

### Week 1
- freeze schemas and validator contracts
- implement source verification and ingestion
- emit canonical JSON for chapter 21.5

### Week 2
- complete annotations/events/scenes for 21.5 and 21.6
- implement event stepper and map sync core
- implement map failure fallback
- add integration tests

### Week 3
- scale to first 10 chapters
- tune mobile and sync stability
- integrate release QA gates

### Week 4-6
- complete remaining Book 21 chapters
- finalize gazetteer coverage and uncertainty notes
- lock performance and accessibility baselines

### Week 7+
- Launch+1 features (indexes, bookmarks, richer sync controls)
- begin Launch+2 visual storytelling assets

## 21) Risk Register and Controls
- Translation/source drift: hash + marker validation in CI.
- Anchor fragility: block + quote + checksum anchoring.
- Ambiguous geography: candidate geometries + confidence + editorial note.
- Scope creep: tiered roadmap and strict MVP gate.
- Mobile complexity: reader-first layout with map drawer.
- Performance degradation: budgets and regression monitoring.
- Data inconsistency: strict schema and cross-reference validation.

## 22) Launch and Growth (After Quality Gates)
- SEO: static chapter pages + entity pages with metadata.
- Initial launch channels: Hacker News, relevant history communities.
- Open-source posture: accept factual corrections and alternative-route proposals with citation requirements.

### 22.1 Feedback and Improvement Loop
1. Immediate fixes: factual errors, broken links, accessibility regressions.
2. Short-cycle refinements: weak explanations, missing references, interaction pain points.
3. Planned enhancements: backlog intake tied to roadmap tier (Launch+1 vs Launch+2).

## 23) Immediate Next Steps
1. Implement `validate:source` with current known hashes.
2. Implement `ingest:book21` with deterministic split, footnote extraction, and stable IDs.
3. Generate canonical chapter JSON for 21.5.
4. Author 21.5 events and scenes using pilot blueprint.
5. Build and run QA gates end-to-end.
6. Extend to 21.6 and lock reusable editorial workflow.

## 24) Definition of Done

### 24.1 MVP Done
MVP is complete when:
- all 63 Book 21 chapters are published in canonical JSON
- every chapter passes release blockers
- reader/map flow answers actor/location/movement/confidence questions in-context
- text-first fallback preserves complete reading continuity

### 24.2 Launch+1 Done
Launch+1 is complete when:
- entity indexes and deep links are stable
- cross-reference and resume/bookmark workflows are reliable
- sync controls and accessibility coverage pass expanded tests

### 24.3 Strategic Success Signal
The project is on-track for enterprise-level success when readers can move from passive reading to active historical reasoning, and every spatial claim in the interface remains transparent, testable, and source-backed.
