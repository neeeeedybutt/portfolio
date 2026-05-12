# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static personal portfolio site (`nidhibhat.xyz`) **exported from Webflow** via Exflow.site. There is no build system, no package manager, no test suite, no linter. Pages are hand-editable HTML, but the markup is machine-generated and very dense (most files are effectively one long line).

To preview locally, serve the `portfolio/` directory with any static server (e.g. `python3 -m http.server` from `portfolio/`) and open `index.html`. Opening files via `file://` will break some Webflow features that expect HTTP.

## Layout

- `index.html` — home page, the only page that loads scripts from the local `js/` folder
- `About/about.html`, `Contact/contact.html` — top-level subpages
- `Project-<name>/project-<name>.html` — one HTML file per case study (beretta-gallery, design-pulse, enroute, praan, shield)
- `css/nidhibhatdesigns.webflow.shared.7933b6920.css` — the entire site's shared stylesheet (~325 KB)
- `js/` — Webflow runtime: `jquery.js`, two `webflow.schunk.*.js` chunks, and `webflow-script.js` (the entry point that imports the chunks)
- `images/` — all assets, served locally
- `script1.js` … `script8.js` at the project root — **orphan duplicates**. `script3` = `js/jquery.js`, `script4`/`script5` = the two webflow chunks, `script6` = `js/webflow-script.js`. Nothing references them. Don't edit these expecting changes to take effect; if you touch them, mirror the change to the corresponding `js/` file (or delete the duplicates). `styles1.css` at root is similarly unreferenced.

## Critical: page → asset wiring is inconsistent

Only `index.html` uses local assets. **Every other page** (`About/`, `Contact/`, all `Project-*/`) loads its Webflow JS from `https://cdn.prod.website-files.com/6709ab44f5d55b23e7ea8476/...` instead. Consequences:

- Subpages need internet to render interactions correctly; the home page does not.
- Edits to `js/webflow.schunk.*.js` or `js/webflow-script.js` only affect the home page. To change behavior on subpages, you must rewrite their `<script>` tags to point at local files (or accept that you can't change CDN-hosted code).
- The shared CSS is loaded locally on every page, so style edits to `css/nidhibhatdesigns.webflow.shared.7933b6920.css` apply everywhere.

When adding a new project page, copy an existing `Project-*/project-*.html` and update: the `<title>`, `data-wf-page` (each page has a unique Webflow page ID; `data-wf-site` is shared across the site), all hero/content text, and any `data-w-id` blocks you want to keep animated.

## Webflow interactions (IX2) — handle with care

Animations are driven by Webflow's IX2 system, not hand-written JS:

- Each animated element has a `data-w-id="<hex>"` attribute. The Webflow runtime (`webflow-script.js` + chunks) finds these IDs at load time and wires up scroll/hover/load animations.
- The `<head>` of each page contains a `<style>` block that sets `opacity:0` for those IDs inside `html.w-mod-js:not(.w-mod-ix)` selectors. This hides elements until the IX2 runtime tags `<html>` with `w-mod-ix` and animates them in. Removing this block causes elements to flash visible before animating; removing a `data-w-id` orphans its animation.
- Inline `style="...opacity:0; transform:translate3d(0,50px,0)..."` on individual elements is the same hide-before-animate pattern. Don't strip it unless you're also removing the corresponding interaction.

If an element appears invisible after an edit, the most likely cause is that you broke a `data-w-id` linkage or removed an inline opacity reset.

## Hand-written JS

Only one place: the bottom of `index.html` (~line 289 onward) initializes Swiper sliders for any `.slider-component` on the page. Swiper is loaded from a jsdelivr CDN. This block is the entry point for any per-component vanilla JS — extend it here rather than creating new script files.

## Known broken link

`Contact/contact.html` and the project pages link to `../Works/works.html`, but `Works/` doesn't exist. Either create the page or repoint to `../index.html#Projects` (which is what other nav links use).

## Editing tips

- Files are minified into single lines. To find content, grep for the visible text or for stable Webflow class names (e.g. `home-hero`, `footer-all`); don't try to read by line number.
- Don't reformat HTML wholesale — Webflow's IX2 selectors and the inline opacity styles are tied to exact attribute order and presence; reflowing the file is safe in principle but makes diffs unreviewable.
- The `data-wf-page` attribute is the page's Webflow ID — keep it stable per page, never copy across pages.
