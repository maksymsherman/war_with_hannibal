# War With Hannibal - Source-Locked V1 Execution Plan

## 1) Objective
Deliver a Book 21 reading site where users can follow all major actors (Roman, Carthaginian, allied, local) with map context in the same reading flow.

If a user asks while reading, they must get answers immediately:
- Who is acting in this passage?
- Where is this happening?
- What moved from where to where?
- How certain is this reconstruction?

## 2) Non-Negotiable Product Rules
1. Reading-first: text must remain fully usable if map fails.
2. Actor-complete: not Hannibal-only; include all major actors in every event.
3. Event-first map logic: map follows narrative events, not raw paragraphs.
4. Explicit uncertainty: interpretive geography always shows confidence + source.
5. Deterministic content pipeline: identical input produces identical chapter JSON.

## 3) Source Lock (Verified)
Primary source for v1 publication:
- Project Gutenberg eBook #10907
- URL: `https://www.gutenberg.org/ebooks/10907`
- Local file: `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`

Observed facts from downloaded file:
- File has UTF-8 BOM at start.
- Gutenberg boundaries are present:
  - `*** START OF THE PROJECT GUTENBERG EBOOK ... ***`
  - `*** END OF THE PROJECT GUTENBERG EBOOK ... ***`
- Book XXI heading present and Book XXII heading present.
- Book XXI chapter markers are sequential `1.` through `63.`.
- Book XXI contains 9 inline `[Footnote: ...]` notes.
- Text is hard-wrapped near ~70 characters.

Integrity fingerprints (current):
- `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`
  - `ff3ede761458ec93d989a18384ad485715fae8b0f6c822cca0f44e874be9430e`
- `data/sources/livy/book_21_pg10907.txt`
  - `33a23b3e36482a4cd23a0c51a730b366c1e0ec399a658acc5e8c3223b0282f3d`
- `data/sources/livy/book_21_chapter_05_pg10907.txt`
  - `e8ee3451ffecf80cba074877eefa11a1f9049ab3570911530a1ab8da5733bd33`

## 4) True V1 Scope
In scope:
- Book 21 only (63 chapters).
- Reader + synchronized map per chapter.
- Four entity classes: place, person, polity, force.
- Event timeline per chapter.
- Source/confidence UI for interpretive map layers.
- Mobile reader-first layout.

Out of scope:
- Accounts, comments, social features.
- Public editor.
- Cross-book expansion.
- Portrait systems and advanced visual extras.

## 5) Canonical Data Contracts

### 5.1 Chapter JSON
Required top-level fields:
- `chapter_id`
- `source`
- `blocks`
- `annotations`
- `events`
- `scenes`

### 5.2 IDs
- Chapter: `21.05`
- Block: `21.05.b07`
- Annotation: `21.05.a19`
- Event: `21.05.e04`
- Scene: `21.05.s04`

Rules:
- IDs never recycled.
- New IDs appended when content is split or inserted.
- All IDs must be globally unique within chapter namespace.

### 5.3 Entity types (required)
- `place`
- `person`
- `polity` (tribes/peoples/civic bodies/confederations)
- `force` (field army/fleet/detachment)

### 5.4 Source metadata shape
Use array for translators.

```json
{
  "translation_id": "pg10907-spillan-edmonds",
  "title": "The History of Rome, Books 09 to 26",
  "translators": ["D. Spillan", "Cyrus R. Edmonds"],
  "edition_or_release": "Project Gutenberg eBook #10907",
  "release_date": "2004-02-01",
  "last_updated": "2024-10-28",
  "license": "public-domain",
  "url": "https://www.gutenberg.org/ebooks/10907",
  "retrieved_at": "YYYY-MM-DD"
}
```

## 6) Deterministic Ingestion Pipeline (Must Be Implemented Exactly)

### Step A - Acquire and verify
Input file path is fixed:
- `data/sources/livy/livy_books_09_to_26_spillan_edmonds_pg10907.txt`

Checks:
1. File exists.
2. SHA matches expected fingerprint unless intentionally updated.
3. Encoding is UTF-8 and BOM is removed in normalized output.

### Step B - Strip Gutenberg wrapper
Extract only text between START and END markers.
Fail if either marker is missing.

### Step C - Extract Book XXI
Book slice boundaries:
- start line matching `^BOOK XXI\.$`
- end line matching `^BOOK XXII\.$` (exclusive)

Fail if boundaries missing or inverted.

### Step D - Normalize wrapped text
Rules:
1. Convert hard-wrapped lines into paragraph strings.
2. Preserve blank-line paragraph boundaries.
3. Keep punctuation spacing normalized.
4. Keep chapter header numbers (`1.`, `2.`, ... `63.`).
5. Preserve footnote content for provenance, but detach from primary reading text (see Step F).

### Step E - Split chapters
Regex anchor:
- `^(\d+)\.\s`

Validation:
1. Exactly 63 chapters.
2. Sequence strictly 1..63.
3. No duplicates, no gaps.

### Step F - Footnote policy
For v1 reading flow:
- Remove inline `[Footnote: ...]` blocks from main text body.
- Store extracted notes in `chapter.notes[]` with anchor to `block_id` + quote snippet.

Rationale:
- Keeps reading/map sync clean while preserving provenance data.

### Step G - Blocking policy (critical)
Do not use raw paragraph blocks only.

Target block sizing:
- 80-220 words preferred.
- Hard maximum: 280 words.

