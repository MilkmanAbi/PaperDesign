# PaperPatterns
### How PaperDesign and PaperImplementation apply to real interaction problems.

Version 0.1 — companion to `PaperDesign.md` and `PaperImplementation.md`

---

## 0. What this document is

This is not a component library. There are no mandatory templates here, no exact pixel specs that override the grid, no code to copy. Each pattern is a worked example — the same principles from `PaperDesign.md` and `PaperImplementation.md`, applied to a real interaction problem that comes up often enough to be worth thinking through once, carefully.

Two apps that both follow this document should not look identical. They should feel like they solved the same problem the same *way* — same density philosophy, same update locality, same honesty about state — while looking like themselves. Each app is a sovereign window (`PaperDesign.md` §0): its own accent, its own typeface, its own personality. These patterns are how the shared *behavior* shows up in practice, not a visual template to be copied across products.

Where this document references a section in the base spec (e.g. `PD §5.1`, `PI §4.3`), those are load-bearing references, not decorative ones. If a decision here doesn't make sense without its rationale, the rationale is in that section.

Patterns are ordered by structural complexity, not importance.

---

## 1. Search Bars & Filter Fields

The most common bar in any tool, and the fastest way to test whether the PD shape language is being applied consistently.

### The core decisions

A search bar is a bar, not a container — it takes `radius-pill` per `PD §5.2`, not `radius-sm`. Its height is derived from the text inside it (the bar-height formula in `PD §5.2`), not chosen to match a neighboring button. The default search bar at Compact density should land around 32–36px tall; anything above that is inflation, not comfort.

A filter field that produces tags/chips from its input is still a bar while typing, but the chips it emits are also pill-radius per `PD §5.1` — because a chip is shaped like a bar, not a box. Chips use `radius-pill` and sit inside or below the search bar depending on layout, not in a modal or a secondary panel.

The icon inside the bar (magnifying glass at rest, X to clear when text is present) sits at `space-2` from the edge, 16px, `ink-secondary` — never 20–24px, which reads as an oversized decoration rather than a tool.

**The state sequence is the most important thing to get right here.** For local filtering (filtering a list already in memory), results update on each keystroke, immediately — this is `PI §1.1` in its most literal form. The user typed a character; the filter result is already known; show it on that frame. No debounce spinner, no "press Enter to search." If the input genuinely requires a network call to resolve, that's a different pattern — the bar is still the same shape, but the response follows `PI §4.1–4.2`: results load with an honest indeterminate state, not a fake instant that later corrects itself.

### Anti-patterns

- ❌ Using `radius-sm` on a search bar — a search bar is a bar, not a box, and the shape difference communicates that.
- ❌ Debouncing local (already in-memory) filtering — debounce is for network calls, not string matching.
- ❌ Showing a spinner on a local filter — if the data is already there, there's nothing to load.
- ❌ A filter that opens a modal or overlay to configure — filters are inline; advanced filter fields are a docked panel (PI §4 / PD §4).
- ❌ A clear button that's always visible — the X should appear only when there's text to clear. A permanently visible clear button in an empty field is noise.

---

## 2. Command Palette

The command palette is PaperDesign's most transparent component — it has to prove it works without any surrounding context, so every principle shows up in concentrated form.

### The core decisions

A command palette is a **true overlay** per `PD §4` — transient, dismissed by Esc or by completing the action, with `elevation-2` and a scrim behind it. It is not a panel, not a widget, and not a sidebar mode. It exists precisely as long as you're using it, and then it's gone.

The overlay itself is narrow and centered: wide enough to read a full command label and its shortcut, no wider. It should not span the full viewport — a command palette that's 800px wide on a 1440px display is padding its way to looking substantial. Aim for a fixed width in the 480–560px range (six to seven columns of the 12-column grid).

The search field at the top is pill-radius, consistent with every other search/filter bar in the system. Results appear in a list below, compact rows, `32px` height in Compact density. The active/focused result uses `accent-primary-muted` background, the same active-state treatment as sidebar nav items (`PD §11.4`) — this makes the selection language consistent whether you're navigating a list or a command result.

Results update on every keystroke, immediately (`PI §1.1`). The list has no loading state for local commands. For a palette that also searches remote content (files, contacts, cloud items), the local results appear first and immediately; remote results append below with an honest indeterminate indicator only on the remote section, not the whole list.

