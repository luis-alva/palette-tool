# Palette Tool

Lightweight web app for creating, editing, and exporting color palettes. Single HTML file, no build step, no framework.

## Features

**Create & Edit**
- Add colors via picker, hex input, or random generation
- HSL/RGB sliders with live preview
- Drag-and-drop reorder (single or group)
- Interpolation between adjacent colors (+ buttons)
- Extrapolation at palette edges
- 1,186 named colors from Wikipedia with autocomplete

**Multi-Selection**
- Cmd/Ctrl+click to toggle, Shift+click for range, Cmd+A for all
- Group drag, batch delete, copy/paste, duplicate
- Batch HSL adjust across selection (Cmd+E)

**Import**
- PNG strip (reads pixels, dedupes adjacent)
- .hex file (one hex per line)
- Image color extraction (k-means, with outlier-aware mode)

**Export**
- PNG strip (.png)
- Procreate (.swatches)
- GIMP/Inkscape/Krita (.gpl)
- Hex list (.hex)
- JSON (.json)

**Grid**
- Configurable: 8x1, 8x2, 4x4, 8x4, 8x8, 16x8, 16x16, 32x4
- Add/remove rows and columns on hover
- Square or free aspect ratio toggle
- Resizable container

**Sort**
- By hue (perceptual bucketing), saturation, lightness, or luminance

## Usage

Open `index.html` in a browser. No server needed.

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Cmd/Ctrl + C` | Copy selected hex to clipboard |
| `Cmd/Ctrl + V` | Paste hex as new color(s) |
| `Cmd/Ctrl + D` | Duplicate selected |
| `Cmd/Ctrl + E` | Batch HSL adjust (2+ selected) |
| `Cmd/Ctrl + A` | Select all |
| `Delete` | Delete selected |
| `Escape` | Deselect / close popup |

## Technical Notes

- ~102KB single HTML file (37KB is embedded color dictionary)
- Vanilla JS, no dependencies except JSZip (CDN, for .swatches export)
- Interpolation uses HSL with chroma damping for distant hues
- Color extraction: k-means clustering with optional outlier-aware mode
