# Project Knowledge: Logseq

## 1. Project Overview
**Logseq** is a privacy-first, open-source platform for knowledge management and collaboration. It is a local-only, offline-first application built on a graph database foundation.

*   **Repository Type**: Multi-part Monorepo
*   **Core Logic**: `src/main/frontend` (Shared ClojureScript Core)
*   **Desktop**: Electron (`src/electron`)
*   **Mobile**: Capacitor (`src/main/mobile`)

## 2. Technology Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Language** | ClojureScript | Primary language for UI and logic. |
| **Frontend Framework** | React (Rum) | Functional React wrapper. |
| **State Management** | DataScript | Immutable, in-memory Datalog database. |
| **Styling** | Tailwind CSS | Utility-first CSS framework. |
| **Desktop Runtime** | Electron | Cross-platform desktop wrapper. |
| **Mobile Runtime** | Capacitor | Cross-platform mobile wrapper (iOS/Android). |
| **Build Tool** | Shadow-cljs | ClojureScript build and dependency management. |
| **Parser** | mldoc | OCaml-based Markdown/Org-mode parser (WASM). |

## 3. Architecture Highlights

### Functional Reactive Programming (FRP)
The application uses a reactive loop ensuring UI consistency:
1.  **State**: Held in DataScript (graph data) and Atoms (UI state).
2.  **Events**: Dispatched via `frontend.handler.events` (see [API Contracts](./api-contracts-frontend.md)).
3.  **Render**: Rum components subscribe to state changes via mixins (`rum/reactive`) and re-render automatically.

### Multi-Platform Core
The `frontend` namespace contains the core logic shared across Web, Desktop, and Mobile. Platform-specific bridges (Electron IPC, Capacitor Plugins) inject capabilities into this core.

## 4. Documentation Artifacts

The following deep-dive details have been generated:

*   **[Data Models](./data-models-frontend.md)**: Details on Block (node) and Page entities, and DataScript schemas.
*   **[API & Events](./api-contracts-frontend.md)**: Usage of the internal event bus and handler patterns.
*   **[Component Inventory](./component-inventory-frontend.md)**: List of reusable UI components in `frontend.ui`.

## 5. Existing Documentation Index

*   [Codebase Overview](../CODEBASE_OVERVIEW.md): Detailed explanation of the tech stack by the maintainers.
*   [Contributing](../CONTRIBUTING.md): Build and setup instructions.
*   [Readme](../README.md): General project introduction.
