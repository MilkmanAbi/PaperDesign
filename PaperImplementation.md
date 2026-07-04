# PaperImplementation
### The behavioral half of PaperDesign — how a PaperDesign interface acts, not just how it looks.

Version 0.1 — companion to `PaperDesign.md`

---

## 0. What this document is

`PaperDesign.md` is a visual and structural spec: grid, shape, color, type, motion timing. It answers "what does this look like, and how big is it."

This document answers a different question: "what does this *do* when you touch it, and what does it do when something goes wrong." Two interfaces can be pixel-identical against `PaperDesign.md` and feel completely different to use — one calm and trustworthy, one fussy and evasive — because that difference lives in behavior, not layout. This is where PaperDesign's principles 9 and 10 (§1 of the base spec) get expanded into rules you can actually build against.

Nothing here overrides `PaperDesign.md`. Where this document talks about motion or feedback, it's assuming the timings, easing, and visual language already defined there (§12). This document is additive: it's the part of the system that isn't a token.

**This document is what creates cross-app familiarity in a Paper ecosystem.** `PaperDesign.md` makes each app coherent within itself — its own grid, its own accent, its own typographic system, its own sovereign identity. This document is what makes a user feel at home in an unfamiliar Paper app: not because it looks like the one they already know, but because it *behaves* the same way. Buttons respond the same way. Failures are honest the same way. Long operations report themselves the same way. Only the affected part of the screen changes. These are habits — and habits transfer between apps even when visuals don't.

**The one sentence this whole document expands:** *the user is to be respected, not served or abandoned.* Every section below is that sentence applied to a different situation — a click, a slow network, a failure, a rebuild, a system preference the user already set.

---

## 1. State Honesty

### 1.1 The feedback is the state change

For any action whose outcome is already known the instant it happens — a checkbox, a toggle, a tab closing, a row reordering locally, a character appearing as you type — there is no meaningful gap between "the interface acknowledged you" and "the thing happened." Collapse them into one event.

```
click  →  new state
```

not

```
click  →  acknowledgment  →  (some delay)  →  new state
```

If you can already compute the new state before you've finished handling the click, render the new state on that same frame. Don't hold it behind a transition just because a transition exists for that component in other contexts.

### 1.2 UI state vs. system state — the honest split

This only gets complicated when the outcome *isn't* fully under the interface's control — saving to a server, syncing, uploading, deleting from the cloud. Here, PaperDesign draws a hard line between two things that are often conflated:

- **UI state** — what the user's intent was. This can and should update immediately: the star fills in, the item moves to the "favorited" list, the toggle flips.
- **System state** — what's actually true on the other end of a network call. This is not known immediately and must not be claimed until it's confirmed.

```
Click "Favorite"
  → ★ appears immediately            (UI state — honest: this is what you asked for)
  → "Syncing…" appears beside it     (system state — honest: not confirmed yet)
  → "Syncing…" clears on success,
    or reverts with an explanation on failure
```

Showing "Saved" before the server has actually saved is not optimism, it's a lie the interface tells the user. Showing nothing at all until the network call resolves is a different failure — see §4.

### 1.3 Don't animate toward a known outcome

Covered as a motion rule in `PaperDesign.md` §12.3 — restated here as the behavioral principle it comes from: motion should never be the mechanism that *delays* an already-known result in order to look smoother. A 150ms ease into "checked" is 150ms of the interface being less honest than it could be for zero benefit. Motion is for the parts of the interaction that are genuinely temporal (a panel physically sliding into its dock, a row easing out of a list because it now occupies different space) — never for gating a value the system already has.

### 1.4 Never fake progress or completion

A progress bar that isn't tracking real progress, a "Saved" state shown before the save resolved, a spinner that keeps spinning past the point where the interface actually gave up and is silently retrying — all of these are the same mistake: performing certainty the interface doesn't have. If you don't have a real progress signal, don't show a progress bar; show an honest indeterminate state (§4.2) instead.

---

## 2. Update Locality — "groups, not colonies"

