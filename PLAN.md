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
- "Scenes" tied to the narrative:
  - Prev/Next scene controls.
  - Optional scroll-sync: as the user scrolls, the active scene updates deterministically.
- Forces + routes are part of v1:
  - Roman/Carthaginian force markers.
  - March routes as polylines (with clear uncertainty signaling).
- Visual storytelling aids:
  - Character portraits integrated with person annotations.
  - Army composition visualizations for major forces.
  - Battle sequence diagrams for key engagements.
  - Timeline context for narrative events.

## Key Design Principle: Canonical Data Is Chapter-Level
We want **Book.Chapter.Section** in the UI, but we do *not* want the underlying data model to become dependent on the final “sectioning” scheme.

Plan:
- Store canonical information per **chapter** as stable paragraph blocks.
- Store anchors as “block + selector” (quote/offset-based).
- Treat `Book.Chapter.Section` as a **view** derived from a mapping file that can change later.

This keeps the implementation fluid: you can re-split text into sections without breaking annotations/scenes.

## Visual Storytelling System

To make ancient history accessible to general audiences, the application will include rich visual aids that support and enhance the narrative without overwhelming the text. These visual elements serve as entry points for readers unfamiliar with Roman military structure, ancient geography, or historical context.

### Character Portrait System
Character portraits appear as small circular avatars next to person annotations and expand into detailed cards when clicked. The portrait system serves multiple functions:

**Portrait Sources and Standards:**
- Commission original artwork in a consistent classical style (think Edward Gibbon-era engravings)
- 400x400px base resolution, optimized WebP with PNG fallbacks
- Neutral historical styling that avoids both Hollywood glamour and modern anachronisms
- Include 2-3 alternate views per major character (profile, full-body for military leaders)

**Integration Points:**
- Small avatar (32px) appears inline with first mention of a character in each chapter
- Larger portrait (120px) in force info cards when that person commands the unit
- Full portrait gallery accessible via character detail pages
- Portrait metadata includes artist attribution, historical basis notes, and confidence levels

**Character Prioritization (Book 21):**
- Tier 1: Hannibal, Scipio Africanus, Flaminius, Varro, Paullus (commissioned art)
- Tier 2: Maharbal, Hanno, Mago, other named commanders (sourced historical depictions)
- Tier 3: Minor figures (text-only cards with genealogy/context)

### Army Composition Visualizations
Battle narratives become more engaging when readers understand what "two legions" or "Numidian cavalry" actually means in practical terms. The app includes interactive diagrams that break down military units:

**Roman Military Structure Diagram:**
- Hierarchical visualization: Legion → Cohort → Maniple → Century
- Soldier count indicators with uncertainty ranges (3,000-5,000 per legion depending on period)
- Equipment visualization: hastati vs. principes vs. triarii gear differences
- Formation diagrams showing battle deployment (triplex acies)
- Hover details explaining tactical roles and historical evolution

**Carthaginian Forces Breakdown:**
- Mercenary composition by origin: Libyans, Spaniards, Gauls, Germans
- Cavalry types: Numidian light horse vs. Spanish heavy cavalry
- Elephant unit structure and tactical employment
- Command structure differences from Roman hierarchy

**Contextual Deployment:**
These visualizations appear in force info cards and scene overlays, triggered by mentions of military units. A mention of "the hastati advanced" triggers a diagram showing where hastati fit in Roman battle order.

### Battle Sequence Diagrams
Major engagements (Trebbia, Lake Trasimene, Cannae) receive step-by-step visual treatment that reveals tactical genius without requiring military expertise:

**Diagram Structure:**
- 3-5 sequential "snapshots" of each battle's key phases
- Top-down view with clearly differentiated unit symbols
- Movement arrows showing tactical maneuvers
- Terrain features highlighted for tactical significance
- Casualty indicators and outcome annotations

