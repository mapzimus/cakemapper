# Cakemapper - Project Context

## What This Is
Cakemapper is a single-file HTML web app for creating colored US state/territory maps. Hosted on GitHub Pages at `https://mapzimus.github.io/cakemapper/`. Repo: `mapzimus/cakemapper` on `master` branch.

## Architecture
- **Single file**: Everything is in `index.html` (~2615 lines) - all CSS, HTML, JS inline
- **No build step**: Edit index.html, commit, push to master, GitHub Pages auto-deploys
- **External CDNs**: topojson-client@3, html2canvas 1.4.1
- **Data source**: `us-atlas@3/states-albers-10m.json` (pre-projected Albers USA, FIPS codes as feature.id)
- **County data**: `us-atlas@3/counties-albers-10m.json` (loaded on demand for Pro county view)

## Key Technical Details

### SVG Structure
```
<svg viewBox="-20 -30 1010 710" preserveAspectRatio="xMidYMid meet">
  <g id="statesGroup" transform="translate(5, -20) scale(0.95)">
    <!-- state paths + labels rendered here by JS -->
  </g>
  <g id="puertoRicoGroup" transform="translate(540, 610) scale(0.45)">
    <!-- manually positioned PR path + label -->
  </g>
  <g id="northArrow" .../>
  <g id="scaleBar" .../>
</svg>
```

### CRITICAL: statesGroup Transform
The `translate(5, -20) scale(0.95)` on statesGroup is load-bearing. It offsets the pre-projected Albers coordinates to fit the viewBox. In county mode, this transform is RESET to `''` (otherwise getBBox calculations are distorted), then RESTORED when returning to state view.

### Label Positioning
Labels use geometric centroid of the largest polygon (not bounding box center). This correctly handles irregular states like FL (peninsula vs panhandle), MI (lower vs upper peninsula), OK (panhandle). A `labelOffsets` object provides small dx/dy fine-tuning for ~14 states.

### Color Application
MUST use `element.style.fill` (not `setAttribute('fill')`) due to CSS specificity. The `.state-path` CSS class sets a default fill, and inline style overrides it.

### State Data
- 52 entries in `stateAbbreviations` (50 states + DC + PR)
- DC is non-colorable (in `nonColorable` Set) - no click listeners, excluded from counts
- Total colorable: 51 (50 states + PR)
- FIPS codes map states: `fipsToState` object (e.g., '01'='Alabama', '72'='Puerto Rico')
- PR is NOT in us-atlas data - manually positioned as separate SVG group

### Ocean Background
The ocean effect is CSS-based (`.map-container.ocean-on` class), not an SVG element. This avoids visible partition lines between the SVG viewport and surrounding page.

### County View (Pro Feature)
- Requires Pro unlock (codes: `CAKEPRO2026` or `cakemapper`)
- Loads counties-albers-10m.json, filters by state FIPS prefix
- Resets statesGroup transform, hides PR/north arrow/scale bar
- Zooms viewBox to state bounds with padding
- `backToStateView()` restores everything

## appState Properties
```javascript
selectedColor, stateColors, stateLabels, legendEntries,
mapTitle, mapSubtitle, titleColor, bgColor, scaleUnit,
legendPosition, arrowPosition, scalePosition, showBasemap,
history, darkMode, showLabels, proUnlocked, topologyData,
urlUpdateTimeout, stateAbbreviations, regions, presetColors
```

## Features
- Click to color states, right-click to clear
- 24-color palette + custom hex/picker + "Add Custom" button
- Legend builder with auto-categorization
- Quick fill: Select All, regions (NE/MW/S/W), Invert
- Undo (Ctrl+Z), Clear All with confirmation
- Dark/light theme toggle (persisted in localStorage)
- Editable title/subtitle with color picker
- Background color picker
- North arrow (4 positions), scale bar (3 positions, imperial/metric)
- Show/hide: labels, ocean background
- Export: PNG (with watermark), SVG, GeoJSON, CSV
- Import: CSV with auto-category-to-color mapping
- Share via URL (base64 encoded state in hash)
- Save/load JSON config files
- County view (Pro) with state selection modal
- Puerto Rico as interactive territory
- Stats bar with progress and legend breakdown

## Git Workflow (Windows CMD)
Commit messages with spaces FAIL in CMD. Use file-based approach:
```cmd
echo Your commit message here> commit_msg.txt
git commit -F commit_msg.txt
del commit_msg.txt
```

## Development Notes
- GitHub Pages caching: use Ctrl+Shift+R to see latest changes
- The `index_b64.txt` file in the repo is unused legacy
- All state paths are rendered in two passes: paths first, then labels on top (ensures labels are never hidden behind paths)
- PR label uses `font-size="22"` because parent group scales at 0.45x (renders at ~10px)
- Scale bar positioned at y=645 max (text extends +14px below, must stay within viewBox)
