# Multi-Shade Graph

A **Cartesian plane** applet for placing **two full lines** through integer lattice points, then **toggling shaded regions** formed by those lines. There is **no question bank, grading, or submit flow**—it is an open manipulation surface (exploration / demonstration).

**Live build:** [https://content-interactives.github.io/multi_shade_graph/](https://content-interactives.github.io/multi_shade_graph/)

---

## Stack

| Layer | Technology |
|--------|------------|
| Build | [Vite](https://vite.dev/) 7 (`@vitejs/plugin-react`) |
| UI | [React](https://react.dev/) 19 |
| Styling | [Tailwind CSS](https://tailwindcss.com/) 3 (`index.css`); `App.css`; `glow.css` (segmented Undo / Redo / Reset control) |
| Lint | ESLint 9 |

---

## Repository layout

```
index.html
vite.config.js          # base: '/multi_shade_graph/' (GitHub Pages)
tailwind.config.js
postcss.config.js
package.json            # dev, build, preview, lint; gh-pages via predeploy/deploy
src/
  main.jsx              # createRoot → <App />
  App.jsx               # Thin wrapper: renders <MultiShadeGraph />
  index.css
  glow.css              # Orbit/glow styles for toolbar buttons
  App.css
  components/
    MultiShadeGraph.jsx # Entire interactive: SVG, state, geometry, interactions
```

---

## Behavior (`MultiShadeGraph.jsx`)

### Coordinate system

- **Canvas:** 500×500 SVG; plot domain **x, y ∈ [-10, 10]** (ticks at integers). Axes extend one unit past ±10 for arrowheads (`EXTENDED_MIN` / `EXTENDED_MAX`).
- **Mapping:** `valueToX` / `valueToY` center the plot with padding; **y** increases upward in value space and **downward** in SVG space.

### Line placement

- User clicks **twice** per line. Each click snaps to the **nearest integer** coordinates (`roundToTick` after `xToValue` / `yToValue`).
- **`REQUIRED_LINES` / `MAX_LINES` = 2:** only two boundary lines are supported. While the second line is animating, only one completed line is shown (`completedLines` slicing).
- After two clicks, a **`requestAnimationFrame`** loop runs **`LINE_ANIMATION_DURATION_MS` (1500)** to grow the segment from the midpoint toward both clipped endpoints; then the segment is appended to **`history`** and **`historyIndex`** advances. New lines reset **`shadingHistory`** to `[[]]`.

### Line rendering

- Each segment is **extended to the SVG rectangle** with **`clipLineToRect`** (parametric intersection with the viewport), ordered with **`orderEndpointsByPoints`**, then drawn with triangular **arrowheads** (`arrowPoints`).

### Shading (after both lines exist)

- Clicks use **2D cross products** relative to each line to classify the pointer into one of **four half-plane combinations** (`side1`, `side2` ∈ {1, -1}), encoded as region keys `"1,1"`, `"1,-1"`, etc.
- Clicks **on** either line (near-zero cross) are ignored.
- **`getRegionPolygon`** builds the visible polygon: half-plane of line 1 (`getHalfPlanePolygon`), then **`clipPolygonToHalfPlane`** with line 2. Active regions render as semi-transparent blue polygons (`#a8d4f0`, opacity 0.5).
- **`shadingHistory`** is an array of region sets; **`shadingIndex`** supports undo/redo over shading toggles (**`MAX_UNDO` = 9** caps memory).

### Toolbar

- **Undo:** In-progress line → remove last point; after both lines → step shading back; otherwise peel last completed line and repopulate first endpoint as a new start.
- **Redo:** Restore undone line commit or advance shading index.
- **Reset:** Clear line history, in-progress points, and shading.
- **`showHistoryGlow`:** Dims the orbit glow on `glow.css` after first toolbar use.

### Accessibility / input

- The graph `div` uses **`role="button"`** and **`tabIndex={0}`**; interaction is **click + hover** (preview circle) only—no keyboard drawing.

---

## `App.jsx`

Renders a single **`MultiShadeGraph`** inside `<div className="App">`. No props, context, or global state.

---

## Scripts

| Command | Purpose |
|---------|---------|
| `npm install` | Install dependencies |
| `npm run dev` | Vite dev server |
| `npm run build` | Output to `dist/` |
| `npm run preview` | Serve `dist/` locally |
| `npm run lint` | ESLint |
| `npm run deploy` | Build then `gh-pages -d dist` |

Update **`base`** in `vite.config.js` if the deployment path changes.

---

## Embedding

- Fixed **500×500** graph; surrounding page should allow the bordered box plus top-right toolbar.
- No `postMessage` or LMS integration.

---

## Extension notes

- More than two lines would require generalizing region keys (currently four keys from two lines), polygon intersection, and UI flow (`REQUIRED_LINES`, `MAX_LINES`, history caps).
- Changing **`MIN`/`MAX`** updates scale and tick generation; axis label positions hard-code `10` in a few `<text>` nodes—adjust if the domain changes.
