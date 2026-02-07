# War With Hannibal — Interactive Map Reader (PLAN_CODEX.md)

## 1) Strategic Intent
Build the best practical digital companion for Livy's war narrative, starting with **Book 21**. The product must help readers answer spatial and tactical questions instantly while preserving scholarly trust through explicit sourcing and uncertainty.

This is a reading product first and a map product second:
- The map exists to explain the text.
- The text flow must remain stable and readable.
- Historical ambiguity is shown, not hidden.

## 2) Vision, Product Principles, and Non-Goals

### Vision
When a reader sees "Hannibal moved from X to Y," they should immediately understand:
- where X and Y are,
- how the move likely happened,
- what constraints mattered (river, pass, season),
- how certain or disputed the reconstruction is,
- and what sources support the claim.

### Product principles
- Reading-first UX: text clarity wins when map and text conflict.
- Canonical content model: chapter-level stable IDs survive reformatting.
- Explicit provenance: map-assertive claims require citations and confidence.
- Fast by default: first readable text is independent of map load.
- Honest uncertainty: no fake precision.

### Non-goals (v1)
- Not a total war simulator.
- No multiplayer, comments, or public editing.
- No attempt to resolve all scholarly disputes.
- No mandatory account system.

## 3) What Success Looks Like

### Reader outcomes
- Readers can follow Book 21 without getting lost geographically.
- Readers can click places/people/forces and immediately get context.
- Scene transitions feel predictable and non-jarring.

### Editorial outcomes
- New chapter onboarding is repeatable: ingest -> annotate -> scene -> validate -> publish.
- Editors can review every claim with confidence metadata and citations.

### System outcomes
- Stable deep links remain valid across content revisions.
- Data errors are caught in CI before deployment.
- Map assets load progressively and do not block reading.

## 4) Audience and Use Cases

### Primary audience: general history readers
Jobs to be done:
- "Show me where this happened."
- "Help me follow army movement without opening a separate atlas."

### Secondary audience: students and teachers
Jobs to be done:
- "Step through a chapter in class with clear visuals."
- "Share a link to a claim with source context."

### Tertiary audience: digital humanists
Jobs to be done:
- "Inspect data structure and provenance."
- "Reuse model for other historical texts."

## 5) Scope and Release Shape

### MVP (v1)
- Book 21 only.
- Chapter reader + synchronized map scenes.
- Clickable place/person/force annotations.
- Force markers and route overlays with uncertainty styling.
- Source/citation display for non-trivial assertions.
- Mobile-compatible reader/map interaction.

### v1.5
- Cross-chapter entity index for Book 21.
- Better scene editing ergonomics.
- Optional present mode improvements.

### v2
- Books 22-30.
- Optional user accounts (bookmarks/progress sync).
- Community contribution workflow.

## 6) Licensing and Text Strategy

### Text licensing
- Use a **public-domain English translation** for v1 publication.
- Store source metadata per chapter: translator, edition year, URL/source.
- Keep text layer swappable so future translations can reuse canonical annotations/scenes.

### Companion mode (future)
- If using copyrighted translations later, support a companion mode that references locations/sections without reproducing copyrighted full text.

### Map and data licensing
- Natural Earth (public domain) for physical geography baseline.
- OpenStreetMap-derived data only where licensing obligations are met.
- Pleiades alignment when useful for authority IDs.
- Attribution manifest generated at build time.

## 7) Information Architecture (Canonical-First)

The core architectural decision is unchanged and remains correct: **canonical data is chapter-based with stable block IDs**.

### 7.1 Canonical entities
- Place
- Person
- Force

Required properties by type:
- Place: `place_id`, `display_name`, `alt_names`, geometry, confidence, sources.
- Person: `person_id`, names, roles, faction, optional dates, sources.
- Force: `force_id`, faction, commander links, optional size/composition, sources.

### 7.2 Chapter as canonical unit
Each chapter file contains:
- `blocks`: stable narrative units.
- `annotations`: references from block selectors to entities.
- `scenes`: ordered map states tied to narrative moments.

### 7.3 Presentation mapping
`Book.Chapter.Section` is a view layer, not canonical storage:
- mapping files group block IDs into display sections.
- section boundaries can change without breaking annotations.

### 7.4 ID and version policy
- Book/chapter: `21.05`
- Block: `21.05.p01`
- Annotation: `21.05.a12`
- Scene: `21.05.s03`

Rules:
- IDs never recycled.
- Data package semantic versioning.
- Breaking changes only when IDs or schemas break compatibility.
- Changelog required for location/route hypothesis changes.

## 8) Data Model (Detailed)

