# Add EIRA Project Entry — Design

## Goal

Add EIRA, the internal generative AI platform built for the Mitsubishi Group, as a
project on the portfolio's projects page.

## Context

Projects are data-driven. `src/collections/projects.json` holds an array of
`{ name, description, image, url }` objects. `src/components/project.astro` renders
each as a card (16:9 image, name, one-line truncated description) linking to `url`.
The projects page iterates the collection, so adding an entry requires no component
changes.

EIRA is not named in Steven's CV; it is the "internal generative AI platform for the
Mitsubishi Group" described under his current role (chat, RAG regulation search, image
generation, meeting-minutes generation, document translation). Confirmed with the user.

## Change

Prepend one object to `src/collections/projects.json`, as the first array element
(before Wellsy), so EIRA appears first as the newest/current work:

```json
{
  "name": "EIRA",
  "description": "Internal generative AI platform for Mitsubishi Group: conversational chat, RAG-powered regulation search, image generation, automated meeting minutes, and document translation.",
  "image": "/assets/images/projects/eira.png",
  "url": "https://eira.bsi.co.id/"
}
```

## Decisions

- **Order:** First in the list (top of projects).
- **URL:** `https://eira.bsi.co.id/`.
- **Image:** References `/assets/images/projects/eira.png`. This file does not exist
  yet — it is a placeholder. The card image will be broken until the user drops an
  image (16:9 recommended, matching `object-cover aspect-[16/9]`) into
  `public/assets/images/projects/eira.png`.
- **Description:** Kept to one sentence since the card truncates to a single line.

## Out of Scope

- No changes to `project.astro` or the projects page.
- Creating/sourcing the actual `eira.png` image (user will add it).

## Verification

- `pnpm build` (runs `astro check && astro build`) succeeds.
- EIRA renders first on the projects page, linking to `https://eira.bsi.co.id/`.