Keyboard shortcut hints (⌘K, ⌘P, etc.) live in the result row at the far right, in monospace, `type-caption`, `ink-tertiary` — present but quiet. They don't need to be visible when a result isn't focused; they appear in the focused row so the user connects shortcut to action.

### Anti-patterns

- ❌ A command palette that opens as a panel or sidebar — it's transient by definition; forcing it into a persistent position defeats its purpose.
- ❌ Debouncing local command filtering (`PI §1.1`).
- ❌ A spinner or loading state for commands that are already in memory.
- ❌ Showing shortcut hints on every row simultaneously — that's a shortcut reference, not a command palette.
- ❌ A command palette wider than ~560px — it's a focused search tool, not a full-screen overlay.
- ❌ Dismissing on anything other than Esc, Enter, or clicking outside the overlay — don't intercept navigation keys that the user expects to move within the list.

---

## 3. Tree Views

Tree views are the most information-dense pattern in this catalog, and density is exactly where they tend to go wrong — either too compact to read under a real data load, or padded to the point of hiding structure.

### The core decisions

A tree view is a **list of rows with indentation hierarchy**, not a set of nested containers. Each row is a flat, grid-height element — `32px` at Compact density — with a disclosure triangle and an icon or glyph, then a label. The indentation per level is one consistent step (`space-5`, 32px, or `space-4`, 24px if the tree is very deep) — chosen once, held throughout. No level of the tree gets more padding than another just because it "feels like a header."

The **disclosure triangle** (or chevron) is a 16px icon, `ink-secondary`, sitting at `space-2` from its indentation level's left edge. It rotates 90° on open, in the Micro duration (100ms) from `PD §12.1` — this is motion acknowledging a real spatial change (children are about to occupy space below), not decoration. A tree node with no children has no disclosure triangle; the space is just empty so labels align.

**Selection** uses `accent-primary-muted` background on the full row width, not just the label. This matches the nav-list active state in `PD §11.4` for consistency — your tree view and your sidebar nav should use visually identical selection language.

**Expand/collapse is immediate** (`PI §1.1`): click the triangle, children appear or disappear on that frame. No animation on the children themselves — they exist or they don't. The only animation is the triangle's rotation and the row height change as the list reflows. If a tree node needs to load its children on demand (remote filesystem, lazy-loaded API), that node shows a spinner only in the row being expanded, never in the surrounding tree (`PI §2.3`), and the expansion honestly represents the loading state (`PI §4.2`).

**Cut/copy/paste, drag-and-drop reordering, and rename** are the behaviors most likely to introduce update-locality violations: a rename should update only the renamed row; a move should animate only the moved subtree collapsing out of its origin and appearing at its target. Nothing else in the tree should visibly shift during these operations.

### Anti-patterns

- ❌ Nested containers (a `surface-raised` card for each folder) instead of flat rows with indentation — it looks organized but destroys density.
- ❌ Inconsistent indentation between levels — if a level-3 child is indented the same as a level-2 child, the hierarchy is invisible.
- ❌ Animating children in/out on expand/collapse instead of showing them immediately.
- ❌ Expanding a node and showing a spinner that blocks the whole tree (`PI §2.3`, `PI §4.4`).
- ❌ Icons that change size between levels — a folder icon at level 1 and a file icon at level 4 should be the same 16px.
- ❌ Hover state that doesn't cover the full row width — a partial-width hover on a deeply-indented row looks like a different component from the same hover on a top-level row.

---

## 4. Tables

Tables are the most "data" component in this catalog — dense by default, and the place where PaperDesign's compact-not-cramped principle gets its hardest test, because real tables contain real heterogeneous content that was not designed to sit neatly in a grid.

### The core decisions

Table rows are **32px at Compact density**, the same as every other bar-height element in the system. This is not a coincidence — a table row is a kind of bar, just a very wide one. The row height rule isn't "whatever the content needs"; the content needs to fit the row, and if it doesn't, that's a signal that this content should be in a detail panel, not a table cell.

**Column headers** are one step up in visual weight from row text — `type-body-sm` at 600 weight, `ink-secondary` color, left-aligned by default, right-aligned for numeric columns. Header cells are not buttons; they look like headers. Clicking a header to sort is an interaction on a header, not a button inside it — the sort indicator (a 12px chevron, `ink-tertiary`) appears inline after the label.

