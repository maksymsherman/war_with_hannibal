# War With Hannibal: Interactive Historical Atlas & Reader — Comprehensive Project Plan

## Executive Summary

**War with Hannibal** is a digital humanities project designed to transform the reading experience of Livy’s *Ab Urbe Condita*, specifically Book 21 (The Second Punic War), into an interactive, visually immersive journey. The core value proposition is to bridge the gap between abstract historical text and concrete geographical understanding. When Livy describes Hannibal’s army moving through Gaul or crossing the Alps, the user will not just read the words but see the movement on a synchronized, period-accurate map.

This project addresses a common pain point for students and history enthusiasts: the difficulty of visualizing complex military maneuvers and ancient geography solely from text. By coupling a clean, readable text interface with a dynamic, state-aware map, we provide immediate context—answering "where is this?" and "how far is that?" instantly.

The initial scope (MVP) focuses on **Book 21**, covering the outbreak of the war, the crossing of the Alps, and the early battles in Italy. The platform is architected to be scalable, allowing for the addition of further books, other ancient texts, or different historical periods in the future.

---

## 1. Vision & Objectives

### 1.1 Core Mission
To create the definitive digital companion for reading Livy’s history of the Second Punic War, making ancient military history accessible, engaging, and spatially intuitive for a modern audience.

### 1.2 Key Objectives
*   **Visual Synchronicity:** Ensure every significant geographical mention in the text is instantly locatable on the map.
*   **Narrative Immersion:** Maintain the flow of reading while providing rich contextual data (troop movements, terrain difficulties, distances).
*   **Scholarly Rigor:** Use confidence intervals and citations for uncertain locations, distinguishing between archaeological fact and historical conjecture.
*   **Open Access:** Utilizing public domain texts and open-source mapping technologies to ensure the tool is free and accessible to educators and students.
*   **Extensibility:** Build a data model that decouples the "text" from the "map state," allowing for future translations or different source texts to drive the same visualization engine.

---

## 2. Target Audience

### 2.1 Primary Audience: History Enthusiasts & Autodidacts
*   **Profile:** Individuals who enjoy podcasts like *The History of Rome* or *Hardcore History*. They are interested in the narrative but often lack the deep geographical knowledge of the ancient Mediterranean.
*   **Needs:** Clear visualization of army movements, understanding of strategic decisions based on terrain, and identifying ancient cities with their modern counterparts.

### 2.2 Secondary Audience: Students & Educators
*   **Profile:** High school or undergraduate students studying Classics or Ancient History.
*   **Needs:** A reliable study aid that helps visualize the text for exams or papers. Teachers need a tool to demonstrate the scale of the conflict in the classroom.

### 2.3 Tertiary Audience: Digital Humanists
*   **Profile:** Researchers interested in spatial history and digital mapping.
*   **Needs:** Access to the underlying data (GeoJSON, entity relationships) and a model for how to digitize other ancient texts.

---

## 3. Product Features & Scope (MVP)

### 3.1 The "Dual-View" Interface
The application’s primary interface is a split-screen view:
*   **Left Panel (The Reader):** A clean, typography-focused rendering of Livy’s text.
    *   **Features:** Infinite scroll (chapter-based), deep-linking to paragraphs, distraction-free reading mode.
    *   **Interactive Elements:** "Entity" links (places, people, armies) are highlighted. Clicking them triggers map actions or opens information cards.
*   **Right Panel (The Atlas):** A dynamic, WebGL-powered map.
    *   **Features:** Smooth zooming/panning, period-accurate basemap (no modern borders/cities), and distinct layers for distinct entity types.
    *   **Context Awareness:** As the user scrolls through the text, the map automatically updates to reflect the current "Scene" (e.g., panning to Saguntum when the siege begins).

### 3.2 The "Scene" System
A "Scene" is a snapshot of the world state at a specific point in the narrative.
*   **Automatic Sync:** The text is divided into logical blocks. Each block is associated with a Scene ID.
*   **State Definition:** A Scene defines:
    *   **Viewport:** Center coordinates and zoom level.
    *   **Active Layers:** Which armies, cities, or paths are visible.
    *   **Troop Positions:** Coordinates for army icons.
    *   **Movement Arrows:** Polylines indicating current maneuvers (e.g., "Hannibal crosses the Rhone").
*   **Manual Override:** Users can toggle "Sync" on/off. If off, they can explore the map freely without it jumping around as they read.

### 3.3 The Entity System (Gazetteer)
A database of people, places, and groups mentioned in the text.
*   **Places:** Cities, rivers, mountains, regions.
    *   *Data:* Ancient name, modern name, lat/long, type (settlement, physical feature).
*   **People:** Historical figures (Hannibal, Scipio, Fabius Maximus).
    *   *Data:* Short bio, faction, role (General, Consul, Politician).
*   **Forces:** Armies or detachments.
    *   *Data:* Composition (infantry/cavalry counts), commander, faction.

