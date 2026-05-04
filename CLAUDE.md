# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **static export from Framer** of the "Gardener — Landscaper & Exterior Template" (Framer site id `5EzV3iUmhZBY9RBJKL8w1l`, generator `Framer 4705d03`). There is **no build system, no package manager, no test suite, and no source other than the exported HTML/CSS/JS and binary assets**. Everything in this directory is generated output, edited in place.

To preview locally, serve the directory with any static server, e.g.:

```powershell
python -m http.server 8000   # then open http://localhost:8000/
```

There is nothing to build, lint, install, or test.

## Repository layout

- `index.html` — the home page. ~250 KB, fully server-side-rendered: contains the inline CSS, font-face declarations, OG/Twitter meta, and the SSR'd component tree for the landing page. **This is the only page with meaningful HTML content in the repo.**
- `about-us.html`, `services.html`, `contact.html` — **byte-identical** thin shells (~25 KB each). They contain only `<head>` boilerplate plus an empty `<div id="main"></div>`; the page content is rendered at runtime by `https://framerusercontent.com/sites/5EzV3iUmhZBY9RBJKL8w1l/script_main.h4KGbDSZ.mjs`, which reads the URL path and fetches/builds the appropriate route.
- `assets/fonts/*.woff2` — Inter font subsets (latin, latin-ext, cyrillic, greek, vietnamese, etc.) referenced from the inline `@font-face` blocks in each HTML file with **hashed filenames** (e.g. `5vvr9Vy74if2I6bQbJvbw7SY1pQ.woff2`).
- `assets/images/*.{jpg,jpeg}` — referenced by hashed filename from `index.html` and from OG/favicon `<link>` tags in every HTML file.
- `assets/video/*.mp4` — single hero video referenced from `index.html`.

## Architecture & important caveats

**The non-index pages depend on Framer's CDN to render anything visible.** They have no SSR'd body — the visible UI is produced by the remote `script_main.h4KGbDSZ.mjs` bundle (and the long list of `modulepreload` modules in the `<head>`) talking to `framerusercontent.com`. Practical consequences:

1. **Editing `about-us.html` / `services.html` / `contact.html` body content has no effect** — there's nothing in the body to edit, and the JS replaces `#main` at runtime. To change those pages' content, you have to change it in the Framer project (site id above) and re-export, OR replace those files with hand-authored HTML and remove the Framer bundle script.
2. **`index.html` is the only page where local HTML edits to the rendered DOM are visible without internet access** — though even then, the Framer JS may hydrate over your changes once it loads. For text/copy edits to the home page, prefer editing the SSR'd HTML *and* expecting the same change in the Framer source.
3. **Editing meta tags (title, description, OG image, favicon) is a 4-file change** because each HTML file has its own `<head>`. The three sister pages are identical, so a common pattern is "edit one, then copy it to the other two" — verify with `cmp` after.
4. **Assets are referenced by hashed filename.** Replacing an image or font requires (a) putting the new file in `assets/` (keeping the old hash filename, or picking a new one) and (b) updating every `url(...)`, `src=`, and `href=` reference across the HTML files. Grep before replacing.
5. **External dependencies that must remain reachable for the site to function fully:** `framerusercontent.com` (JS bundles, search index), `fonts.gstatic.com` (Google-hosted Figtree subsets), `events.framer.com` (analytics), and `framer.com` (the injected badge link). The local `assets/fonts/` files cover Inter only; Figtree is loaded from Google.
6. **There is a Framer promo badge** injected as `<div id="__framer-badge-container">` near the bottom of every HTML file. It is the visible "Made in Framer" link to framer.com — remove it from all four files together if branding it out.
7. **Inline design tokens** are defined as CSS custom properties on `body { ... }` in each HTML file (e.g. `--token-feecde84-...: rgb(33, 133, 68)` is the brand green). These are duplicated across all four files; edit consistently.

## Working tips

- When asked to "change the site" globally (color, title, favicon, font, script, badge): expect to touch all 4 HTML files. Use `Grep` across `*.html` first to find every occurrence, then `Edit` each.
- When asked to add a new page: copying one of the thin shells will produce a page that **will not render its own content** — it'll fall through to whatever route the Framer runtime decides. Adding a real new page generally means writing static HTML from scratch (and likely dropping the `script_main` bundle reference).
- Never modify files inside `assets/` by editing them — they are binary. Replace them whole.
- The HTML files contain very long single lines (inline `<style>` and `<script>`). When editing, prefer `Edit` with a small, unique surrounding snippet rather than reading and rewriting whole lines.
