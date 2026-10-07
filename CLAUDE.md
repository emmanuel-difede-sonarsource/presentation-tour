# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo gives Sonar Solutions Engineers an alternative to slide decks for introducing Sonar to customers: a single dynamic HTML page ("Sonar Coding Loops") that the SE drives live, in front of the customer. It serves two needs on the same page:

1. **Introduction**: the overview and the three loop views (Agentic, CI verification, Code maintenance) tell the Sonar story at a high level, in presentation order (←/→).
2. **Product detail on demand**: each product opens a detail panel (what it does, Server vs Cloud, a key stat) so the SE can go deeper only where the customer is interested.

Keep that in mind when changing anything:

- **Everything on screen is customer-facing.** The customer sees the whole page, so don't add SE-only content (talk tracks, discovery questions) or internal-only information (pricing, roadmap dates not yet public, competitor bashing, internal code names).
- **Product claims must come from an official Sonar source.** The current wording follows the 2026 SonarQube deck and the Server-vs-Cloud comparison (SonarQube Server 2026.5 / Cloud, Oct 2026), as the comment above `LOOPS` says. Don't invent features, availability (GA, Server vs Cloud) or stats. If a change needs a fact you can't verify, ask the user and update the source comment when the source changes.
- **Stay presentable live.** The tour must keep working with keyboard only (←/→, Esc, 0–3, F), read well on a projector or a shared screen, and keep the intro short: depth belongs in the panels, not on the overview.

## Branding source

The official Sonar brand guide (zeroheight) is the source of truth for logos, colors, typography and iconography. **Always fetch the relevant page below before adding or changing a logo, a color, a font or an icon**, instead of guessing values or copying them from other repos or decks (several internal docs carry outdated values, e.g. a plum `#290042`). The `.md` suffix returns the page as raw Markdown; without it the page renders via JavaScript and comes back empty.

- Logos: https://zeroheight.com/3bcff7bd1/p/162843-logos/b/33110e.md
- Logo marks (without wordmark): https://zeroheight.com/3bcff7bd1/p/162843-logos/b/05a32b.md
- Color palette: https://zeroheight.com/3bcff7bd1/p/8307fd-colors.md
- Typography: https://zeroheight.com/3bcff7bd1/p/747502-typography.md

```bash
curl -sL https://zeroheight.com/3bcff7bd1/p/8307fd-colors.md
```

- Image links in these pages are pre-signed S3 URLs that expire: download the asset into the repo instead of hotlinking it, and never recolor or stretch logos.
- Iconography: no direct `.md` link yet. Ask the user for it before introducing brand icons.

## Language

**Everything committed to this repository must be in English**: page content, code, comments, `README.md`, `CLAUDE.md` and any other Markdown, commit messages and PR descriptions. This applies even when instructions are given in another language: reply in the user's language, but write files and git messages in English, translating any wording the user provides.

## Running

No build step, no package manager, no tests. Serve the folder over HTTP (ES modules + import map don't work from `file://`):

```bash
python3 -m http.server   # then open http://localhost:8000/
```

- `?instant` skips the intro flight and panel animation (screenshots, slow machines).
- Deep links: `#2` → loop 2, `#2.1` → loop 2, first product, `#2.2.1` → that product's deep dive. Useful to send a customer straight to a product after a meeting. The hash is kept in sync with navigation via `history.replaceState`.
- Three.js 0.170.0 comes from jsDelivr (see the `<script type="importmap">`) and fonts from Google Fonts, so presenting needs network access.

## Architecture (all inside the `<script type="module">` in `index.html`)

- **Content lives in `LOOPS`** at the top of the script (the "Content — edit here" block). Each loop has `title`, `dir` (flow direction), `stage`, `tagline`, `caps` (guide/verify/solve captions) and `products`. A product shows either an official logo (`logo`, a file in `assets/logos/`) or, when the brand guide has none, an assembled lockup: the SonarQube Server mark (`mark`) plus `brand`/`sub` text. Each product drives three views: the overview wordmark (`pos`), the loop-focus card (`tag`, `bullets`), and the detail panel (`what`, `runs`, `runsNote`, `stat`). An optional `deep` object adds a level below the panel (a `cta` button in the panel opens a `kind: 'deep'` panel with `cols`, `ecosystems` and highlighted `ecosystemsHl` chips); only SonarQube Advanced Security has one. Strings are injected as HTML, so inline `<b>`/`<code>` work. Most content changes should only touch this block.
- **Two render layers stacked in `#stage`:** a WebGL `renderer` (`#gl`) for loops, dashed `Line2` arcs, nodes, dot field and the flowing token particles (amber = AI-generated change, blue = verified change, matching `#legend`); and a `CSS3DRenderer` (`#css`) for all text/cards/panels, created via `css3d(html, cls, x, y, z, { kind, loop, act })`. CSS3D objects are scaled by `S = 0.01` (CSS px → world units), so CSS sizes in the 3D-layer styles are very large (e.g. 300px headings).
- **Layout geometry:** loops are spaced by `GAP` with radius `R`; stage node angles in `ANG`. Product card positions per loop come from `CARD_LAYOUT` (keyed by product count 1/2/3). Adding a 4th product to a loop requires a new entry, and the overview wordmark `pos` must be placed by hand.
- **Visibility is by `kind`:** each CSS item has a `kind` (`title`, `connector`, `loopStatic`, `wordmark`, `focus`, `panel`, `deep`) and the frame loop fades it via `setOpacity` based on camera distance / current state. Interactive items get `pointer-events` only when mostly opaque.
- **Navigation state:** `state = { level: 'overview' | 'loop' | 'product' | 'deep', loop, prod }`. `tour` is the linear presentation order (overview → each loop → its products, each followed by its deep dive if any), built from `LOOPS`, and used by ←/→ and the HUD dots. `go(state)` sets panel targets and calls `flyTo` on the `cam` object; framing distances `Zo/Zf/Zd` come from `fitZ` and are recomputed on resize. Free wheel-zoom/drag updates `state` back via `syncStateFromCamera`. `stateFromHash` only accepts loops 1–3: update its regex if loops are added.
- **Clicks are routed through `data-act` attributes** (`loop:i`, `prod:i:j`, `deep:i:j`, `step:k`, `overview`, `close`, `next`, `prev`) handled by `act()`; loop discs in WebGL are picked by raycasting `hitDiscs`.
