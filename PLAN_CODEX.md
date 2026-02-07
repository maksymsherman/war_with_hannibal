# War With Hannibal - V1 Execution Plan

## 1) Product Goal
Build a reading companion for Livy Book 21 where users can understand who is where, who moved, and why it matters without opening external maps.

Primary user promise:
- While reading any passage, users can immediately see the relevant geography.
- Users can track movements and actions of all major actors, not only Hannibal.
- Users can distinguish secure facts from uncertain reconstructions.

## 2) V1 Scope
In scope:
- Livy Book 21 only.
- Chapter reader with synchronized map context.
- Entity linking for places, people, groups/polities, and military forces.
- Scene steps for major narrative events (movement, siege, battle, diplomacy).
- Source and confidence on map-assertive claims.
- Mobile support with reader-first behavior.

Out of scope for v1:
- Accounts, comments, public editing.
- Full editor UI.
- Community contribution workflow.
- Character portraits and advanced battle diagrams.
- Books 22-30.

## 3) User-Centered Requirements (Must Ship)
1. Read text comfortably on desktop and mobile.
2. Tap/click any annotated mention and see map focus + context card.
3. Step through narrative events with clear map changes.
4. See all relevant actors in each event (Roman, Carthaginian, allied, local groups).
5. See confidence and sources whenever the map reflects interpretation.
6. Continue reading even if map fails to load.

## 4) Success Criteria
A new user can complete one chapter and correctly explain:
- Which actors are involved.
- Where key events occur.
- What moved between locations.
- Which points are disputed or uncertain.

Operational targets:
- First readable text in under 2 seconds on a good connection.
- No broken deep links after data-only updates.
- All shipped scenes have source and confidence fields.

## 5) Canonical Content Model (Minimal but Sufficient)

### 5.1 Chapter unit
Each chapter is canonical and stored as:
- `source`
- `blocks`
- `annotations`
- `events`
- `scenes`

### 5.2 Stable IDs
- Chapter: `21.05`
- Block: `21.05.b03`
- Annotation: `21.05.a14`
- Event: `21.05.e04`
- Scene: `21.05.s04`

Rules:
- IDs are never recycled.
- New IDs are appended when splitting/adding blocks.
- Historical interpretation changes require changelog notes.

### 5.3 Entities (v1)
Use four entity types:
- `place`
- `person`
- `polity` (tribe, city-state, people, confederation, senate/body politic)
- `force` (army/detachment/fleet)

Why this is required:
- Book 21 frequently names peoples/groups (example: Olcades, Vaccaei, Carpetani) that are not cleanly place/person/force.

### 5.4 Event model
`events` are narrative beats users step through.
Each event should include:
- `event_id`
- `block_ids` (one or more)
- `title`
- `summary`
- `actors` (entity references)
- `event_type` (`movement|battle|siege|diplomacy|logistics|political|other`)
- `sources`
- optional `time_note`

### 5.5 Scene model
Each event may have one primary scene.
A scene includes:
- `scene_id`
- `event_id`
- `anchor_block_id`
- `viewport`
- `layers`

Layer types (v1):
- `highlight_place`
- `route`
- `force_marker`
- `battle_area`
- `label`

All non-trivial layers must include:
- `confidence` (`high|medium|low`)
- `sources`
- optional `note`

## 6) Text and Anchor Strategy

### 6.1 Translation policy
Use one public-domain English translation for v1 publication, with full metadata:
- `translation_id`
- `title`
- `translators` (array, not single value)
- `edition_or_release`
- `license`
- `url`
- `retrieved_at`

Initial source to use now:
- Project Gutenberg eBook #10907 (`https://www.gutenberg.org/ebooks/10907`)

### 6.2 Blocking policy
Do not use raw paragraph as the only block strategy.
Use narrative beats:
- target 80-220 words per block,
- split long paragraphs at clear event pivots,
- preserve chapter order and reference mapping.

### 6.3 Anchoring policy
Avoid offset-only anchors.
Use resilient anchors:
- primary: `block_id` + normalized quote snippet,
- secondary: token window checksum,
- optional offsets for rendering optimization only.

Rationale:
- offsets break across text revisions and translation swaps.

## 7) Map Data Strategy (Practical)

### 7.1 Basemap
- Ancient-friendly physical map style.
- No default modern political borders.
- Modern reference labels optional in entity cards/toggle.

### 7.2 Gazetteer requirements
`places.geojson` place entries must support uncertainty:
- `place_id`
- `display_name`
- `alt_names`
- `authority_refs` (Pleiades etc.)
- `geometry_candidates` (one or many)
- `preferred_geometry_index`
- `confidence`
- `sources`
- `editorial_note`

