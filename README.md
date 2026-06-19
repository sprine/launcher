# Launcher

A dead-simple home screen of big, clear buttons that open websites. Built for an elderly
audience (80s+) where **clarity and low cognitive load** matter most — usually set up by a
family member (e.g. a daughter) for a parent or grandparent.

- **One self-contained file** — `index.html`. No build step, no server, no account, no
  password. Double-click it or host it anywhere; it works instantly.
- **Local only** — every setting lives in `localStorage` (key `galt.launcher.v1`). Nothing
  leaves the device.
- **Same Galt Software treatment** — cream paper (`#fffff1`) + black ink, the `GS` favicon
  monogram, Inter for chrome. Default reading face is **Atkinson Hyperlegible** (built for low
  vision).

## What's on screen

- A grounding **clock + greeting** ("Good afternoon · Friday, June 19") at the top.
- Big **website tiles**, each showing the site's real logo (favicon, fetched from Google's
  favicon service) with a black-on-cream letter monogram as the fallback. Tapping a tile opens
  that site in the same tab; the browser's back button returns to the Launcher.
- A small, cornered **Settings** button (deliberately hard to hit by accident).

## Settings (the caregiver's panel)

Kept intentionally short:

1. **Text size** — a `−` / `+` stepper, 16–40px, scales the *entire* UI live (everything is
   `rem`-based off one root variable).
2. **Reading font** — Clear (Atkinson Hyperlegible), Dyslexia-friendly (OpenDyslexic, loaded
   from a CDN with a Comic Sans / system fallback), or Classic (Inter). Each chip previews its
   own face.
3. **Your websites** — add / edit / reorder / remove. Name + address per row. A bare address
   like `iheartradio.ca` is auto-promoted to `https://`; `javascript:`/`data:`/`file:` URLs are
   rejected. Blank rows are pruned on close.

All changes auto-save and apply immediately — there is no separate "save" step to forget. Seeds
with **iHeartRadio** (`iheartradio.ca`) and **Pluto TV** (`pluto.tv`) on first run.

## Accessibility notes

- Maximum contrast (black on cream), large hit targets, visible 3px focus rings.
- Pinch-zoom left enabled; `prefers-reduced-motion` disables transitions.
- Tiles are real `<a href>` links, so they work even if the clock script fails.

## Deploy (optional)

Not yet wired into the combined Cloudflare build (`../build-site.sh`). To ship it under
`/launcher/`, copy this folder into `_site/launcher/` after the Brochure copy step, e.g.:

```sh
rsync -a --exclude='*.md' "$ROOT/Launcher/" "$OUT/launcher/"
```

Because everything is path-relative and self-contained, no other changes are needed.
