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
│   ├── icons/*.svg         # Line icons (stone mark, arrows, +/−)
│   └── images/*.png        # Photography from the Figma file
├── .env.example            # Template for MCP credentials (not committed)
├── .gitignore
├── AGENTS.md               # Agent working conventions (local only)
└── opencode.jsonc          # OpenCode tool config (local only)
```

## Running locally

Any static file server from the repo root works:

```sh
python3 -m http.server 8321 --bind 127.0.0.1
# or
bunx serve -l 8321
```

Then open <http://127.0.0.1:8321/>. There is no build/transpile step.

## Color palette system

All colors are **CSS custom properties** declared once under `:root` in
`assets/css/styles.css`. Every component (badges, buttons, hairlines, SVG
icons via `currentColor` masks) derives from these tokens, so the entire site
re-colors from ~10 values.

### The five palettes

Default (no parameter) is **Moss**, the palette from the Figma design.

| Theme   | Activate                 | Character                          |
| ------- | ------------------------ | ---------------------------------- |
| Moss    | `/` (default)            | Sage green on warm paper (design)  |
| Slate   | `/?theme=slate`          | Cool industrial blue-slate         |
| Clay    | `/?theme=clay`           | Warm terracotta on cream           |
| Coral   | `/?theme=coral`          | Client palette peach/blush/coral   |
| Wine    | `/?theme=wine`           | Client palette cream/rose/burgundy |

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

Delete the four `:root[data-theme=…]` blocks and the `<script>` in
`index.html` — Moss continues to work untouched as the default.

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

## Toolchain (OpenCode MCP servers)

| Server       | Purpose                        | Config                          |
| ------------ | ------------------------------ | ------------------------------- |
| `framelink`  | Figma design data + assets     | `npx figma-developer-mcp`       |
| `playwright` | Browser verification           | `bunx @playwright/mcp` headless |
