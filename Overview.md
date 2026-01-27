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
- [x] Responsive color squares (expand to fill space)
- [ ] Remove Overview file from Github. Unnecessary.

### Phase 6: Advanced Extraction
- [x] "Rare color" extraction mode — catch distinct outlier colors (e.g., bright red lipgloss in a muted photo) that k-means would average away
- [x] Extraction algorithm options: K-means (current) vs Outlier-aware

### Phase 7: Export Fixes
- [x] Debug & fix Procreate .swatches export
- [x] ~~Debug & fix Adobe ASE export~~ — Removed (not needed)

### Future Ideas
- [ ] Dark/light theme toggle
- [ ] Local storage save
- [ ] Color count display
- [x] Sort colors (by hue, saturation, lightness)
- [ ] Color harmony suggestions
- [ ] Import from URL (image or palette)
- [ ] Shareable palette links
- [ ] Order palette based on tonality
- [ ] Row/column hover controls:
  - Bottom: button to add new row of empty tiles
  - Right: button to add new column of empty tiles
  - Top: small × per column to delete that column
  - Left: small × per row to delete that row

---

## Current Status

**Phases 1-7 complete.** App is fully usable:
- Grid layout with 8 size options (8×1 to 32×4)
- Full CRUD: add, edit, delete, duplicate, reorder (drag)
- HSL sliders + color picker + hex input
- Interpolation between colors (+ buttons)
- Multi-selection with keyboard shortcuts
- Import: PNG strip, .hex file, k-means image extraction (standard + outlier-aware modes)
- Export: PNG, .gpl (GIMP), .hex, JSON, .swatches (Procreate) — all working

**Repo:** https://github.com/luis-alva/palette-tool

---

## Next Session — Pending Items

### High Priority
1. ~~**Fix Procreate .swatches export**~~ — Done!
2. ~~**Fix Adobe ASE export**~~ — Removed (not needed)

### UI/UX — Completed
- [x] **Cmd+D duplicate shortcut** — duplicates selected colors
- [x] **Sort colors** — dropdown to sort by hue/saturation/lightness/luminance
- [x] **Responsive color squares** — grid uses flexible sizing, expands to fill container
- [x] **Drag to expand view** — container is resizable with drag handle
- [x] **Standardize color picker** — HSL sliders are primary, system picker is secondary "Pick" button
- [x] **Color name dictionary** — 1,186 names from Wikipedia, shows nearest match in edit popup

### UI/UX — Fixes (Completed)
- [x] **Fix Y-axis resize** — grid now uses `grid-template-rows: 1fr` for vertical sizing
- [x] **Sort dropdown persist selection** — removed reset, shows current sort method
- [x] **HSL/RGB toggle** — button with ⇄ arrows toggles between HSL and RGB sliders
- [x] **Color preview opens picker** — click the large color rectangle to open system picker
- [x] **Color name autocomplete** — editable field with dropdown (up to 5 suggestions)
- [x] **Squares toggle** — checkbox top-right of grid to lock tiles to square aspect ratio (default: on)
- [x] **Color picker position fix** — edit popup aligned left to give room for picker
- [x] **Grid resize full width** — removed max-width limit, now uses viewport width
- [x] **Consistent grid gaps** — squares mode uses auto row height, free mode uses 1fr
- [x] **Interpolation buttons inside tiles** — buttons now positioned inside cells, not in gap

### Bugs to Fix
~~3. **Interpolation button position** — FIXED: buttons now centered in the gap between tiles~~

### Core Features (Pending)
4. ~~**Outlier-aware color extraction**~~ — Done!
5. **Undo/redo** — track history stack

### Nice to Have (Pending)
5. **Drag to move selected group**
6. **Batch HSL adjust** on multi-selection
7. **Color count display**
8. **Local storage save** — persist palette between sessions

---

## File Structure

```
Palette Tool/
├── Overview.md          # This file
├── index.html           # The app (single file)
├── color-names.json     # 1,186 color names from Wikipedia (reference)
├── color-names.min.json # Compact version (46KB, embedded in HTML)
└── test-palettes/       # Sample palettes for testing
    └── splendor128.png
```

---

## Notes

- Keep it under 50KB total (excluding JSZip CDN)
- Mobile-friendly (responsive grid)
- No server needed — pure client-side
- **Git commits:** Do not add Co-Authored-By lines or email addresses

---

## Keyboard Shortcuts

| Key | Action | Status |
|-----|--------|--------|
| `Cmd/Ctrl + C` | Copy selected color(s) hex to clipboard | ✓ |
| `Cmd/Ctrl + V` | Paste hex from clipboard as new color | ✓ |
| `Delete / Backspace` | Delete selected color(s) | ✓ |
| `Cmd/Ctrl + A` | Select all | ✓ |
| `Escape` | Deselect / close popup | ✓ |
| `Cmd/Ctrl + D` | Duplicate selected | ✓ |
| `Cmd/Ctrl + Z` | Undo | Pending |
| `Cmd/Ctrl + Shift + Z` | Redo | Pending |

---

## Multi-Selection

**Selection methods (all implemented ✓):**
- `Click` — Select single color, open edit popup
- `Shift + Click` — Range select
- `Cmd/Ctrl + Click` — Toggle add/remove from selection
- `Cmd/Ctrl + A` — Select all

**Multi-selection actions:**
- ✓ Delete all selected
- ✓ Copy all selected (as hex list)
- Pending: Drag to move group
- Pending: Batch HSL adjust

**UI indication:** Selected colors get purple border/glow.

---

## Log

*Add entries below, newest first*

**2025-01-27** — Fixed Procreate .swatches export (correct JSON structure), removed broken ASE export, added outlier-aware color extraction mode
**2025-01-25** — Fixed: Consistent grid gaps (squares vs free mode), interpolation buttons now inside tiles
**2025-01-25** — Fixed: Grid overflow on resize, squares toggle moved to top-right, removed max-width limit, edit popup positioned left for color picker room, HSL/RGB button now shows ⇄ arrows
**2025-01-25** — Added: Squares toggle (locks tiles to square aspect ratio), fixed color picker position overlay
**2025-01-25** — Fixed: Y-axis resize, sort dropdown persists, HSL/RGB toggle, color preview opens picker, color name autocomplete
**2025-01-25** — Added Cmd+D duplicate, sort colors dropdown, responsive grid with resize handle, standardized color picker (HSL primary), color name dictionary (1,186 Wikipedia colors)
**2025-01-24** — Git repo initialized, pushed to GitHub (luis-alva/palette-tool)
**2025-01-24** — Phase 5 complete: Multi-selection, keyboard shortcuts (copy/paste/delete/select all), fixed grid spacing
**2025-01-24** — Procreate export still broken, deprioritized to Phase 7
**2025-01-24** — Rewrote Procreate/ASE exports (testing pending). Documented Phase 5-6 plans.
**2025-01-24** — Phase 4 complete: All exports (.swatches, .ase, .gpl, JSON)
**2025-01-24** — Phase 3 complete: Drag reorder, HSL sliders, duplicate color
**2025-01-24** — Phase 2 complete: Import (PNG, .hex, k-means extraction), interpolation buttons
**2025-01-24** — Phase 1 complete: Grid layout, color editing, PNG/.hex export
**2025-01-24** — Project created, spec drafted
