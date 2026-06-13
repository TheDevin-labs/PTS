# Picture This (.pts) Language Specification
**Version:** 0.1.2
**File Extension:** `.pts`
**Full Name:** Picture This

---

## Overview

Picture This is a web-native presentation format. `.pts` files are rendered directly in the browser via a browser extension — no compilation to HTML required. The format is designed to be minimal, readable, and easy to write by hand.

---

## File Structure

Every `.pts` file begins with a version header, followed by optional metadata, then slide content.

```
<pts=0.1>
~title = "My Presentation"

; slide content here
```

---

## Coordinate System

Picture This uses a **center-origin coordinate system**, identical to Scratch:

- `(0, 0)` is the **center** of the screen
- X increases to the **right**, decreases to the **left**
- Y increases **upward**, decreases **downward**

```
         +Y
          |
 -X ------+------ +X
          |
         -Y
```

---

## Syntax

### Comments

Lines beginning with `;` are comments and are ignored by the renderer. Inline comments are also supported after any statement.

```
; this is a full line comment
~text (0, 0, "Hello") ; this is an inline comment
```

---

### Version Header

Required. Must be the first line of every `.pts` file.

```
<pts=0.1>
```

---

### Setters

Setters configure global or slide-level properties. They use the `=` operator and do not render anything on the canvas.

```
~setter_name = value
```

**Available setters:**

| Setter | Description | Example |
|---|---|---|
| `~title` | Browser tab title | `~title = "My Slides"` |
| `~set_bg_color` | Slide background color | `~set_bg_color = #1a1a2e` |

---

### Elements

Elements are rendered on the canvas. All renderable elements require `x` and `y` as their first two arguments, defining the position relative to the center of the screen.

```
~element (x, y, ...args, ...optional_named_params)
```

**Rule:** If it appears on the canvas → first two args are always `x, y`.

---

### Slide Breaks

| Command | Description |
|---|---|
| `~break` | Start a new slide, **inheriting** the previous slide's background |
| `~breaknew` | Start a new slide, **resetting** the background to default |

---

### Labels (Anchors)

Labels are invisible jump targets, defined with a backtick prefix. They mark a position that `~jump` can navigate to.