### 3.4 Visual Storytelling Aids
*   **Route Visualization:** Dashed lines for uncertain routes, solid lines for well-attested roads.
*   **Battle Schematics:** For major battles (Trebbia, Ticinus), the map zooms in and overlays a schematic view of the troop formations, rather than just a generic icon.
*   **Uncertainty Visualization:** Color-coding or opacity changes to indicate historical uncertainty (e.g., "The exact pass used is debated").

---

## 4. Technical Architecture

### 4.1 Tech Stack Selection
We will use a modern, jamstack-inspired architecture for performance, SEO, and ease of deployment.

*   **Frontend Framework:** **Next.js (React)**.
    *   *Reasoning:* Excellent support for static site generation (SSG) for text content (great for SEO and speed), robust ecosystem, and easy integration with mapping libraries.
*   **Language:** **TypeScript**.
    *   *Reasoning:* Strict typing is essential for managing the complex data structures of the Scene/Entity models and ensuring data integrity across the app.
*   **Mapping Engine:** **MapLibre GL JS**.
    *   *Reasoning:* High-performance vector tile rendering (WebGL). Open-source fork of Mapbox GL, free from usage fees (critical for a non-profit/hobby project). Supports custom styling for that "ancient map" aesthetic.
*   **State Management:** **Zustand** or **React Context**.
    *   *Reasoning:* We need a lightweight way to share the "Active Scene ID" and "Hovered Entity ID" between the Reader and Map components.
*   **Styling:** **Tailwind CSS**.
    *   *Reasoning:* Rapid UI development, utility-first approach helps maintain consistency.
*   **Deployment:** **Vercel** or **Netlify**.
    *   *Reasoning:* Seamless Git integration, automatic preview builds, global CDN for fast asset delivery.

### 4.2 Data Architecture
The data strategy decouples the narrative content from the geospatial data.

#### 4.2.1 File Structure
The project will use a file-based CMS approach (Git as the single source of truth).
```
/data
  /books
    /21
      /chapters
        01.json       # Text content, blocked by paragraph
        02.json
      /scenes
        01.json       # Map states corresponding to Chapter 1
        02.json
  /gazetteer
    places.geojson    # Master list of all locations
    people.json       # Master list of historical figures
    forces.json       # Master list of military units
```

#### 4.2.2 Data Models

**Chapter Model (`01.json`):**
```json
{
  "id": "21.01",
  "title": "The Oath",
  "blocks": [
    {
      "id": "21.01.p01",
      "text": "It implies no lack of respect...",
      "scene_id": "21.01.s01",
      "entities": [
        { "id": "rome", "start": 15, "end": 19 },
        { "id": "carthage", "start": 45, "end": 53 }
      ]
    }
  ]
}
```

**Scene Model (`21.01.s01` in `scenes/01.json`):**
```json
{
  "id": "21.01.s01",
  "viewport": {
    "center": [10.2, 36.8], // Carthage
    "zoom": 6
  },
  "layers": [
    { "type": "highlight", "entity_id": "carthage" },
    { "type": "marker", "entity_id": "hannibal_army", "loc": [10.2, 36.8] }
  ],
  "narrative_note": "Hannibal swears his oath of eternal enmity."
}
```

### 4.3 Map Stack Details
To achieve a "historical" look:
1.  **Basemap Generation:** We cannot use Google Maps. We will generate custom vector tiles using **Planetiler** or **Tippecanoe**.
2.  **Data Source:** Natural Earth (physical features) + OpenStreetMap (water bodies). Modern labels/cities will be stripped out.
3.  **Hosting:** The generated `.pmtiles` (Protomaps) archive (containing the Mediterranean basin) will be hosted as a static asset. MapLibre will fetch tiles directly from this file using HTTP Range Requests, eliminating the need for a tile server.

---

## 5. UI/UX Design Strategy

