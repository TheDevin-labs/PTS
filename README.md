# Picture This

**Picture This** (`.pts`) is a web-native presentation format that renders directly in your browser via a Chrome/Edge extension — no compilation, no exports, no PowerPoint.

Write a `.pts` file, open it in Chrome, and it just works.

---

## Install

1. Download `picture-this-v0.1.0.zip` from [Releases](../../releases)
2. Unzip it
3. Go to `chrome://extensions` and enable **Developer mode**
4. Click **Load unpacked** and select the unzipped `pts-extension` folder
5. Go to **Details → Allow access to file URLs** and enable it
6. Open any `.pts` file in Chrome — it will render automatically

---

## Example

```
<pts=0.1>
~title = "My Presentation"

; Slide 1
~set_bg_color = #1a1a2e
~text (0, 60, "Hello World", size=48, color=white, bold)
~text (0, 10, "Made with Picture This", size=18, color=#888)
~jump (0, -80, slide2 "Next →", size=16, color=lime)

~break

; Slide 2
`slide2
~text (0, 0, "That's it!", size=36, color=white)
~link (0, -60, {github.com} "View on GitHub", color=#00e676)
```

---

## Syntax

### Version header *(required, first line)*
```
<pts=0.1>
```

### Metadata
```
~title = "Tab title here"
~set_bg_color = #1a1a2e
```

### Text
```
~text (x, y, "content")
~text (x, y, "content", size=32, color=red, bold, italic)
```

### Image
```
~image (x, y, "file.png", width=300, height=200)
```

### Internal link
```
`my_label
~jump (x, y, my_label "Click me", color=lime)
```

### External link
```
~link (x, y, {github.com} "GitHub", color=blue, size=18)
```

### Slide breaks
```
~break      ; new slide, inherits background
~breaknew   ; new slide, resets background
```

### Style params *(all optional, order-free)*
| Param | Example |
|---|---|
| `color` | `color=red`, `color=#ff0000` |
| `size` | `size=24` |
| `font` | `font=arial` |
| `width` | `width=300` |
| `height` | `height=200` |
| `opacity` | `opacity=0.5` |
| `bold` | bare flag |
| `italic` | bare flag |
| `underline` | bare flag |

### Coordinates
Center-origin like Scratch: `(0, 0)` is the middle of the screen. X goes right, Y goes up.

---

## Navigation

| Input | Action |
|---|---|
| `→` / `Space` / `↓` | Next slide |
| `←` / `↑` | Previous slide |
| Click canvas | Next slide |
| `~jump` button | Jump to label |
| `~link` button | Open URL |

---

## Spec

See [`SPEC.md`](SPEC.md) for the full language specification.

---