```
`label_name
```

Labels are **global** across all slides — a jump can target any label anywhere in the file.

---

## Elements Reference

### `~text`

Renders text on the canvas.

```
~text (x, y, "content", ...optional)
```

**Example:**
```
~text (0, 0, "Hello world")
~text (0, 50, "Big Title", size=48, color=white, bold)
~text (-100, -30, "Subtitle", size=18, color=#888, italic)
```

---

### `~image`

Renders an image on the canvas.

```
~image (x, y, "filename.png", ...optional)
```

**Example:**
```
~image (0, 0, "logo.png", width=300, height=200)
```

---

### `~jump`

Renders a clickable element that navigates to an internal label.

```
~jump (x, y, label_name "button text", ...optional)
```

**Example:**
```
~jump (0, -100, slide2 "Next →", color=lime, size=16)
```

---

### `~link`

Renders a clickable element that opens an external URL. URLs are wrapped in `{}`.

```
~link (x, y, {url} "button text", ...optional)
```

**Example:**
```
~link (0, 0, {github.com} "Source Code", color=blue, size=18)
```

---

## Style Parameters

All renderable elements accept optional named style parameters. Parameters are **order-free** and appended after required arguments.

### Named Parameters

| Parameter | Type | Description | Example |
|---|---|---|---|
| `color` | color | Text or element color | `color=red`, `color=#ff0000` |
| `size` | number | Font or element size | `size=24` |
| `font` | string | Typeface | `font=arial` |
| `width` | number | Element width | `width=300` |
| `height` | number | Element height | `height=200` |
| `opacity` | number | Transparency (0–1) | `opacity=0.5` |

### Bare Flags

Boolean style properties are written as bare words — their presence means `true`.

| Flag | Description |
|---|---|
| `bold` | Bold text |
| `italic` | Italic text |
| `underline` | Underlined text |

**Example:**
```
~text (0, 0, "Hello", bold, italic, size=32, color=red)
```

### Color Values

Colors can be written as:
- Named colors: `red`, `blue`, `lime`, `white`, `black`
- Hex codes: `#ff0000`, `#1a1a2e`, `#888`

---

## Animations

Animations are appended after an element in square brackets `[...]`. Multiple animations can be chained with `&`, and chains can be looped.

### Basic Animation

```
~element (...) [effect duration]
```

**Available effects:**

| Effect | Description |
|---|---|
| `fade-in` | Fades element from invisible to visible |
| `fade-out` | Fades element from visible to invisible |
| `slide-in-left` | Slides in from the left while fading in |
| `slide-in-right` | Slides in from the right while fading in |
| `zoom-in` | Scales up from 50% while fading in |

**Duration** is in seconds.

```
~text (0, 0, "Hello", bold) [fade-in 1]
~text (0, -40, "World") [slide-in-left 0.6]
```

---

### Sequencing

By default, animated elements on a slide play **one after another** in the order they appear in the file — each starts after the previous finishes.

```
~text (0, 80, "First", bold) [fade-in 1]      ; plays at t=0
~text (0, 20, "Second") [slide-in-left 0.6]   ; plays at t=1s
~text (0, -40, "Third") [zoom-in 0.5]         ; plays at t=1.6s
```

---

### `+wait_until` — Timing Control

`+wait_until` overrides the default sequencing for an element or animation step.

| Syntax | Meaning |
|---|---|
| `+wait_until` or `+wait_until=0` | Start at the same time as the previous animation (parallel) |
| `+wait_until=N` | Start N seconds after the previous animation finishes (extra gap) |

```
; all three zoom in at the same time
~text (-120, 0, "A") [zoom-in 0.5]
~text (0,    0, "B") [zoom-in 0.5 +wait_until=0]
~text (120,  0, "C") [zoom-in 0.5 +wait_until=0]

; 1 second pause before this one starts
~text (0, -80, "Done!") [fade-in 0.8 +wait_until=1]
```

---

### Chaining — `&`

Multiple animations can be chained on a single element using `&`. They play in order, each starting after the previous finishes (unless `+wait_until` overrides this).

```
~text (0, 0, "Blink") [fade-in 0.5] & [fade-out 0.5]
```

`+wait_until` applies within a chain too:

```
~text (0, 0, "Pulse") [fade-in 0.5] & [fade-out 0.5 +wait_until=1]
; fade-out starts 1s after fade-in finishes
```

---

### Loops — `|` and `<` / `>`

Animation chains can be looped using loop markers.

| Marker | Description |
|---|---|
| `\|` | Loop start — placed before the first `[...]` |
| `<` | Loop end — infinite loop |
| `<N` | Loop end — loop exactly N times, then stop |
| `>` | Skip to next loop iteration |

```
; infinite blink
~text (0, 40, "BLINKING!") | [fade-in 0.5 +wait_until] & [fade-out 0.5] <

; blink 3 times, then stay in final state (invisible)
~text (0, -40, "3 blinks") | [fade-in 0.4 +wait_until] & [fade-out 0.4] <3
```

When a finite loop finishes, the element stays in whatever state the last animation left it.

---

### Animation Syntax Summary

```
; single
~element (...) [effect duration]

; with timing
~element (...) [effect duration +wait_until=N]

; chained
~element (...) [effect duration] & [effect duration]

; infinite loop
~element (...) | [effect duration] & [effect duration] <

; finite loop
~element (...) | [effect duration] & [effect duration] <3

; skip iteration
~element (...) | [effect duration] & [effect duration] >
```

---

## Navigation

| Input | Action |
|---|---|
| `→` / `Space` / `↓` | Next slide |
| `←` / `↑` | Previous slide |
| Click canvas | Next slide |
| `~jump` button | Jump to label |
| `~link` button | Open URL in new tab |

---

## Full Example

```
<pts=0.1>
~title = "My Presentation"

; Slide 1 - Title
~set_bg_color = #1a1a2e
~text (0, 60, "My Presentation", size=48, color=white, bold) [fade-in 1]
~text (0, 10, "by Jane Doe", size=18, color=#888) [slide-in-left 0.6]
~text (-80, -40, "Fast", size=20, color=lime) [zoom-in 0.4]
~text (0,   -40, "Simple", size=20, color=lime) [zoom-in 0.4 +wait_until=0]
~text (80,  -40, "Elegant", size=20, color=lime) [zoom-in 0.4 +wait_until=0]
~jump (0, -100, slide2 "Get Started →", size=16, color=lime) [fade-in 0.5]

~break

; Slide 2 - Content (inherits dark bg)
`slide2
~text (0, 80, "Why Picture This?", size=36, color=white, bold) [fade-in 0.8]
~text (-180, 20, "No compilation", size=22, color=lime) [slide-in-left 0.5]
~text (-180, -20, "Web native", size=22, color=lime) [slide-in-left 0.5]
~text (-180, -60, "Easy syntax", size=22, color=lime) [slide-in-left 0.5]
~text (0, -30, "!", size=48, color=white, bold) | [fade-in 0.3 +wait_until] & [fade-out 0.3] <
~jump (0, -120, slide3 "Resources →", size=16, color=lime)

~breaknew

; Slide 3 - Links (reset bg)
`slide3
~set_bg_color = #f0f0f0
~text (0, 80, "Resources", size=36, color=#1a1a2e, bold) [fade-in 0.6]
~link (0, 20, {github.com} "Source Code", size=20, color=blue) [slide-in-right 0.5]
~link (0, -20, {example.com} "Live Demo", size=20, color=blue) [slide-in-right 0.5]
~jump (0, -80, slide2 "← Back", size=16, color=#888)
```

---

## Quick Reference

| Syntax | Renders? | Purpose |
|---|---|---|
| `<pts=0.1>` | ❌ | Version header |
| `; comment` | ❌ | Comment (full line or inline) |
| `` `label `` | ❌ | Jump anchor |
| `~title = "..."` | ❌ | Browser tab title |
| `~set_bg_color = color` | ❌ | Background color |
| `~break` | ❌ | New slide, inherit bg |
| `~breaknew` | ❌ | New slide, reset bg |
| `~text (x, y, "...")` | ✅ | Text element |
| `~image (x, y, "file")` | ✅ | Image element |
| `~jump (x, y, label "text")` | ✅ | Internal link |
| `~link (x, y, {url} "text")` | ✅ | External link |
| `[effect duration]` | — | Animation |
| `[...] & [...]` | — | Chained animations |
| `+wait_until=N` | — | Timing offset |
| `\| [...] <` | — | Infinite loop |
| `\| [...] <N` | — | Finite loop |
| `\| [...] >` | — | Skip loop iteration |
