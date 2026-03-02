**WeftWarp**

Product Specification  ·  v6

A local-first spatial knowledge workspace. Content exists once; context is infinite.

| **Status**           | Active development                                                        |
| -------------------- | ------------------------------------------------------------------------- |
| **Version**          | v6 — JS-native stack, UI/UX-led                                           |
| **Stack**            | Plain JS · wa-sqlite (WASM) · JSZip · File System Access API              |
| **Distribution**     | Static file serve · zero backend · PWA-capable · Tauri for native desktop |
| **Network activity** | None post-load, except optional file sync (post-v1)                       |
| **File format**      | .wwz — self-contained zip archive                                         |

**v6 Change Summary**

The ECS data model, file format, cross-reference system, and design language are unchanged from v1. This version replaces the Go backend with a JS-native storage layer, reorders the specification to lead with user experience and derive technical requirements from it, and formally documents the distribution strategy.

 

# **1. What It Is**

WeftWarp is a local-first visual workspace for organizing creative and technical projects. It combines Joplin-style hierarchical navigation with a Milanote-style spatial canvas, unified by a wiki-style cross-reference system and a flat ECS data model.

It is not a node graph, not a flowchart tool, and not a document editor. It is a spatial knowledge organizer — a place where the same idea can appear in multiple contexts without duplication, where every named thing can be referenced from anywhere, and where the structure of your knowledge is as visible as the content itself.

### **Core Philosophy**

Content exists once; context is infinite. 

Every entity lives in a flat ECS store and is placed, referenced, or linked from anywhere in the workspace. Nothing is copied. Everything stays in sync.

WeftWarp enforces bounded pages. There is no infinite canvas. Every page has a defined size — 8.5×11 by default — and content must be organized within it. Bounded pages force the organizational decision that infinite canvas defers indefinitely. Pages are exportable as PDF or image as a direct consequence of this constraint.

### **The Problem It Solves**

Creative and technical projects rarely fit into neat hierarchies. A character might belong to three different works. A design decision might be referenced in a spec, a planning board, and a journal entry simultaneously. A game design doc needs both structured prose and spatial relationship maps.

| **Tool**  | **Gap**                                             |
| --------- | --------------------------------------------------- |
| Joplin    | Good hierarchy, poor cross-referencing              |
| Milanote  | Good spatial canvas, poor hierarchy and portability |
| Obsidian  | Good wiki links, poor visual layout                 |
| All three | SaaS-adjacent or cloud-dependent fragility          |

WeftWarp is the intersection: hierarchy for navigation, canvas for spatial thinking, wiki links for cross-referencing, and a local file for portability.

### **Deployment Model**

WeftWarp is a static application. Once the HTML, JS, and CSS files are served to the browser, the server's job is finished. There is no API, no backend process, no database server, and no persistent server-side session. All computation happens in the browser. All state lives in the .wwz file on the user's machine.

This puts WeftWarp in the same category as Photopea or similar converter-style tools: served once, runs indefinitely, works offline, requires no infrastructure to operate beyond a file host. The practical consequences are significant — the application can be hosted on any static CDN or file server, self-hosted trivially, and run without an internet connection after the initial load.

The only legitimate network activity the application will ever perform is file sync, which is a post-v1 feature and optional by design. Everything else — opening files, editing, saving, exporting — is entirely local.

 

# **2. User Experience**

This section defines the application as a user experiences it. Technical requirements in later sections are derived from what these flows demand — not the reverse.

## **2.1  The Three Views**

Navigation is a flat stack of three destination views. The left sidebar is persistent across all of them.

### **Workspace View**

The default state when no binder is selected. The sidebar shows all Binders. No canvas is visible — the center area is empty or shows a welcome state. This is the orientation layer: where am I, what projects do I have?

### **Binder Gallery View**

Clicking a Binder in the sidebar opens a thumbnail grid of every Page in that Binder. Pages are shown at their defined aspect ratios — a landscape page appears wider than a portrait page. This is the shape-of-the-work view.

| **Drag**            | Reorder pages                                      |
| ------------------- | -------------------------------------------------- |
| **Click**           | Rename page inline                                 |
| **Double-click**    | Enter Page Edit view for that page                 |
| **Right-click**     | Context menu: duplicate, delete, change dimensions |
| **New Page button** | Creates page at default 8.5×11                     |

### **Page Edit View**

The bounded canvas editor. One page, its layers, the entity strip, and the full toolbar. This is where work happens.

## **2.2  Page Edit Layout**

┌────────────────────────────────────────────────────────────────┐

│  ◆ WeftWarp        |  File  Edit  View  Export     ○ autosaved 2m    │  Menubar (27px) 

├────────────────────────────────────────────────────────────────┤

