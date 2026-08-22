# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing site for "Остров Кэма" (Ostrov Kema), a trout farm and
alcohol-free family restaurant in Taraz, Kazakhstan. It is a static site with
no build step, no package manager, and no dependencies — just `index.html`
plus an `images/` folder, deployed directly via GitHub Pages (`.nojekyll`
disables Jekyll processing so files are served as-is).

## Running / previewing

There is no build or dev server tooling in this repo. To preview locally, serve
the directory with any static file server and open it in a browser, e.g.:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`. There are no lint, test, or build commands
to run — changes are verified by opening the page in a browser.

## Structure

- `index.html` — the entire site: all CSS (in a `<style>` block) and all JS
  (in a single IIFE `<script>` block) live inline in this one file. There are
  no separate `.css`/`.js` files.
- `images/` — photos (`hero-terrace.jpg`, `lake-view.jpg`, `kabinka.jpg`,
  `hall-family.jpg`), favicons, and `images/menu/menu-01.jpg` … `menu-25.jpg`
  (25 sequentially numbered full-menu photo pages, rendered as a gallery — see
  below).
- `.nojekyll` — required for GitHub Pages to serve the repo as a plain static
  site.

## Architecture: single-page "carousel" of sections

The page is not a normal scrolling page — it's a horizontal, full-viewport
carousel. Each `<section class="slide" data-slide-id="...">` inside
`#track` is one "slide" (`hero`, `about`, `fishing`, `bar`, `events`,
`contact`). JS in the inline `<script>` translates `.track`'s `transform:
translateX(...)` to move between them, driven by:

- Prev/next arrow buttons, dot indicators, and a `"n / total"` counter.
- Keyboard arrow keys, touch swipe, and trackpad horizontal wheel scroll.
- Any element with `data-slide="<slide-id>"` (nav links, the logo, in-page
  CTA buttons) — clicking it calls `goToSlide()` for that slide, so adding a
  new internal link means adding `data-slide` with a matching
  `data-slide-id`, not an `href="#..."`.

When adding a new slide/section: add a new `<section class="slide"
data-slide-id="...">` inside `#track`, and the carousel wiring
(`slides`/`slideIds`, dot generation, counter) picks it up automatically
since it queries `.slide` elements — no JS changes needed unless the new
slide needs bespoke behavior.

## i18n: RU / KK (Kazakh) / EN

All user-facing copy is duplicated three times inside the `translations`
object in the inline script (`ru`, `kk`, `en` keys), keyed by dot-path strings
like `"about.card1.title"`. Markup elements carry `data-i18n="that.key"` and
`applyLang()` walks the DOM setting `textContent` from the active
dictionary. The chosen language persists in `localStorage` (`ok_lang`).

**When changing any visible text, you must update it in all three language
blocks** (`ru`, `kk`, `en`) and keep the `data-i18n` key on the corresponding
HTML element in sync — the HTML element's own text content is just the RU
fallback/initial render before JS runs.

## Theming

Colors are CSS custom properties on `:root` (light palette), overridden both
by `@media (prefers-color-scheme: dark)` and by `:root[data-theme="dark"]`
(no explicit dark-mode toggle UI is present, but the CSS supports one).
Follow the existing token names (`--bg`, `--text`, `--accent`, `--surface`,
etc.) rather than hardcoding new colors.

## Menu gallery

The 25 menu photos are not individually referenced in HTML — `menuGallery`
is populated at runtime by a loop in the script that generates
`images/menu/menu-01.jpg` through `menu-25.jpg` (zero-padded to 2 digits).
To add/remove menu pages, add/remove the actual image files and update the
loop bound (`mi <= 25`) in the script — don't hand-edit gallery markup.

## Content notes

- Contact/pricing details (address, hours, phone) are still placeholders in
  places (e.g. `contact.addr.value` says "уточняется" / "to be confirmed")
  — a `.draft-banner` at the top of the page currently flags the whole site
  as a draft prototype. When real business details are supplied, update the
  same string in all three language dictionaries and consider removing the
  banner.
- The WhatsApp number (`+7 776 888 07 30` / `wa.me/7768880730`) and
  Instagram handle (`@ostrovkema`) appear in multiple places (hero CTA,
  events CTA, contact section, floating FAB button) — keep them consistent
  if they change.
