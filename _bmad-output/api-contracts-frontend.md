# Frontend API & Event Contracts

Logseq uses an internal event bus system (`state/pub-event!`) to handle application logic, decoupling UI from state updates.

## Event Bus System

*   **Dispatcher**: `frontend.handler.events/run!` listens on a core.async channel.
*   **Publisher**: `frontend.state/pub-event!` puts events onto the channel.
*   **Handler**: `defmethod handle :event/name` implementation.

## Key Event Contracts

### Graph Operations
| Event | Payload | Description |
| :--- | :--- | :--- |
| `:graph/switch` | `[graph-repo opts]` | Switches the active graph/repository. Triggers persistence cleanup. |
| `:graph/ready` | `[repo]` | Fired when graph is fully loaded and ready for user interaction. |
| `:graph/save-db-to-disk`| `opts` | Forces a flush of the in-memory DataScript DB to disk (JSON/Transit). |

### Page Operations
| Event | Payload | Description |
| :--- | :--- | :--- |
| `:page/create` | `[page-name opts]` | Creates a new page. Handles "Today" journal creation logic. |
| `:page/deleted` | `[page-name tx-meta]`| Handles cleanup after a page deletion. |
| `:page/renamed` | `[repo data]` | Handles references update after page rename. |

### Editor Operations
| Event | Payload | Description |
| :--- | :--- | :--- |
| `:editor/save-current-block`| `_` | Triggers save of the currently edited block. |
| `:editor/toggle-own-number-list`| `[blocks]` | Toggles ordered list formatting. |
| `:editor/quick-capture` | `args` | Handles quick capture input. |

### RTC (Real Time Collaboration)
| Event | Payload | Description |
| :--- | :--- | :--- |
| `:rtc/sync-state` | `state-map` | Updates the local RTC sync state. |
| `:rtc/download-remote-graph` | `[name uuid version]` | Initiates download of a remote graph for sync. |

## Plugin System
*   **Event**: `:exec-plugin-cmd`
*   **Payload**: `{:keys [pid cmd action]}`
*   **Description**: Bridges internal events to the plugin API.

## Source Reference
*   `src/main/frontend/handler/events.cljs`: Main event registry.
*   `src/main/frontend/state.cljs`: Event bus implementation.
