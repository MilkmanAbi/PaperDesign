# PaperDesign

A personal design system built over years of chasing software that felt good to use, not just good to screenshot.

---

## What this is

PaperDesign is not a component library. There is no npm package, no Figma plugin, no versioned release. It is a set of design and behavioral principles I developed for myself, documented so I stop re-arguing the same decisions every time I start a new project.

The name is literal. Think of an engineering notebook or a sheet of graph paper: every element sits on a rule, corners are square because paper corners are square, and the only softness comes from the writing on top of the page, not the page itself.

The system lives in three companion documents:

- `PaperDesign.md` - the visual and structural spec. Grid, shape language, color, typography, motion, density, components.
- `PaperImplementation.md` - the behavioral spec. How a Paper app acts when you click it, how it handles state, how it fails honestly, how it respects OS preferences.
- `PaperPatterns.md` - worked examples. Common interaction patterns (search bars, command palettes, tables, tree views, docking panels, etc.) showing how the principles apply to real problems.

---

## Why I made it (╯°□°）╯

The short version: I got exhausted.

I have been designing and building interfaces for myself for about five years. Not professionally, just obsessively. And somewhere along the way the dominant UI trends stopped feeling like progress and started feeling like a regression dressed up in a new coat.

Material Design was exciting when it arrived. Compared to the skeuomorphic era, it was cleaner, more consistent, easier to reason about. But over time a lot of software started adopting the surface characteristics instead of the underlying principles. You ended up with 48px minimum-height everything, giant rounded rectangles, acres of whitespace, and interfaces that show half as much information on the same monitor, primarily used by people sitting at a desk with a keyboard and mouse.

Electron made it worse. The specific flavor of "we took a website and put a window around it" UI became so common that you can identify it in three seconds. Large touch targets on a non-touch device. Cards inside cards. A spinner that takes 400ms to tell you a fetch failed. The whole page re-renders when a single badge count changes.

I was not nostalgic for Windows XP or flat gray Win32 chrome. I did not want to go back. What I wanted was what old productivity software got right, specifically the part where it assumed you were there to work, and it got out of your way so you could, combined with the modern things that are genuinely good: consistent design tokens, accessible contrast, intentional motion, readable typography.

PaperDesign is my attempt to hold both of those things at the same time.

---

## The core ideas (˘•ω•˘)

### Compact, not cramped

Default to the smallest size that still reads clearly. Not the smallest that technically fits, the smallest where nothing is harder to understand. Density comes from tight internal spacing, not removed spacing between unrelated groups. A dense layout still breathes between sections; it just refuses to waste space inside them.

Most first drafts of a PaperDesign layout run 15-25% too large. If it looks like a template from a website builder, every element was sized for "comfortable default" instead of "as small as clarity allows."

### Workflow beats decoration

If a choice makes the interface prettier but slower to scan or operate, the workflow wins. Every time, no exception. This is the one sentence that settles most design arguments.

### Groups, not colonies

The interface is a tree of independent groups, not one giant surface that happens to have parts. A state change propagates only through the group that actually changed. If a download widget updates its progress bar, the sidebar, the toolbar, and the document should not repaint. If a note syncs in the background, the editor should not jitter.

Old desktop software felt calm not because it had no animation, but because changes were local. One thing changed. Only that one thing moved.

### The feedback is the state change

For any action whose outcome the interface already knows (a checkbox, a toggle, a character typed), there is no gap between "acknowledged" and "happened." They are the same event. Motion exists to acknowledge an interaction or explain a spatial change, not to narrate a result that is already true. When you write on paper, the graphite exists the instant the pencil touches down. It does not fade in.

### Coherence over uniformity

Two PaperDesign apps should look nothing like each other and feel unmistakably related. Not because they share an icon pack or a corner radius, but because the reasoning that produced them is the same. Uniformity is copying assets. Coherence is sharing values.

A browser, a notes app, a PDF editor, and a CAD tool can all be PaperDesign apps. None of them should look like the others. All of them should feel like they were built by someone who solved problems the same way.

### Every app is a sovereign window

An application is not a fragment of the operating system. It has its own identity, its own accent color, its own typographic character. PaperDesign does not mandate a shared accent across apps or a shared visual theme. What unifies Paper apps is behavior, not paint. Behavioral conventions carry over between apps because they become habits, and habits reduce cognitive load. Visual conventions do not need to carry over, because visual identity is what makes each app feel like itself.

