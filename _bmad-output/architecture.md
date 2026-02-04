# Logseq Architecture Document

**Version:** 1.0  
**Date:** 2026-02-04  
**Author:** Architecture Analysis

---

## 1. Executive Summary

Logseq is a **privacy-first, local-only knowledge management platform** built on a graph database foundation. The architecture emphasizes:

- **Offline-first**: All data stored locally, optional cloud sync
- **Multi-platform**: Shared core logic across Web, Desktop (Electron), and Mobile (Capacitor)
- **Functional Reactive Programming (FRP)**: Immutable state with reactive UI updates
- **Graph-based data model**: DataScript (in-memory Datalog) for flexible querying

**Key Architectural Decisions:**
1. ClojureScript for type safety and functional programming benefits
2. DataScript for powerful graph queries and immutability
3. Shared core with platform-specific bridges for code reuse
4. Event-driven architecture for decoupled components

---

## 2. System Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    User Interface Layer                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Web App    │  │   Desktop    │  │    Mobile    │  │
│  │  (Browser)   │  │  (Electron)  │  │ (Capacitor)  │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼──────────────────┼──────────────────┼─────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
┌────────────────────────────┼─────────────────────────────┐
│              Shared Frontend Core (ClojureScript)        │
│  ┌──────────────────────────────────────────────────┐   │
│  │           React Components (Rum)                 │   │
│  └────────────────────┬─────────────────────────────┘   │
│  ┌────────────────────┴─────────────────────────────┐   │
│  │         Event Handlers & Business Logic          │   │
│  └────────────────────┬─────────────────────────────┘   │
│  ┌────────────────────┴─────────────────────────────┐   │
│  │     State Management (DataScript + Atoms)        │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────┬───────────────────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────┐
│               Platform Abstraction Layer                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Browser API │  │ Electron IPC │  │  Capacitor   │  │
│  │              │  │              │  │   Plugins    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└───────────────────────────────────────────────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────┐
│                  Storage Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Local Files  │  │   IndexedDB  │  │ Cloud Sync   │  │
│  │ (Markdown)   │  │              │  │  (Optional)  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└───────────────────────────────────────────────────────────┘
```

### 2.2 Repository Structure

**Multi-part Monorepo:**

- **`src/main/frontend/`** - Shared core logic (ClojureScript)
- **`src/electron/`** - Desktop-specific code (Electron main process)
- **`src/main/mobile/`** - Mobile-specific code (Capacitor)
- **`deps/graph-parser/`** - Internal library for parsing graphs
- **`packages/ui/`** - Component system (shadcn-based)
- **`android/`, `ios/`** - Native mobile app shells

---

## 3. Core Architectural Patterns

### 3.1 Functional Reactive Programming (FRP)

**Pattern:** Unidirectional data flow with reactive components

```
User Action → Event Handler → State Update → UI Re-render
                                    ↓
                              DataScript Transaction
```

**Implementation:**
- **State**: Immutable data in DataScript (graph) + Clojure Atoms (UI state)
- **Events**: Dispatched via `frontend.handler.events/handle` multimethod
- **Reactivity**: Rum components subscribe to state via `rum/reactive` mixin

**Benefits:**
- Predictable state changes
- Time-travel debugging (via DataScript history)
- Automatic UI consistency

### 3.2 Event-Driven Architecture

**Core Event Bus:** `frontend.handler.events`

**Key Event Categories:**
- **Graph Operations**: `:graph/switch`, `:graph/ready`, `:graph/save-db-to-disk`
- **Page Operations**: `:page/create`, `:page/deleted`, `:page/renamed`
- **Editor Operations**: `:editor/save-current-block`, `:editor/quick-capture`
- **RTC (Real-Time Collaboration)**: `:rtc/sync-state`, `:rtc/download-remote-graph`

**Pattern:**
```clojure
(defmulti handle first)  ; Dispatch on event type

(defmethod handle :graph/switch [event]
  ;; Handle graph switching logic
  )