### 2.1 The interface is a tree of independent groups, not one surface

`PaperDesign.md` §3.1 already treats a layout as a tree of groups for *responsive* purposes — a toolbar reflows independently of a content grid. This section generalizes the same tree structure to *rendering*: a state change should propagate only through the branch of the tree that actually changed, not ripple through the whole page because the whole page happens to share one render pass.

```
Window
 ├── Sidebar         — untouched
 ├── Toolbar         — untouched
 ├── Document        — untouched
 └── Download widget
      └── Progress bar   — this is the only thing that changed
```

Progress moving from 34% to 35% should not cause the sidebar to repaint, the toolbar to re-measure, or the scrollbar to visibly reset. If it does, the group boundaries were drawn in the wrong place, not "React/Flutter/the framework is just like that."

### 2.2 Why this is a felt quality, not just a performance one

A page that quietly re-renders unrelated regions on every small update produces a specific, hard-to-name feeling: the software seems to be doing something, all the time, even when you didn't ask it to. That's the opposite of the "quiet tool" this system is trying to build (`PaperDesign.md` §0, §1 principle 8). The desk analogy holds exactly: moving one notebook on a desk doesn't make the pens jump or the lamp flicker. Only the notebook moved. An interface should have that same locality.

### 2.3 Practical guidance, framework-agnostic

- **Colocate state with the smallest component that owns it.** A progress bar's percentage should not live in a store that also re-triggers the whole page's render tree on every tick.
- **Treat "does this group need to know about that change" as a real design question**, the same way you'd ask "does this group need to reflow at this breakpoint" in `PaperDesign.md` §3.2. Most of the time, the answer is no.
- **Prefer fine-grained reactivity (signals, scoped state, targeted diffing) over broad state objects that fan out to everything subscribed to "the app."** This is an architecture decision, but it's a PaperDesign decision too — the visual stillness principle 10 describes is downstream of it.
- **A loading or error state belongs to the group that's loading or erroring**, not to a global boolean that disables the whole screen. See §4.4.
- **Test for this the same way you'd test spacing:** watch the actual screen (not just the diff/profiler) while triggering a small, unrelated-looking update. If anything outside the group you touched visibly shifts, flickers, or repaints, that's a locality bug, not a cosmetic one.

### 2.4 This is not an anti-consistency rule

Locality doesn't mean groups are isolated from every design-system rule — density mode, theme, and accent color are still global and should update everywhere at once when the user actually changes them (that's a real, user-intended global change, and it should look like one). The rule is about *incidental* propagation — updates rippling outward because of how the code happens to be wired, not because the user asked for something app-wide.

---

## 3. Motion Philosophy — animate actions, not outcomes

This is the plain-language version of §1.3 and `PaperDesign.md` §12.3, worth stating once on its own because it's the single sentence that explains most of this document's motion-adjacent rules:

> **Animate the interaction. Never animate the information.**

