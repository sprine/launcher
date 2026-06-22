# Launcher

A dead-simple home screen of big, clear buttons that open websites. Built for an elderly
audience (80s+) where **clarity and low cognitive load** matter most — usually set up by a
family member (e.g. a daughter) for a parent or grandparent.

- **One self-contained file** — `index.html`. No build step, no server, no account, no
  password. Double-click it or host it anywhere; it works instantly. (A handful of sibling
  files — `manifest.webmanifest`, `sw.js`, `icon-*.png` — add the optional "install as an app"
  feature below; they're pure enhancement and the page works fine without them.)
- **Local only** — every setting lives in `localStorage` (key `galt.launcher.v1`). Nothing
  leaves the device.
- **Same Galt Software treatment** — cream paper (`#fffff1`) + black ink, the `GS` favicon
  monogram, Inter for chrome. Default reading face is **Atkinson Hyperlegible** (built for low
  vision).

## What's on screen

- A warm **greeting in the header** ("Good afternoon", rolling over through the day) with a large
  **clock and date** centred below it ("6:52 PM" over "Friday, June 19"). Both follow the chosen
  language, formatted via `Intl`.
- Big **website tiles**, each showing the site's real logo (favicon, fetched from Google's
  favicon service) with a black-on-cream letter monogram as the fallback. Tapping a tile opens
  that site in the same tab; the browser's back button returns to the Launcher.
- **Music & radio tiles play right on the page.** Paste an **iHeartRadio** live-station or a
  **YouTube** video/playlist link as a website, and instead of a link the tile becomes an inline
  player — the listener just taps the player's big play button, no new site to get lost in. The
  **power button stops both**. (A YouTube *Mix* / "start radio" link — its address contains
  `list=RD…` — can't be embedded by YouTube, so the tile plays just that one video; use a normal
  saved playlist whose address has `list=PL…` to play straight through.)
- A small, cornered **Settings** button (deliberately hard to hit by accident).

## Settings (the caregiver's panel)

Opened from the small, cornered **Settings** button. Each section is a **collapsible accordion** —
the heading is the tappable summary, with a "Click to open / Click to close" hint and a chevron.
Everything starts **collapsed except "Your websites"**, so the panel opens short and uncluttered.

1. **Language** — 10 languages (English, Français, Español, Deutsch, Italiano, Português, Русский,
   中文, हिन्दी, العربية), each listed by its own name. Picking one re-localizes the greeting, the
   date, and the whole settings panel. Defaults to the device's language (falling back to English);
   Arabic switches the page to right-to-left.
2. **Text size** — a `−` / `+` stepper, 16–40px, scales the *entire* UI live (everything is
   `rem`-based off one root variable).
3. **Reading font** — Clear (Atkinson Hyperlegible), Dyslexia-friendly (OpenDyslexic, loaded
   from a CDN with a Comic Sans / system fallback), or Classic (Inter). Each chip previews its
   own face.
4. **Clock format** — 12-hour ("6:52 PM") or 24-hour ("18:52"); the choice also adjusts the date
   (24-hour adds the year). Defaults to the device/locale convention.
5. **Your websites** — add / edit / reorder / remove. Name + address per row. A bare address
   like `iheartradio.ca` is auto-promoted to `https://`; `javascript:`/`data:`/`file:` URLs are
   rejected. Blank rows are pruned on close.

All changes auto-save and apply immediately — there is no separate "save" step to forget. Seeds
with **iHeartRadio** (`iheartradio.ca`) and **Pluto TV** (`pluto.tv`) on first run.

## Install as an app (instead of a browser homepage)

Browsers give web pages **no way to set themselves as the homepage** — it was a hijacking
vector, so the old API was removed. The better fit is to **install** the launcher as a PWA: on a
Chromebook it gets its own icon on the shelf and opens full-screen with no address bar, which is
far easier for an 80-something to find than a homepage.

- Open **Settings → Add to this device → Install Launcher**. (The button only appears when the
  browser can actually install it.)
- Requires the launcher to be **served over `https`** (e.g. the Cloudflare `/launcher/` path
  below). On a double-clicked `file://` copy there's nothing to install, so the section stays
  hidden — everything else still works.
- The `sw.js` service worker also makes the launcher open **offline** after the first visit.
- *Truly* automatic startup is only possible on a **managed Chromebook**, where an admin sets
  `HomepageLocation` / `RestoreOnStartup` by policy in the Google Admin console.

## Accessibility notes

- Maximum contrast (black on cream), large hit targets, visible 3px focus rings.
- Pinch-zoom left enabled; `prefers-reduced-motion` disables transitions.
- Tiles are real `<a href>` links, so they work even if the clock script fails.
- Settings sections are native `<details>` accordions — fully keyboard-operable — and every label
  (plus the greeting and date) is localized, with right-to-left layout for Arabic.

## Deploy (optional)

Not yet wired into the combined Cloudflare build (`../build-site.sh`). To ship it under
`/launcher/`, copy this folder into `_site/launcher/` after the Brochure copy step, e.g.:

```sh
rsync -a --exclude='*.md' "$ROOT/Launcher/" "$OUT/launcher/"
```

Because everything is path-relative and self-contained, no other changes are needed.
