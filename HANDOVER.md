# Cakemapper Handover Document

## Project Summary
Cakemapper is a single-file HTML web app for creating colored US state maps. Users pick colors, click states, add legends, and export their maps. It runs entirely client-side via GitHub Pages.

- **Live site**: https://mapzimus.github.io/cakemapper/
- **Repo**: https://github.com/mapzimus/cakemapper (branch: `master`)
- **Local path (Windows)**: `C:\Users\mhowe\cakemapper\index.html`
- **Single file**: `index.html` (~2615 lines, all CSS/HTML/JS inline)

## Current State (as of commit 7f3dded)
The app is fully functional with all core features working:

- 50 US states + Puerto Rico colorable (DC visible but non-colorable)
- 24-color palette with custom color support
- Legend builder, quick fill by region, undo, dark/light themes
- Export to PNG, SVG, GeoJSON, CSV; import from CSV
- Shareable URL encoding, save/load config files
- County-level view (Pro feature, unlock codes: CAKEPRO2026 or cakemapper)
- Editable title/subtitle, north arrow, scale bar with position controls
- Ocean background toggle (CSS-based, no visible partition)

## Architecture Overview
Everything lives in one HTML file. No build tools, no framework. External dependencies loaded via CDN: `topojson-client@3` for parsing TopoJSON, `html2canvas` for PNG export. Map data comes from `us-atlas@3` (pre-projected Albers USA).

The SVG uses a viewBox of `-20 -30 1010 710`. State paths are rendered inside a `<g>` with `transform="translate(5, -20) scale(0.95)"` which offsets the pre-projected coordinates. Labels use geometric centroid of the largest polygon for positioning, with fine-tune offsets for ~14 irregular states.

## Key Code Sections (line numbers approximate)

| Section | Lines | Description |
|---------|-------|-------------|
| CSS styles | 7-520 | All styling, dark/light mode, sidebar, map, modals |
| Sidebar HTML | 521-984 | Controls: palette, legend, display, quick fill, export |
| SVG + map HTML | 985-1030 | SVG element, statesGroup, PR group, north arrow, scale bar |
| Modals | 1048-1095 | Upgrade, county select, clear confirm |
| appState | 1106-1156 | Central state object with all app data |
| init/loadUSMap | 1162-1186 | Initialization and TopoJSON loading |
| fipsToState | 1189-1204 | FIPS code to state name mapping |
| renderStatesFromTopology | 1206-1330 | Main render function with centroid label positioning |
| setupPuertoRico | 1332-1371 | PR path setup with scaled label |
| Color/interaction | 1444-1560 | Palette, click handlers, tooltips |
| Legend | 1565-1648 | Legend builder and display |
| Quick fill | 1654-1712 | Select all, regions, invert, clear |
| URL state | 1740-1798 | Base64 encode/decode for shareable URLs |
| Export/Import | 1804-2235 | PNG, SVG, GeoJSON, CSV export; CSV import |
| County view | 2240-2442 | Pro county mode with zoom and back navigation |
| Event listeners | 2448-2597 | All UI event bindings |

## Technical Decisions and Gotchas

1. **statesGroup transform**: The `translate(5,-20) scale(0.95)` is critical. County view resets it to `''` and restores it on exit. If you forget to restore it, all state label positions break.

2. **Color via style.fill**: CSS `.state-path` sets default fill. Coloring must use `element.style.fill` (inline style) to override CSS. `setAttribute('fill')` will NOT work.

3. **Puerto Rico**: Not in us-atlas data. Manually placed as a separate SVG group with `scale(0.45)`. Its label uses `font-size="22"` to render at ~10px after scaling.

4. **DC non-colorable**: Uses a `nonColorable` Set. DC is excluded from click listeners, Select All, region fills, Invert, and the stats count. It still renders on the map.

5. **Ocean background**: Implemented as CSS class (`ocean-on`) on map-container, NOT as an SVG rect. This prevents visible partition lines between SVG viewport and page background.

6. **Git on Windows CMD**: Commit messages with spaces break. Write message to file, use `git commit -F filename`, then delete the file.

7. **GitHub Pages caching**: Hard refresh (Ctrl+Shift+R) needed after deploy to see changes. Deploys take ~15-30 seconds after push.

8. **Scale bar positioning**: Text extends +14px below the bar's y position. Max y for bottom positions is 645 to keep text within viewBox (max y = 680).

## Commit History
```
7f3dded Fix WA/MA labels, remove background partition by moving ocean to CSS
a834229 Remove basemap, fix label centering with geometric centroids
8ad52e2 Make DC non-colorable and fix count to 51
1ced28e Fix basemap with real geographic data and fix state count text
1cb2a15 Fix county view centering, scale bar clipping, PR label, and UI bugs
4794192 Make Puerto Rico larger and more visible on the map
b4270a6 Add county view, CSV import/export, basemap, Puerto Rico, more exports
5f38b81 Fix MD/DE labels, UI polish
dc29f93 Label contrast, NE cluster fix, remove DC, hide subtitle, better hover
da37b1e Fix FL label, enlarge north arrow and scale bar
e71f4a9 Fix FL label, add north arrow and scale bar toggles
2b6fc60 Add title color, north arrow, scale bar, watermark, fix labels
a60e8f4 Fix export: hide stats bar, prevent AK/HI clipping
987ed29 Fix color fills, label positions, add label toggle
bba059f Fix critical bugs: FIPS mapping, SVG sizing, tooltip shaking, color selection
bd45e26 Initial release of Cakemapper
```

## Known Issues / Future Work

- **County tooltips**: Show FIPS codes instead of county names (us-atlas counties lack name properties; would need a separate FIPS-to-name lookup)
- **Puerto Rico shape**: Uses a simplified hand-drawn SVG path, not real geographic data
- **Small state labels**: CT, RI, NJ, DE, MD labels crowd in the northeast at small viewports
- **Mobile**: Not optimized for mobile/touch (sidebar takes full width, no responsive layout)
- **Pro system**: Unlock codes are hardcoded client-side (no real payment/auth)
- **index_b64.txt**: Unused legacy file in repo, can be deleted

## How to Continue Development

1. Clone/pull the repo: `git clone https://github.com/mapzimus/cakemapper.git`
2. Edit `index.html` directly - it's the only file that matters
3. Test locally by opening `index.html` in a browser (most features work, but CORS may block TopoJSON fetch from CDN - use a local server like `python -m http.server`)
4. Commit and push to master for GitHub Pages deployment
5. The `.claude/CLAUDE.md` file in the repo contains detailed technical context for AI assistants

## File Structure
```
cakemapper/
  index.html          # The entire app (2615 lines)
  index_b64.txt       # Unused legacy file
  .claude/
    CLAUDE.md          # AI assistant context for this project
  HANDOVER.md          # This document
```