**Battle of Cannae Example:**
1. Initial deployment: Roman numerical superiority, standard formation
2. Hannibal's weak center: deliberate vulnerability with strong flanks
3. Double envelopment: the center gives way, flanks close
4. Complete encirclement: tactical masterpiece visualization
5. Aftermath: casualty numbers and strategic consequences

**Integration with Narrative:**
Battle diagrams appear as expandable overlays triggered by scene navigation. As readers progress through Livy's account of Cannae, they can reference the visual sequence to understand tactical descriptions.

### Timeline Visualization
Ancient narratives can be disorienting for modern readers accustomed to precise dates. The timeline system provides chronological anchors and broader historical context:

**Temporal Context Bar:**
- Horizontal timeline spanning 218-201 BCE (Second Punic War)
- Current narrative position highlighted with precision indicators
- Major concurrent events: other Roman campaigns, domestic politics, Hellenistic affairs
- Seasonal indicators: ancient campaigns were seasonal, winter quarters matter

**Multi-Scale Timeline:**
- Year view: major campaign phases and strategic turning points
- Month view: specific battles and movements within campaign seasons
- Day view: detailed battle sequences and diplomatic exchanges
- Cross-referencing: "while Hannibal was at Cannae, Romans were doing X in Sicily"

**Uncertainty Representation:**
Ancient chronology involves scholarly debate. Timeline elements include confidence indicators:
- Solid bars: well-established dates (major battles, consular years)
- Dashed bars: approximate dating based on narrative sequence
- Dotted bars: speculative reconstruction with citation to scholarly sources

### Asset Management and Performance
**File Organization:**
```
/public/visual-assets/
├── portraits/
│   ├── hannibal/
│   │   ├── profile-400.webp
│   │   ├── full-body-600.webp
│   │   └── metadata.json
│   └── [character-id]/
├── diagrams/
│   ├── military-units/
│   │   ├── roman-legion.svg
│   │   └── carthaginian-army.svg
│   └── battles/
│       ├── trebbia/
│       └── cannae/
└── timeline/
    ├── textures/
    └── icons/
```

**Responsive Delivery:**
- SVG for diagrams (infinite scalability, small file sizes)
- WebP with PNG fallbacks for portraits
- Lazy loading for visual assets outside viewport
- Progressive image enhancement based on connection speed

**Attribution and Licensing:**
- All visual assets require clear attribution metadata
- Commissioned artwork contracts specify perpetual educational use rights
- Historical image sourcing documented with museum/library permissions
- Creative Commons licensing for user-contributed content (future consideration)

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

## Map Stack: Historical Cartography for Ancient Narratives

The map is the heart of this application - it transforms abstract geographical references in ancient texts into vivid spatial understanding. However, creating an accurate and engaging map for 3rd-century BCE events requires careful consideration of historical geography, anachronistic features, and user expectations.

### Base Map Data Strategy: Balancing Accuracy and Usability

**The Fundamental Challenge:**
Modern geographical data contains thousands of anachronisms irrelevant or confusing for ancient history readers: modern cities, political boundaries, infrastructure, and even coastline changes over 2,000+ years. However, ancient-specific cartographic datasets are either non-existent, incomplete, or too specialized for general audiences.

**Recommended Approach: Modern Foundation with Historical Overlay**
1. **Base Layer**: Use **Natural Earth 1:10m Physical** data as the foundation
   - Provides clean, generalized coastlines and major physical features
   - Includes major rivers essential for ancient geography (Tiber, Po, Rhone, Ebro)
   - Mountain ranges clearly defined (Alps, Pyrenees, Apennines)
   - Simplified enough to avoid excessive modern detail

2. **Historical Corrections**: Apply targeted adjustments for known changes
   - **Po Delta**: Reconstruct 3rd century BCE coastline (significantly different from modern)
   - **Minor Syrtis**: Adjust North African coast around modern Tunisia/Libya
   - **Alpine Passes**: Emphasize historically attested routes (Great St. Bernard, others)
   - **Sicily/Strait of Messina**: Ensure accurate representation of this crucial strategic waterway