**Sorting is immediate** for data already in memory (`PI §1.1`). Click a header → the rows reorder on that frame. For server-sorted tables, the sort request follows `PI §4.1–4.2`: the header shows a loading state, the rows that were visible remain visible (they're stale, not gone), and new rows replace them when the response arrives.

**Selection** (row-level) uses `accent-primary-muted` background, consistent with tree views and nav lists. A checkbox in the first column is the affordance for multi-select; it appears on hover of any row in a selectable table (not permanently visible on every row, which creates visual noise). The "select all" checkbox in the header column follows the three-state convention (unchecked / partial / checked) honestly — if any rows are hidden by a filter, "select all" selects only visible rows, and says so.

**Resizable columns** snap to the 8px grid when released — drag freely, snap on drop. A column resize handle is a 4px-wide hit target centered on the column boundary, visible as a 1px `ink-tertiary` line on hover. It does not appear as a permanent separator between every column; PaperDesign tables use row striping or hover highlight for row identity, not heavy vertical borders.

**Horizontal scrolling** is a panel-level behavior, not a cell-level one. The table scrolls as a unit; headers stay sticky. Frozen columns (e.g., a name column that stays fixed as the rest scroll) use a 1px `ink-tertiary` shadow-replacement border to signal the freeze boundary, per `PD §9` (hairline borders over shadows where possible).

### Anti-patterns

- ❌ Rows taller than 32px (Compact) or 40px (Comfortable) when the data doesn't genuinely require it.
- ❌ Centered text in non-numeric columns — center alignment implies symmetry; most table content is asymmetric.
- ❌ A "Loading…" state that blanks out the whole table while a sort request is in flight (`PI §4.4`) — keep the stale rows visible; replace them when the real rows arrive.
- ❌ Permanent checkboxes visible on every row when multi-select isn't actively in use — show them on row hover in selectable tables.
- ❌ Column resize handles that don't snap to the grid.
- ❌ Vertical column borders as a default style — they add visual weight without adding information in most tables; use row hover instead.
- ❌ A table inside a card inside a table — never nest tables.

---

## 5. Docking Panels

The docking panel is PaperDesign's most opinionated pattern, because PaperDesign has a very specific opinion about the difference between a panel and an overlay that most design systems treat as a stylistic choice. Here it's a structural one.

### The core decisions

A panel is **grid-aligned, grid-column-wide, and pushes or occupies real layout space** — it is not a float that overlaps content. When a panel opens, the content it sits beside gets narrower; it does not get covered. When it closes, content gets wider. This is "docked" in the architectural sense: the panel is a column in the layout, not a layer above it.

There are two legitimate panel positions: edge-docked (from the left or right of the window, spanning full height) and content-docked (occupying a column beside a specific content pane, not the whole window). What's not legitimate: a panel docked to a specific trigger point (a popover that stays open while you work), a panel that floats above other panels, or a panel with `elevation-2` (that's an overlay, not a panel — per `PD §4`).

**Slide animation** uses the Macro duration (260ms) from `PD §12.1` — panels are large elements with significant travel distance, and 260ms is the ceiling appropriate to that. The motion is a single axis slide (the panel enters along its docked edge) with no simultaneous opacity change — per the "one property at a time" rule in `PD §12.1`. Content doesn't animate to fill the vacated space; it simply resizes (the layout negotiates, the panel doesn't perform).

**Multiple open panels** is a valid state. A left sidebar and a right inspector can both be open; they each occupy their column, and the content pane between them is narrower but still functional. The spec for this is the 12-column grid in `PD §2.3` — a left sidebar at 3 columns, right inspector at 3 columns, and a 6-column content pane is a legal, intentional layout, not an emergency. Design for it explicitly, don't let it happen by accident.

**State honesty for panels (`PI §1.1`):** whether a panel is open or closed is a local UI state, determined entirely by the user's click. It updates immediately — no request, no server, nothing to wait for. The content *inside* the panel may have its own loading state if it fetches data; the panel's *open/closed* state does not.

### Anti-patterns

- ❌ A panel with a scrim or backdrop (`PD §4`) — a panel isn't an overlay and should never dim what's behind it.
- ❌ A panel that opens with `elevation-2` — panels are `elevation-0` or `elevation-1` at most; they belong to the page.
- ❌ Content that animates to fill the space when a panel closes — the layout resizes, that's it.
- ❌ A panel that's actually a persistent overlay masquerading as a panel (floats above content, closes on outside click) — if it closes on outside click, it's an overlay, and should be built and styled as one.
- ❌ A panel that opens at an arbitrary pixel width instead of whole column units.
- ❌ Horizontal slide for a panel docked to a vertical edge, or vertical slide for a panel docked to a horizontal edge — slide direction follows the dock axis.

