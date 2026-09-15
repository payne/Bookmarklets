# Bookmarklets Project - Interaction Log

## Overview

This document records the interactions and development history of the Bookmarklets project.

---

## Interaction 1: Laser Pointer Bookmarklet

**Date:** May 23, 2026
**Commit:** `f067cb0`
**Commit Message:** CLAUDE made a laser pointer bookmarklet.

### Description

Created an HTML page (`index.html`) containing a collection of useful bookmarklets. The initial bookmarklet added was a **Laser Pointer** tool.

### Bookmarklet Details

**Name:** Laser Pointer

**Functionality:**
- Adds a Google Slides-style laser pointer to any web page
- Displays a red glowing dot that follows the cursor
- Press `L` to toggle the laser pointer on/off
- Hides the normal cursor while the laser is active
- Useful for presentations or highlighting content on screen

**Technical Implementation:**
- Creates a fixed-position red dot element with CSS glow effects
- Tracks mouse movement to position the dot
- Listens for the `L` key to toggle activation
- Properly avoids triggering in input fields and text areas
- Uses maximum z-index to ensure visibility above all content

### Files Created

| File | Description |
|------|-------------|
| `index.html` | Main bookmarklets page with styled interface and instructions |

---

## Interaction 2: Documentation

**Date:** May 23, 2026

### Description

Created this markdown file (`INTERACTIONS.md`) to document all interactions and development history in the Bookmarklets project folder.

---

## Interaction 3: Confetti Bookmarklet

**Date:** May 23, 2026

### Description

Added a **Confetti** bookmarklet to `index.html` that creates a celebratory confetti effect on any web page.

### Bookmarklet Details

**Name:** Confetti

**Functionality:**
- Press `C` to trigger a colorful confetti burst
- 80 confetti pieces fall from the top of the screen
- Effect lasts approximately 1 second
- Mix of circles and rectangles in rainbow colors
- Pieces rotate and fade as they fall

**Technical Implementation:**
- Creates 80 div elements with random colors, sizes, and shapes
- Uses CSS transitions for smooth falling animation with cubic-bezier easing
- Random horizontal drift for natural confetti spread
- Auto-cleanup: each piece removes itself after animation completes
- Ignores key press in input fields and textareas
- Uses maximum z-index to appear above all content

---

## Interaction 4: Snow Bookmarklet

**Date:** May 23, 2026

### Description

Added a **Snow** bookmarklet to `index.html` that creates a continuous snowfall effect that can be toggled on and off.

### Bookmarklet Details

**Name:** Snow

**Functionality:**
- Press `S` to toggle snowfall on/off
- White snowflakes continuously fall from the top of the screen while active
- Snowflakes have varying sizes and opacity
- Subtle horizontal drift for realistic effect
- Toggling off immediately stops new flakes and clears existing ones

**Technical Implementation:**
- Uses setInterval to spawn new snowflakes every 50ms while active
- Each snowflake is a white circular div with subtle glow/shadow
- CSS transitions for smooth linear falling animation (2-5 seconds per flake)
- Tracks all active flakes in an array for cleanup on toggle off
- Auto-cleanup: each flake removes itself after animation completes
- Ignores key press in input fields and textareas
- Uses maximum z-index to appear above all content

---

## Interaction 5: Collapsible List Items

**Date:** May 23, 2026

### Description

Made each bookmarklet item in the ordered list collapsible using HTML5 `<details>` and `<summary>` elements. All items start collapsed by default.

### Changes Made

- Wrapped each list item's content in `<details>` and `<summary>` elements
- The bookmarklet link appears in the summary (always visible)
- The description is hidden until expanded
- Added CSS styling for the collapse/expand arrow indicator
- Arrow rotates 90 degrees when expanded
- Smooth transition animation on the arrow

---

## Interaction 6: Labels Bookmarklet

**Date:** May 23, 2026

### Description

Added a **Labels** bookmarklet to `index.html` that allows users to create draggable, editable labels on any web page.

### Bookmarklet Details

**Name:** Labels

**Functionality:**
- Press `L` to create a new label at the center of the screen
- Labels are draggable - click and drag to reposition
- Double-click a label to edit its text via prompt dialog
- Labels are numbered sequentially (Label 1, Label 2, etc.)
- Yellow sticky-note style appearance

**Technical Implementation:**
- Creates fixed-position div elements styled as yellow labels
- Implements drag-and-drop via mousedown/mousemove/mouseup events
- Double-click event triggers prompt() for text editing
- Tracks label count for sequential numbering
- Ignores key press in input fields and textareas
- Uses maximum z-index to appear above all content

**Note:** Uses the same hotkey (`L`) as Laser Pointer - cannot use both simultaneously on the same page.

---

## Interaction 7: Reload300 Bookmarklet

**Date:** September 15, 2026

### Description

Added a **Reload300** bookmarklet to `index.html` that automatically reloads the current page every 300 seconds, forever, with console logging of its status.

### Bookmarklet Details

**Name:** Reload300

**Functionality:**
- Reloads the page every 300 seconds (5 minutes)
- Logs to the console when it reloads
- Logs the time remaining until the next reload every 30 seconds
- Keeps looping indefinitely - no need to re-click after each reload
- Click the bookmarklet again to stop it

**Technical Implementation:**
- A real `location.reload()` destroys the entire JS context, so nothing can survive it to automatically resume the countdown - this is a fundamental browser limitation, not something code can work around
- To satisfy "keeps working after the reload," it instead does a "soft reload": `fetch()`s the current URL fresh (bypassing cache) and swaps the content in via `document.open()`/`document.write()`/`document.close()`, which replaces the document but leaves the `window` object (and therefore the bookmarklet's timers) intact
- Uses one `setInterval` (30s) for the countdown log and one `setTimeout` (300s) for the reload itself, re-arming both after every successful cycle
- Falls back to a real one-time `location.reload()` if the fetch fails (e.g. CORS/CSP blocking the refetch), though the timer won't survive that fallback
- Created `docs/reload300.md` documenting the approach and its trade-offs versus a true browser reload
