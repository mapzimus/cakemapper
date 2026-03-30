# Mapparatus Handover Document

## Project Summary
Mapparatus (formerly Cakemapper) is a single-file HTML web app for creating colored US state and county maps. Users pick colors, click states, build legends, and export publication-ready maps. It runs entirely client-side, currently hosted on GitHub Pages with a custom domain.

- **Live site**: https://mapparatus.org (GitHub Pages with custom domain)
- **Repo**: https://github.com/mapzimus/cakemapper (branch: `master`, to be renamed `mapparatus`)
- **Local path (Windows)**: `C:\Users\mhowe\cakemapper\index.html`
- **Single file**: `index.html` (~2825 lines, all CSS/HTML/JS inline)
- **Domain**: mapparatus.org (registered via Squarespace/Google Domains)
- **Email**: max@mapparatus.org (Google Workspace)

## Current State
The app is fully functional and rebranded as Mapparatus with tiered subscription model (Free/Pro/Enterprise):

- 50 US states colorable (DC visible but non-colorable by design)
- Puerto Rico removed entirely
- 24-color palette with custom color support (12 colors on Free tier)
- Legend builder, quick fill by region, undo, dark/light themes
- Export to PNG and SVG (no GeoJSON/CSV export)
- Pro unlock codes: `MAPPRO2026` or `mapparatus`
- County-level view (Pro feature)
- State zoom for individual state county maps (Pro feature)
- Editable title/subtitle, north arrow, scale bar with position controls
- Ocean background toggle (CSS-based)
- Inline SVG logo watermark on map (always on for Free; Pro users can hide)
- Mobile-friendly export (detects mobile UA, opens image for long-press save)
- Responsive layout (breakpoints at 900px and 600px)
- Brand colors: teal (#2B7A8C primary, #3A9BB0 hover, #1D5A6A dark)
- Small NE state labels use leader lines (DE, CT, RI, MD)
- Shareable URL encoding, save/load config files (Pro)

## Architecture Overview
Everything lives in one HTML file. No build tools, no framework. External dependencies loaded via CDN: `topojson-client@3` for parsing TopoJSON, `html2canvas` for PNG export. Map data comes from `us-atlas@3` (pre-projected Albers USA).

The SVG uses a viewBox of `-20 -30 1010 710`. State paths are rendered inside a `<g>` with `transform="translate(5, -20) scale(0.95)"` which offsets the pre-projected coordinates. Labels use bbox center (`getBBox()`) with hand-tuned manual offsets in the `labelOffsets` object.

## Key Code Sections (line numbers approximate)

| Section | Description |
|---------|-------------|
| CSS styles (~7-860) | All styling, dark/light mode, sidebar, map, modals, responsive media queries |
| Sidebar HTML | Controls: palette, legend, display, quick fill, export, logo toggle |
| SVG + map HTML | SVG element, statesGroup, north arrow, scale bar, logo watermark |
| Modals | Upgrade, county select, clear confirm |
| appState | Central state object with all app data |
| init/loadUSMap | Initialization and TopoJSON loading |
| fipsToState | FIPS code to state name mapping (50 states + DC, no PR) |
| renderStatesFromTopology | Main render with centroid labels + leader lines for NE states |
| isPro() | Returns `appState.proUnlocked` — used by export and logo toggle |
| Color/interaction | Palette, click handlers, tooltips (DC shows "not colorable") |
| Legend | Legend builder and display |
| Quick fill | Select all, regions, invert, clear |
| URL state | Base64 encode/decode for shareable URLs |
| Export (PNG/SVG) | PNG via html2canvas, SVG via XMLSerializer; mobile detection for camera roll |
| County view | Pro county mode with zoom, tooltips show "Name County, State" |
| State zoom | Pro feature for individual state county maps |
| Event listeners | All UI event bindings incl. leader line toggle, logo toggle |

## Technical Decisions and Gotchas

1. **statesGroup transform**: `translate(5,-20) scale(0.95)` is critical. County view resets it to `''` and restores on exit. Forgetting to restore breaks all label positions.

2. **Color via style.fill**: CSS `.state-path` sets default fill. Coloring MUST use `element.style.fill` (inline style). `setAttribute('fill')` will NOT work.

3. **Puerto Rico**: REMOVED. PR SVG group, `setupPuertoRico()`, FIPS entry, abbreviation, and all PR references deleted.

4. **DC non-colorable**: Uses `nonColorable` Set. Excluded from click listeners, Select All, region fills, Invert, and stats count. Tooltip shows "not colorable" note. Label hidden via `labelOffsets` `hide: true`.

5. **Ocean background**: CSS class (`ocean-on`) on map-container, NOT an SVG rect. Prevents visible partition lines.

6. **Git on Windows CMD**: Commit messages with spaces break. Write message to file, use `git commit -F filename`, then delete.

7. **GitHub Pages caching**: Hard refresh (Ctrl+Shift+R) after deploy. Deploys take ~15-30 seconds.

8. **Scale bar**: Text extends +14px below bar's y position. Max y for bottom positions is 645.

9. **SVG namespace**: `innerHTML` on SVG `<g>` elements creates children in the HTML namespace, breaking rendering and export. Use the `setSVGContent()` helper which creates a temp `<svg>` element for proper namespace parsing.

10. **Label positioning**: Labels use bbox center with manual offsets in `labelOffsets`. These are hand-tuned and locked — do NOT change without visual verification. See CLAUDE.md for the full offset table.

11. **Logo watermark**: Inline SVG elements (not external image) at y=580 with scale(0.40). Text splits as "map" (dark teal) + "paratus" (light teal). Logo visibility controlled by `appState.showLogo`; hiding requires Pro.

12. **Mobile export**: Detects mobile via user agent regex. Opens image blob in new tab for long-press camera roll save instead of triggering download (which fails on mobile Safari/Chrome).

## Brand Identity
- **Name**: Mapparatus
- **Tagline**: "simple maps, simple tools"
- **Logo**: Hexagon with map pin icon. Three versions: stacked, horizontal, icon-only
- **Primary color**: Teal (#2B7A8C / dark #1D5A6A)
- **Secondary color**: Light teal (#12A6C8 / #3A9BB0)
- **Typography**: Georgia serif for wordmark, Helvetica Neue for tagline/UI
- **Logo files**: `/assets/logo-horizontal.svg` (others to be added)

## Subscription Tiers

| Feature | Free | Pro ($5/mo) | Enterprise ($20/mo) |
|---------|------|-------------|---------------------|
| State coloring | 50 states | 50 states | 50 states |
| County view | - | Yes | Yes |
| State zoom | - | Yes | Yes |
| Colors | 12 curated | 24 + custom hex | 24 + custom + saved palettes |
| Subtitle | - | Yes | Yes |
| Legend | - | Yes | Yes |
| PNG export | 3/month + watermark | Unlimited, no watermark | Unlimited, white-label |
| SVG export | - | Yes | Yes |
| Logo on map | Always on | Can hide | Can hide |

## Commit History
```
818b4b7 Rebrand to Mapparatus: tier system, state zoom, remove PR/CSV, professional polish
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

- **Small NE state labels**: CT, RI, DE, MD use leader lines but still crowd at very small viewports
- **Mobile**: Responsive layout exists but not fully optimized for touch interactions
- **Pro system**: Unlock codes are hardcoded client-side (no real payment/auth yet)
- **Subscription infrastructure**: Needs Vercel + Supabase/Clerk auth + Stripe (Phase 2)
- **Custom domain**: mapparatus.org configured via Squarespace DNS → GitHub Pages
- **Social media pipeline**: Instagram (@mappratus), TikTok, X accounts created. OpenClaw automation planned (Phase 4)
- **Repo rename**: GitHub repo still named "cakemapper", needs rename to "mapparatus"

## Roadmap

1. **Phase 1** (current): Rebrand + polish — mostly complete, remaining: onboarding UX, repo rename
2. **Phase 2**: Subscription & auth — Vercel hosting, Supabase auth, Stripe payments
3. **Phase 3**: Hosting & domain — Vercel deployment, SSL, DNS finalization
4. **Phase 4**: Social media content engine — OpenClaw automation, idea database, headless Puppeteer export
5. **Phase 5**: Growth & iteration — analytics, A/B testing, gallery, SEO

## How to Continue Development

1. Clone/pull the repo: `git clone https://github.com/mapzimus/cakemapper.git`
2. Edit `index.html` directly — it's the only file that matters
3. Test locally by opening `index.html` in a browser (use `python -m http.server` for CORS)
4. Commit and push to master for GitHub Pages deployment
5. See `.claude/CLAUDE.md` for full technical context, label offsets, and phase details

## File Structure
```
cakemapper/
  index.html              # The entire app (~2825 lines)
  assets/
    logo-horizontal.svg   # Horizontal logo (icon + wordmark)
  .claude/
    CLAUDE.md             # AI assistant context (detailed technical docs + roadmap)
  HANDOVER.md             # This document
```

## Owner
- **Name**: Max
- **Email**: max@mapparatus.org / mhowe.gis@gmail.com
- **GitHub**: mapzimus