---

## 6. Notifications

Notifications are the hardest pattern to get right because there are genuinely two very different things that get called "notifications," and the difference between them determines everything about how they're built.

### Transient vs. persistent: make the call first

Before designing any notification, answer: does the user need to acknowledge this, act on it later, or just know it happened and move on?

- **Toast / inline confirmation** — for things that happened and are over. Auto-dismisses. No action required. Never blocks input to anything. Lives at a screen corner, `elevation-2`, per `PD §11.6`. Duration: 3–4 seconds for informational, 6–8 seconds if it contains a secondary action ("File saved · Undo"). It does not live as a permanent badge anywhere after it dismisses — when it's gone, it's gone.
- **Badge / unread count** — for things that happened and are waiting for the user's attention. Lives on a nav item (sidebar row, tab, icon) as a count pill. `radius-pill`, `type-caption`, accent or semantic color, sits at the top-right corner of the icon it belongs to. Never more than one badge per nav item. Clears when the user visits the relevant section (`PI §1.1` — the clear is immediate and local).
- **Notification center (docked panel)** — for things that need to be reviewable later. This is a panel per `PD §4`, opened explicitly, not an overlay that appears on a badge click. The badge click navigates to the notification center; it doesn't conjure a floating panel the user didn't ask to open.

### The behavioral requirement

Toasts and badges share one `PaperImplementation` rule: **a notification should represent a true system state**, not an optimistic claim. A "File saved" toast appears when the file is actually saved (`PI §1.2`). A sync badge clears when sync is actually complete. A "Message sent" toast does not appear before the message is confirmed delivered — if the operation might fail, the honest pattern is the toast appearing on success and an inline error appearing on failure, not a premature "Sent" and a later silent correction.

For notification-triggered state changes that depend on the network (a push message arriving, a badge count updating), the update is scoped to the nav item or notification center widget that shows the count — nothing else repains (`PI §2.1`).

### Anti-patterns

- ❌ A toast that requires explicit dismissal — if it requires action, it's a dialog, not a toast.
- ❌ More than one toast visible at the same time — if two events happen simultaneously, queue the second.
- ❌ A floating notification panel that appears on badge-click and stays open while the user tries to work behind it — that's an overlay, which means it should be for transient decisions. A persistent notification inbox is a panel.
- ❌ A "sent" or "saved" toast before the operation has confirmed (`PI §1.4`).
- ❌ A badge that doesn't clear when the relevant section is visited — a badge is an unread count, not a permanent feature indicator.
- ❌ A toast that slides in from a direction inconsistent with where it lives — if the toast lives bottom-right, it enters from the right or bottom, not the top.

---

## 7. File Picker

A file picker is not a single component — it's a composition of several patterns (tree view, table or grid, search bar, breadcrumb) assembled into one surface. Getting it right is almost entirely about which patterns to use and how their borders connect.

### The core decisions

The file picker is a **panel or a page-level overlay**, depending on context — never a small modal. A modal file picker is one of the most common UX violations in desktop software because it forces the user to choose a file in a context where they might need to reference what they were just doing to know which file to choose. At a minimum, the file picker occupies enough space to show a tree (or at least a breadcrumb) and a list simultaneously.

**Structure:** breadcrumb navigation at the top (plain text path, not a button row), a filter/search bar (pill-radius, per §1), a tree view in a left panel for folder navigation (per §3), and the selected directory's contents in the main area as a list or grid. Two views are legitimate: a list (table rows, per §4) and a thumbnail grid (cards, `radius-sm`, `elevation-1`). The toggle between them is in the toolbar, a pair of icon buttons, not a settings option buried elsewhere.

**Breadcrumb** rows use `type-body` for the last segment (the current directory) and `type-body-sm ink-secondary` for all preceding segments — they read as navigation context, not page titles. Clicking any breadcrumb segment is immediate local navigation; no loading state unless the directory requires a network fetch.

**Directory navigation is immediate** when navigating within an already-loaded tree (local filesystem, cached directories). For network locations or lazy-loaded directories, the folder's contents show an honest indeterminate state (per `PI §4.2`) in the list area only — the tree and breadcrumb remain interactive (`PI §4.4`).

