# CLAUDE.md — Launcher

Working notes for editing this project. See `README.md` for the product / user-facing description.

## Architecture
- **Everything lives in `index.html`** — markup, CSS (`<style>`), and JS (`<script>`) in one file.
  No build step, no framework, no bundler. Edit the file, reload.
- Sibling files are optional PWA enhancement only: `manifest.webmanifest`, `sw.js`, `icon-*.png`,
  `favicon.svg`. The page works fully as a double-clicked `file://`.
- The JS is one classic (non-module) script under `"use strict"`. Top-level `function`/`let` are
  effectively globals, so an injected `<script>` can call them — handy for testing (see below).

## State
- A single object persisted to `localStorage["galt.launcher.v1"]`:
  `{ fontPx, font, lang, clockFormat, sites: [{ id, name, url }] }`.
- `defaults()` builds the initial object (and is the fallback source inside `load()`).
- `load()` validates **every field** against defaults and ignores unknown/invalid values; a removed
  setting simply stops being read and is dropped on the next `save()`.
- **To add a setting:** add it to `defaults()` + validate it in `load()`, add UI + a render path,
  and `save()` on change.

## i18n (10 languages)
- `I18N` = `{ langCode: { key: "string", … } }` for `en, fr, es, de, it, pt, ru, zh, hi, ar`.
- `t(key)` returns the current language's string, falling back to English, then the raw key —
  missing keys degrade gracefully, never throw.
- Static markup: `data-i18n="key"` sets `textContent`; `data-i18n-aria="key"` sets `aria-label`.
  `applyI18n()` walks these and also sets `<html lang>` / `<html dir>`.
- JS-built UI (language/font chips, site rows, install notes, the clock) calls `t()` at render time.
- `detectLang()` picks the first supported `navigator.languages` entry; `RTL` (a `Set`, currently
  just `ar`) drives `dir="rtl"`.
- **To add a UI string:** add the key to **all 10** language objects, then reference it via
  `data-i18n`/`data-i18n-aria` or `t()`. Use non-ASCII quotes (curly “”, « », „ “) inside the
  strings — never a raw `"` inside a double-quoted JS string.

## Clock
- `renderClock()` formats time and date with `Intl.DateTimeFormat(state.lang, …)`, so day/month
  names localize for free. Runs on `setInterval(renderClock, 10000)`.
- The greeting (morning/afternoon/evening) lives in the **header** (`#greeting`), not in a brand label.
- `clockFormat` is `"12h"` or `"24h"`: it sets `hour12` for the time AND swaps the date options
  (12h = short, no year; 24h = fuller, with year). The default comes from the locale via
  `defaultClockFormat()`.

## Settings UI = accordions
- Each section is a native `<details class="section">` with a `<summary class="sum">`; the heading
  *is* the summary. No JS toggling — `<details>` handles open/close (keyboard + a11y included).
- The "Click to open / Click to close" helptext is two spans (`.sum-open` / `.sum-close`, keys
  `hint_open` / `hint_close`) swapped purely by CSS `details[open]`; the chevron rotates the same way.
- **Default open state:** all collapsed except the one section carrying the `open` attribute
  (Your websites). The open/closed state is per-session only — it resets on reload.

## Icons
- Inline SVGs in the `ICONS` object, injected by `paintIcons(root)` into any
  `<span class="ic" data-icon="name">`. Add an icon = add to `ICONS` + use the span.

## Embedded media players
- A site URL can render as an **inline player** — a `.tile-radio` (header with favicon + name, then
  the provider's iframe) instead of a link tile. `renderGrid()` probes the normalized URL through
  two detectors and uses the first that matches:
  - `iHeartEmbedUrl` — iHeart `/live/` station → `?embed=true` iframe (audio; fixed `height=200`).
  - `youTubeEmbedUrl` — `watch?v=`, `youtu.be/`, `/live|embed|shorts/`, `music.youtube.com`, or a
    bare `?list=` → an `/embed/…` src; the frame also gets `.video-frame` (16:9 via `aspect-ratio`).
    It **drops auto-generated mix/radio playlists** (`list` id starting with `RD`, e.g. from "start
    radio") because YouTube refuses to embed them — falls back to the seed video.
- **No auto-start — the listener taps the provider's *own* play button inside the iframe.** A
  custom button + hidden iframe for "audio-only" YouTube was tried and abandoned: Firefox won't let
  a cross-origin frame play unless the tap lands inside it (it logs `Feature Policy: Skipping
  unsupported feature name "autoplay"` and ignores `allow="autoplay"`), so nothing ever started.
  (Spotify was tried too and removed — its embeds only play 30-second previews unless signed in.)
- **Every player iframe carries `class="radio-frame"`**, which is how power-off silences them:
  `stopRadios()` stashes the src and swaps to `about:blank` (the only way to stop a cross-origin
  player), `resumeRadios()` restores it (YouTube reloads paused). **A new player type must use
  `radio-frame`** or the power button won't stop its audio.
- YouTube embeds need a real origin: on a `file://` page the player shows "Error 153" (null
  origin); over `http(s)://` it loads fine.

## Gotchas
- **`sw.js` is network-first for navigations:** edits to `index.html` ship on the next load — no
  cache bump needed. Only changes to **icons/manifest** (cache-first assets) require bumping the
  `CACHE` constant (`"launcher-v1"`).
- `@media (prefers-reduced-motion: reduce)` disables transitions. macOS "Reduce motion" is common,
  so a CSS animation/opacity effect you add will be suppressed unless you deliberately exempt it.
  (A blinking-colon feature was added then removed, partly over exactly this.)
- In Chrome, **`file://` counts as a secure context** (`isSecureContext === true`), so the install
  section appears there; it's hidden only on truly non-secure contexts and when already installed.

## Testing (headless Chrome)
- Validate JS by extracting the script and running `node --check`:
  `node --check <(awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html)`
- Screenshot:
  `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --screenshot=/tmp/x.png --window-size=W,H --hide-scrollbars --virtual-time-budget=600 "file://$PWD/index.html"`
- To drive state, inject a `<script>` before `</body>` with `sed` and load the temp copy — e.g.
  `openSettings()`, `state.lang="ar";applyI18n();renderSettings();`, or set a `<details>.open=true`.
- **Headless Chrome reports `prefers-reduced-motion: reduce` by default** — useful for catching
  motion the media query suppresses.

## Conventions
- Cream paper `#fffff1` + black ink via CSS vars; all sizing is `rem` off `--base` (the text-size
  stepper scales the whole UI).
- Comments explain *why*, matching the existing terse style. Build DOM with the
  `el(tag, props, kids)` helper.
- Audience is elderly (80s+): favour clarity, large hit targets, low cognitive load.