### 8.1 Chapter schema (conceptual)
```json
{
  "chapter_id": "21.05",
  "source": {
    "translation_id": "roberts-1912",
    "title": "The History of Rome",
    "translator": "Canon Roberts",
    "year": 1912,
    "license": "public-domain",
    "url": "..."
  },
  "blocks": [
    {
      "block_id": "21.05.p01",
      "text": "..."
    }
  ],
  "annotations": [
    {
      "annotation_id": "21.05.a04",
      "anchor": {
        "block_id": "21.05.p01",
        "selector": {
          "quote": "Saguntum",
          "startOffset": 123,
          "endOffset": 131
        }
      },
      "entity": {
        "type": "place",
        "id": "saguntum"
      },
      "confidence": "high",
      "sources": [
        "Livy 21.5",
        "Pleiades 246637"
      ],
      "note": "Widely accepted site near modern Sagunto."
    }
  ],
  "scenes": [
    {
      "scene_id": "21.05.s02",
      "anchor": {
        "block_id": "21.05.p03"
      },
      "viewport": {
        "lng": 1.1,
        "lat": 41.0,
        "zoom": 5.8,
        "bearing": 0,
        "pitch": 0
      },
      "layers": [
        {
          "type": "highlight_place",
          "place_id": "saguntum",
          "confidence": "high",
          "sources": ["Livy 21.5"]
        }
      ],
      "narrative_note": "Roman and Carthaginian strategic focus converges on Saguntum."
    }
  ]
}
```

### 8.2 Scene layer vocabulary (opinionated minimal set)
- `highlight_place`
- `force_marker`
- `route`
- `battle_area`
- `label`
- `note`

All non-trivial layers include:
- `confidence`: `high|medium|low`
- `sources`: citation array
- optional `note`

### 8.3 View mapping schema
```json
{
  "chapter_id": "21.05",
  "sections": [
    {
      "label": "21.5.1",
      "block_ids": ["21.05.p01", "21.05.p02"]
    }
  ]
}
```

### 8.4 Gazetteer structure
- `data/gazetteer/places.geojson`
- `data/gazetteer/people.json`
- `data/gazetteer/forces.json`

Recommended place properties:
- `place_id`
- `display_name`
- `alt_names`
- `authority_refs` (e.g., Pleiades)
- `confidence`
- `sources`
- `editorial_note`

## 9) Editorial Standards and Workflow

### 9.1 Editorial conventions (must exist before scaling)
Define and publish:
- naming and transliteration style,
- citation format,
- confidence criteria,
- scene granularity rules,
- ambiguous toponym handling.

### 9.2 Manual-first pipeline (v1)
1. Ingest chapter text to canonical blocks.
2. Annotate place mentions, then people, then forces.
3. Create scenes at narrative pivots.
4. Add confidence and sources for map assertions.
5. Validate and review.

### 9.3 Internal editor (v1.5)
Build behind dev flag:
- text span annotation tool,
- entity picker with fuzzy search,
- map-driven scene builder,
- viewport capture,
- route drawing and editing,
- PR-friendly change summary.

### 9.4 Review checklist per chapter
- Coverage: major places and actors annotated.
- Every scene claim has source and confidence.
- No dangling IDs.
- No unexplained speculative geometry.
- Mobile and accessibility pass completed.

## 10) Historical Map Strategy

### 10.1 Basemap intent
Use an ancient-friendly physical map that avoids modern political and urban clutter.

### 10.2 Data strategy
- Base: Natural Earth physical layers.
- Curated overlays: ancient places/routes relevant to Book 21.
- Historical corrections (iterative):
  - Po delta/coastal context where materially relevant.
  - Alpine pass emphasis where routes are debated.
  - Key river crossings and strategic corridors.

### 10.3 Rendering stack
- Map engine: MapLibre GL JS.
- Tile packaging: PMTiles.
- Hosting: static CDN with HTTP range requests.

### 10.4 Label strategy
- Default labels from project gazetteer only.
- Optional modern reference hints in cards or toggle.
- No modern country borders in default style.

### 10.5 Zoom strategy
- 0-3: basin overview.
- 4-6: regional campaigns.
- 7-10: battle corridor and pass detail.

## 11) Reader Experience and Interaction Design

### 11.1 Desktop layout
- Split reader/map.
- Reader remains primary focus width.
- Scene controls and sync status always visible.

### 11.2 Mobile layout
- Reader-first layout with map drawer/sheet.
- Map can be expanded for investigation, collapsed for reading continuity.
- Scene controls remain reachable without precision tapping.