**Selection state** follows the same row-highlight convention as every other list in the system (`accent-primary-muted` background). Multi-select uses the same checkbox-on-hover pattern as tables (§4). Confirming a selection with "Open" or "Select" is a primary button at the bottom of the picker, not inside the list — the action lives in chrome, not in content.

### Anti-patterns

- ❌ A file picker that opens as a small modal (400×300px) — it should be large enough to actually show the filesystem.
- ❌ No way to navigate back from the current directory other than clicking the breadcrumb — a "Back" nav button in the toolbar is worth adding.
- ❌ Breadcrumb that shows only the current directory name, not the path — a breadcrumb without ancestors is just a label.
- ❌ Loading state on the whole picker while a subfolder loads — only the list area should show a loader (`PI §4.4`).
- ❌ A thumbnail grid with `radius-pill` on image thumbnails — thumbnails are containers, not bars, so `radius-sm` per `PD §5.3`.
- ❌ A "Select" button that activates before a file is chosen — it should be disabled until there's a valid selection, with an honest explanation if appropriate (`PI §4.2`).

---

## 8. Settings

Settings surfaces are where design systems most often reveal whether they have a real opinion about *information architecture* or just surface aesthetics. PaperDesign has an opinion.

### Panels and pages, not dialogs

**Settings should not live in a modal dialog.** A settings dialog implies settings are a temporary interruption — you open them, change a thing, close them. Real settings in any serious application are a destination: you browse them, you look for something, you change multiple things across sections, you come back next week. That's a panel or a page, not an overlay.

For simple apps (fewer than ten distinct settings), a docking panel (§5) on the right is appropriate — open from a gear icon, occupies a right column, close when done. For complex apps (sections, subsections, deeply nested preferences), settings is its own route/page with a left-nav sidebar and a content pane — exactly the same structure as any other content-heavy section of the app.

Either way, the pattern is: **sidebar (or section nav) on the left, settings form on the right.** Not tabs across the top. Not a flat scrolling list with section headers. Not a tree that opens subpanels. Sidebar nav → content pane is the one pattern that scales from ten settings to two hundred.

### Settings form layout

Settings are forms. Forms in PaperDesign use the same grid as everything else — labels left-aligned to a column boundary, inputs spanning the remaining columns, help text / description in `type-body-sm ink-secondary` below the label. Groups of related settings are visually separated by `space-5` between groups, never by cards with `surface-raised` — a card around a settings group implies the settings have a separate "elevation" from the page, which they don't.

**Toggles and checkboxes** are the native control for binary settings — never a "Save" button per toggle. A toggle flips immediately (`PI §1.1`) for settings that apply instantly (theme, density, a display preference). For settings that require a restart or re-authentication to take effect, the toggle still flips immediately (the intent is recorded), but a banner or inline note appears explaining that the change takes effect after the required action. That's the honest version of "you changed it but it isn't applied yet" (`PI §1.2`).

**Dangerous settings** (delete account, reset all, clear cache) live in their own section, clearly labeled "Danger zone" or equivalent in `ink-primary` weight (not in `accent-danger` — the color coding should appear only in the confirmation dialog, not as a permanent red section header). The action lives behind a confirmation overlay (`PD §11.5`) — this is one of the few cases in PaperDesign where a modal is the correct affordance, because a destructive action genuinely requires a decision before proceeding.

### Anti-patterns

