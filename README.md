# Viana Stone — viana-site

Static marketing site for **A. C. Viana Pedras** — natural marble blocks and
mining machinery. Pure HTML + CSS, no build step, deployed on **Cloudflare**.

## Project rules

- **Commit only files required to deploy on Cloudflare** — HTML, CSS, fonts,
  images. Credentials, tooling config and generated files are excluded via
  `.gitignore` and never committed.
- Never stage/commit without being explicitly asked.
- Work happens on feature branches; `main` carries deploy-ready state.

## Repository structure

```
viana-site/
├── index.html              # Homepage (single page so far)
├── assets/
│   ├── css/styles.css      # All styling: tokens, layout, themes
│   ├── icons/*.svg         # Arrows, +/−, brand wordmark + favicon
│   └── images/*.png        # Photography from the Figma file
├── .opencode/
│   └── skills/design-review/   # Playwright review-pass skill (local only)
├── .env.example            # Template for MCP credentials (not committed)
├── .gitignore
├── AGENTS.md               # Agent working conventions (local only)
└── opencode.jsonc          # OpenCode tool config (local only)
```

## Running locally

Any static file server from the repo root works — full instructions in the
[Usage guide](#usage-guide) below.

## Usage guide

Step-by-step flows for day-to-day work on this site.

### Start the preview server

```sh
python3 -m http.server 8321 --bind 127.0.0.1
```

- Open <http://127.0.0.1:8321/>. The server serves live files — just save and
  refresh; there is nothing to rebuild.
- `Address already in use` means a previous instance is still running; since
  it serves the same live directory, it is normally fine to keep using it.
  To fully restart: `pkill -f "http.server 8321"`, then run the command again.

### Preview the color palettes

Append `?theme=<id>` to any URL: `slate`, `clay`, `coral`, `wine`, `viana`,
or load `/` plain for the default Moss. The five presentation links are:

```
/?theme=slate   /?theme=clay   /?theme=coral   /?theme=wine   /?theme=viana
```

The theme is read once per page load from the URL — share the exact link to
present the exact palette. Unknown values silently fall back to Moss.

### Prepare a designer review dossier

Ask the agent to run the `design-review` skill (e.g. *"run a design review"*
or `/design-review`). Scopes:

| Scope | Result |
| ----- | ------ |
| *(none)* | Full pass: 6 themes × 390/1024/1440, close-ups, hover/FAQ states, QA table |
| `theme:coral` | All viewports for one theme only |
| `tablet` / `mobile` | Quick single-viewport pass across all themes |
| `overlay:<figma-export.png>` | Side-by-side + blend overlay against a Figma frame export |

Output is always written to **`/tmp/opencode/design-review/`** — one HTML
dossier plus a `shots/` folder of screenshots, never inside the repository.
Open `index.html` directly in a browser; to share it, zip the folder
(the HTML references its shots). The dossier header records the git commit
and uncommitted changes the captures reflect, so a reviewer always knows
what they are looking at.

### Edit content

- All copy lives in `index.html`; sections are commented
  (`<!-- ===== Section name ===== -->`) in design order: intro/hero,
  capabilities, overview, marble blocks, machinery, story, process,
  responsibility, FAQ, inquiry, footer.
- **Add a marble block card**: copy one `<article class="block-card">…</article>`
  inside `.cards--blocks`, then edit name/`NM–…` code, variant line,
  dimensions, weight, and point `img src` at the new photo. The grid is
  3-up; a fourth card wraps to a new row automatically.
- **Add a machine listing**: same pattern under `.cards--machines`
  (2-up grid) — photo, tag, name, spec triple (year / hours / weight).
- **FAQ**: each question is a `<details class="faq">`; add one and keep the
  first `open` attribute on whichever question should show expanded by
  default.

### Replace placeholder assets

| Asset | Current state | How to replace |
| ----- | ------------- | -------------- |
| Photography | Concept renders from the Figma file | Drop real photos into `assets/images/` with the same filenames (or update `src`), target display sizes ≈1440×568 (panorama), ~440×330 (cards), ~690×340 (machines), 624×580 (portrait), 720×580 (operations) |
| Wordmark / favicon | Traced SVGs from the sign photo | Drop in the client's original vector, same filenames, preserve the wordmark's ~4.07:1 aspect ratio |
| Contact details | `hello@viana.example`, `+46 …`, Quarry Road 12 | Search `index.html` for the placeholder strings and replace |

### Requirement checklist before go-live

1. Replace placeholder contact details + address.
2. Write final FAQ answers (2–4 are drafted placeholders).
3. Replace concept photography and, ideally, the traced wordmark.
4. Decide the final palette and remove the theme switcher (see below).
5. QA pass (`design-review`) with 0 console/network issues, then commit to
   `main` and deploy.

## Color palette system

All colors are **CSS custom properties** declared once under `:root` in
`assets/css/styles.css`. Every component (badges, buttons, hairlines, SVG
icons via `currentColor` masks) derives from these tokens, so the entire site
re-colors from ~10 values.

### The six palettes

Default (no parameter) is **Moss**, the palette from the Figma design.
**Viana** is the true brand palette, sampled from the company logo.

| Theme   | Activate                 | Character                          |
| ------- | ------------------------ | ---------------------------------- |
| Moss    | `/` (default)            | Sage green on warm paper (design)  |
| Slate   | `/?theme=slate`          | Cool industrial blue-slate         |
| Clay    | `/?theme=clay`           | Warm terracotta on cream           |
| Coral   | `/?theme=coral`          | Client palette peach/blush/coral   |
| Wine    | `/?theme=wine`           | Client palette cream/rose/burgundy |
| Viana   | `/?theme=viana`          | Brand purple from the company logo |

Each theme is a `:root[data-theme="…"]` block overriding only color tokens:

```css
:root[data-theme="wine"] {
  --paper: #FCF5EE;     /* page background */
  --ink: #2E1219;       /* primary text */
  --muted: #7C4B53;     /* secondary text */
  --accent: #850E35;    /* links, markers, mono captions, solid button */
  --accent-strong: #6B0A2A;  /* solid button hover */
  --line: #F1D9DA;      /* hairline borders */
  --line-dark: #A04A5E; /* hairlines inside dark sections */
  --dark: #4A0F26;      /* machinery section background */
  --dark-text: #F5E2E6; /* text on dark sections */
  --wash: #FFC4C4;      /* responsibility section background */
  --wash-cta: #F5CFD3;  /* inquiry CTA background */
}
```

The switcher is a small inline script at the end of `index.html` that reads
`?theme=` and sets `data-theme` on `<html>`. Only whitelisted values apply;
anything else silently falls back to Moss.

### Adding a new palette

1. Pick values for all 12 tokens above (see mapping notes in `styles.css`).
2. **Contrast rules**: `--accent` used *as text* must reach 4.5:1 against
   `--paper` (small mono captions and links). `--accent` as the solid button
   background must give white text ≥ 4.5:1. In the Coral theme the source
   palette's `#FD7979` fails both, so the caption/link coral is darkened to
   `#BE4242` while the washes keep the palette's original values.
3. Append the block next to the other themes in `styles.css`.
4. Add the key to the whitelist array in the `index.html` switcher script.
5. Screenshot and eyeball it (see QA workflow).

### Going to production with one palette

Delete the five added `:root[data-theme=…]` blocks and the `<script>` in
`index.html` — Moss continues to work untouched as the default.

## Brand assets

`assets/icons/viana-wordmark.svg` and `viana-favicon.svg` are **traced from a
photo of the old company signage** (`viana_logo_photo.jpg`, untracked) with a
potrace pipeline — flat single-color plum fill `#671E4B`. The logo's photo
gradient cannot survive a trace; a flat fill matches how the sign prints.
**Upgrade path:** when the client supplies the original vector brand asset,
replace both files with it (wordmark aspect ratio ~4.07:1, header height 24px).

The logo colors also feed the `viana` theme (see palettes above).

## Design ↔ code workflow (Figma MCP)

The site is translated from the Figma file via the **Framelink**
(`figma-developer-mcp`) MCP server:

- Credentials: personal access token in `.env` as `FIGMA_API_KEY`
  (template in `.env.example`; token scope *File content: read-only*).
  Never committed.
- One Figma frame = one page build. Fetch layout data
  (`get_figma_data`), download photography/icons
  (`download_figma_images`), then translate tokens into `styles.css`.

## QA workflow (Playwright MCP)

Verification runs against a local server through the **Playwright MCP**
server (Chromium, headless, `--isolated`):

1. Serve the site (`python3 -m http.server 8321`).
2. Navigate, take a full-page screenshot, dump console + network requests.
3. Acceptance: **0 console errors/warnings**, no non-200 asset requests,
   render matches the Figma frame. Screenshot artifacts live in
   `/tmp/opencode/playwright-mcp/` (outside the repo, never committed).

For reviewers and presentation prep, the same machinery is wrapped in the
**`design-review` skill** (see *Usage guide → Designer review dossier*),
which produces the shareable dossier instead of ad-hoc screenshots.

## Deployment (Cloudflare)

Static site, no build output. Cloudflare Pages / Workers static assets can
serve the repo root directly; the repo intentionally contains only
deploy-required files. When a production setup is chosen, wire it up with:

```sh
bunx wrangler pages deploy . --project-name=viana-site
```

(Or connect the repository in the Cloudflare dashboard — no config needed.)

## Content placeholders (TODO before go-live)

- Phone `+46 (0) 10 000 00 00` and email `hello@viana.example` are placeholders.
- FAQ answers 2–4 are drafted placeholder copy (the Figma file only defines
  the first answer).
- All copy is English-only; the `EN / SE` language switch is not functional yet.
- Photography is concept imagery from the design file.
- The wordmark/favicon are photo traces — replace when the client provides
  the original vector logo.

## Toolchain (OpenCode MCP servers)

| Server       | Purpose                        | Config                          |
| ------------ | ------------------------------ | ------------------------------- |
| `framelink`  | Figma design data + assets     | `npx figma-developer-mcp`       |
| `playwright` | Browser verification           | `bunx @playwright/mcp` headless |

Skills: `design-review` (`.opencode/skills/design-review/`) — Playwright
review pass producing the designer dossier (local only, not committed).
