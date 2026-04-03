# Mapparatus Handover Document

## Project Summary
Mapparatus (formerly Cakemapper) is a single-file HTML web app for creating colored US state and county maps. Users pick colors, click states, build legends, and export publication-ready maps. It runs client-side with Vercel serverless functions for auth/payments.

- **Live site**: https://mapparatus.org (Vercel with custom domain)
- **Repo**: https://github.com/mapzimus/mapparatus (branch: `master`)
- **Local path (Windows)**: `C:\Users\mhowe\cakemapper\index.html`
- **Single file**: `index.html` (~4070 lines, all CSS/HTML/JS inline)
- **Domain**: mapparatus.org (Squarespace DNS → Vercel)
- **Email**: max@mapparatus.org (Google Workspace)

## Current State (April 2026)

The app is fully functional and rebranded as Mapparatus:

### Core Features
- 50 US states colorable (DC visible but non-colorable by design)
- 24-color default palette with custom color support
- 30 themed palettes in expandable dropdown (Nature, Pastels, Bold, Muted, Fun/Pride/USA, Colorblind)
- Colorblind-safe palette button (Wong 2011, one-click)
- "Reset to Default" palette button (always visible)
- 10 color ramps (7 sequential + 3 diverging) with visual picker, adjustable steps, and reverse (Pro)
- Legend builder with click-to-paint (click legend color → selects as active painting color), edit button for color changes, custom title, 4-position placement
- Quick fill by region, undo, dark/light themes
- Export to PNG and SVG (3 free exports/month with watermark, unlimited for Pro)
- Pro unlock codes: `MAPPRO2026` or `mapparatus`
- County-level view and state zoom (Pro features)
- Editable title/subtitle, north arrow, scale bar with position/style controls
- Ocean background toggle (CSS-based)
- Inline SVG logo watermark (always on for Free; Pro can hide)
- Mobile-friendly export (detects mobile UA, opens image for long-press save)
- Responsive layout (breakpoints at 900px and 600px)
- Shareable URL encoding, save/load config files (Pro)
- Guided onboarding overlay for first-time users
- Brand colors: teal (#2B7A8C primary, #3A9BB0 hover, #1D5A6A dark)

### Infrastructure (Phase 2 & 3 — Partially Complete)
- **Vercel**: Project deployed, custom domain configured, serverless functions working
- **Supabase**: Project created, `user_subscriptions` table with RLS policies
- **Stripe**: Product with two prices ($5/mo, $48/yr), webhook configured
- **3 API routes**: `create-checkout.js`, `verify-subscription.js`, `webhook.js`
- **Auth UI**: Sign up/in/out in upgrade modal, Supabase client via CDN
- **7 env vars**: All set in Vercel dashboard

### What Needs Testing / Fixing
1. **END-TO-END AUTH TEST**: Sign up → confirm email → sign in → Stripe checkout → Pro unlock. This has NOT been tested yet.
2. **Supabase anon key verification**: The key in `index.html` (~line 2537) is `sb_publishable_wRYolYcDWeiUzobTmaCnrQ__J0quzOq`. Standard Supabase anon keys are long JWTs starting with `eyJ...`. Check Supabase dashboard → Settings → API → `anon public` key.
3. **CORS**: API routes have CORS hardcoded to `https://mapparatus.org`. Won't work from localhost.

## Architecture Overview

Everything lives in one HTML file plus 3 Vercel serverless functions. No build tools, no framework.

**External dependencies** (CDN):
- `@supabase/supabase-js@2.43.4` — Auth client (UMD in `<head>`)
- `topojson-client@3` — TopoJSON parsing
- `html2canvas` — PNG export

**Map data**: `us-atlas@3` (pre-projected Albers USA)

**SVG setup**: viewBox `-20 -30 1010 710`, statesGroup has `transform="translate(5, -20) scale(0.95)"`

## File Structure
```
mapparatus/
  index.html                      # The entire app (~4070 lines)
  api/
    stripe/
      create-checkout.js          # Stripe Checkout session creation
      verify-subscription.js      # Subscription status check
      webhook.js                  # Stripe webhook handler
  assets/
    logo-horizontal.svg           # Horizontal logo (icon + wordmark)
  .claude/
    CLAUDE.md                     # AI assistant context (detailed technical docs + roadmap)
  .env.example                    # Template for 7 env vars
  package.json                    # Dependencies: stripe, @supabase/supabase-js
  vercel.json                     # Vercel config with API rewrites
  HANDOVER.md                     # This document
```

## Key Code Sections (line numbers approximate)

| Section | Description |
|---------|-------------|
| CSS styles (~7-890) | All styling, dark/light mode, sidebar, map, modals, responsive, legend-color-edit |
| Sidebar HTML | Palette (colorblind/reset/more buttons), legend, display, quick fill, export |
| Modals | Upgrade (auth + pricing + legacy code), county select, clear confirm |
| appState (~1570) | Central state: `currentUser`, `subscription`, `proUnlocked`, colors, legend, etc. |
| init/loadUSMap (~1634) | Async init: loads map, then auth, then checkout return handler |
| Extra Palettes (~2056) | 26 themed + 4 colorblind palettes, categorized dropdown builder |
| Color Ramps (~2145) | 10 ramps, visual picker, steps slider, reverse, apply/reset |
| Legend (~2450) | Legend builder with click-to-paint, edit button on hover |
| Auth functions (~2565) | Supabase init, sign up/in/out, subscription check, Stripe checkout |
| Export (~2960) | PNG via html2canvas (mobile detection), SVG via XMLSerializer |
| Event listeners (~3990) | All UI bindings incl. auth, palette, legend, keyboard shortcuts |

## Technical Gotchas

1. **statesGroup transform**: `translate(5,-20) scale(0.95)` is critical. County view resets and restores it.
2. **Color via style.fill**: Must use `element.style.fill` (inline). `setAttribute('fill')` won't work.
3. **Puerto Rico**: REMOVED entirely.
4. **DC non-colorable**: Uses `nonColorable` Set. Excluded from all fill operations.
5. **Ocean background**: CSS class (`ocean-on`), not SVG rect.
6. **Vercel caching**: Hard refresh (Ctrl+Shift+R) after deploy. Cache-bust with `?v=N` if needed.
7. **SVG namespace**: Use `setSVGContent()` helper, not `innerHTML` on `<g>` elements.
8. **Label offsets**: Hand-tuned, locked. See CLAUDE.md for full table.
9. **Logo watermark**: Inline SVG, not external image. Controlled by `appState.showLogo`.
10. **Mobile export**: Opens blob in new tab for long-press save.

## Commit History (recent)
```
e3a0351 Expand palette collection: 26 themed palettes with categorized dropdown
81df43d Add always-visible reset default palette button
9543789 Add legend click-to-paint, more palettes, and colorblind-safe palette
965d6ae Fix truncated HTML: add missing </script></body></html> closing tags
fb19538 Add Supabase auth and Stripe checkout integration
c50846a Add Vercel serverless API routes for Stripe payments and Supabase auth
496f6a1 Update docs: repo rename complete, Phase 1 done, new features documented
db0834d Visual ramp picker with colored strips and reset palette button
1be6498 Add color ramps, fix legend positioning, custom legend title, county mode guards
818b4b7 Rebrand to Mapparatus: tier system, state zoom, remove PR/CSV, professional polish
```

## Roadmap

1. **Phase 1** (COMPLETE): Rebrand + polish — repo renamed, onboarding, color ramps, legend fixes, palette expansion, colorblind mode
2. **Phase 2** (IN PROGRESS): Subscription & auth — Supabase + Stripe integrated, needs end-to-end testing
3. **Phase 3** (COMPLETE): Hosting & domain — Vercel deployed, DNS configured, SSL active
4. **Phase 4** (NOT STARTED): Social media content engine — idea database, automation pipeline, headless Puppeteer export
5. **Phase 5** (NOT STARTED): Growth & iteration — analytics, A/B testing, gallery, SEO

## How to Continue Development

1. Pull the repo: `git pull origin master`
2. Edit `index.html` directly — it's the main file
3. Test locally: `python -m http.server` (note: API routes won't work locally without Vercel dev)
4. For API route testing: `npx vercel dev` (requires Vercel CLI + env vars)
5. Commit and push to master — Vercel auto-deploys
6. See `.claude/CLAUDE.md` for full technical context, label offsets, and phase details

## Owner
- **Name**: Max
- **Email**: max@mapparatus.org / mhowe.gis@gmail.com
- **GitHub**: mapzimus