### 5.1 Design Aesthetic: "Digital Vellum"
The design should evoke a sense of history without being kitschy.
*   **Typography:** A high-legibility serif for the body text (e.g., *Crimson Pro* or *EB Garamond*) paired with a clean sans-serif for UI controls (e.g., *Inter*).
*   **Color Palette:** Warm, earthy tones.
    *   *Background:* Off-white/cream (#FDFBF7).
    *   *Text:* Dark charcoal (#2D2D2D), never pure black.
    *   *Map Water:* Desaturated teal/blue.
    *   *Map Land:* Parchment texture or solid beige.
    *   *Highlights:* Gold/Amber for Carthage, Imperial Red for Rome.

### 5.2 User Flows

**Flow 1: The "Follow Along" Reader**
1.  User lands on `/book/21/chapter/1`.
2.  User starts reading.
3.  As they scroll past paragraph 3, the "Active Scene" changes.
4.  The map on the right smoothly pans from Carthage to Saguntum.
5.  An arrow animates on the map showing the direction of the march.
6.  User hovers over "Saguntum" in the text; the city on the map pulses.

**Flow 2: The "Deep Dive" Investigator**
1.  User pauses reading to explore the map.
2.  They click the "Unlock Map" button.
3.  They zoom into the Alps.
4.  They click a "Pass" marker.
5.  A sidebar opens: "Col de la Traversette. Elevation: 2947m. Note: This is the controversial but likely route proposed by Gavin de Beer..."
6.  User clicks "Back to Text" to snap the map back to the current reading position.

### 5.3 Responsive Design
*   **Desktop:** Split view (Text 40% | Map 60%).
*   **Tablet:** Split view (Text 50% | Map 50%).
*   **Mobile:** Stacked view is difficult. We will implement a "Drawer" pattern.
    *   Text is full screen.
    *   A "Map" floating action button (FAB) or bottom sheet handle is visible.
    *   Tapping it slides the map up to cover 70% of the screen.
    *   Alternatively, "Inline Maps": small static images inserted between paragraphs for key moments, with a "View Interactive Map" button.

---

## 6. Implementation Roadmap

### Phase 1: Prototype (Weeks 1-3)
*   **Goal:** A single chapter (e.g., Chapter 21, The Crossing of the Alps) working end-to-end.
*   **Tasks:**
    *   Set up Next.js repo.
    *   Configure MapLibre with a basic Natural Earth style.
    *   Manually transcribe Chapter 21 text into JSON.
    *   Manually code 5 scenes for Chapter 21.
    *   Implement the Scroll-to-Scene sync logic.

### Phase 2: Content Pipeline & CMS (Weeks 4-6)
*   **Goal:** Streamline the creation of data so we don't have to hand-code JSON.
*   **Tasks:**
    *   Write a script to parse raw text and split it into blocks.
    *   Create a simple "Dev Mode" overlay in the app:
        *   Alt-click a word in the text -> "Create Entity".
        *   Move the map -> "Save Viewport to Scene".
    *   Populate the Gazetteer with major locations.

### Phase 3: The Full Book (Weeks 7-12)
*   **Goal:** Complete Book 21 content.
*   **Tasks:**
    *   Systematic entry of all chapters.
    *   Geocoding all identifiable places.
    *   Creating "Routes" for the major army movements.
    *   Refining the mobile experience.

### Phase 4: Polish & Launch (Weeks 13-14)
*   **Goal:** Public release.
*   **Tasks:**
    *   Performance tuning (image optimization, bundle splitting).
    *   SEO (meta tags, Open Graph images for every chapter).
    *   Accessibility audit (ARIA labels, keyboard navigation).
    *   Deploy to production URL.

---

## 7. Content Strategy & Licensing

### 7.1 Text Source
We will use the **Canon Roberts (1912)** translation (available in the Perseus Digital Library or similar) as it is firmly in the public domain.
*   *Caveat:* The language can be archaic. We might consider a "Modernized" toggle where we lightly edit archaic phrasing, but strictly storing the original as the source of truth.

### 7.2 Map Data
*   **Natural Earth:** Public Domain.
*   **OpenStreetMap:** ODbL (requires attribution).
*   **Historical Data:** We will source coordinates from the **Pleiades Project** (Creative Commons Attribution 3.0). This is critical for academic accuracy.

---

## 8. Risk Management

### 8.1 Ambiguity of Ancient Geography
*   *Risk:* Livy mentions a town that no longer exists or whose location is unknown.
*   *Mitigation:* The "Uncertainty" flag in the data model. We will represent these as large, fuzzy circles rather than precise points. We will add editorial notes explaining the debate (e.g., "Polybius says X, Livy says Y").

### 8.2 Mobile Usability
*   *Risk:* Split-screen map/text is notoriously bad on phones.
*   *Mitigation:* Prioritize the "Drawer" interface. The map is secondary context on mobile, only pulled up when the user needs orientation. The text is primary.

### 8.3 Scope Creep
*   *Risk:* Trying to map *every* minor skirmish.
*   *Mitigation:* Stick to "Strategic" movements. If a patrol of 5 men moves 1km, we don't map it. We only map movements of armies or key characters that change the strategic situation.

---

## 9. Marketing & Growth

### 9.1 Launch Strategy
*   **Show HN:** Post on Hacker News. The intersection of "Tech" and "History/Classics" is very strong there.
*   **Reddit:** r/history, r/ancientrome, r/dataisbeautiful.
*   **Twitter/X:** Tag history influencers and digital humanities accounts.

### 9.2 Community Contribution
Once the core engine is built, the project can be open-sourced to allow the community to:
*   Fix typo/geocoding errors.
*   Suggest "Alternative Routes" for debated paths.
*   Contribute mappings for *other* books (e.g., Book 22, Cannae).

### 9.3 SEO
*   Each chapter will be a static page.
*   Entity pages (e.g., `/place/saguntum`) will be generated. These act as "Landing Pages" for people searching for specific historical locations, driving traffic to the book reader.

---

## 10. Conclusion

**War with Hannibal** is more than just a digitized book; it is a contextual engine. By anchoring the narrative in physical space, we transform a 2,000-year-old text from a dry recitation of names into a vivid, understandable conflict. The technology stack (Next.js, MapLibre, Vector Tiles) is mature and capable of delivering this experience performantly. The greatest challenge lies in the data curation—transforming vague Latin descriptions into coordinates—but with a structured "Uncertainty" system, even the ambiguities of history can be visualized and understood.

This plan provides a clear path from concept to MVP, prioritizing user experience and scholarly integrity equally.
