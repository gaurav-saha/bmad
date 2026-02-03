# Frontend Component Inventory

This document lists the reusable UI components found in `frontend.ui`, built with **Rum** (React wrapper).

## Core Components (`frontend.ui`)

### Input/Form Controls
*   `ls-textarea`: Auto-resizing textarea with selection/composition handling.
*   `select`: Styled select dropdown.
*   `checkbox`: Checkbox with support for `disabled` state (e.g., publishing mode).
*   `radio-list`: List of radio buttons.
*   `checkbox-list`: List of checkboxes.
*   `toggle`: Switch/Toggle component.

### Navigation/Overlay
*   `dropdown`: Generic dropdown with trigger and content.
*   `dropdown-with-links`: Specialized dropdown for menus.
*   `menu-link`: Standard menu item styling.
*   `menu-background-color`: Color picker specific menu item.
*   `auto-complete`: Generic autocomplete list with grouping support.

### Feedback/Status
*   `notification`: Toast notification system.
*   `notification-content`: Individual toast item.
*   `loading`: Loading spinner with text.
*   `block-error`: Styled error message for block rendering failures.
*   `admonition`: Callout blocks (Note, Tip, Warning, etc.).

### Layout/Structure
*   `foldable`: Collapsible section with header and content.
*   `foldable-title`: Header for foldable sections with rotation arrow.
*   `main-node`: App scroll container reference.

## External Wrappers
*   `TransitionGroup`, `CSSTransition`: `react-transition-group`
*   `Virtuoso`, `VirtuosoGrid`: `react-virtuoso` (Virtual scrolling)
*   `ReactTweetEmbed`: `react-tweet-embed`

## Source Reference
*   `src/main/frontend/ui.cljs`: Component definitions.