```

**Benefits:**
- Decoupled components
- Extensible via plugins
- Clear audit trail of system actions

### 3.3 Multi-Platform Core with Platform Bridges

**Shared Core:** `src/main/frontend/` (95%+ of logic)

**Platform-Specific Bridges:**
- **Electron**: IPC for file system, native menus, auto-updates
- **Capacitor**: Plugins for camera, filesystem, status bar
- **Web**: Browser APIs (IndexedDB, Service Workers)

**Abstraction Layer:** `frontend.fs` namespace provides unified file system API

**Benefits:**
- Code reuse across platforms
- Single source of truth for business logic
- Platform-specific optimizations where needed

---

## 4. Component Architecture

### 4.1 Frontend Component Hierarchy

```
frontend.components/
├── root.cljs                 # App root
├── page.cljs                 # Page container
├── block.cljs                # Block (core editing unit)
├── editor.cljs               # Rich text editor
├── sidebar.cljs              # Navigation sidebar
├── search.cljs               # Search interface
└── settings.cljs             # Settings panel
```

**Component Pattern:** Rum functional components with mixins

```clojure
(rum/defcs component-name < 
  rum/reactive              ; Subscribe to state changes
  db-mixins/query           ; DataScript query mixin
  [state props]
  [:div ...])
```

### 4.2 Handler Architecture

```
frontend.handler/
├── events.cljs               # Event bus dispatcher
├── page.cljs                 # Page CRUD operations
├── block.cljs                # Block operations
├── editor.cljs               # Editor state management
├── file.cljs                 # File I/O operations
├── db.cljs                   # Database transactions
└── plugin.cljs               # Plugin lifecycle
```

**Handler Pattern:** Pure functions that return state transformations

### 4.3 Database Layer

**DataScript Schema:**
- **`:block/uuid`** - Unique block identifier
- **`:block/page`** - Reference to parent page
- **`:block/parent`** - Reference to parent block (for nesting)
- **`:block/content`** - Markdown/Org-mode content
- **`:block/properties`** - Key-value metadata
- **`:block/refs`** - Outgoing references (backlinks)

**Query Patterns:**
- **Pull API**: Retrieve entity trees
- **Datalog Queries**: Complex graph traversals
- **Reactive Queries**: Auto-update on data changes

---

## 5. Data Flow Architecture

### 5.1 Application Startup Flow

```
1. Platform Initialization (Electron/Capacitor/Browser)
   ↓
2. Load User Configuration (from local storage)
   ↓
3. Initialize DataScript Database (empty schema)
   ↓
4. Load Graph Files (Markdown/Org from filesystem)
   ↓
5. Parse Files → DataScript Transactions
   ↓
6. Build Query Cache (for performance)
   ↓
7. Render Initial UI (React components)
   ↓
8. Register Event Handlers
   ↓
9. Ready State (user can interact)
```

### 5.2 User Edit Flow

```
User Types in Block
   ↓
Editor Handler Triggered
   ↓
┌─────────────────────────────┐
│ 1. Save to Disk (debounced) │  ← Prevent data loss
│ 2. Update UI State (Atom)   │  ← Immediate feedback
│ 3. DataScript Transaction   │  ← Update graph
└─────────────────────────────┘
   ↓
Query Cache Invalidation
   ↓
Reactive Components Re-render
   ↓
UI Updated
```

### 5.3 File Synchronization Flow

```
File Change Detected (File Watcher)
   ↓
Parse Changed File
   ↓
Diff with DataScript State
   ↓
Apply Incremental Transaction
   ↓
Trigger Re-render of Affected Components
```

---

## 6. Technology Stack Decisions

### 6.1 Core Technologies

| Technology | Rationale | Trade-offs |
|:---|:---|:---|
| **ClojureScript** | Functional programming, immutability, Lisp macros, excellent tooling | Smaller talent pool, learning curve |
| **DataScript** | Powerful graph queries, immutability, time-travel debugging | In-memory only (requires persistence layer) |
| **Rum** | Lightweight React wrapper, Clojure-friendly state management | Less ecosystem than React libraries |
| **Shadow-cljs** | Best-in-class ClojureScript build tool, hot reload, REPL | Requires Node.js and npm |
| **Electron** | Cross-platform desktop, native APIs, auto-update | Large bundle size (~150MB) |
| **Capacitor** | Cross-platform mobile, web-first, plugin ecosystem | Performance vs native |

### 6.2 Build Pipeline

```
ClojureScript Source Files
   ↓
Shadow-cljs Compiler
   ↓
JavaScript Bundles (Code Splitting)
   ├── app.js (Main UI)
   ├── db-worker.js (Database operations)
   └── inference-worker.js (AI features)
   ↓
Webpack (Asset Bundling)
   ↓
Platform-Specific Packaging
   ├── Electron Builder (Desktop)
   ├── Capacitor Build (Mobile)
   └── Static Files (Web)