### Honesty above everything

An interface should never leave the user wondering whether it understood them. Not through animation, not through flourish, just through immediate and unmistakable feedback followed by an accurate representation of what is actually true. If something is saving, say it is saving. If something failed, say what failed and what to do about it. If an operation is taking longer than expected, say so and give the user a way out. Never discard local work because a network request failed.

---

## What PaperDesign is not

- A replacement for your framework's component library. It is a set of principles you apply when building with whatever you are already using.
- Retro or nostalgic. It borrows the structural priorities of older productivity software without bringing back beveled buttons, Win95 gray, or fake textures.
- Minimalist. Minimalism asks what can be removed. PaperDesign asks what actually needs to be here and how to organize it well. Those are different questions.
- Versioned. There is no PaperDesign 2.0. It is a living set of principles. When better accessibility research arrives or a new interaction pattern surfaces, the principles absorb it. No reinvention cycle, no migration guide.

---

## The document structure

```
PaperDesign.md          Visual + structural spec
                        Grid (8px base, 12-column desktop)
                        Shape language (two radii, period)
                        Color method (one undertone, four neutral steps, one muted accent)
                        Typography (voices by topic, not component)
                        Motion (duration scaled to element, no bounce, no decorative easing)
                        Density modes (Compact default, Comfortable, Expressive)
                        Components, iconography, theming philosophy

PaperImplementation.md  Behavioral spec
                        State honesty (feedback = state change when the outcome is certain)
                        Update locality (change propagates only through the affected group)
                        Long-running operations (transparency over optimism)
                        Failure handling (die gracefully, preserve local work)
                        System preferences (OS dark/light, reduced motion, high contrast
                        treated as behavioral requirements, not visual options)

PaperPatterns.md        Worked examples
                        Search bars, command palettes, tree views, tables,
                        docking panels, file pickers, settings, notifications, text editors
```

---

## Who this is for

Honestly, mostly me (•‿•)

But if you are also exhausted by the same things and find the principles useful for your own work, that is completely fine. The whole point is that PaperDesign is intentionally loose enough to produce apps that look nothing like each other while feeling like they came from the same school of thought. You are not expected to copy a specific button style. You are expected to ask whether your button respects the grid, gives honest feedback, and earns its size.

The kinds of software PaperDesign was designed for: browsers, IDEs, file managers, dashboards, CAD tools, terminal frontends, note-taking apps, PDF editors, developer tools. Anything where someone is going to spend six hours in it and needs it to get out of their way.

If you are building a social app or a playful consumer product with big gestures and expressive animation, PaperDesign is probably not a good fit. It optimizes for the eighth hour of using the app, not the first thirty seconds.

---

## Philosophy in one paragraph

PaperDesign is a design system built around the idea that software should earn the user's trust by consistently reflecting reality. Not by smoothing over rough edges with animation, not by hiding errors behind optimistic UX, not by making every element oversized so it feels "friendly." By being honest: state changes when the interaction happens, failures say what failed, long operations say how long they are taking, and only the thing that changed is the thing that visibly changes. Structure gives the system its clarity. Restraint about where to spend warmth keeps that structure from feeling like a spreadsheet. And each application gets to be its own thing, with its own identity, as long as it reasons about its design the same way.

---

## Five years of chasing this ᕙ(⇀‸↼‶)ᕗ

The concepts here did not arrive all at once. They accumulated over five years of building personal projects, getting annoyed at specific things in existing tools, fixing them, getting annoyed at the fix, and iterating. The grid came from realizing that eyeballing spacing produces layouts that look fine in isolation and inconsistent in aggregate. The two-radius rule came from noticing that a third radius always ended up decorative. The behavioral principles in PaperImplementation came from actually losing work because an interface lied about saving it.

None of this is claimed to be original. Most of it is a synthesis of things that better designers have known for decades, just assembled in a way that fits how I actually think about building software. The paper metaphor kept surviving every attempt to replace it with something more abstract, so I kept it.

If anything here is useful to you, use it. If you disagree with something, you are probably right for your context, because PaperDesign is not trying to tell you what to build. It is trying to tell you how to think about building it.

---

*PaperDesign, PaperImplementation, PaperPatterns. No npm. No Figma link. Just documents.*
