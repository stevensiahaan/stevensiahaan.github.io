# Add Hedz Psikologi Project Entry — Design

## Goal

Add Hedz Psikologi, an online psychology assessment platform, as the first (newest)
project on the portfolio's projects page.

## Context

Projects are data-driven. `src/collections/projects.json` holds an array of
`{ name, description, image, url }` objects. `src/components/project.astro` renders
each as a card (16:9 image via `aspect-[16/9] object-cover`, name, one-line truncated
description) linking to `url`. The projects page iterates the collection, so adding an
entry requires no component changes.

Hedz Psikologi (https://hedzpsikologi.com/) is an online psychology platform offering
psychological tests: Intelligence Structure Test (Tes Kecerdasan), personality tests
(Tes Kepribadian), and early screening for depression, anxiety, and stress — with
guidance from professional psychologists. Built by Steven with Claude Code.

## Change

1. **Image:** Copied the homepage screenshot from
   `D:\Users\bsi90950\Pictures\Screenshots\Screenshot 2026-07-03 090311.png` to
   `public/assets/images/projects/hedz.png`. The wide screenshot suits the card's
   `aspect-[16/9] object-cover` framing.

2. **Data:** Prepend one object to `src/collections/projects.json`, as the first array
   element (before EIRA):

```json
{
  "name": "Hedz Psikologi",
  "description": "An online psychology assessment platform for intelligence, personality, and mental-health tests — from the Intelligence Structure Test to depression, anxiety, and stress screening — guided by professional psychologists.",
  "image": "/assets/images/projects/hedz.png",
  "url": "https://hedzpsikologi.com/"
}
```

## Decisions

- **Order:** First in the list (before EIRA), as the newest work.
- **URL:** `https://hedzpsikologi.com/`.
- **Image:** Real homepage screenshot saved as `hedz.png` (not a placeholder).
- **Description:** One sentence covering the three service pillars visible on the site.

## Out of Scope

- No changes to `project.astro` or the projects page.

## Verification

- `pnpm build` (runs `astro check && astro build`) succeeds.
- Hedz Psikologi renders first on the projects page, linking to
  `https://hedzpsikologi.com/`.