### 7.3 Routes and uncertain geography
For disputed routes/locations:
- store alternative hypotheses explicitly,
- render alternatives with reduced emphasis,
- explain basis in scene notes.

## 8) Reader and Map Experience

### 8.1 Desktop
- Split view, reader prioritized.
- Sticky event controls.
- Active event badge visible at all times.

### 8.2 Mobile
- Reader-first with expandable map sheet.
- Event controls remain thumb-reachable.
- Map defaults to current event focus when opened.

### 8.3 Interaction contract
- Click annotation -> focus map and open entity card.
- Click event step -> scroll to event blocks and update map scene.
- Reset -> canonical scene viewport.
- If map fails -> show event summaries and linked place cards in text-only mode.

## 9) Sync Behavior (Simple for v1)
Use only two modes:
- `auto` (default)
- `manual`

Auto mode rules:
- Active event updates when its anchor block crosses upper viewport threshold.
- Debounce scroll updates.
- User map interactions pause auto updates briefly.

Manual mode rules:
- Event controls drive scene changes.
- Scroll does not alter active scene unless user re-enables auto.

## 10) Editorial Workflow (Manual-First)
Per chapter workflow:
1. Ingest chapter text and split into narrative blocks.
2. Annotate places, people, polities, forces.
3. Define events and actor participation.
4. Build one scene per event (plus optional secondary scenes).
5. Add confidence and sources for all interpretive map layers.
6. Run validation and chapter QA checklist.
7. Publish.

## 11) QA Gates (Ship Blockers)
A chapter cannot ship unless all checks pass:
1. No dangling entity references.
2. All events have at least one actor.
3. All event-linked scenes resolve.
4. All non-trivial layers include confidence + sources.
5. All disputed geographies are flagged.
6. Annotation and event controls are keyboard accessible.
7. Mobile smoke test passes.
8. Text-only fallback is functional.

## 12) Technical Architecture
Stack:
- Next.js + TypeScript
- MapLibre GL JS
- PMTiles (or static GeoJSON for earliest pilot)
- Zod for schema validation
- Zustand (or equivalent) shared UI state

Target structure:
```text
/data
  /books/21/chapters/*.json
  /books/21/views/default/*.json
  /gazetteer/places.geojson
  /gazetteer/people.json
  /gazetteer/polities.json
  /gazetteer/forces.json
/public
  /tiles/war-with-hannibal.pmtiles
/scripts
  /ingest
  /validate
  /compile
/src
  /app
  /components
  /lib
```

Constraints:
- Static-first delivery.
- Build fails on invalid content.
- Data-driven rendering only.

## 13) Implementation Roadmap

### Phase 0 (Week 1): Foundations
Deliverables:
- Freeze schemas for chapter/entities/events/scenes.
- Lock translation metadata model (`translators` array).
- Build minimal reader + map shell.
- Implement text-only fallback state.

Exit criteria:
- App loads one stub chapter with one stub scene.

### Phase 1 (Weeks 2-3): One Complete Pilot Chapter
Deliverables:
- Ingest one Book 21 chapter end to end.
- Full annotation for places/people/polities/forces.
- Event timeline and scene stepping.
- Confidence/source UI.
- Validation scripts for schema + reference integrity.

Exit criteria:
- User can read and map-follow the chapter without external atlas.

### Phase 2 (Weeks 4-5): Core Book 21 Pipeline
Deliverables:
- Repeatable chapter onboarding scripts/workflow.
- Deep links for block, event, and scene.
- Performance pass (bundle size, map init, interaction latency).
- Accessibility pass (keyboard, labels, reduced motion).

Exit criteria:
- Multiple chapters shipped with consistent QA results.

### Phase 3 (Week 6+): Completion and Hardening
Deliverables:
- Remaining Book 21 chapters.
- Regression test suite (unit + integration + E2E smoke).
- Changelog discipline for interpretation updates.

Exit criteria:
- Book 21 complete and stable for public release.

## 14) Metrics to Track
Product metrics:
- Event step usage rate.
- Annotation click-through rate.
- Chapter completion rate.
- Drop-off by chapter and event index.

Quality metrics:
- Validation failures per chapter PR.
- Map load error rate.
- Text-only fallback usage rate.

## 15) Immediate Next Actions
1. Freeze schema files for chapter, entities, events, and scenes.
2. Convert one real chapter (recommend 21.5 or 21.6) into narrative blocks.
3. Produce first actor-complete event map for that chapter.
4. Run QA gates and tune UX based on observed friction.
5. Scale chapter by chapter through Book 21.

## 16) v1 Decision Log (Hard Rules)
- The product is reading-first.
- All major actors are represented, not only Hannibal.
- Event-level mapping is the primary comprehension tool.
- Uncertainty is shown explicitly, never hidden.
- Shipping a smaller, reliable v1 is preferred over feature breadth.
