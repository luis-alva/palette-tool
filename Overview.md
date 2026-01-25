# Palette Tool

Lightweight web app for creating, editing, and exporting color palettes.

## Goal

A single-page web app that lets you:
- Import palettes from images or files
- Extract dominant colors from any image
- Edit and organize colors
- Export to formats for Procreate, game dev, and design tools

## Use Cases

| Context | Palette Size | Export Format |
|---------|--------------|---------------|
| 3D game models | 32-128 colors | PNG strip, .hex |
| Video/image reference | 4-16 colors | PNG strip |
| Procreate drawing | 8-32 colors | .swatches |
| Design tools | varies | .ase, .gpl |

---

## Features

### Import

| Source | Method |
|--------|--------|
| PNG strip | Read pixels horizontally, dedupe adjacent |
| Any image | K-means clustering for dominant colors |
| .hex file | Parse hex codes (one per line) |
| JSON | `{ name, colors: ["#hex", ...] }` |
| Manual | Color picker to add individual colors |

### Edit

- **Add** — Color picker, or paste hex code
- **Delete** — Click to select, delete key or button
- **Adjust** — HSL sliders + RGB input
- **Rearrange** — Drag and drop
- **Name** — Palette title for export

### Export

| Format | Extension | Used By |
|--------|-----------|---------|
| PNG strip | `.png` | Universal visual reference |
| Procreate | `.swatches` | iPad drawing |
| Adobe | `.ase` | Photoshop, Illustrator, Affinity |
| GIMP | `.gpl` | GIMP, Inkscape, Krita |
| Hex list | `.hex` | Scripts, code |
| JSON | `.json` | Programmatic use |

---

## UI Layout

```
┌─────────────────────────────────────────────────────┐
│  [Palette Name]          Grid: [4x4 ▼]  [Import] [Export]│
├─────────────────────────────────────────────────────┤
│                                                     │
│   ┌───┐ + ┌───┐ + ┌───┐ + ┌───┐                    │
│   │   │   │   │   │   │   │   │                    │
│   └───┘   └───┘   └───┘   └───┘                    │
│     +       +       +       +                       │
│   ┌───┐ + ┌───┐ + ┌───┐ + ┌───┐                    │
│   │   │   │   │   │   │   │   │                    │
│   └───┘   └───┘   └───┘   └───┘                    │
│                                                     │
│   Click color → edit popup                          │
│   Click + → add interpolated color                  │
│   Drag to rearrange                                 │
│                                                     │
└─────────────────────────────────────────────────────┘

Color Edit Popup (appears on color click):
┌─────────────────────────────┐
│  #FF5733            [×]     │
├─────────────────────────────┤
│  ┌─────────────────┐        │
│  │  Color Picker   │        │
│  └─────────────────┘        │
│  H: [====●===] 15°          │
│  S: [======●=] 79%          │
│  L: [====●===] 60%          │
│  Hex: [#FF5733]             │
├─────────────────────────────┤
│  [Delete] [Duplicate]       │
└─────────────────────────────┘
```

**Grid Options:** 8×1, 8×2, 4×4, 8×4, 8×8, 16×8, 16×16, 32×4 (cols×rows)

**Interpolation:** Clicking the + between two colors inserts a new color that's the midpoint (in HSL space). Works both horizontally and vertically in grid.

---

## Technical Spec

### Stack

- **Single HTML file** with embedded CSS/JS
- **No framework** — vanilla JS
- **Dependencies:**
  - JSZip (for .swatches export) — CDN link
  - No build step

### Data Model

```javascript
palette = {
  name: "My Palette",
  colors: [
    { hex: "#FF5733", name: "Coral" },  // name optional
    { hex: "#3498DB" },
    ...
  ]
}
```

### File Formats

**PNG Strip:**
- 1 pixel height, N pixels width (one per color)
- Or 32px squares in a row for visibility

**.swatches (Procreate):**
- ZIP file containing:
  ```
  Swatches.json  →  [{ "hue": 0.1, "saturation": 0.8, "brightness": 0.6, "alpha": 1 }]
  name.txt       →  "Palette Name"
  ```

**.ase (Adobe Swatch Exchange):**
- Binary format
- Header + color entries (RGB float values)

**.gpl (GIMP Palette):**
```
GIMP Palette
Name: My Palette
#
255  87  51	Coral
 52 152 219	Blue
```

**.hex:**
```
FF5733
3498DB
```

**JSON:**
```json
{
  "name": "My Palette",
  "colors": ["#FF5733", "#3498DB"]
}
```

---

## Color Extraction Algorithms

### K-Means (Current)
1. Load image to canvas
2. Sample pixels (every Nth pixel for performance)
3. Run k-means with user-specified k (default 8)
4. Return cluster centers as palette colors
5. Sort by luminance (dark to light)

**Pros:** Fast, captures dominant colors well
**Cons:** Rare but visually important colors get averaged away

### Outlier-Aware Extraction (Planned)
Designed to catch rare but distinct colors (e.g., bright red lipstick in a muted portrait).

