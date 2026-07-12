# Work Section — Studio Cards

**Date:** 2026-07-12
**Status:** Approved
**Source design:** Claude Design project `019e00c4-fd18-71d6-bac1-9f0c7f4e11e0` ("Personal Portfolio"), file `Portfolio v2 - Studio Cards.html`

## Problem

The site already runs on the Studio design system. `src/layouts/main.astro` defines its tokens (`--ink`, `--rule`, `--accent: #1B3A6B`, Newsreader / Geist / Geist Mono), and the hero, about, capabilities, experience, and contact sections in `src/pages/index.astro` match the source design class-for-class.

The Work section is the one section still in its old form: `.work-list`, a set of flat text rows (`num | name | desc | tags | arrow`) with no imagery. The design specifies `.work-grid` — a two-column grid of cards, each fronted by a device-framed screenshot.

Two defects surface alongside the redesign:

1. `index.astro` holds two hardcoded lookup maps, `workTags` and `workBlurbs`, keyed by project name. EIRA and Hedz Psikologi have no entry in either, so they render with zero tags. The card design foregrounds tags, so this becomes visible.
2. `projects.json` references `/assets/images/projects/eira.png`, which does not exist on disk.

## Approach

Replace the row list with a card grid. Move per-project card data out of the page and into `projects.json`. Make the card adapt to however many images a project actually has, so the section can ship with today's assets and improve as screenshots are added — with no code change.

### Files

| File | Change |
| --- | --- |
| `src/components/work-card.astro` | **New.** Renders one card. |
| `src/pages/index.astro` | Work section becomes `.work-grid` + `<WorkCard>` map. Delete `workTags`, `workBlurbs`, and all `.work-*` list CSS. |
| `src/collections/projects.json` | Each entry gains `frame`, `images`, `tags`. |

`image-slot.js` from the source design is an authoring shim and is **not** ported. Real `<img>` elements replace `<image-slot>`.

## Data model

Each entry in `src/collections/projects.json`:

```json
{
  "name": "Wellsy",
  "description": "Digitises gym body composition, tracks progress, and surfaces AI-powered insights.",
  "url": "https://wellsy.site/",
  "frame": "phone",
  "images": ["/assets/images/projects/wellsy.png"],
  "tags": ["Flutter", "AI", "Health"]
}
```

- `frame` — `"phone"` (portrait bezel) or `"wide"` (landscape screen bezel).
- `images` — 0 to 3 paths. Index 0 is the hero; 1 and 2 are side shots.
- `tags` — replaces the `workTags` map.
- `description` — existing field; becomes the card blurb, replacing the `workBlurbs` map.

### Per-project values

| Project | frame | images | tags |
| --- | --- | --- | --- |
| WhatsApp Channel Automation | wide | `whatsapp-ai-channel.png` | Hermes, WAHA, Automation |
| Hedz Psikologi | wide | `hedz.png` | Web, Psychology, Assessment |
| EIRA | wide | *(none)* | GenAI, RAG, Azure |
| Wellsy | phone | `wellsy.png` | Flutter, AI, Health |
| Runchise POS | wide | `runchise.jpg` | Flutter, POS, Bluetooth |
| Skul.id | phone | `skulid.webp` | Flutter, EdTech, BLoC |
| Andal Linkage | phone | `andal_linkage.webp` | Flutter, HRIS, Geo |
| Baboo | phone | `baboo.png` | Android, Firebase, Payments |

Paths are relative to `/assets/images/projects/`. EIRA ships with `"images": []` — see below.

## Component contract

`work-card.astro` accepts `{ name, description, url, frame, images, tags, index }`.

### Image count drives the media layout

- **1 image** — hero bezel alone, centered, spanning the full media width. No side rail.
- **2–3 images** — hero bezel plus the side-shot rail, per the source design.
- **0 images** — the media area is omitted entirely. The card renders as a text-only card.

The zero case is deliberate: it degrades cleanly instead of rendering a broken image. **EIRA uses it.** This resolves the missing-`eira.png` defect without blocking the section on new artwork. Backfilling an EIRA screenshot later is a one-line JSON edit that upgrades the card automatically.

### Links

- `url` starting with `http` — the name and arrow are anchors with `target="_blank" rel="noopener"`.
- `url` of `"#"` (**Andal Linkage** and **Baboo** — both are `"#"` in the current data) — the name and arrow render as `<span>`, not links. The source design does this for the same two projects; preserving it avoids two dead links.

The card is not a single wrapping anchor. The name and arrow are the interactive elements, matching the design and keeping tab order sane. Hover styling on `.work` still drives the name and arrow via CSS descendant selectors.

## Layout

- `.work-grid` — `repeat(2, 1fr)`, `gap: 24px`; collapses to one column at `max-width: 900px`.
- `.work-media` — `height: 300px`, `background: var(--surface)`, `border-bottom: 1px solid var(--rule)`. At `max-width: 600px`, height drops to 240px and padding tightens.
- `.phone` — 6px `var(--ink)` border, `border-radius: 26px 26px 0 0`, no bottom border; bezel bleeds off the bottom of the media area.
- `.screen` — 5px `var(--ink)` border, `border-radius: 12px 12px 0 0`, no bottom border. Used by `frame: "wide"`.
- `.work-body` — number, name + arrow row, description, tags.

## Accessibility

- Hero image: descriptive alt, e.g. `"Wellsy — app screenshot"`.
- Side shots: `alt=""`. They are supplementary; announcing three near-identical images adds noise.

## Verification

1. `pnpm build` passes.
2. Drive the built page in a browser at desktop (1280px) and mobile (390px) widths and confirm: the grid renders two-up then collapses, phone and wide bezels frame their screenshots correctly, single-image cards center their hero with no empty side rail, the EIRA card renders text-only with no media area, and no image requests 404.

Build success alone is not sufficient evidence — the page gets driven.

## Out of scope

- Dark mode. The design system is light-only today; `tailwind.config.mjs` sets `darkMode: "class"` but no dark styles exist for these tokens.
- Any change to the hero, about, capabilities, experience, CV, or contact sections.
- Producing new screenshots. The adaptive card exists so this can happen independently, later.