- The mouse press — animate it (§12.2 of the base spec).
- A panel physically occupying new space — animate it (it's genuinely moving).
- A row collapsing because something above it was removed — animate it (the layout genuinely changed).
- The document, the value, the checked state, the saved flag — never animate *becoming* true. It either already is, or it honestly isn't yet (§1.2).

The paper metaphor is literal here: when you write on paper, the graphite exists the instant the pencil touches down. It doesn't fade in. When you erase, the mark is gone — it doesn't dissolve over a tasteful 200ms. The physical action (your hand moving) is what has duration and deserves acknowledgment. The result is not a performance; it's a fact.

---

## 4. Long-Running Operations — transparency over optimism

### 4.1 No unbounded waiting

An operation that might take a while should tell the user roughly what "a while" means, and update that expectation as reality diverges from it:

```
Connecting…
(a few seconds pass)
Still trying…
(the expected window passes)
Couldn't connect.        [Retry]   [Work offline]
```

Nothing here should sit in an unlabeled spinner for an unbounded amount of time. If you don't know how long something will take, say that too ("This is taking longer than usual") rather than saying nothing.

### 4.2 Active feedback, not a passive spinner

A disabled control with no explanation forces the user to guess why it's disabled. Pair the disabled state with a short, specific label of what's happening:

```
[ Saving… ]     (button disabled, label explains why)
```

not

```
[ Save ]     (button disabled, spinner somewhere nearby, no stated reason)
```

The user should never have to infer "is this frozen or just busy" — the interface should just say which one it is.

### 4.3 Always give the user a way out

Not every operation is precious to the user just because the software initiated it. Someone who clicked "Sync" may not care if it fails; someone mid-upload may just want to keep typing. Long-running or uncertain operations should offer a real alternative to indefinite waiting, not just a spinner and patience:

```
Syncing…
Couldn't reach the server.
Your changes are safe on this device.
[ Retry ]   [ Keep working ]   [ Work offline ]
```

Cancel, "keep working," and "do it later" are first-class outcomes, not failure states to be hidden behind a retry loop the user can't see or stop.

### 4.4 One failing operation never takes the rest of the app hostage

If cloud sync fails, the editor, sidebar, and search should keep working. If a save fails, the document isn't locked. Scope loading/error/disabled states to the smallest group that's actually affected (§2.3) — this is update locality (§2) applied specifically to failure states. A single background operation failing should never be indistinguishable, from the user's seat, from the whole application being broken.

---

## 5. Respect the User's Work

### 5.1 The core ethic

> **The user is to be respected — not served, and not abandoned.**

"Served" software tries to be clever on the user's behalf: it hides errors, retries silently forever, assumes what the user wants, and smooths over its own failures with animation instead of information. "Abandoned" software does the opposite failure: it loses work, freezes, or gives up with no explanation and no path forward. Respect is the position between those two — tell the truth, protect what's already been created, and let the user decide what happens next.

### 5.2 Preserve local work before protecting the network's pride

The user's work is the most valuable thing in the application. The network, the server, and the sync service are the least reliable things in the application. When something has to give, it's never the user's work:

- Keep unsent changes locally, visibly, and recoverable — never discard a draft because the request that would have sent it failed.
- Make "your changes are safe on this device" an explicit, stated fact when a remote operation fails, not an assumption the user has to trust silently.
- Never close a document, clear a field, or navigate away as a side effect of an error — an error should narrow what's broken, not widen what's lost.

### 5.3 Die gracefully

Borrowed from the way well-behaved command-line tools fail: a failure should report itself clearly, clean up after itself, and leave everything else usable. In a GUI, "dying gracefully" doesn't mean the app exits — it means the *feature* that failed admits it failed, says so in one clear sentence, and gets out of the way without taking anything else down with it (§4.4).

```
Bad:   Error saving.  [OK]  → document silently closes
Good:  Couldn't save to the server. Your draft is kept locally.
       [ Retry ]   [ Save a copy ]   [ Dismiss ]
```

### 5.5 System preferences — behavioral requirements, not visual options

Respecting OS-level user preferences is an expression of the same principle as everything else in this section: the user made a decision about their environment; the application does not override it silently. These are not optional courtesies or "nice-to-have accessibility features" — they are behavioral requirements at the same level as state honesty and update locality.

**Reduced motion** is the most important. A user who has set "reduce motion" at the OS level has made a statement about their perceptual needs — and possibly a medical one (vestibular disorders, motion sensitivity, migraine). A Paper app must honor this unconditionally:
- All decorative transitions → duration zero.
- Spatial transitions necessary to understand a layout change (a panel opening, a list reordering) may remain, at reduced duration and with no secondary animation layered on top.
- There are no exceptions for "this one is really subtle" or "this one communicates something." Reduced motion means reduced motion.

**Dark/light mode** follows the OS default at first launch, always. The user should never be asked on first run to choose a theme they already specified at the OS level. If the user wants to override this in-app later, that override lives in settings — it never replaces the OS default as the first-run behavior.

**High-contrast mode** — if the platform exposes this preference, the app's neutrals, borders, and focus rings shift to maintain the required contrast ratios without any manual user action inside the app.

**Text/display size** — where the platform exposes this preference, respect it. An app that silently overrides the user's system font size is making an accessibility decision it doesn't own.

Implementation note: check these preferences at app startup, and re-check them if the OS fires a preference-change event (e.g., the user switches dark mode mid-session). A Paper app that requires a restart to pick up an OS dark-mode change has implemented this as a one-time read, not a behavioral commitment.

### 5.4 A short test for any failure state

Before shipping an error, loading, or timeout state, check it against three questions:

1. **Does the user know what happened?** (not "an error occurred" — the actual problem, in one sentence, per `PaperDesign.md` §14)
2. **Does the user know what they can do about it?** (retry, work offline, save a copy, dismiss — a real option, not just "OK")
3. **Is anything the user made still safe?** If the honest answer to any of these is no, the state isn't finished.

---

## 6. Anti-Patterns

- ❌ Animating a checkbox, toggle, or other locally-certain state change instead of showing it immediately (§1.1, §1.3).
- ❌ Displaying "Saved," "Synced," or similar before the remote operation has actually confirmed it (§1.2, §1.4).
- ❌ A progress indicator that isn't tied to real progress.
- ❌ An update to one component causing a visible repaint, re-layout, or flicker in an unrelated sibling component (§2.1–§2.3).
- ❌ A global loading flag that disables the entire screen for one small, scoped operation (§2.3, §4.4).
- ❌ A spinner or "Loading…" state with no time bound, no explanation, and no way to cancel or work around it (§4.1).
- ❌ A disabled control with no label explaining why it's disabled (§4.2).
- ❌ An operation that retries silently and indefinitely in the background after the interface has stopped being honest about it still trying (§4.1, §4.3).
- ❌ A failure state whose only option is "OK" when a real alternative (retry, work offline, save locally) exists (§4.3).
- ❌ Any error path that discards, closes, or loses user-created content as a side effect (§5.2).
- ❌ One failing background operation (sync, upload, autosave) disabling or breaking unrelated parts of the application (§4.4, §5.3).
- ❌ An error message that states there was a problem without stating what it was or what to do next (§5.4).
- ❌ Ignoring the OS dark/light preference on first launch — always follow the OS; never ask the user to configure what they already set at the system level (§5.5).
- ❌ Playing decorative transitions for a user who has set "reduce motion" at the OS level — this is a behavioral requirement with no exceptions, not an accessibility checkbox (§5.5).
- ❌ Requiring an app restart to pick up an OS preference change (dark mode toggle, reduced motion toggle) — these changes should take effect in the running session (§5.5).
- ❌ Overriding the user's OS text/display size preference with a hardcoded font size (§5.5).

---

## 7. Quick Reference Card

```
STATE HONESTY   feedback = state change when the outcome is already known
                UI state (intent) updates instantly; system state (confirmed) updates honestly
UPDATE SCOPE    a change propagates only through the group that changed — never the whole tree
MOTION RULE     animate the interaction, never the information (see PaperDesign.md §12.3)
SLOW OPERATIONS state expected duration → escalate honestly → offer retry/alternative, never
                wait silently forever (no unbounded spinners)
DISABLED STATE  always paired with an active label explaining why ("Saving…" not just greyed out)
FAILURE         one clear sentence on what happened + a real option, never just "OK"
USER'S WORK     never discarded, closed, or lost as a side effect of a failure
ISOLATION       one failing operation never disables or breaks unrelated parts of the app
OS PREFS        reduced motion · dark/light · high contrast · text size — all behavioral
                requirements, not visual options; check at startup and on pref-change events
ECOSYSTEM ROLE  PI is what creates cross-app familiarity: shared behavior, not shared appearance
CORE ETHIC      the user is respected — not served, and not abandoned
```

---

*PaperImplementation is `PaperDesign.md`'s companion, not a subset of it — revise both together. If a new interaction pattern surfaces a case neither document anticipated, it probably belongs here rather than in the visual spec, since the question is almost always "what does this do," not "what does this look like."*