3. **Contextual Filtering**: Remove or de-emphasize anachronistic features
   - **Modern Cities**: Completely remove all settlements not existing in 3rd century BCE
   - **Political Boundaries**: No modern country borders whatsoever
   - **Infrastructure**: Remove modern roads, railways, canals (Suez, etc.)
   - **Reservoirs/Dams**: Remove all artificial water bodies post-dating antiquity

### Basemap Technology Stack

**Rendering Engine: MapLibre GL JS**
- **Rationale**: Open-source, performant vector tile rendering without vendor lock-in
- **Performance**: Hardware-accelerated rendering essential for smooth pan/zoom during reading
- **Styling Control**: Complete control over visual presentation for antique aesthetic
- **Self-Hosting**: No external dependencies or usage limits for educational content

**Tile Format: PMTiles**
- **Single File Deployment**: Entire Mediterranean basin in one .pmtiles file
- **Range Request Support**: Efficient loading without tile server infrastructure
- **Static Hosting**: Compatible with CDN, GitHub Pages, or simple web hosting
- **Size Management**: Target <100MB for entire Mediterranean basin (zoom 0-10)

**Tile Generation Pipeline: Planetiler + Tippecanoe**
```bash
# Conceptual pipeline for tile generation
planetiler --osm-path=mediterranean-extract.osm.pbf \
           --natural-earth=ne_10m_physical.zip \
           --bounds="5,-5,45,47" \  # Mediterranean basin
           --output=base-tiles.mbtiles

# Apply historical filtering and styling
tippecanoe --force \
           --output=war-with-hannibal.pmtiles \
           --maximum-zoom=10 \
           --base-zoom=5 \
           --drop-fraction-as-needed \
           --extend-zooms-if-still-dropping \
           processed-features.geojson
```

### Visual Design: Achieving Authentic Antiquity

**Parchment Aesthetic Implementation:**
- **Base Texture**: Seamless parchment/vellum texture as map background
- **Color Palette**: Sepia-toned with earth tones (browns, tans, muted blues)
- **Typography**: Classical serif fonts for any remaining labels (Trajan Pro or similar)
- **Aging Effects**: Subtle map border irregularities, occasional "stain" overlays