│  [Grid][Layers][Clean]  |  [Page: 8.5×11] [Portrait] [#bg] [Grid] │  Toolbar + Dynamic Edit Bar (38px)

├────────┬──────┬─────────────────────────────────────┬──────────┤

│                   │              │                                                                                     │                       │

│Sidebar      │ Ent.     │                             · · · · · · · · · · · · · · · · ·                        │  Right           │

│                    │Strip     │                                                                                     │  Panel           │

│Binders      │(52px)   │           ┌──────────────────────┐                  │                      │

│+ Pages      │              │           │             Bounded Page            │                   │  Layers        │

|                    |              |           |                                                    |                   |   Entities     |

│                    │Markup│           │                 8.5 × 11 in                │                   │  Properties │

│(196px)       │Tools    │           └──────────────────────┘                   │  Backlinks  │

│                    │              │                    · · · · · · · · · · · · · · · · ·                                 │  (200px)    │

├────────┴──────┴─────────────────────────────────────┴──────────┤

│  ← →  │  Project Alpha / Char Notes → Novel: Ashfall / Lore   75% −  +  │  Bottombar (42px)

└────────────────────────────────────────────────────────────────┘

## **2.3  Chrome Regions**

### **Menubar  ·  27px  ·  #090909**

The darkest surface. WeftWarp wordmark (Syne 800, 12px, rotated red square mark), native-style menu items (File, Edit, View, Export — Barlow Condensed 500, 11px, #686868), right-aligned save indicator (Space Mono 9px with a 5px dot that pulses red when unsaved).

### **Toolbar + Dynamic Edit Bar  ·  38px  ·  #111**

Left region: icon button groups (tbtn, 26×26px, border-radius 4px) for grid toggle, layer model toggle, and clean view toggle. Groups separated by 1px dividers.

The Clean View toggle (eye icon with a slash, or a distinct 'clean' glyph) switches the canvas between edit mode and clean mode. In clean mode, all entity chrome is suppressed — card headers, type labels, resize handles, selection indicators, corner bracket marks, and the dot grid are hidden. Only content is visible. The toggle is a persistent app-level state, not per-selection. Its active state uses the standard --red-glow fill like other active toolbar buttons.

Center region — the Dynamic Edit Bar — reconfigures based on active selection:

| **Nothing selected**        | Page dimension pills (8.5×11 in, Portrait), background color picker, grid toggle                |
| --------------------------- | ----------------------------------------------------------------------------------------------- |
| **Content entity selected** | Entity type label + type controls (text: Bold/Italic/Link; image: Crop/Fit; checklist: reorder) |
| **Markup entity selected**  | Stroke weight, stroke color, fill, arrowhead style                                              |
| **Multi-select**            | Align (L/R/T/B/Center), Distribute, Match Size, Delete                                          |

### **Sidebar  ·  196px  ·  #0e0e0e**

Search row at top (26px, #151515 fill). Scrollable tree below, organized into labeled sections (Space Mono 8px, #505050, all-caps):

- BINDERS — collapsible tree items, folder icon, binder name (Syne 600, 11.5px), page count badge. Active binder: 2px red left border stripe.

- Pages under open binder — indented (padding-left: 26px), 8×8px dot fills red on active page. Syne 500, 11px.

- TAGS — flat tag list.

- PINNED — pinned entities, no nesting.

Footer: Save, Sync, Import buttons (full-width, 24px, Barlow Condensed 600, 10px).

### **Entity Strip  ·  52px  ·  #0d0d0d**

Narrow vertical rail between sidebar and canvas. The primary mechanism for adding content entities to the canvas. Each type is a 40×40px button (es-btn) with icon (20px) above a 6.5px Space Mono label. Drag to place on canvas; click to place at page center.

Top to bottom: Text, Jot, Checklist, Image, Video, Audio, Link, Document, File, Color Swatch.

Markup tools below (same panel): Arrow, Line, Rect, Label, Comment — 32×32px buttons, one active at a time.

### **Canvas Area  ·  flex: 1  ·  #141414**

The workspace surround. Pan: middle-click drag or Space+drag. Zoom: scroll wheel or pinch, centered on cursor. The page floats centered in this area with a box-shadow (0 8px 40px rgba(0,0,0,0.7)) lifting it off the surround. A dot grid (rgba(255,255,255,0.045)) fills the surround.

### **Right Panel  ·  200px  ·  #0e0e0e**

Context-sensitive. Four tab sections accessible by icon strip at top:

| **Layers**     | Layer stack for current page. Visibility and lock toggles. Drag to reorder user layers.                                                                  |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Entities**   | Ordered list of all placement and markup entities on the active layer. Primary surface for z-index management and grouping. See §2.3 Entities Tab below. |
| **Properties** | Selected entity or group properties: position (x, y), size (w, h), rotation, style overrides, tags.                                                      |
| **Backlinks**  | Flat list of entities that reference the selected entity via [[e:uuid]]. With context excerpts.                                                          |

### **Bottombar  ·  42px  ·  #111**

The bottombar contains the navigation controls and path trail, using the otherwise-idle space in this strip effectively. Left to right: back [←] and forward [→] buttons (tbtn, 22×22px), a 1px vertical divider, the path trail filling all remaining width, and the zoom percentage readout (Space Mono 9px) with − and + controls flush right.

Back and forward are disabled (dimmed, non-interactive) when at the start or tip of the history stack respectively.

The path trail is a horizontal sequence of <ww-nav-crumb> elements (Barlow Condensed 500, 11px, --white-dim) separated by two types of dividers:

| **/ (slash, --muted)**       | Structural separator — binder name / page name within the same navigation origin.                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **→ (arrow, --muted-light)** | Cross-reference hop separator — the user followed a [[e:uuid]] link to reach the next destination, potentially crossing binder boundaries. |

Each crumb is individually clickable — clicking navigates directly to that point in the trail. Hovering shows a tooltip with the full name if truncated, and a miniature thumbnail for page crumbs. When the trail exceeds available width, older crumbs collapse into a '…' overflow element from the left — the current destination is always fully visible.

→ *In Workspace view and Binder Gallery view the path trail is hidden and only the zoom controls remain visible. The bottombar is always present but its nav content is only meaningful in Page Edit view.*

### **Entities Tab**

The Entities tab shows every placement and markup entity on the currently active layer as a named, ordered list — mirroring the z-stack from top to bottom. It is the primary interface for two operations that are impractical to perform directly on a dense canvas: z-index management and grouping.

**Z-index management:**

- Each row shows the entity type icon, its name (or type label if unnamed), and a drag handle on the left.

- Drag rows to reorder — this directly updates the z component of the placement_data or the markup entity's order.

- The active selection is highlighted in the list; clicking a row selects that entity on the canvas and vice versa. Selection is bidirectional.

- Right-click a row: Bring to Front, Send to Back, Move Up One, Move Down One.

**Grouping and nesting:**

Atomic content entities can be grouped within a layer to form a named cluster. Groups are a first-class concept in the Entities tab but are transparent to the ECS store — a group is represented as a lightweight container entity (type: group) whose children reference their members via membership components scoped to that group.

| **Create group**          | Select two or more entities on canvas or in the list → Group button (or Cmd/Ctrl+G) → entities nest under a new group row, collapsed by default.                          |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ungroup**               | Select group row → Ungroup. Children return to flat layer membership.                                                                                                     |
| **Rename group**          | Double-click group row label → inline rename.                                                                                                                             |
| **Collapse / expand**     | Click the disclosure triangle on a group row. Collapsed groups still render normally on canvas.                                                                           |
| **Move group**            | Drag group row to reorder it relative to other entities and groups. All children move with it.                                                                            |
| **Nest into group**       | Drag an entity row onto a group row → entity becomes a child of that group.                                                                                               |
| **Select group contents** | Clicking a group row on canvas selects all children as a multi-select. Individual children remain independently selectable with a single click inside the group boundary. |

→ *Groups affect z-ordering and canvas selection behavior only. They do not create a new ECS hierarchy level — child entities remain on the same layer and are fully individually addressable. A group is a view-layer organizational unit, not a structural one.*

## **2.4  The Canvas & Content Cards**

### **Bounded Page**

Hard page boundary with four corner bracket marks (12×12px L-shapes, rgba(232,57,42,0.35), 1px stroke) always visible in edit mode. Content beyond the boundary renders in a desaturated bleed style while editing; hard-clipped on export. No infinite canvas.

### **Content Cards**

Content entities on the canvas are placement-wrapped cards (c-card):

┌─ [ico] TYPE · Entity Name ─────────────────────────────┐

│  (Space Mono 9px, #5a5a6e — type label in red at 60%)              │

├─────────────────────────────────────────────────┤

│  Heading (Syne 700, 12px, #eaeaea)                                                 │

│  Body copy (Barlow Condensed 11px, #8a8a9a)                             │

│  [[inline ref]] — red text, dotted underline, cursor: pointer          │

└─────────────────────────────────────────────────┘

Card background: #1a1a22. Border-radius: 5px. Min dimensions: 80×40px.

Selected: red border (var(--red)) + 0 0 0 1px rgba(232,57,42,0.4) ring + 0 4px 24px rgba(0,0,0,0.5) elevation. Eight resize handles (rh): 7×7px circles, #1a1a22 fill, 1.5px red border, at corners and midpoints.

### **Markup Entities**

Arrows, rects, and labels render as SVG elements on an overflow:visible SVG layer — no card chrome. Directly manipulable on the canvas.

### **Clean Mode**

When the Clean View toggle is active, the canvas enters a read-only display state optimized for visual review and export. The following elements are suppressed:

| **Card headers**                    | Type icon, type label, entity name strip — hidden. Content body fills the full card area.                                                            |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Resize handles**                  | All eight rh handles hidden. No selection affordances.                                                                                               |
| **Selection state**                 | Active selection border and red ring suppressed. Cards appear unselected regardless of internal state.                                               |
| **Corner bracket marks**            | Page boundary brackets hidden.                                                                                                                       |
| **Dot grid**                        | Canvas surround dot grid hidden.                                                                                                                     |
| **Bleed zone styling**              | Out-of-bounds content renders normally rather than desaturated — useful for visual review before deciding to crop.                                   |
| **Entity Strip & toolbar edit bar** | Entity strip and Dynamic Edit Bar are dimmed and non-interactive while clean mode is active. The clean view toggle itself remains clickable to exit. |

Canvas interaction in clean mode is intentionally limited: pan and zoom work normally, but drag, resize, and entity creation are disabled. Clicking entities does not trigger selection. The intent is a stable, low-noise viewing surface.

→ *Clean mode state is not persisted. It resets to off on workspace reload. It is not a layer visibility toggle — it is a rendering mode that affects all layers simultaneously.*

## **2.5  Interaction Model**

### **Placing Entities**

- Drag from entity strip → drop onto canvas → creates placement at drop position

- Click entity strip button → creates placement at page center

- Drag-and-drop image file from OS → ingests as image entity, creates placement at drop

### **Selecting & Manipulating**

- Click card → single select, shows resize handles

- Shift+click → add to selection

- Drag empty canvas area → marquee select

- Drag card → reposition (snaps to grid if grid is on)

- Drag resize handle → resize placement

- Double-click card → enter content edit mode for that entity

### **Cross-Reference Input**

Type @ anywhere in a text or document entity → fuzzy search dropdown over all entity names in the workspace. Selecting inserts [[e:uuid]]; displayed text shows the entity's current name. Works across binder boundaries.

### **Reference Navigation**

Inline entity references ([[e:uuid]]) that point to a page or binder entity are navigable — they are not just display labels. The navigation behavior depends on the target type:

| **Click a page ref**           | Navigates to that page in Page Edit view. Pushes to history stack with via: 'ref'. Bottombar trail gains a '→ Binder Name / Page Name' segment.                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Click a binder ref**         | Navigates to that binder's Gallery view. Pushes to history stack. Bottombar trail gains a '→ Binder Name' segment.                                                                   |
| **Click a content entity ref** | Does not navigate. Opens the entity's hover preview inline, or opens its detail modal (for document-type entities). Navigation only occurs for structural entities (pages, binders). |
| **Cmd/Ctrl+Click any ref**     | Opens in the background — no navigation, no history push. Preview panel updates to show the referenced entity.                                                                       |

→ *The distinction betwe**en navigating and previewing is intentional. Cross-referencing is the core of the workspace's value — navigating to every referenced entity on a click would make dense pages unworkable. Hover previews satisfy casual curiosity; a deliberate click navigates.*

### **Hover Previews**

| **Content entity ref** | Compact tooltip: entity name, type icon, content excerpt     |
| ---------------------- | ------------------------------------------------------------ |
| **Page ref**           | Miniature canvas thumbnail — postage stamp view of the board |
| **Binder ref**         | Binder description + list of contained pages                 |

## **2.7  Navigation Bar**

The navigation trail in the bottombar implements a browser-style history model scoped to the WeftWarp session. It is the primary affordance for moving through a web of cross-referenced pages without losing your place.

### **History Stack**

Every navigation action pushes an entry onto a history stack owned by <ww-app>. The stack records:

| **type**        | 'page' \| 'gallery' \| 'workspace'                                                         |
| --------------- | ------------------------------------------------------------------------------------------ |
| **binderId**    | UUID of the active binder, or null for workspace view                                      |
| **pageId**      | UUID of the active page, or null for gallery and workspace views                           |
| **via**         | 'sidebar' \| 'gallery' \| 'ref' \| 'back' \| 'forward'                                     |
| **refEntityId** | UUID of the [[e:uuid]] token that triggered the navigation, if via: 'ref'. Null otherwise. |

Back navigation decrements the stack pointer. Forward navigation increments it. Any new navigation (not via back/forward) truncates the forward history and pushes a new entry — identical to browser history behavior.

→ *The history stack is session-only. It is not persisted in the .wwz file. On workspace reload, history resets to a single entry for the last-open page (read from workspace.meta).*

### **Path Trail vs. History Stack**

The history stack and the path trail are related but distinct:

| **History stack**                  | Full record of every page visited this session, in order. Powers the back/forward buttons. May contain many entries spanning multiple binders and many sidebar navigations.                                                                          |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Path trail (bottombar display)** | A condensed view of the current cross-reference chain since the last sidebar navigation. Resets to a single crumb whenever the user navigates via the sidebar or gallery — those are treated as new origins, not continuations of a reference chain. |

The path trail is derived from the history stack on each navigation: scan backward from the current stack entry until a via: 'sidebar' or via: 'gallery' entry is found. Everything from that origin forward is the current trail. This means the trail always starts at a 'home base' the user deliberately chose, and shows only the cross-reference hops taken from there.

Example:

| **User opens 'Project Alpha / Character Notes' from sidebar** | Trail: Project Alpha / Character Notes                                                          |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Clicks [[e:uuid]] ref to 'Novel: Ashfall / Backstory'**     | Trail: Project Alpha / Character Notes → Novel: Ashfall / Backstory                             |
| **Clicks [[e:uuid]] ref to 'World Building / Lore Map'**      | Trail: Project Alpha / Character Notes → Novel: Ashfall / Backstory → World Building / Lore Map |
| **Opens 'Game Dev / Sprint Board' from sidebar**              | Trail resets: Game Dev / Sprint Board                                                           |
| **Presses back ←**                                            | Returns to World Building / Lore Map. Trail shows previous chain again.                         |

### **Crumb Interaction**

| **Click crumb**  | Navigates directly to that page/binder. Pushes a new history entry (via: 'ref' is not appropriate here — use via: 'crumb'). Trail truncates to the clicked crumb.     |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hover crumb**  | Shows tooltip with full binder + page name (useful when crumbs are truncated). For a page crumb, shows the same miniature thumbnail as hover previews on entity refs. |
| **Overflow '…'** | Clicking opens a dropdown list of the collapsed older crumbs, each clickable for direct navigation.                                                                   |

## **2.6  File Operations (as the User Experiences Them)**

| **Open workspace** | File → Open or drag .wwz file onto the window → workspace loads                                                                                                                                     |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Save**           | File → Save or Cmd/Ctrl+S → saves in place, save indicator updates                                                                                                                                  |
| **Autosave**       | 30s after last change, save indicator shows pulse then settles                                                                                                                                      |
| **Save As**        | File → Save As → OS file picker, saves copy to new path                                                                                                                                             |
| **Export page**    | Export → PNG or PDF → OS save dialog, crops to page boundary. Export always renders in clean mode regardless of current toggle state — card chrome and corner marks never appear in exported files. |

→ *These interactions define what the storage layer must support. Implementation follows.*

 

# **3. Design Language**

## **3.1  Philosophy**

The visual language is tool-first and unsentimental. The application reads as a professional desktop instrument — not a productivity app, not a notes app, not a consumer creative product. Every surface earns its real estate. Nothing decorates; everything communicates.

The palette is near-monochrome with a single high-signal accent. Structure is flat but not sterile — the distinction between chrome and content is spatial and tonal, not achieved through gradients or drop shadows on UI elements. The page itself is the one place depth is allowed, emerging from its dark surround by contrast and a layered box-shadow.

## **3.2  Color Palette**

| **Role**              | **Token**      | **Value**               |
| --------------------- | -------------- | ----------------------- |
| App background        | --bg           | #161616                 |
| Sidebar / right panel | --sidebar-bg   | #0e0e0e                 |
| Secondary panels      | --panel-bg     | #1a1a1a                 |
| Card fill             | --card-bg      | #1e1e1e                 |
| Card border           | --card-border  | #2a2a2a                 |
| Border default        | --border       | #242424                 |
| Border hover/active   | --border-light | #333333                 |
| Red accent            | --red          | #e8392a                 |
| Red dimmed            | --red-dim      | #9c2820                 |
| Red glow (fills)      | --red-glow     | rgba(232,57,42,0.15)    |
| Primary text          | --white        | #eaeaea                 |
| Secondary text        | --white-dim    | #bbbbbb                 |
| Muted text            | --muted        | #6a6a6a                 |
| Canvas surround       | --canvas-bg    | #141414                 |
| Page surface          | --page-bg      | #1c1c24                 |
| Dot grid              | --dot-color    | rgba(255,255,255,0.045) |

The red accent (#e8392a) appears exclusively on: active selection borders, active state fills (15% opacity), the logo mark, active sidebar item indicator, and entity type labels in card headers. Its presence is a signal, not decoration.

## **3.3  Typography**

| **Syne (800, 600, 500)**       | Display and primary UI. Wordmark, binder names, page titles, card headings, sidebar labels. Geometric, slightly wide, confident.                                                        |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Space Mono**                 | Data and metadata. Section headers, card header type labels, property rows, save indicator, zoom readout, badge counters. 6.5–9px. Monospaced to signal 'this is a reading, not prose.' |
| **Barlow Condensed (500–700)** | Controls and body text. Buttons, edit bar actions, dropdown items, checklist text, card body copy, footer actions. Packs more label into less width at small sizes.                     |

Scale: UI chrome operates 8–12px. Card body text 11–12px. Nothing exceeds 14px except card headings (12px bold Syne). No body copy role in chrome.

## **3.4  Icon System**

All icons are custom SVG sprites in a <defs> block, referenced via <use href="#ico-...">. Exactly three fill values: #111 (base mass), #353434 (mid shadow), #ef4436 (red accent detail). White highlights use #fefbfa. This three-value system produces icons that read clearly at 14–20px.

Icons sized via --icon-size CSS property on a wrapper .icon element: 18px for toolbar, 11–14px for sidebar and panel. No icon uses more than four distinct shapes.

 

# **4. ECS Architecture**

Everything in WeftWarp is an Entity — a row in a flat store with an ID and a type. Binders, Pages, Layers, text blocks, images, checklists, link references, and canvas placements are all entities. Structure and content are composed from Components attached to entities.

## **4.1  Content Entities vs. Placement Entities**

This is a core architectural distinction — the foundation that makes the canvas extensible without per-type reimplementation.

| **Content Entity**                                    | **Placement Entity**                                        |
| ----------------------------------------------------- | ----------------------------------------------------------- |
| Content Entity                                        | Placement Entity                                            |
| Independently meaningful data object                  | Canvas-level wrapper instance                               |
| Lives in the flat store regardless of placement       | References one content entity + carries canvas metadata     |
| Carries content components (text, image_ref, etc.)    | Carries: position, size, z-order, rotation, style overrides |
| Participates in [[e:uuid]] reference system           | Does not participate in reference system                    |
| Appears in @ search, has backlinks, ww:// addressable | Scoped to one layer via membership component                |
| Deleting removes content everywhere                   | Deleting removes only this appearance; content persists     |

The same content entity can appear on five different Pages simultaneously via five different placement entities. Edit the content once — it updates everywhere. Move or restyle one placement — other Pages are unaffected.

→ *Frontend embed logic: a placement div is rendered* *with all UX applied (drag, resize, rotate, select, style). The content entity is injected into that div by a per-type renderer. The renderer is pure display — zero UX code. Adding a new content type requires only a new renderer; all canvas UX is automatic.*

## **4.2  Markup Entities**

Arrows, rects, labels, and comments are purely presentational and carry their own positional data directly in their component. They use membership to belong to a layer but do not use placement entities or content_ref. They do not participate in the reference system.

## **4.3  The Hierarchy**

Workspace

  └── Binder            (a project — clicking opens page gallery)

        └── Page        (bounded canvas with defined dimensions)

              └── Layer (z-ordered surface: Markup, Content, Background)

                    ├── Placement Entity  (references a content entity)

                    ├── Markup Entity     (arrow, rect, label, comment)

                    └── Group Entity      (named cluster, children via membership)

                          ├── Placement Entity

                          └── Markup Entity

Every level is itself an entity in the flat store, related to its parent via a membership component. The hierarchy is navigational, not storage-structural.

## **4.4  Layer Model**

| **Markup (top)**        | Arrows, rects, labels, comments. Always topmost. Position locked.   |
| ----------------------- | ------------------------------------------------------------------- |
| **Content (middle)**    | Placed content entities. Default user-facing layer.                 |
| **Background (bottom)** | Color fills, background images. Always bottommost. Position locked. |

Additional user-defined content layers can be inserted between Markup and Background. Toggle visibility and lock per layer.

## **4.5  Cross-Reference System**

### **Addressing**

Every entity has a stable URI: ww://entity/{uuid}. IDs are UUIDs, never positional or name-based. Rename or move anything — links never break. Broken links (deleted target) surface visually with a distinct style.

### **Inline Syntax**

Text content uses BBML — a markdown-adjacent format supporting inline entity references:

The [[e:a3f9c2]] first appeared in [[e:8b21d7]]'s domain.

Resolved at render time to the entity's current name. Renames propagate automatically everywhere the entity is referenced.

### **@ Input**

Type @ anywhere in a text or document entity → fuzzy search dropdown over all entity names. Selecting inserts [[e:uuid]]; displayed text shows the entity or page or binder name. Works across binder boundaries.

### **Reference Index**

A reverse index of all [[e:uuid]] tokens is maintained in the entity_refs table. Rebuilt on every text or document component write by diffing existing rows against parsed tokens. This is the source for backlinks and is maintained by the storage layer — never written directly by the UI.

 

# **5. Frontend Architecture**

WeftWarp is built entirely on native browser APIs — no framework, no bundler, no virtual DOM. The organizational system that replaces a framework is Web Components: every distinct UI concern is a self-contained custom element with its own template, styles, and behavior. The platform enforces the boundary; convention does not have to.

## **5.1  Why Web Components**

The ECS data model already describes a system where entity types are independent and composable. Web Components apply the same principle to the UI layer. Each component is a black box: it has defined inputs (attributes and properties), defined outputs (custom events), and internal encapsulation (Shadow DOM). Nothing outside can accidentally break it; it cannot accidentally break anything outside.

This maps directly to the renderer architecture already described in Section 4: a content entity renderer is pure display with zero UX code. A custom element is exactly this — it receives data, renders markup into its shadow root, and dispatches events upward. The placement wrapper handles all UX. The renderer never sees or touches it.

| **Principle**       | **Consequence**                                                                                                                   |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Isolation           | Shadow DOM scopes styles per component. A renderer's CSS cannot leak. The canvas's CSS cannot intrude. No specificity wars.       |
| Replaceability      | Swapping a renderer is registering a new custom element. Nothing else changes. Debug one file, ship one file.                     |
| No build step       | Custom elements are registered and used directly. No transpilation. No module bundler required. Files are what the browser loads. |
| Platform longevity  | Web Components are a browser standard. No framework deprecation risk.                                                             |
| Tauri compatibility | The same components work in a Tauri WebView unchanged. No adaptation layer.                                                       |

## **5.2  Component Inventory**

Every named UI region in the layout is a registered custom element. The component tree mirrors the layout hierarchy directly.

### **Application Shell**

| **<ww-app>**          | Root element. Owns router state, navigation history stack, navigation index, workspace reference, global event bus. Renders <ww-sidebar>, <ww-canvas> or <ww-gallery>, <ww-right-panel>. Pushes history entries on all navigation events and updates <ww-bottombar> accordingly.                                             |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **<ww-menubar>**      | Wordmark, File/Edit/View/Export menus, save indicator dot. Dispatches menu actions as custom events to <ww-app>.                                                                                                                                                                                                             |
| **<ww-toolbar>**      | Left icon groups + center Dynamic Edit Bar slot. Owns clean-mode state. Listens for selection-change events from canvas; reconfigures edit bar accordingly. Dispatches clean-mode-change to <ww-app> on toggle.                                                                                                              |
| **<ww-clean-toggle>** | Clean view toggle button (tbtn). Active state: --red-glow fill. When active, dispatches clean-mode-enter; on deactivate, dispatches clean-mode-exit. <ww-app> propagates these to <ww-canvas> and <ww-entity-strip>.                                                                                                         |
| **<ww-bottombar>**    | Back/forward nav buttons, path trail of <ww-nav-crumb> elements, and zoom controls. Receives the current trail array as a property from <ww-app> on every navigation. Dispatches ww:nav-back, ww:nav-forward, and ww:zoom-change. Trail and nav buttons hidden in Workspace and Gallery views; zoom controls always visible. |
| **<ww-nav-crumb>**    | A single segment in the navigation trail. Displays binder name and/or page name (Barlow Condensed 500, 11px). Attribute: binder-id, page-id, separator ('/' or '→'). Clickable — dispatches ww:nav-to with { binderId, pageId }. Shows thumbnail tooltip on hover for page crumbs.                                           |

### **Sidebar**

| **<ww-sidebar>**     | Outer sidebar shell. Search row + scrollable tree + footer buttons. Owns collapsed/expanded state per binder. |
| -------------------- | ------------------------------------------------------------------------------------------------------------- |
| **<ww-binder-item>** | Single binder row. Folder icon, name, badge. Attribute: binder-id. Dispatches binder-select, binder-toggle.   |
| **<ww-page-item>**   | Single page row under an open binder. Dot indicator, name. Attribute: page-id. Dispatches page-select.        |
| **<ww-tag-item>**    | Single tag row. Dispatches tag-filter.                                                                        |

### **Binder Gallery**

| **<ww-gallery>**    | Page thumbnail grid. Drag-to-reorder, inline rename, new-page button. Owns drag state.                                                                |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **<ww-page-thumb>** | Single page thumbnail. Renders a scaled-down canvas preview. Attribute: page-id. Handles click (rename), dblclick (open), right-click (context menu). |

### **Canvas & Placement**

| **<ww-canvas>**     | The canvas surround. Owns pan/zoom transform, dot grid, page drop target. Renders <ww-page> inside.                                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **<ww-page>**       | The bounded page surface. Owns layer stack. Renders <ww-layer> for each layer. Enforces clip boundary. Handles marquee selection.                                                                             |
| **<ww-layer>**      | A single z-ordered surface. Renders its placement entities and markup entities. Attribute: layer-id, layer-class (markup/content/background).                                                                 |
| **<ww-placement>**  | The universal placement wrapper. Handles drag, resize (8 handles), rotate, select, z-order. Injects the appropriate content renderer into its slot. Attribute: placement-id. Dispatches select, move, resize. |
| **<ww-markup-svg>** | SVG overlay layer for markup entities (arrows, rects, labels). Owns the SVG coordinate space synchronized with the page.                                                                                      |

### **Content Entity Renderers**

Each renderer is a custom element that receives an entity and its components, renders pure display markup into its shadow root, and dispatches content-edit events when the user interacts with editable regions. Zero UX code — no drag, no resize, no selection logic.

| **<ww-render-text>**         | BBML rich text. Parses [[e:uuid]] tokens into <ww-entity-ref> inline elements. Dispatches text-edit on double-click. |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **<ww-render-jot>**          | Single-line note. Minimal chrome.                                                                                    |
| **<ww-render-checklist>**    | Checkable item list. Dispatches item-toggle, item-edit.                                                              |
| **<ww-render-image>**        | Embedded image from embed hash. Handles non-destructive crop display.                                                |
| **<ww-render-video>**        | Compact video player. Controls inside shadow root.                                                                   |
| **<ww-render-audio>**        | Compact waveform + playback controls.                                                                                |
| **<ww-render-document>**     | Icon + word count preview. Dblclick opens <ww-document-editor> modal.                                                |
| **<ww-render-link>**         | Renders target entity name as a clickable chip. Dispatches entity-navigate.                                          |
| **<ww-render-color-swatch>** | Color preview chip with hex/name display.                                                                            |
| **<ww-render-file>**         | Icon, filename, download trigger.                                                                                    |

### **Cross-Reference Elements**

| **<ww-entity-ref>**    | Inline reference chip inside BBML text. Displays resolved entity name. Shows <ww-hover-preview> on hover. Attribute: entity-id.                      |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **<ww-hover-preview>** | Tooltip preview card. Renders content excerpt for entities, miniature canvas for pages, description for binders. Positioned relative to its trigger. |
| **<ww-at-suggest>**    | @ mention autocomplete dropdown. Appears inside text editors. Dispatches ref-insert on selection.                                                    |

### **Right Panel**

| **<ww-right-panel>**      | Panel shell with tab strip (Layers / Entities / Properties / Backlinks). Owns active tab state.                                                                                                                                   |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **<ww-layer-panel>**      | Layer stack list. Drag-to-reorder user layers. Visibility and lock toggles per layer.                                                                                                                                             |
| **<ww-entities-panel>**   | Ordered entity list for the active layer. Drag rows to reorder z-index. Bidirectional sync with canvas selection. Group/ungroup controls. Disclosure tree for nested groups. Updates on layer-change and selection-change events. |
| **<ww-entity-row>**       | Single row in the entities list. Type icon, name label, drag handle, disclosure triangle (if group). Attribute: entity-id, is-group, depth. Dispatches row-select, row-reorder, row-group.                                        |
| **<ww-properties-panel>** | Position, size, rotation inputs. Style override controls. Tag chip list. Updates on selection-change events. Works for both single entities and groups (shows shared/mixed values).                                               |
| **<ww-backlinks-panel>**  | Flat list of backlink rows with context excerpts. Queries db.getBacklinks() on selection-change.                                                                                                                                  |

### **Entity Strip**

| **<ww-entity-strip>** | Outer rail. Renders content type buttons above markup tool buttons.                                            |
| --------------------- | -------------------------------------------------------------------------------------------------------------- |
| **<ww-es-btn>**       | Single entity strip button. Icon + Space Mono label. Drag source for canvas placement. Attribute: entity-type. |
| **<ww-tool-btn>**     | Single markup tool button. Activates draw mode. One active at a time. Attribute: tool-type.                    |

## **5.3  Communication Model**

Components communicate upward via custom events dispatched with bubbles: true, composed: true. They receive data downward via attributes and properties set by parent components. No component reaches into a sibling or cousin — all cross-tree communication routes through a common ancestor or the global event bus on <ww-app>.

| **Downward (parent → child)** | Set properties or attributes directly. e.g. placementEl.entity = entityData                                                                                                |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Upward (child → parent)**   | Dispatch custom events. e.g. dispatchEvent(new CustomEvent('ww:select', { bubbles: true, composed: true, detail: { id } }))                                                |
| **Cross-tree**                | Route through <ww-app>'s event bus for events that need to cross from canvas to sidebar or panel.                                                                          |
| **Storage updates**           | Components call db.js and storage.js directly — they do not proxy through a parent. The event model is for UI state; data operations go directly to the storage interface. |

## **5.4  File Organization**

Each custom element lives in its own file, named after the element. No file contains more than one component definition. The directory structure mirrors the component inventory above.

js/

  app/

    ww-app.js

    ww-menubar.js

    ww-toolbar.js

    ww-clean-toggle.js

    ww-nav-crumb.js

    ww-bottombar.js

  sidebar/

    ww-sidebar.js

    ww-binder-item.js

    ww-page-item.js

    ww-tag-item.js

  gallery/

    ww-gallery.js

    ww-page-thumb.js

  canvas/

    ww-canvas.js

    ww-page.js

    ww-layer.js

    ww-placement.js

    ww-markup-svg.js

  renderers/

    ww-render-text.js

    ww-render-jot.js

    ww-render-checklist.js

    ww-render-image.js

    ww-render-video.js

    ww-render-audio.js

    ww-render-document.js

    ww-render-link.js

    ww-render-color-swatch.js

    ww-render-file.js

  refs/

    ww-entity-ref.js

    ww-hover-preview.js

    ww-at-suggest.js

  panel/

    ww-right-panel.js

    ww-layer-panel.js

    ww-entities-panel.js

    ww-entity-row.js

    ww-properties-panel.js

    ww-backlinks-panel.js

  strip/

    ww-entity-strip.js

    ww-es-btn.js

    ww-tool-btn.js

  storage/

    storage.js

    db.js

→ *storage/ is the only directory whose files are not custom elements. It is the abstraction boundary described in Section 7.*

 

# **6. Data Model**

All data lives in SQLite inside the .wwz archive. The schema is type-agnostic — no migrations are needed to add new entity types.

## **6.1  entities**

CREATE TABLE entities (

    id          TEXT PRIMARY KEY,

    type        TEXT NOT NULL,

    class       TEXT NOT NULL DEFAULT 'content',  -- 'content' | 'markup'

    name        TEXT,

    created_at  INTEGER NOT NULL,

    updated_at  INTEGER NOT NULL

);

| **Structural types** | binder, page, layer                                                                |
| -------------------- | ---------------------------------------------------------------------------------- |
| **Content types**    | text, jot, checklist, image, video, audio, svg, link, color_swatch, document, file |
| **Markup types**     | placement, arrow, line, rect, label, comment, group                                |

## **6.2  components**

CREATE TABLE components (

    id          TEXT PRIMARY KEY,

    entity_id   TEXT NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    type        TEXT NOT NULL,

    data        TEXT NOT NULL,   -- JSON

    created_at  INTEGER NOT NULL,

    updated_at  INTEGER NOT NULL

);

### **Structural Components**

| **membership**    | { "parent_id": "uuid", "order": 12 }                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **layer_config**  | { "z_order": 2, "visible": true, "locked": false }                                                                        |
| **page_config**   | { "width_in": 8.5, "height_in": 11.0, "orientation": "portrait", "background": "#1a1a2e", "grid": true, "grid_size": 10 } |
| **binder_config** | { "icon": "folder", "color": "#ff4444", "description": "" }                                                               |

### **Placement Components**

| **content_ref**    | { "entity_id": "uuid" }                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| **placement_data** | { "x": 120, "y": 340, "w": 280, "h": 160, "z": 1, "rotation": 0 }                                         |
| **style**          | { "background": null, "border_color": null, "border_width": null, "border_radius": null, "opacity": 1.0 } |
| **tags**           | { "tags": ["important", "wip"] }                                                                          |

### **Markup Components**

| **arrow**   | { "x1": 100, "y1": 200, "x2": 400, "y2": 350, "style": "solid", "head": "arrow", "tail": "none", "weight": 2, "color": "#ffffff" }                         |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **rect**    | { "x": 80, "y": 120, "w": 500, "h": 300, "fill": "#ffffff10", "stroke": "#ffffff40", "stroke_width": 1, "radius": 4 }                                      |
| **label**   | { "x": 100, "y": 80, "body": "Section header", "size": 14, "color": "#aaaaaa" }                                                                            |
| **comment** | { "x": 200, "y": 150, "body": "Check this layout", "color": "#ffee88" }                                                                                    |
| **group**   | { "name": "Hero section", "collapsed": false }  — children linked via membership components (parent_id = group entity id, order = z position within group) |

### **Content Components**

| **text**                  | { "body": "...", "format": "wwml" }                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| **jot**                   | { "body": "..." }                                                                                |
| **checklist**             | { "items": [{ "id": "uuid", "text": "...", "checked": false, "order": 0 }] }                     |
| **image_ref**             | { "hash": "a3f9c2...", "alt": "...", "caption": "", "crop": { "x": 0, "y": 0, "w": 1, "h": 1 } } |
| **video_ref / audio_ref** | { "hash": "...", "caption": "" }                                                                 |
| **svg**                   | { "data": "<svg>...</svg>" }                                                                     |
| **link_ref**              | { "target_id": "uuid", "display": "alias" }                                                      |
| **color_swatch**          | { "hex": "#ff4444", "name": "Crimson", "format": "hex" }                                         |
| **document**              | { "body": "...", "format": "wwml", "word_count": 1240 }                                          |
| **file_ref**              | { "hash": "...", "original_name": "spec.pdf", "mime_type": "application/pdf", "size": 204800 }   |

## **6.3  entity_refs**

Reverse index of all [[e:uuid]] tokens. Maintained by the storage layer on every text/document write. Never written directly by UI code.

CREATE TABLE entity_refs (

    id              TEXT PRIMARY KEY,

    source_entity   TEXT NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    target_entity   TEXT NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    component_id    TEXT NOT NULL REFERENCES components(id) ON DELETE CASCADE,

    context         TEXT   -- short excerpt around the reference

);

CREATE INDEX idx_refs_target ON entity_refs(target_entity);

## **6.4  embeds**

CREATE TABLE embeds (

    hash        TEXT PRIMARY KEY,   -- SHA-256 of file contents

    mime_type   TEXT NOT NULL,

    size        INTEGER NOT NULL,

    created_at  INTEGER NOT NULL

);

Files in embeds/ are named by their content hash. Deduplication is structural — multiple entities referencing the same image share one file on disk. The embed table is an index only.

## **6.5  workspace**

CREATE TABLE workspace (

    id          INTEGER PRIMARY KEY CHECK (id = 1),

    name        TEXT NOT NULL DEFAULT 'Workspace',

    created_at  INTEGER NOT NULL,

    updated_at  INTEGER NOT NULL,

    meta        TEXT   -- JSON: theme, last_open_binder, zoom, etc.

);

 

# **7. Storage Layer**

The storage layer is the only code in the application that knows about SQLite, zip archives, and file system access. All other modules interact with it through a clean interface. This boundary is what makes the Tauri native build a single-file swap rather than a rewrite.

## **7.1  Serving Model**

WeftWarp has no server-side component. The application is a set of static files — HTML, JS, CSS, and SVG sprites — that are served once and then run entirely in the browser. There is no API endpoint, no server process to keep alive, and no network dependency after the initial load.

| **Hosting requirement** | Any static file server or CDN. GitHub Pages, Netlify, S3 with static hosting, a self-hosted nginx — all equivalent. No Node.js, no Docker, no database server.                |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Offline capability**  | After the first load, the application functions without network access. A Service Worker caches all application assets on first visit, enabling fully offline use thereafter. |
| **Network activity**    | None during normal use. The only planned network activity is file sync (WebDAV/Nextcloud, post-v1), which is opt-in and operates on .wwz files as atomic units.               |
| **Self-hosting**        | A user can download the static bundle and serve it from localhost or a local network server. The application behaves identically.                                             |

This model is the direct consequence of the JS-native stack decision. Because all computation (SQLite via WASM, zip I/O via JSZip, hashing via Web Crypto) runs in the browser, there is simply nothing left for a server to do. The architecture did not aim for this property — it emerged from the correct stack choice.

## **7.2  PWA and Service Worker**

WeftWarp registers a Service Worker on first load to cache all application assets. This enables reliable offline use and fast subsequent loads.

| **Cache strategy** | Cache-first for all static assets (JS, CSS, fonts, SVG sprites). Network-first is not needed — the app bundle does not change between user sessions unless the user explicitly updates.                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Update flow**    | On load, the Service Worker checks for a new bundle version in the background. If found, it caches the update and prompts the user to reload — matching the pattern used by Photopea and similar tools.            |
| **File API scope** | The Service Worker does not intercept .wwz file reads or writes — those go through the File System Access API directly to the OS, bypassing the Service Worker entirely. The cache is for application assets only. |
| **Installability** | A Web App Manifest enables 'Add to Home Screen' / 'Install App' on supporting platforms, giving WeftWarp a native-app launch experience from the browser without Tauri.                                            |

→ *The Service Worker and PWA manifest are infrastructure concerns, not product features. They ship as part of the static bundle and require no user configuration.*

| **SQLite runtime**     | wa-sqlite or sql.js — SQLite compiled to WASM, runs in-browser with no native dependency  |
| ---------------------- | ----------------------------------------------------------------------------------------- |
| **Archive I/O**        | JSZip — reads and writes .wwz (zip) archives in JS                                        |
| **File system access** | File System Access API — native OS open/save dialogs with write-back, drag-to-open        |
| **Hashing**            | Web Crypto API (SubtleCrypto.digest) — SHA-256 for embed deduplication, no library needed |

→ *This is the same approach Obsidian uses internally. It is production-proven for local-first desktop-like workspaces.*

## **7.4  The storage.js Interface**

All file lifecycle operations live in storage.js. No other module imports JSZip or touches the File System Access API directly. The interface is async throughout.

| **storage.open()**                           | OS file picker or drag-drop handler → reads .wwz, inflates to in-memory SQLite, returns workspace                                                                                                                                  |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **storage.save()**                           | Flushes in-memory SQLite → re-zips to .wwz → writes back to original file handle                                                                                                                                                   |
| **storage.saveAs()**                         | Same as save() but triggers a new OS save dialog first                                                                                                                                                                             |
| **storage.autosave()**                       | Debounced (30s) call to save(). Drives the save indicator in the menubar.                                                                                                                                                          |
| **storage.exportPage(pageId, format, opts)** | Renders page to an offscreen canvas element in clean mode (chrome suppressed, marks hidden) → exports as PNG or PDF blob → triggers OS save dialog. opts: { clean: true } is always forced regardless of current app toggle state. |
| **storage.ingestEmbed(file)**                | SHA-256 hashes file → checks embeds table → writes to embeds/ if new → returns hash                                                                                                                                                |
| **storage.getEmbed(hash)**                   | Returns embed file as Blob/URL for renderer use                                                                                                                                                                                    |

## **7.5  The db.js Interface**

All SQLite queries live in db.js. No other module constructs SQL. The interface mirrors the query patterns the UI actually needs, derived from the UX flows in Section 2.

### **Queries derived from Workspace View**

| **db.getWorkspace()** | Returns workspace meta              |
| --------------------- | ----------------------------------- |
| **db.getBinders()**   | Returns all binders with page count |

### **Queries derived from Binder Gallery View**

| **db.getPages(binderId)**            | Returns all pages in a binder, ordered                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| **db.createPage(binderId, config)**  | Creates new page entity + layer entities + membership components |
| **db.deletePage(pageId)**            | Cascades: removes page, layers, all placements on those layers   |
| **db.reorderPage(pageId, newOrder)** | Updates membership order component                               |

### **Queries derived from Page Edit View**

| **db.getLayer(layerId)**                  | Returns all placement + markup entities on a layer, with content entities resolved |
| ----------------------------------------- | ---------------------------------------------------------------------------------- |
| **db.createEntity(type, class, name)**    | Inserts entity row, returns new id                                                 |
| **db.addComponent(entityId, type, data)** | Inserts component row                                                              |
| **db.updateComponent(componentId, data)** | Updates component data JSON                                                        |
| **db.deleteEntity(entityId)**             | Cascades per rules below                                                           |

### **Queries derived from Cross-Reference UX**

| **db.search(q)**                      | Full-text search across all text content                                                                                                                 |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **db.getEntity(id)**                  | Single entity with all components                                                                                                                        |
| **db.getBacklinks(entityId)**         | All entity_refs rows targeting this entity                                                                                                               |
| **db.getPlacements(contentEntityId)** | All placement entities referencing this content entity                                                                                                   |
| **db.rebuildRefs(componentId, body)** | Parses [[e:uuid]] tokens, diffs against existing entity_refs, inserts/deletes to maintain consistency. Called on every text or document component write. |

### **Queries derived from Entities Tab (grouping and z-order)**

| **db.getLayerEntities(layerId)**                      | Returns all placement, markup, and group entities on a layer, with group membership resolved into a tree. Used to populate the Entities tab list. |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **db.createGroup(layerId, name, memberIds)**          | Creates a group entity on the layer, sets membership components for all members with parent_id = group id. Returns new group id.                  |
| **db.ungroup(groupId)**                               | Re-parents all children to the layer directly (updates membership parent_id). Deletes the group entity.                                           |
| **db.reorderEntity(entityId, newOrder, newParentId)** | Updates the membership component order (and optionally parent_id if moving into or out of a group). Called on Entities tab drag-reorder.          |
| **db.renameGroup(groupId, name)**                     | Updates the group entity's name field.                                                                                                            |

## **7.6  Delete Cascade Rules**

| **Delete placement entity** | Removes placement and its components. Referenced content entity is unaffected.                                                                                                                                           |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Delete content entity**   | Removes content entity, its components, its entity_refs entries, and all placement entities whose content_ref.entity_id matches.                                                                                         |
| **Delete markup entity**    | Removes entity and its components.                                                                                                                                                                                       |
| **Delete group entity**     | By default: removes group entity and re-parents children to the layer (equivalent to ungroup then delete). Optionally 'delete with contents': removes group and all child entities with their own cascade rules applied. |

## **7.7  File Format**

workspace.wwz

  ├── project.db       SQLite — all structure, content, and references

  └── embeds/

        ├── a3f9c2...  Hash-named binary files

        └── 8b21d7...

A .wwz file is a complete, self-contained workspace. No external dependencies. Drag it onto the window to open it. The storage layer inflates it to an in-memory SQLite instance on open and re-zips on save.

## **7.8  Tauri Native Build**

Tauri is not a different application — it is the same zero-network-activity, static-asset application running inside a native WebView shell instead of a browser tab. The product model is identical. The only things that change are the storage primitives, because Tauri provides native file I/O that is more reliable than the File System Access API on some platforms, and because shipping as a native binary removes the browser sandbox entirely.

| **Browser build**      | **Tauri native build**                                |
| ---------------------- | ----------------------------------------------------- |
| Browser build          | Tauri native build                                    |
| wa-sqlite WASM         | rusqlite via Tauri invoke                             |
| JSZip                  | Rust zip crate via Tauri invoke                       |
| File System Access API | Tauri fs + dialog plugins                             |
| Web Crypto SHA-256     | Rust sha2 crate via Tauri invoke (or keep Web Crypto) |
| PWA install            | Native .app / .exe / .deb installer                   |
| Service Worker cache   | Not needed — binary ships with all assets             |

All ECS logic, rendering, and UI remain unchanged. The swap is entirely within storage.js and db.js. The Rust surface area is intentionally minimal — it is a thin file I/O shell, not a business logic layer.

→ *The browser build is not a fallback or a prototype. It is a fully viable distribution target. Tauri is an optimization for users who prefer a native installer, not a requirement for the application to function.*

 

# **8. Content Type Registry**

## **8.1  Adding a New Content Type**

The schema is type-agnostic. Adding a new content type requires:

- A component schema entry (type string + data shape) in this registry

- A renderer custom element: js/renderers/ww-render-{type}.js — receives entity and components via properties, renders into shadow root. Pure display, zero UX code.

- Register the element: customElements.define('ww-render-{type}', WWRender{Type})

- A case in ww-placement.js's renderContent() that instantiates the correct renderer element

- An entry in ENTITY_DEFAULTS and a <ww-es-btn> in ww-entity-strip.js

All canvas UX (drag, resize, rotate, select, style, tags, z-order) is handled universally by <ww-placement>. The renderer element never sees or touches it. No schema migrations.

## **8.2  Content Entity Types**

| **Type**     | **Component** | **Display**                  | **Notes**                             |
| ------------ | ------------- | ---------------------------- | ------------------------------------- |
| Text         | text          | Inline rich text card        | BBML, supports [[e:uuid]]             |
| Jot          | jot           | Compact single-line note     | Quick capture, no formatting          |
| Checklist    | checklist     | Checkable item list          | Nested indent, [[e:uuid]] in items    |
| Image        | image_ref     | Embedded image               | Hash-referenced, non-destructive crop |
| Video        | video_ref     | Embedded video player        | Hash-referenced                       |
| Audio        | audio_ref     | Compact audio player         | Hash-referenced                       |
| SVG          | svg           | User-drawn vector            | Raw SVG stored in component           |
| Link         | link_ref      | Reference to another entity  | Renders as clickable entity name      |
| Document     | document      | Icon + preview → full editor | Long-form BBML, popup editor          |
| File         | file_ref      | Icon + filename + download   | Any non-media file type               |
| Color Swatch | color_swatch  | Color preview chip           | Hex, name, format display             |

## **8.3  Markup Entity Types**

| **Type** | **Behavior**                                               |
| -------- | ---------------------------------------------------------- |
| Arrow    | Two-point directed connector, configurable head/tail style |
| Line     | Two-point undirected connector, dashed or solid            |
| Rect     | Filled/stroked rectangle for grouping, configurable radius |
| Label    | Floating text annotation, no content-system integration    |
| Comment  | Sticky note annotation, color-coded                        |

Markup entities are excluded from @ search, the reference index, and backlinks. Stored and exported identically to content entities.

 

# **9. Architectural Considerations**

This section documents known design challenges that require deliberate resolution before or during implementation. These are not deferred problems — they are decisions that, if left implicit, will produce either a brittle architecture or a significant rewrite. Each is identified early so the chosen resolution can be treated as a first-class constraint.

## **9.1  State Management**

### **Navigation History**

The navigation history stack is app-level state owned exclusively by <ww-app>. No other component reads or writes it directly — they dispatch navigation intent events and receive the derived trail as a property.

| **State shape**      | { stack: NavigationEntry[], index: number }. index points to the current position; entries after index are the forward history.                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Push**             | On any non-back/forward navigation: truncate stack to index+1, push new entry, increment index. <ww-app> recomputes the trail and pushes it to <ww-bottombar>.                                                              |
| **Back**             | <ww-app> handles ww:nav-back by decrementing index and navigating to stack[index]. Does not push a new entry. Recomputes trail.                                                                                             |
| **Forward**          | <ww-app> handles ww:nav-forward by incrementing index and navigating to stack[index]. Does not push a new entry. Recomputes trail.                                                                                          |
| **Trail derivation** | Scan backward from current index until a via: 'sidebar' or via: 'gallery' entry. The trail is the slice from that entry to the current index. Performed in <ww-app> on every stack mutation before updating <ww-bottombar>. |
| **Guard**            | Before any navigation push, check if the new entry matches the current entry (same binderId + pageId). If it matches, do not push — this prevents duplicate entries from double-clicks or redundant ref clicks.             |

→ *Navigation history does not interact with the undo/redo inverse operation log. Navigating to a different page does not constitute an undoable edit. The two stacks are fully independent.*

### **High-Frequency Events and the Global Bus**

Cross-tree communication routes through a global event bus on <ww-app>. This is appropriate for low-frequency state transitions — navigation, selection, tab changes, save state. It is not appropriate for high-frequency canvas interactions.

During drag, resize, and pan/zoom operations, state updates fire on every animation frame. If <ww-placement> dispatches a move event to the global bus on every pointermove, the bus becomes a throughput bottleneck and unrelated listeners execute on every frame. The resolution is strict phase separation:

| **During interaction (pointermove / rAF)**   | State updates are local to the component only. No global events dispatched. The placement moves on screen by updating its own transform directly.                              |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **On interaction end (pointerup / dragend)** | A single global event is dispatched with the final state. Persistence (db.updateComponent) happens here. The Entities tab and Properties panel update here.                    |
| **Pan and zoom**                             | Owned entirely by <ww-canvas> internally. No global events during gesture. <ww-app> is notified of final viewport state on gesture end for persistence in workspace.meta only. |

→ *This is the standard pattern for performant drag implementations in the browser. Treating pointerup as the commit boundary also maps naturally onto a future undo/redo system — each committed interaction is one undoable operation.*

### **Bidirectional Selection Sync**

The Entities tab and the canvas maintain synchronized selection state — clicking a card selects its row in the panel, and clicking a row selects its card on the canvas. Without a strict protocol, this creates an event loop: canvas dispatches select, panel updates and dispatches row-select, canvas reacts and dispatches select again.

The resolution is a single source of truth with identity-checked dispatch:

| **Selection state owner** | <ww-app> owns the canonical selection set (a Set of entity ids). No component owns selection independently.                                                                                                                           |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Dispatch guard**        | Before dispatching a ww:select event, a component checks whether its proposed selection matches the current selection in <ww-app>. If it matches, no event is dispatched. This breaks the loop at the source.                         |
| **Update direction**      | Components receive selection state as a property pushed down from <ww-app>. They do not pull it. When <ww-app> updates selection, it pushes to both <ww-canvas> and <ww-entities-panel> simultaneously — neither reacts to the other. |

### **Multi-Placement Content Reactivity**

The ECS model allows the same content entity to appear on multiple placements simultaneously — including multiple placements on the same page. If a user edits a text entity that appears twice on the same canvas, both placement cards must update instantly without a full page re-render.

Because storage operations bypass the component hierarchy and go directly to db.js, the standard parent-to-child property flow does not cover this case. A purpose-built pub/sub channel for database mutations is required:

| **db.on(entityId, callback)**     | Any rendered component can subscribe to mutations on a specific entity id. <ww-render-text> subscribes to its content entity's id when it connects.                                                |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **db.emit(entityId, components)** | Called internally by db.js after any component write. Notifies all active subscribers for that entity id with the updated component data.                                                          |
| **Lifecycle**                     | Subscription is registered in connectedCallback, removed in disconnectedCallback. When a placement is removed from the DOM, its renderer unsubscribes automatically — no stale listeners.          |
| **Scope**                         | This channel is for content entity mutations only. Structural changes (placement position, layer order) are not broadcast through it — those are owned by placement and layer components directly. |

→ *This pa**ttern is essentially a minimal reactive store scoped to entity ids. It is intentionally narrow — it does not become a general application state manager. Everything outside of content entity reactivity continues to use the event bus and property-down model.*

### **Undo / Redo**

Undo/redo is explicitly parked for post-MVP, but the implementation approach needs to be decided now because it constrains how db.js writes are structured. Two viable strategies:

| **Inverse Operation Log**                                                                                                                 | **SQLite Snapshot**                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Inverse operation log                                                                                                                     | SQLite snapshot                                                                                                                  |
| Each db write logs its inverse operation (e.g. UPDATE components SET data=old WHERE id=x). Undo replays the inverse log.                  | On each committed interaction, serialize the full SQLite db to a binary snapshot in memory. Undo restores the previous snapshot. |
| Granular, low memory, composable into named operations.                                                                                   | Simple to implement, zero diffing logic required.                                                                                |
| Requires every db.js write to produce a well-formed inverse. Complex for cascade deletes and multi-step operations.                       | Memory-intensive for large workspaces. Snapshot size grows with embed count even if no embeds changed.                           |
| Recommended: implement this approach. The pointerup commit boundary (§9.1) maps each user gesture to a single inverse-loggable operation. | Viable for MVP-era workspaces where embed count is low. Revisit if workspace sizes grow significantly.                           |

The recommended approach is the inverse operation log, because it aligns with the commit-boundary pattern already established for high-frequency events, and because it produces named, human-readable undo history as a natural byproduct.

## **9.2  Mobile and Touch Considerations**

The browser-first distribution model means mobile browsers are a valid target. WeftWarp is designed as a professional desktop instrument, and a full mobile port is post-v1 — but several decisions made now will either ease or foreclose that path.

### **Hover State Dependency**

The spec relies on hover states for entity reference tooltips, page thumbnail previews, and binder description previews. Hover does not exist on touch interfaces.

| **Inline references (<ww-entity-ref>)** | Map to long-press (300ms touch timer, or contextmenu event on supporting browsers). The tooltip appears on press-hold and dismisses on release or tap-elsewhere.         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Toolbar and strip tooltips**          | These are orientation aids, not primary interaction. On touch, suppress tooltips entirely — the label text on es-btn buttons is sufficient.                              |
| **Hover previews in general**           | Implement as a tap-to-reveal pattern where hover previews are triggered by a dedicated info affordance rather than proximity. Avoids needing to invent a hover metaphor. |

### **Chrome vs. Canvas Real Estate**

The desktop layout commits 196px (sidebar) + 52px (entity strip) + 200px (right panel) = 448px of horizontal chrome. On a 390px wide mobile viewport, this leaves no room for the canvas.

| **Sidebar**      | Move behind a swipeable drawer (swipe-right from left edge to open). A collapsed state shows only binder icons in a 40px rail.                |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Entity Strip** | Rotate to a horizontal scrolling bottom bar above the system navigation area. Entity type icons become horizontally scrollable.               |
| **Right Panel**  | Move behind a bottom sheet (swipe up from bottom edge) or a secondary drawer from the right. Activated by a panel icon in the toolbar.        |
| **Toolbar**      | Consolidate to essential actions only on mobile. Dynamic Edit Bar collapses to a context-sensitive floating action button above the keyboard. |

→ *These are post-v1 concerns. The desktop layout should not be d**esigned around mobile constraints. However, using CSS custom properties for all dimensional tokens (panel widths, strip width, toolbar height) from the start means the mobile layout becomes a different set of token values rather than a parallel stylesheet.*

### **Touch Gesture Conflicts**

The spec defines panning via middle-click drag or Space+drag, and single-finger interactions include: tap to select, drag to move a placement, and drag on empty canvas for marquee select. On a touch device, single-finger drag is overloaded with conflicting intents.

The resolution is a strict pointer intent heuristic evaluated at the start of each gesture, using the Pointer Events API throughout (never raw Touch Events):

| **Two-finger gesture**            | Always pan/zoom. No ambiguity. Pointer Events captures both pointers; if pointercount > 1 on canvas, enter pan/zoom mode immediately.             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Single-finger on a placement**  | Short movement threshold (< 4px in 150ms) → tap/select. Exceeds threshold → drag/move. The threshold prevents accidental moves on tap.            |
| **Single-finger on empty canvas** | Immediately begins marquee select. No conflict with pan because pan is two-finger only on touch. On desktop, Space+drag remains the pan shortcut. |
| **Long-press on empty canvas**    | Context menu (add entity at position). Consistent with the long-press pattern used for hover previews.                                            |

→ *Pointer Events API (pointerdown, pointermove, pointerup, pointercance**l) should be used exclusively throughout <ww-canvas> and <ww-placement> — never addEventListener('touchstart'). Pointer Events unify mouse, touch, and stylus into one model and provide pointerId for multi-touch tracking without separate Touch Events logic.*

 

# **10. MVP Scope**

The MVP supports the core loop: create a Binder, navigate its page gallery, add Pages, place content and markup on a bounded canvas, save and reopen, reference content entities across Pages.

## **10.1  MVP Includes**

- Workspace sidebar with Binder list

- Binder Gallery view — page thumbnail grid, reorder, add, delete, rename

- Page Edit view — bounded 8.5×11 canvas, pan, zoom

- Hard clip at page boundary, bleed zone visible during editing

- Three default layers per page: Markup, Content, Background

- Content entities: Text (WWML + @ reference input), Jot, Checklist, Image

- Markup entities: Rect, Arrow, Label

- Placement: drag from entity strip, resize, z-ordering within layer

- Embed ingestion: drag-and-drop image onto canvas

- [[e:uuid]] rendering and hover previews (entity card tooltips)

- Backlinks panel (flat list)

- Save/load .wwz, drag-to-open

- Export page to PNG (crops to page boundary)

- Autosave (30s debounced)

- Static deployable bundle — no server required, self-hostable

## **10.2  Explicitly Parked**

- Video, audio, document, file, SVG, color swatch content entities

- Markup: Line, Comment

- Additional user-defined content layers

- Layer visibility and lock toggles

- Binder gallery markup (annotating the gallery view itself)

- Board thumbnail previews on hover references

- Tagging and full-text search

- Export to PDF

- Undo/redo history

- Keyboard shortcuts and theming

- Sync (WebDAV/Nextcloud)

- Service Worker / PWA offline caching and installability

- Tauri native desktop build

## **10.3  Future Directions**

| **PWA / offline**        | Service Worker caches all assets on first load. Web App Manifest enables native-style install. No change to application code required — purely infrastructure.                          |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tauri native build**   | Same application, native WebView shell. Swap storage.js and db.js primitives only. Ships as .app / .exe / .deb, etc. installer.                                                         |
| **Sync**                 | WebDAV or Nextcloud target. The .wwz file is the sync unit — no proprietary cloud format. The browser build gets sync for free via Dropbox/Syncthing/any file sync tool in the interim. |
| **Evermore integration** | Replace plain-text body with token-ID sequences referencing a global dictionary. Compression falls out structurally. Semantic analysis becomes a query.                                 |
| **Semantic boards**      | Auto-generated pages assembled by tag, reference cluster, or date range. Same entities, different placement — no duplication.                                                           |
| **Platform ports**       | Android and iOS use the same .wwz format. The PWA install path may satisfy mobile without a native build.                                                                               |
| **Analysis view**        | Visualize the entity_refs knowledge graph — most referenced, orphaned, natural clusters.                                                                                                |