```

### 6.3 Parser Architecture

**mldoc (OCaml → WASM):**
- Parses Markdown and Org-mode
- Compiled to WebAssembly for performance
- Shared across all platforms

**Benefits:**
- Single parser implementation
- Near-native performance
- Consistent parsing across platforms

---

## 7. Deployment Architecture

### 7.1 Desktop (Electron)

```
┌─────────────────────────────────────┐
│         Electron Main Process       │
│  ┌───────────────────────────────┐  │
│  │   File System Access          │  │
│  │   Auto-Update                 │  │
│  │   Native Menus                │  │
│  │   IPC Bridge                  │  │
│  └───────────────────────────────┘  │
└─────────────────┬───────────────────┘
                  │ IPC
┌─────────────────┴───────────────────┐
│      Electron Renderer Process      │
│  ┌───────────────────────────────┐  │
│  │   Frontend Core (ClojureScript)│ │
│  │   React UI (Rum)              │  │
│  │   DataScript Database         │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Distribution:**
- **macOS**: DMG, auto-update via Squirrel
- **Windows**: NSIS installer, auto-update
- **Linux**: AppImage, Snap, deb/rpm

### 7.2 Mobile (Capacitor)

```
┌─────────────────────────────────────┐
│      Native App Shell (iOS/Android) │
│  ┌───────────────────────────────┐  │
│  │   WebView (Chromium-based)    │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │  Frontend Core          │  │  │
│  │  │  (Same as Desktop)      │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │   Capacitor Plugins           │  │
│  │   - Filesystem                │  │
│  │   - Camera                    │  │
│  │   - StatusBar                 │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Distribution:**
- **iOS**: App Store
- **Android**: Google Play, F-Droid

### 7.3 Web (Self-Hosted)

```
┌─────────────────────────────────────┐
│         Static File Server          │
│  ┌───────────────────────────────┐  │
│  │   index.html                  │  │
│  │   app.js (ClojureScript)      │  │
│  │   styles.css                  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────┐
│         User's Browser              │
│  ┌───────────────────────────────┐  │
│  │   Frontend Core               │  │
│  │   IndexedDB (Local Storage)   │  │
│  │   Service Worker (Offline)    │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

## 8. Security Architecture

### 8.1 Data Privacy

**Principles:**
- **Local-first**: All data stored on user's device by default
- **End-to-end encryption**: Optional cloud sync with E2E encryption
- **No telemetry**: No usage data collected without explicit consent

### 8.2 Plugin Security

**Sandbox Model:**
- Plugins run in isolated context
- API surface limited to safe operations
- No direct file system access (must go through Logseq API)

---

## 9. Performance Considerations

### 9.1 Optimization Strategies

**DataScript Query Caching:**
- Reactive queries cached until invalidated
- Incremental updates for large graphs

**Code Splitting:**
- Main app bundle
- Separate worker bundles (db-worker, inference-worker)
- Lazy-loaded features (PDF viewer, Excalidraw)

**Virtual Scrolling:**
- `react-virtuoso` for large lists
- Only render visible blocks

### 9.2 Performance Targets

- **Startup Time**: < 3 seconds (cold start)
- **Block Rendering**: < 16ms (60 FPS)
- **Search**: < 200ms for 10,000 blocks
- **File Save**: < 100ms (debounced)

---

## 10. Extension Points

### 10.1 Plugin API

**Capabilities:**
- Custom commands
- UI extensions (toolbar buttons, context menus)
- Custom renderers (block types)
- Event listeners (page created, block edited)

**API Surface:** `src/main/logseq/` namespace

### 10.2 Theme System

**CSS Variables:**
- Color palette
- Typography scale
- Spacing system

**Custom Themes:** User-provided CSS files

---

## 11. Future Architecture Considerations

### 11.1 Scalability

**Current Limitations:**
- In-memory DataScript (limited by RAM)
- Single-threaded JavaScript (UI blocking on large operations)

**Potential Solutions:**
- Persistent DataScript backend (SQLite)
- Web Workers for heavy computation
- Incremental parsing for large files

### 11.2 Real-Time Collaboration

**Challenges:**
- CRDT integration with DataScript
- Conflict resolution for concurrent edits
- Network layer for sync

**Approach:**
- Operational Transformation (OT) or CRDT
- WebSocket-based sync server
- Offline-first with eventual consistency

---

## 12. References

- [Codebase Overview](../CODEBASE_OVERVIEW.md)
- [Data Models](./data-models-frontend.md)
- [API Contracts](./api-contracts-frontend.md)
- [Component Inventory](./component-inventory-frontend.md)
- [Shadow-cljs Documentation](https://shadow-cljs.github.io/docs/UsersGuide.html)
- [DataScript Documentation](https://github.com/tonsky/datascript)
- [Rum Documentation](https://github.com/tonsky/rum)
