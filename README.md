# Grid Morph

A uniform grid of rounded rectangles that resolves into a silhouette: cells deep inside the shape
become large squares, cells on the outline shrink into circles, and cells outside vanish.

No build step, no dependencies. Open `index.html` in a browser.

```
open index.html
```

## Usage

- Type into **Text instead** for an instant silhouette (defaults to `A`), or drop an SVG / PNG onto
  the canvas.
- **Morph** animates from the full grid to the silhouette; **Reset grid** animates back;
  **Loop** ping-pongs between the two.
- `sample.svg` is a black-on-white test file.

Black-on-white art is expected. For light-on-dark art, tick **Invert**.

## How it works

1. The source is rasterised to an offscreen canvas at ~420px on the short side, then thresholded on
   luminance into a binary ink mask.
2. Two exact Euclidean distance transforms (Felzenszwalb–Huttenlocher, `distanceTo`) give the
   distance to the nearest ink pixel and to the nearest background pixel. Combined, they form a
   signed distance field: negative inside the shape, positive outside.
3. Each cell bilinearly samples the field at its centre and normalises that distance by the cell
   pitch, so the look is resolution-independent. From that single number the cell derives its size
   (`pow(coverage, falloff)`), its corner radius (interpolating toward a full circle at the
   outline), and — with **Align to edge** on — a rotation from the field gradient plus a stretch
   along the edge tangent.
4. Every cell is appended to one `Path2D` and filled in a single draw call, which keeps tens of
   thousands of cells cheap.

Using a distance field rather than raw ink coverage is what makes the transition band read
smoothly: coverage saturates a pixel inside the shape, whereas distance keeps increasing, so the
size ramp stays continuous however far a cell sits from the outline.

## Controls

| Control | Effect |
| --- | --- |
| Columns / Rows | Grid density |
| Gap | Spacing between cells, as a fraction of the pitch |
| Roundness | Corner radius deep inside the shape (0.5 = circle) |
| Edge band | Width of the transition band, in cell pitches |
| Size falloff | Exponent on the size ramp; higher = tighter, harder edge |
| Max size | Size of cells deep inside the shape |
| Wobble | Value-noise jitter on cell size for an analog feel |
| Edge stretch | Elongation along the edge tangent (needs Align to edge) |
| Duration / Stagger / Stagger order | Animation timing and the order cells fire in |
| Paper grain | Multiply-blended noise overlay for a printed look |

## Notes

- Grids above roughly 150×150 (>20k cells) stay interactive but start to cost a few milliseconds
  per frame. For much larger counts, move the field sampling into a WebGL fragment shader and draw
  the cells as instanced quads.
- Changing the source, invert, or the grid size rebuilds the distance field, which is a synchronous
  pass over the raster; at the default resolution it takes a few milliseconds.