Split triggers:
- chapter sentence boundaries.
- discourse pivots (new actor/action/location).
- transition phrases (`then`, `after`, `meanwhile`, `in the meantime`, `having`, `whereupon`).

Validation:
- no empty blocks.
- block order strictly monotonic.
- >95% blocks within size target.
- any oversize block must include justification flag.

### Step H - Event extraction
Each chapter gets event beats that users step through.

Event requirements:
- `event_id`
- `block_ids[]`
- `title`
- `summary`
- `event_type`
- `actors[]`
- `sources[]`

Event types:
- `movement`
- `battle`
- `siege`
- `diplomacy`
- `logistics`
- `political`
- `other`

Validation:
- every event references >=1 block.
- every event has >=1 actor.
- every block belongs to >=1 event.

### Step I - Scene build
Each event has one primary scene.

Required scene fields:
- `scene_id`
- `event_id`
- `anchor_block_id`
- `viewport`
- `layers[]`

Layer types in v1:
- `highlight_place`
- `route`
- `force_marker`
- `battle_area`
- `label`

For `route`, `force_marker`, `battle_area`, `label`:
- `confidence` required.
- `sources` required.

## 7) Gazetteer Strategy

### 7.1 Files
- `data/gazetteer/places.geojson`
- `data/gazetteer/people.json`
- `data/gazetteer/polities.json`
- `data/gazetteer/forces.json`

### 7.2 Place uncertainty model
Each place can have one or more geometry candidates.

Required keys:
- `place_id`
- `display_name`
- `alt_names`
- `authority_refs`
- `geometry_candidates[]`
- `preferred_geometry_index`
- `confidence`
- `sources`
- `editorial_note`

### 7.3 Actor completeness rules
If a chapter mentions a major actor in an event summary, the actor must appear in `events[].actors[]` and at least one scene layer or entity card.

## 8) UI Behavior Spec (v1)

### Reader
- Clean chapter text.
- Inline annotation highlights.
- Event stepper fixed in viewport.

### Map
- Reflect active event, not just scroll position.
- Reset button restores canonical event viewport.
- Entity card shows: name, type, role in current event, source, confidence.

### Sync modes
- `auto` (default): active event updates on anchor threshold crossing.
- `manual`: stepper controls event; scrolling does not auto-switch.

### Failure mode
If map initialization fails:
- show text and event summaries.
- show place cards with textual location context.
- keep deep links working for block and event.

## 9) QA Gates (Release Blockers)
Chapter fails release if any rule fails:
1. Schema validity pass.
2. Cross-reference validity pass.
3. 63-chapter book integrity pass (Book 21 dataset).
4. No dangling entity/event/scene IDs.
5. All interpretive layers include confidence + sources.
6. All events have actor coverage.
7. Disputed geographies flagged.
8. Keyboard support for annotation and event controls.
9. Mobile smoke test pass.
10. Text-only fallback pass.

## 10) CI Validation Commands (to implement)
- `npm run validate:source` - checks source hash + boundary markers.
- `npm run ingest:book21` - emits canonical chapter JSON.
- `npm run validate:data` - schema + reference + integrity checks.
- `npm run test:integration` - chapter render + scene sync.
- `npm run test:e2e:smoke` - open chapter, click annotation, step events, deep-link restore.

Pipeline order:
1. `validate:source`
2. `ingest:book21`
3. `validate:data`
4. tests
5. publish

## 11) Pilot Chapter Blueprint (21.5)
Use chapter 21.5 as the first full template because it includes:
- multi-actor operations,
- movement and combat,
- explicit geography,
- quantitative military claims.

Minimum event set for 21.5:
1. Hannibal commits to war strategy toward Saguntum.
2. Olcades campaign and capture of Carteia.
3. Winter at New Carthage and spring campaign vs Vaccaei.
4. Hermandica/Arbocala actions.
5. Carpetani coalition engagement near Tagus.
6. River-crossing tactical setup and cavalry strike.
7. Carthaginian victory and regional submission beyond Iberus.

Pilot completion criteria:
- all events mapped,
- all actors represented,
- sources/confidence complete,
- user can narrate chapter sequence without external map.

## 12) Execution Timeline (Realistic)

Week 1:
- Freeze schemas and validators.
- Implement source verification + ingestion for Book XXI.
- Emit canonical JSON for one chapter (21.5).

Week 2:
- Complete annotations/events/scenes for 21.5 and 21.6.
- Build stepper + map sync + text-only fallback.
- Add integration tests.

Week 3:
- Scale to first 10 chapters.
- Tune performance and mobile interaction.
- Harden QA gates.

Week 4+:
- Complete remaining Book 21 chapters.
- Stabilize release quality and changelog process.

## 13) Risk Controls
- Translation drift risk: source hash check in CI.
- Anchor fragility risk: block + quote + checksum anchoring.
- Overlong prose risk: enforced block size validation.
- Ambiguous geography risk: candidate geometry model + explicit confidence.
- Scope creep risk: strict v1 scope and blocker gates.

## 14) Immediate Next Steps
1. Implement `validate:source` with current known hashes.
2. Implement `ingest:book21` with deterministic chapter split and footnote extraction.
3. Generate canonical JSON for chapter 21.5.
4. Author events/scenes for 21.5 using pilot blueprint.
5. Run full QA gates before adding more chapters.

## 15) Definition of V1 Done
V1 is done when:
- all 63 Book 21 chapters are published in canonical JSON,
- every chapter passes QA gates,
- reader+map flow answers actor/location/movement questions without external atlas,
- map failure mode still provides complete reading continuity.
