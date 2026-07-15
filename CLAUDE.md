# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environnement

Le propriétaire travaille sous **Windows avec PowerShell**. Toutes les commandes shell proposées ou exécutées doivent être adaptées à PowerShell (et non à bash/POSIX) : utiliser la syntaxe PowerShell (cmdlets, `$env:VAR`, `;` ou `if ($?) {}` pour chaîner, etc.).

## What this is

Single-page marketing site (in French) for **Les Jardiniers des Collines**, a gardening service in the hills around Nice. Static site — no build step, no package manager, no tests. Deployed to Netlify (`jardiniers-des-collines.netlify.app`).

## Running locally

There is nothing to build. But you **must serve over HTTP**, not open `index.html` from the filesystem — the page fetches `content/site.json` at runtime, and a `file://` origin blocks that fetch. Use any static server, e.g.:

```
python -m http.server 8000
```

Then open `http://localhost:8000/`. Opening the file directly still renders, but falls back to the hardcoded `CONTENU_SECOURS` object instead of the real content.

## Architecture

Content and presentation are deliberately separated so a non-technical owner can edit the site without touching code:

- **`content/site.json`** — the single source of all site content (services, tarifs, réalisations, équipe, outils, communes, contact info). This is the file that actually changes in day-to-day use.
- **`index.html`** — the entire site: markup, inline CSS (`:root` custom properties define the green/earth palette), and inline JS. On load it `fetch`es `content/site.json` and renders each section. Two rendering mechanisms:
  - Simple text fields use `data-champ="<key>"` attributes, filled from `d.general`, `d.note_tarif`, `d.note_zone`.
  - List sections (services, réalisations, tarifs, équipe, outils, communes) are built as HTML strings into `#liste-*` containers. All interpolated values go through `esc()` — keep using it when adding fields.
  - `CONTENU_SECOURS` (top of the script) is the offline fallback; keep its shape in sync with `site.json` if you rely on it.
  - The contact/footer phone + SMS links are derived from `general.telephone`; the tab title is derived from `general.nom` + `tagline`.
- **`admin/`** — Decap CMS. `/admin` is the owner's editing UI; `admin/config.yml` defines the form fields that map onto `content/site.json`. Auth is git-gateway via **DecapBridge** (not Netlify Identity). Saving in the CMS commits to `content/site.json` on `main` and re-deploys.
- **`images/uploads/`** — media library for the CMS (`media_folder` in config). Images referenced from `site.json` live here.

External CDN dependencies (no bundling): Leaflet 1.9.4 for the intervention-zone map, and Decap CMS 3.8.x for the admin.

## Editing rules that matter

- **Changing site content** (text, prices, adding a service, etc.) → edit `content/site.json`. Do not hardcode content into `index.html`.
- **Adding a new editable field or section** requires three coordinated changes: the field in `content/site.json`, a matching widget in `admin/config.yml` (so the owner can edit it), and rendering logic in `index.html`. Field `name`s in `config.yml` must match the JSON keys exactly.
- The site is French throughout — keep all user-facing copy in French.
- Commits to `content/site.json` are often authored by the CMS ("via DecapBridge" in the message); expect this file to change outside of code edits.