### 11.3 Core interactions
- Click text annotation -> map highlight + entity card.
- Click scene step -> synchronized text anchor and map transition.
- Reset view returns to canonical scene viewport.
- Present mode enlarges map + large controls for teaching.

### 11.4 Accessibility and motion
- Keyboard support for scene stepping and entity focus.
- Screen-reader labels for annotations and map alternatives.
- `prefers-reduced-motion` disables large animations.

## 12) Reader-Map Sync Engine (Detailed)

### 12.1 Sync modes
- Auto-sync (default)
- Manual scene mode
- Annotation-only sync
- Sync disabled

### 12.2 Activation rules
- Scene becomes candidate when its anchor crosses top third of viewport.
- Hysteresis threshold prevents rapid oscillation.
- Debounce rapid scrolling.
- User interaction interrupts auto animation immediately.

### 12.3 Practical defaults
- Scene switch threshold: ~120-180px
- Scroll debounce: ~250-350ms
- Scene transition duration: 800-1200ms depending on trigger

### 12.4 State model
Single shared store (e.g., Zustand):
- reading state: chapter, block, scroll position
- map state: active scene, viewport, animation status
- sync prefs: mode, speed, pause

### 12.5 Deep linking
- `#block=21.05.p03`
- `#scene=21.05.s02`
- `?ref=21.5.3` resolved via view mapping

Deep links restore both text and map state.

## 13) Visual Storytelling System

These additions materially improve comprehension and engagement when implemented with strict quality controls.

### 13.1 Timeline ribbon (v1)
- show consular year and campaign season,
- show concrete dates only when confidence supports it,
- visually mark uncertainty.

### 13.2 Battle sequence diagrams (v1 for major engagements)
- 3-5 step schematic overlays for key battles in covered books,
- concise tactical explanation per step,
- explicit source basis and uncertainty.

### 13.3 Army composition visuals (v1.5)
- force cards include composition ranges and role summaries,
- diagrams clarify terms like legions, cavalry mix, elephants.

### 13.4 Character portrait system (v2 path)
- do not block v1 on commissioned art,
- support portraits where rights and historical basis are clear,
- include attribution and provenance metadata.

### 13.5 Asset policy
- SVG for diagrams.
- WebP/PNG for portraits.
- Lazy load all non-critical assets.
- Keep attribution data machine-readable.

## 14) Cross-Referencing, Search, and Reading Progress

### 14.1 Entity-centric navigation
- Entity pages aggregate mentions, scenes, and related references.
- "See also" links help readers jump to connected passages.

### 14.2 Cross-reference graph (v1.5)
- Precomputed links:
  - temporal (earlier/later mentions),
  - spatial (nearby/co-located events),
  - thematic (related tactical/political contexts).

### 14.3 Search
- by place/person/force name,
- by chapter/section ref,
- by source citation.

### 14.4 Progress and bookmarks
- v1: local storage resume state.
- v2: optional account sync for bookmarks/notes.
- preserve reader + map state in resume snapshot.

## 15) Technical Architecture

### 15.1 Stack
- Next.js + TypeScript
- MapLibre GL JS
- PMTiles
- shared state store (Zustand or equivalent)
- schema validation (Zod or JSON Schema)

### 15.2 Project structure (target)
```text
/data
  /books/21/chapters/*.json
  /books/21/views/default/*.json
  /gazetteer/places.geojson
  /gazetteer/people.json
  /gazetteer/forces.json
/public
  /tiles/war-with-hannibal.pmtiles
  /visual-assets/...
/scripts
  ingest/
  validate/
  compile/
/src
  app/
  components/
  lib/
```

### 15.3 Architecture constraints
- Static-first delivery.
- Data-driven scene rendering.
- Build fails on invalid content.
- No runtime dependence on fragile ad-hoc parsing.

## 16) Data Compilation Pipeline

At build time:
1. Validate schemas.
2. Validate cross-references.
3. Validate geospatial bounds and geometry sanity.
4. Build chapter/entity indexes.
5. Precompute route lengths and scene bounds.
6. Emit optimized per-chapter bundles.
7. Generate attribution manifest and data changelog.

Optional derived outputs:
- search index,
- entity mention index,
- scene timeline index.

## 17) Testing and Quality Gates

### 17.1 Unit tests
- schema validators,
- id and reference checks,
- route/distance calculations,
- view mapping resolvers.

### 17.2 Integration tests
- chapter render with annotations,
- scene -> map layer derivation,
- deep-link restore behavior.

### 17.3 E2E tests (Playwright)
- open chapter,
- click annotation and assert map update,
- step scenes and verify layer changes,
- load deep-link URL and verify restored state.