**Approach options:**
1. **Two-pass method:** Run k-means first, then scan for pixels far from all cluster centers. Add those as bonus colors.
2. **Color histogram peaks:** Build histogram in HSL space, find peaks (even small ones) that are isolated in hue.
3. **Saturation/brightness outliers:** After k-means, scan for pixels with unusually high saturation or brightness that weren't captured.

**UI:** Toggle in extraction dialog: "Include rare colors" checkbox

---

## Build Phases

### Phase 1: Core (MVP)
- [x] Grid layout with configurable dimensions
- [x] Add color (button + color picker)
- [x] Delete color
- [x] Color edit popup (picker + hex input)
- [x] Export as PNG strip
- [x] Export as .hex

### Phase 2: Import + Interpolation
- [x] Import PNG strip
- [x] Import .hex file
- [x] Image → k-means extraction
- [x] Add interpolated color between two colors (horizontal + vertical)

### Phase 3: Full Edit
- [x] Drag and drop reorder
- [x] HSL adjustment sliders
- [x] Duplicate color
- [x] Name palette (was already in Phase 1)

### Phase 4: All Exports
- [x] .swatches (Procreate)
- [x] .ase (Adobe)
- [x] .gpl (GIMP)
- [x] JSON

### Phase 5: Polish & UX
- [x] Keyboard shortcuts (copy, paste, delete, select all)
- [x] Multi-selection (shift+click, cmd+click)
- [x] Multi-delete
- [x] Symmetrical grid spacing
- [ ] Undo/redo
- [ ] Local storage save
- [ ] Responsive color squares (expand to fill space)

### Phase 6: Advanced Extraction
- [ ] "Rare color" extraction mode — catch distinct outlier colors (e.g., bright red lipgloss in a muted photo) that k-means would average away
- [ ] Extraction algorithm options: K-means (current) vs Outlier-aware

### Phase 7: Export Fixes
- [ ] Debug & fix Procreate .swatches export
- [ ] Debug & fix Adobe ASE export

### Future Ideas
- [ ] Dark/light theme toggle
- [ ] Color count display
- [ ] Sort colors (by hue, saturation, lightness)
- [ ] Color harmony suggestions
- [ ] Import from URL (image or palette)
- [ ] Shareable palette links
- [ ] Order palette based on tonality

---

## Current Status

**Phases 1-5 complete.** Core functionality working:
- Grid layout with 8 size options
- Full CRUD: add, edit, delete, duplicate, reorder (drag)
- HSL sliders + color picker + hex input
- Interpolation between colors (horizontal + vertical)
- Import: PNG strip, .hex file, k-means image extraction
- Export: PNG, .swatches (Procreate), .ase (Adobe), .gpl (GIMP), .hex, JSON

**Testing needed:** Procreate and ASE exports (rewritten, awaiting verification)

---

## File Structure

```
Palette Tool/
├── Overview.md          # This file
├── index.html           # The app (single file)
└── test-palettes/       # Sample palettes for testing
    └── splendor128.png
```

---

## Notes

- Keep it under 50KB total (excluding JSZip CDN)
- Mobile-friendly (responsive grid)
- No server needed — pure client-side

---

## Keyboard Shortcuts (Planned)

| Key | Action |
|-----|--------|
| `Cmd/Ctrl + C` | Copy selected color(s) hex to clipboard |
| `Cmd/Ctrl + V` | Paste hex from clipboard as new color |
| `Delete / Backspace` | Delete selected color(s) |
| `Cmd/Ctrl + D` | Duplicate selected |
| `Cmd/Ctrl + Z` | Undo |
| `Cmd/Ctrl + Shift + Z` | Redo |
| `Escape` | Deselect / close popup |
| `Cmd/Ctrl + A` | Select all |

---

## Multi-Selection (Planned)

**Selection methods:**
- `Click` — Select single color (deselects others)
- `Shift + Click` — Range select (from last selected to clicked)
- `Cmd/Ctrl + Click` — Toggle individual selection (add/remove from selection)
- `Cmd/Ctrl + A` — Select all

**Multi-selection actions:**
- Delete all selected
- Copy all selected (as hex list)
- Drag to move group
- Adjust HSL (apply delta to all selected)

**UI indication:** Selected colors get a colored border/glow. Selection count shown in status area.

---

## Log

*Add entries below, newest first*

**2025-01-24** — Phase 5 complete: Multi-selection, keyboard shortcuts (copy/paste/delete/select all), fixed grid spacing
**2025-01-24** — Procreate export still broken, deprioritized to Phase 7
**2025-01-24** — Rewrote Procreate/ASE exports (testing pending). Documented Phase 5-6 plans.
**2025-01-24** — Phase 4 complete: All exports (.swatches, .ase, .gpl, JSON)
**2025-01-24** — Phase 3 complete: Drag reorder, HSL sliders, duplicate color
**2025-01-24** — Phase 2 complete: Import (PNG, .hex, k-means extraction), interpolation buttons
**2025-01-24** — Phase 1 complete: Grid layout, color editing, PNG/.hex export
**2025-01-24** — Project created, spec drafted