**Cartographic Style Choices:**
- **Coastlines**: Thick (2-3px), dark brown strokes with subtle inner glow
- **Water Bodies**: Muted blue-green (#4A6741) with subtle texture overlay
- **Mountain Ranges**: Hachure-style relief rather than modern hillshading
- **Rivers**: Emphasized thickness for major waterways (Po, Rhone, etc.)
- **Land Mass**: Subtle texture variation suggesting different terrain types

**Information Hierarchy:**
- **Physical Features**: Most prominent (mountains, rivers, major water bodies)
- **Ancient Settlements**: Project gazetteer locations only, styled as period-appropriate symbols
- **Route Indications**: Faint traces suggesting major ancient roads/paths
- **Modern Features**: Completely absent or so faint as to be nearly invisible

### Zoom Level Strategy and Detail Progression

**Zoom 0-3: Mediterranean Overview**
- Entire basin visible: Iberia to Anatolia, North Africa to Gaul
- Only major physical features: coastlines, major mountains, largest rivers
- Perfect for showing strategic scope of Hannibal's campaigns

**Zoom 4-6: Regional Theater**
- Individual campaign theaters: Northern Italy, Southern France, Eastern Spain
- Secondary rivers and mountain passes become visible
- Suitable for showing army movements between major regions

**Zoom 7-10: Battle-Scale Detail**
- Individual valleys, specific mountain passes, detailed river systems
- Perfect for battle sites: Cannae plain, Lake Trasimene, Trebbia valley
- Supports tactical-scale scene visualization

**Beyond Zoom 10: Future Considerations**
- Not implemented in v1 to control file size and complexity
- Could support individual battlefield details in future versions

### Data Sources and Attribution

**Primary Geographic Data:**
- **Natural Earth**: Physical features baseline (public domain)
- **OpenStreetMap**: Selective use for historical river courses (ODbL license)
- **Ancient World Mapping Center**: Consultation for historical accuracy (with permission)
- **Pleiades Project**: Ancient place name authority for gazetteer alignment

**Historical Correction Sources:**
- **Academic Literature**: Peer-reviewed papers on ancient Mediterranean geography
- **Archaeological Evidence**: Excavation reports affecting coastline/settlement interpretation
- **Primary Sources**: Coordinate ancient geographical descriptions with modern understanding

### Performance and Technical Considerations

**File Size Management:**
- **Geometric Simplification**: Coastline generalization appropriate for zoom levels
- **Feature Selection**: Only include geographical features relevant to ancient context
- **Compression**: Optimize pmtiles compression for web delivery

**Caching and Delivery:**
- **HTTP Range Requests**: Essential for pmtiles efficiency
- **CDN Integration**: Distribute tiles globally for consistent performance
- **Preloading**: Intelligent prefetch of likely-needed tiles based on reading progress

**Fallback Strategies:**
- **Graceful Degradation**: Static image tiles for browsers without WebGL support
- **Offline Capability**: Service worker caching for core Mediterranean region
- **Low Bandwidth**: Reduced detail tiles for slow connections

### Future Enhancement Pathways

**Advanced Cartographic Features:**
- **Relief Visualization**: Classical hachure-style terrain representation
- **Seasonal Variation**: Different styling for campaign seasons vs. winter quarters
- **Uncertainty Visualization**: Different styling confidence levels for geographical reconstructions

**Extended Geographic Coverage:**
- **Broader Context**: Extend to cover Gaul, Germania, broader Iberia for complete Punic War context
- **Higher Detail**: Zoom levels 11-14 for individual battlefield analysis
- **3D Terrain**: Elevation model integration for Alpine crossing visualization

**Integration with Historical Sources:**
- **Multiple Cartographic Traditions**: Compare Ptolemaic vs. modern geographical understanding
- **Temporal Layers**: Show geographical changes over the course of the war
- **Source Transparency**: Visual indicators showing which parts of the map are well-attested vs. reconstructed

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

## Cross-Referencing and Progress Systems

Ancient historical narratives are densely interconnected - the same places, people, and armies appear throughout the text in different contexts. The app's cross-referencing system helps readers track these connections while maintaining reading momentum, transforming Livy's linear narrative into an explorable network of relationships.

### Entity Linking Strategy
Every mention of a place, person, or force becomes a node in a rich web of connections that span the entire work:

**Cross-Chapter Entity Tracking:**
- **Place Evolution**: Track how the narrative significance of locations changes (e.g., Saguntum as cause → battlefield → symbol)
- **Character Development**: Follow people through different roles (Hannibal as general → diplomat → fugitive)
- **Force Continuity**: Trace army units through campaigns, showing how they grow, diminish, and reorganize

**Reference Relationship Types:**
1. **Temporal**: "First mentioned in", "Last seen in", "Contemporary appearances"
2. **Spatial**: "Other events at this location", "Nearby locations in same timeframe"
3. **Causal**: "Events leading to this", "Consequences of this event"
4. **Thematic**: "Similar battles", "Parallel diplomatic situations", "Comparable speeches"

**Implementation in Data Model:**
Extend the chapter annotation format to include relationship metadata:
```json
{
  "annotation_id": "21.05.a12",
  "entity": { "type": "place", "id": "cannae" },
  "relationships": {
    "foreshadowing": ["21.03.a05"], // Earlier omens/preparations
    "consequences": ["21.06.a01", "21.07.a03"], // Aftermath mentions
    "spatial_context": ["aufidus_river", "apulia_region"]
  }
}
```

**User Interface for Cross-References:**
- **Hover Preview**: Hovering over any annotation shows mini-card with "See also: 3 other mentions"
- **Entity Detail Pages**: `/place/cannae` aggregates all mentions with context snippets and scene links
- **Relationship Graph**: Visual network diagram showing connections between entities (optional advanced feature)
- **"Find in Text" Feature**: Search for all instances of an entity across the work with context

### Reading Progress Implementation
Long-form digital reading requires different progress indicators than modern books. Ancient texts have uneven pacing - some chapters contain multiple major events, others cover months of inactivity:

**Multi-Level Progress Tracking:**
1. **Narrative Progress**: Track major story beats independently of chapter structure
   - "Crossing the Alps" (spans chapters 21.3-21.6)
   - "First Italian Victories" (21.7-21.10)
   - "Winter Quarters" (21.11)

2. **Chapter Progress**: Traditional chapter-by-chapter completion with subsection granularity
3. **Time-Based Progress**: Show narrative timespan (months/years covered) alongside page progress
4. **Map Exploration Progress**: Track which regions/scenes have been visited

**Bookmark and Resume Functionality:**
- **Smart Bookmarks**: Automatically capture reading position + map state every 30 seconds
- **Multiple Bookmark Categories**:
  - Auto-save (invisible, for resume functionality)
  - User bookmarks (explicit saves with optional notes)
  - Scene bookmarks (capture specific map states for later reference)
  - Research bookmarks (save particularly interesting cross-references)

**Progress Data Structure:**
```json
{
  "user_progress": {
    "book_21": {
      "chapters_completed": ["21.01", "21.02", "21.03"],
      "current_position": {
        "chapter": "21.05",
        "block": "21.05.p04",
        "scene": "21.05.s03",
        "timestamp": "2026-02-03T14:30:00Z"
      },
      "reading_sessions": [
        {
          "date": "2026-02-01",
          "duration": 1800, // 30 minutes in seconds
          "chapters_touched": ["21.01", "21.02"],
          "scenes_viewed": ["21.01.s01", "21.01.s02"]
        }
      ],
      "narrative_milestones": {
        "alps_crossing": { "completed": true, "first_read": "2026-02-02" },
        "cannae_battle": { "completed": false }
      }
    }
  }
}
```

**Visual Progress Indicators:**
- **Chapter TOC**: Completion status with partial progress bars for in-progress chapters
- **Narrative Timeline**: Story milestone completion overlaid on chronological timeline
- **Map Coverage**: Subtle visual indication of which map regions have been explored through reading

### Cross-Chapter Navigation Enhancements
Help readers discover connections and maintain orientation across the large text:

**"See Also" System:**
- **Intelligent Suggestions**: When reading about Cannae, suggest related passages about Roman tactical reforms or Hannibal's other victories
- **Contextual Links**: "For background on Roman military structure, see Chapter 21.2"
- **Follow-up References**: "This event is referenced again in Chapter 21.8"

**Character Journey Tracking:**
- **Person Pages**: Timeline view of character's appearances with context
- **Character Arcs**: Visual representation of how key figures develop throughout the narrative
- **Relationship Networks**: Show alliances, enmities, and political connections between historical figures

**Thematic Reading Paths:**
Beyond linear chapter progression, offer curated reading experiences:
- **"Hannibal's Perspective"**: Scenes and passages that focus on Carthaginian viewpoint
- **"The Roman Response"**: How Rome adapted to the Hannibalic threat
- **"Geography of War"**: Focus on strategic importance of terrain and logistics
- **"Diplomatic Dimensions"**: Treaties, negotiations, and alliance-building

**Implementation Considerations:**
- **Performance**: Pre-compute relationship graphs during build time to avoid runtime computation
- **Storage**: Use efficient indexing for cross-references to support fast lookups
- **Privacy**: Store progress data locally with optional cloud sync (respect user privacy)
- **Accessibility**: Ensure all cross-reference features work with screen readers and keyboard navigation

## Reader ↔ Map Sync Behavior

The synchronization between reading progress and map state is crucial for maintaining narrative immersion. The system must feel responsive and predictable while avoiding jarring transitions that break concentration.

### Scroll-Triggered Scene Detection
The app continuously monitors scroll position to determine which scene should be active, using a sophisticated algorithm that balances responsiveness with stability:

**Active Scene Algorithm:**
1. **Anchor Point Detection**: Scan for scene anchors currently in or recently passed through the viewport
2. **Proximity Weighting**: Scenes closer to viewport center receive higher activation priority
3. **Hysteresis Buffer**: Require 150px of scroll movement before switching scenes (prevents oscillation)
4. **Reading Direction Awareness**: Distinguish between scrolling down (reading) vs. scrolling up (reviewing)
5. **Viewport Threshold**: Scene becomes active when its anchor reaches the top 1/3 of viewport

**Edge Case Handling:**
- **Multiple Scenes in Viewport**: Prioritize the scene whose anchor is closest to the top 1/3 line
- **No Scene in Viewport**: Maintain last active scene until a new anchor is reached
- **Rapid Scrolling**: Debounce scene changes with 300ms delay, but immediately update if user stops scrolling
- **Scene Density**: In sections with many scenes, implement minimum dwell time (2 seconds) before auto-switching

**Implementation Approach:**
```javascript
// Simplified pseudocode for scene detection logic
const detectActiveScene = throttle(() => {
  const viewportTop = window.scrollY;
  const viewportCenter = viewportTop + window.innerHeight * 0.33;

  const candidateScenes = scenes
    .filter(scene => scene.anchor.offsetTop <= viewportCenter)
    .sort((a, b) => b.anchor.offsetTop - a.anchor.offsetTop);

  const newActiveScene = candidateScenes[0];

  if (newActiveScene && newActiveScene.id !== currentActiveScene?.id) {
    const scrollDistance = Math.abs(viewportTop - lastScrollPosition);
    if (scrollDistance > HYSTERESIS_THRESHOLD) {
      setActiveScene(newActiveScene);
    }
  }
}, 100);
```

### Animation Framework and Timing
Map transitions must feel smooth and purposeful, providing visual continuity that supports rather than distracts from the reading experience:

**Animation Categories:**
1. **Scene Transitions** (auto-scroll triggered):
   - Duration: 1200ms (allows reading to continue)
   - Easing: cubic-bezier(0.25, 0.46, 0.45, 0.94) - smooth deceleration
   - Coordinate animation: simultaneous pan + zoom using interpolation
   - Layer transitions: 300ms fade crossfade between scene layers

2. **User-Initiated Navigation** (click on annotation):
   - Duration: 800ms (faster since user expects immediate response)
   - Easing: cubic-bezier(0.25, 0.8, 0.25, 1) - quick acceleration, smooth landing
   - Highlight emphasis: 200ms bounce effect on target location

3. **Manual Scene Stepping** (prev/next buttons):
   - Duration: 1000ms (medium speed for deliberate navigation)
   - Text synchronization: scroll text to scene anchor simultaneously with map animation
   - Progress indication: subtle progress bar during transition

**Animation Interruption Handling:**
- **User Interaction During Animation**: Immediately cancel current animation and start new one
- **Scroll During Auto-Sync**: Pause scene synchronization until scrolling stops for 500ms
- **Multiple Rapid Triggers**: Queue animations but cancel intermediate steps (only animate from current to final state)

**Performance Optimizations:**
- **GPU Acceleration**: Use `transform3d()` for all map movements to trigger hardware acceleration
- **Frame Rate Targeting**: Monitor requestAnimationFrame timing and reduce animation complexity if dropping below 30fps
- **Reduced Motion Preference**: Respect `prefers-reduced-motion` by using instant cuts instead of transitions

### User Control Options
Users need control over the sync behavior to accommodate different reading preferences and technical constraints:

**Sync Mode Settings:**
1. **Full Auto-Sync** (default): Map follows reading position with smooth transitions
2. **Manual Scenes Only**: Map only updates when user clicks scene navigation buttons
3. **Annotation Sync Only**: Map responds to place/force clicks but ignores scroll position
4. **Disabled**: Map state completely independent from reading position

**User Interface for Controls:**
- **Sync Toggle**: Prominent toggle switch in map header with clear state indication
- **Settings Panel**: Expandable settings with animation speed controls (0.5x, 1x, 2x, instant)
- **Keyboard Shortcuts**:
  - `S` to toggle sync mode
  - `←/→` for manual scene navigation
  - `Space` to pause/resume auto-sync temporarily

**Preference Persistence:**
- Store user preferences in localStorage with graceful degradation
- Session-level preferences (override saved settings temporarily)
- URL parameters for sharing specific sync states: `?sync=manual&speed=2x`

**Adaptive Behavior:**
- **Touch Devices**: Reduce animation duration by 25% for more responsive feel
- **Reduced Bandwidth**: Automatically disable particle effects and use simpler transitions
- **Screen Size**: Adjust sync sensitivity based on viewport height (more generous thresholds on mobile)

### State Management Architecture
Coordinating reader and map state requires careful synchronization to prevent race conditions and ensure consistent user experience:

**Central State Store:**
```javascript
// Simplified state structure
const appState = {
  reading: {
    activeChapter: "21.05",
    activeBlock: "21.05.p04",
    scrollPosition: 1234,
    userScrolling: false
  },
  map: {
    activeScene: "21.05.s03",
    viewport: { lng: 12.4, lat: 41.9, zoom: 6 },
    animating: false,
    userInteracting: false
  },
  sync: {
    mode: "auto", // "auto" | "manual" | "annotations" | "disabled"
    animationSpeed: 1.0,
    paused: false
  }
}
```

**State Synchronization Rules:**
1. **Reader → Map**: Scroll events trigger scene evaluation and potential map updates
2. **Map → Reader**: Scene navigation buttons trigger text scrolling to scene anchors
3. **Conflict Resolution**: User interactions always take precedence over automatic sync
4. **URL Synchronization**: Browser back/forward maintains consistent reader + map state

**Deep Linking Support:**
- **Scene URLs**: `/book/21/chapter/5#scene=21.05.s03` restores exact map + text state
- **Block URLs**: `/book/21/chapter/5#block=21.05.p04` positions text and activates relevant scene
- **Shareable States**: Generate URLs that capture current reading position and map viewport for sharing specific moments

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

## Implementation Strategy: Chapter-by-Chapter Release

The MVP-first approach focuses on delivering complete, polished chapters rather than partial features across multiple chapters. This strategy maximizes user value from the earliest release and provides concrete feedback on the reading experience before scaling content creation.

### Chapter Prioritization and Sequencing
Not all chapters of Book 21 are equally compelling or suitable for demonstrating the app's capabilities. The rollout sequence prioritizes dramatic narrative moments that showcase both the reading experience and the map's storytelling potential:

**Phase 1: Proof of Concept (2 chapters)**
- **Chapter 21.4: The Crossing of the Alps** - Iconic moment with clear geographical storytelling opportunities
- **Chapter 21.10: Battle of Cannae** - Military masterpiece that benefits enormously from visual analysis

*Rationale*: These chapters are universally recognized as climactic moments in ancient history. They test all core features: complex geography (Alpine passes), major battles (tactical diagrams), character development (Hannibal's genius), and dramatic pacing (life-or-death struggles).

**Phase 2: Narrative Context (3 chapters)**
- **Chapter 21.1: Causes of War** - Essential background, tests diplomatic/political annotation types
- **Chapter 21.6: Descent into Italy** - Connects Alps to first victories, tests army movement visualization
- **Chapter 21.11: Winter Quarters** - Quieter chapter testing different pacing and content types

*Rationale*: Provides essential context for the flagship chapters while testing the system with different narrative rhythms and content types.

**Phase 3: Complete Arc (remaining chapters)**
- Fill gaps in chronological order: 21.2, 21.3, 21.5, 21.7, 21.8, 21.9, 21.12+
- Focus on consistency and completeness rather than individual chapter polish

### Content Completion Criteria
Each chapter release meets specific quality standards before moving to the next:

**Annotation Coverage Standards:**
- **Geographic References**: 90%+ of place names annotated with gazetteer links
- **Person Mentions**: All historically significant figures annotated with biographical info
- **Military Units**: Force annotations for all army movements and battle formations
- **Cross-References**: Connections established to other chapters where the same entities appear

**Scene Development Requirements:**
- **Major Events**: Every significant military, diplomatic, or narrative moment has an associated scene
- **Geographic Context**: Minimum one scene per major location mentioned in the chapter
- **Narrative Pacing**: Scene density matches narrative importance (more scenes for battles, fewer for exposition)
- **Visual Integration**: Each scene includes appropriate visual aids from the storytelling system

**Quality Assurance Checklist:**
- [ ] All annotations link to valid gazetteer entries
- [ ] All scenes include confidence levels and source citations
- [ ] Visual assets (portraits, diagrams) properly integrated and attributed
- [ ] Cross-chapter references verified and reciprocal
- [ ] Mobile layout tested and responsive
- [ ] Accessibility features verified (screen reader, keyboard navigation)
- [ ] Performance benchmarks met (page load < 3s, map transitions < 1s)

### User Feedback Integration Workflow
Each chapter release includes mechanisms for gathering and incorporating reader feedback:

**Feedback Collection Points:**
- **Chapter Completion Survey**: Brief questionnaire about pacing, clarity, and engagement
- **Annotation Feedback**: Report errors or suggest improvements for specific historical claims
- **Feature Usage Analytics**: Track which map features are most/least used
- **Performance Monitoring**: Automated detection of slow load times or interaction delays

**Feedback Processing Pipeline:**
1. **Immediate Issues**: Factual errors, broken links, accessibility problems → hotfix within 48 hours
2. **Content Improvements**: Better explanations, additional cross-references → patch within 1 week
3. **Feature Enhancements**: UI improvements, new visualization types → queue for next chapter release
4. **Systemic Changes**: Fundamental workflow or data model changes → major version planning

### Technical Implementation Milestones

**Phase 0: Foundation Setup**
- Choose public domain translation source for Book 21
- Create automated ingestion pipeline from source text to chapter JSON
- Set up development environment and initial project structure

**Phase 1: Core Reading Experience**
- Next.js application with TOC and chapter navigation
- Text rendering with stable block IDs and view mapping system
- Basic responsive layout for desktop and mobile

**Phase 2: Map Integration Foundation**
- MapLibre GL setup with self-hosted pmtiles basemap
- Antique styling applied (parchment texture, muted palette)
- Places overlay system reading from gazetteer GeoJSON

**Phase 3: Interactive Features**
- Annotation system with clickable text references
- Map pan/zoom responses to place clicks
- Info cards and entity detail display

**Phase 4: Scene System and Synchronization**
- Scene data model and rendering system
- Force markers and route polylines per scene
- Scroll-sync between reading position and map state
- Scene navigation controls and smooth transitions

**Phase 5: Editorial Tooling**
- Dev-only annotation editor with text selection
- Scene editing interface with map interaction
- Data validation pipeline integrated with build process

**Phase 6: Visual Storytelling Integration**
- Character portrait system with responsive delivery
- Battle diagram integration with scene system
- Army composition visualizations for force annotations
- Timeline visualization for narrative context

**Phase 7: Advanced Features**
- Cross-referencing system with entity relationship tracking
- Reading progress and bookmark functionality
- Full accessibility compliance and performance optimization
- Advanced search and filtering capabilities

## Defaults And Opinionated Choices For v1
- Text blocks are paragraphs.
- Basemap has no modern labels; all labels come from the project gazetteer and scenes.
- Everything “map-assertive” includes confidence + citations.