### 17.4 Content QA gates
A chapter cannot ship unless:
- all referenced entities exist,
- all non-trivial scene layers include confidence/sources,
- required accessibility labels exist,
- mobile layout smoke test passes.

## 18) Performance Budgets and Reliability

### 18.1 Performance budgets
- first readable text: <2s on good connection,
- map initialization: progressive, non-blocking,
- scene transition frame rate: target 60fps, floor 30fps,
- chapter payload budget enforced by CI.

### 18.2 Reliability
- robust fallback when map fails to initialize,
- deterministic scene state from URL,
- error boundaries around map and card subsystems.

### 18.3 Observability
Track:
- chapter completion,
- annotation click-through,
- scene navigation usage,
- error rates by route/component.

## 19) Chapter Delivery Strategy (Pragmatic)

### 19.1 Pilot-first sequence
Start with a small high-value subset of Book 21 chapters that stress core capabilities:
- one chapter centered on long-range movement (Iberia -> Gaul corridor),
- one chapter centered on Alpine crossing uncertainty,
- one chapter centered on first major Italian engagements.

Then complete remaining chapters in order.

### 19.2 Chapter completion criteria
- >=90% meaningful place mentions annotated.
- all principal figures annotated.
- all major movement events have scenes.
- disputed geographies explicitly flagged.

### 19.3 Release cadence
- weekly or biweekly chapter drops,
- each release includes fixes + one measurable quality improvement.

## 20) Feedback and Iteration Workflow

### 20.1 Feedback channels
- inline error report links,
- chapter-level survey after completion,
- usage analytics for friction detection.

### 20.2 Triage policy
- factual/data errors: hotfix target <=48h
- UX clarity issues: next patch
- feature requests: roadmap queue by impact

### 20.3 Versioning
- app version and data package version both visible.
- changelog entries for historical interpretation changes.

## 21) Deployment and Operations

### 21.1 Hosting
- static-capable frontend host (Vercel/Netlify/Cloudflare Pages).
- PMTiles and large assets served via CDN.

### 21.2 Delivery requirements
- HTTP range requests enabled for PMTiles.
- strong caching with content-hash invalidation.
- preview deployments for editorial QA.

### 21.3 Security and governance
- no secrets in client bundles,
- content-only PR permissions for editors where possible,
- signed release tags for reproducibility.

## 22) Growth and Distribution (Post-MVP)

### 22.1 Launch channels
- history and digital-humanities communities,
- technical communities interested in mapping + data journalism,
- educator outreach with present-mode examples.

### 22.2 SEO and discoverability
- static chapter pages,
- place/person pages as entry points,
- rich metadata for share previews.

### 22.3 Contribution model (v2)
- contribution guidelines for data edits,
- review rubric for uncertain claims,
- templates for source citations and route hypotheses.

## 23) Risk Register and Mitigations

- Text rights risk -> public-domain text first, companion mode later.
- Geographic dispute risk -> confidence + citations + explicit alternatives.
- Scope creep risk -> Book 21 gate, chapter completion criteria.
- Performance risk -> static-first, PMTiles, strict budgets.
- Editorial inconsistency risk -> style guide + review checklist + CI validation.
- Mobile complexity risk -> reader-first drawer model and device testing.

## 24) Milestones

### Phase 0: Foundations (1-2 weeks)
- repo setup,
- schema scaffolding,
- reader skeleton,
- map pane with sample data.

### Phase 1: Core MVP (3-6 weeks)
- ingest and model Book 21,
- gazetteer baseline,
- annotation pipeline,
- scene system,
- source/confidence UI.

### Phase 2: Quality and Usability (2-4 weeks)
- sync refinements,
- accessibility pass,
- performance tuning,
- validation and E2E hardening.

### Phase 3: Editorial Leverage (2-4 weeks)
- dev-only annotation/scene editor,
- release checklist automation,
- cross-reference index.

### Phase 4: Expansion (ongoing)
- Books 22-30,
- optional accounts,
- contribution workflow,
- advanced visuals.

## 25) Immediate Next Actions
1. Lock translation source and record source metadata schema.
2. Freeze chapter/annotation/scene schemas and ID policy.
3. Build one full pilot chapter from ingestion to publish with all gates on.
4. Measure reader flow and tune sync thresholds.
5. Scale to full Book 21 with chapter release cadence.

## 26) Explicit v1 Defaults
- Text blocks are paragraph-based stable units.
- All map labels come from project gazetteer/scenes.
- Every map-assertive layer includes confidence + sources.
- Reader remains usable even if map fails.
- Historical honesty overrides visual spectacle.