- ❌ Settings in a modal with tabs — the moment there are tabs, it's not a modal, it's a page that's pretending to be a modal.
- ❌ A "Save Changes" button at the bottom of settings — for settings that apply instantly, don't require a save; for settings that don't, explain why inline, don't hide it behind a global save.
- ❌ Cards around settings groups (settings don't have elevation).
- ❌ Dangerous actions in the main settings flow without a clear section boundary and a confirmation step.
- ❌ A settings sidebar with items that open sub-panels that open further sub-panels — if the hierarchy is that deep, the information architecture is wrong, not the component.

---

## 9. Text Editors

A text editor is the pattern where chrome/content separation (`PD §4.1`) matters most, because the editor's content is authored by the user — it's not the application's own UI, even though it lives inside the application's window.

### The core decisions

The text editor surface is **content, not chrome** — it does not use `surface-raised`, system typography tokens, or `accent-primary` link styling unless those happen to match the user's document. The editor area is a viewport into something the user is creating; the application's job is to provide the furniture around it (toolbar, sidebar, scrollbar, title bar) and then stay out of the way.

**The toolbar** (formatting controls) lives above the content area, inside a `surface-raised` bar, using the standard icon-row reflow rules from `PD §3.2`. Toggle state for active formatting (Bold is active because the cursor is in bold text) uses the same row-active treatment as nav items — `accent-primary-muted` background on the icon button, not a border or a color change on the icon itself. The toolbar does not float over the content. It is not contextual (appearing only when text is selected) for a document editor — that interaction works for inline rich-text components but not for a primary editing surface the user is in for hours.

**Scroll position is sacred.** When the user's document updates (a collaborator makes a change, autosave triggers a local refresh, a spell-check pass runs), the visible scroll position must not change unless the change happened in the currently visible region and the change is large enough to shift line positions significantly. This is update locality (`PI §2.1`) applied to the editor specifically: a background autosave does not reset the user to the top of the document. Rebuilding the whole document tree on a small change is a locality violation.

**Autosave** follows `PI §1.2`: local write is immediate and cheap — confirm it to the user immediately with a quiet inline state ("Saved locally"). The remote sync is honest separately — "Syncing…" beside the title while in progress, cleared when confirmed, replaced with an error and options if it fails (`PI §4.3`). The user's draft is never hostage to the network's availability (`PI §5.2`).

**Line length.** The editor content area should constrain its text measure to roughly 60–75 characters per line for prose (a `max-width` on the content column, not a constraint on the window) — this is a readability rule borrowed from Sora's reader mode, and it applies equally to any prose-primary editor. Code editors and log viewers are exempt; their content is not prose.

### Anti-patterns

- ❌ A floating toolbar that appears on text selection in a full document editor — reserve contextual toolbars for inline rich-text components, not primary editors.
- ❌ A "Save" button that's the only path to persisting the document — editors should autosave, with an explicit manual save as a supplement, not the primary mechanism.
- ❌ An autosave that resets scroll position or reloads visible content (`PI §2.1`).
- ❌ Styling editor content with system accent colors, surface tokens, or system link styles — content is content; Sora's §4 rule applies here too.
- ❌ A "Syncing…" state that blocks editing — writing and syncing are independent operations; only the sync indicator changes while syncing (`PI §4.4`).
- ❌ A full-document re-render triggered by a single small remote change — the change should propagate only to the affected region.
- ❌ Text that doesn't have a line-length constraint in a prose editor — 120-character-wide paragraphs spanning the full window are a readability failure, not a density feature.

---

## 10. Quick Reference

| Pattern | PD Rule most in play | PI Rule most in play |
|---|---|---|
| Search bar / filter | `§5.2` (bar height) + `§5.1` (pill radius) | `§1.1` (local filter = immediate) |
| Command palette | `§4` (true overlay) + `§12.1` (250ms ceiling) | `§1.1` (local = immediate, remote = honest) |
| Tree view | `§13` (Compact default) + `§12.3` (expand = immediate) | `§2.3` (lazy node loader = scoped state only) |
| Table | `§13` (row height) + `§11.3` (no nesting) | `§1.1` (local sort = immediate) + `§2.1` (sort = only table rerenders) |
| Docking panels | `§4` (panel ≠ overlay) + `§12.1` (260ms macro) | `§1.1` (open/close = local, immediate) |
| File picker | `§4.1` (chrome/content) + `§5.3` (thumbnails = radius-sm) | `§4.4` (subfolder load = list only, not whole picker) |
| Settings | `§4` (settings ≠ modal for complex cases) + `§11.5` (destructive = confirm dialog) | `§1.1–1.2` (toggles = immediate; restart-required = honest) |
| Notifications | `§4` (toast vs. widget vs. panel) + `§11.6` (toasts) | `§1.2` (toast = after actual confirmation, never before) |
| Text editor | `§4.1` (content ≠ chrome) | `§2.1` (scroll position) + `§5.2` (draft ≠ hostage to network) |
| Theme/prefs | `§15` (app identity, user freedom) | `§5.5` (OS prefs = behavioral requirements) |

---

*Patterns are worked examples, not laws. When a real product surfaces a case this document didn't anticipate, the right move is to reason from `PaperDesign.md` and `PaperImplementation.md` directly — then, if the solution is general enough, add it here. The catalog should grow with the system, not constrain it.*
