# Frontend Data Models (DataScript)

This document outlines the core data entities and relationships used in the Logseq Frontend application, based on `frontend.db.model` and DataScript usage.

## Core Entities

### Block (`:block/*`)
The fundamental unit of content in Logseq.

| Attribute | Type | Description |
| :--- | :--- | :--- |
| `:block/uuid` | UUID | Unique identifier for the block. |
| `:block/page` | Ref | Reference to the `Page` entity this block belongs to. |
| `:block/parent` | Ref | Reference to the parent block (if nested). |
| `:block/children` | Ref (Many) | **Derived**: List of child blocks (via `_parent`). |
| `:block/content` | String | The Markdown/Org-mode content of the block. |
| `:block/title` | String | Title (usually for pages or properties). |
| `:block/collapsed?` | Boolean | UI state: whether the block's children are hidden. |
| `:block/properties` | Map | Property map (key-value pairs stored in the block). |
| `:block/refs` | Ref (Many) | **Derived**: Pages/Blocks referenced in this block. |

### Page (`:page/*`)
Represents a document or a page. In Logseq, a Page is often a special type of Block or strongly linked to one.

| Attribute | Type | Description |
| :--- | :--- | :--- |
| `:block/name` | String | The unique name of the page (url-friendly). |
| `:block/journal-day` | Integer | (Optional) Date integer (YYYYMMDD) if this is a Journal page. |
| `:block/original-name`| String | The original case-preserved name of the page. |

## Relationships

*   **Hierarchy**: Blocks form a tree structure via `:block/parent` and `:block/children`.
*   **Ownership**: Every block belongs to a `:block/page`.
*   **References**: Blocks can reference other blocks or pages via `[[:block/refs ...]]`.

## Query Patterns (DataScript)

*   **Pull syntax** is frequently used to retrieve trees: `(pull db '[*] eid)`.
*   **Recursive Queries**: Used for fetching children: `{:block/children ...}`.

## Source Reference
*   `src/main/frontend/db/model.cljs`: Accessors and logic.
*   `logseq/db/frontend/schema`: Schema entry point (inferred).
