# Add WhatsApp Channel Automation Project Entry — Design

## Goal

Add WhatsApp Channel Automation — an automated WhatsApp channel publishing daily AI
news and updates — as the first (newest) project on the portfolio.

## Context

Projects are data-driven. `src/collections/projects.json` holds an array of
`{ name, description, image, url }` objects. The homepage (`src/pages/index.astro`)
renders each as a row in the work list: number, name, description (via the
`workBlurbs` map with a fallback to `project.description`), and optional tags from
the `workTags` map. The `image` field is only referenced by the unused template
component `src/components/home/projects.astro`, so images do not render anywhere
today; the file is kept for consistency and future use.

The channel (https://whatsapp.com/channel/0029Vb5wchg60eBWja1uj83K, "AI News &
Updates 🧠") is populated end-to-end by an automation pipeline built with Hermes
and WAHA.

## Change

1. **Image:** Copy the channel screenshot from
   `D:\Users\bsi90950\Pictures\Screenshots\Screenshot 2026-07-09 081740.png` to
   `public/assets/images/projects/whatsapp-ai-channel.png`.

2. **Data:** Prepend one object to `src/collections/projects.json`, as the first
   array element (before Hedz Psikologi):

```json
{
  "name": "WhatsApp Channel Automation",
  "description": "An automated WhatsApp channel delivering daily AI news and updates — content sourced, summarized, and published end-to-end with Hermes and WAHA.",
  "image": "/assets/images/projects/whatsapp-ai-channel.png",
  "url": "https://whatsapp.com/channel/0029Vb5wchg60eBWja1uj83K"
}
```

3. **Tags:** Add one entry to the `workTags` map in `src/pages/index.astro`:

```ts
"WhatsApp Channel Automation": ["Hermes", "WAHA", "Automation"],
```

No `workBlurbs` entry is needed — the map falls back to `project.description`.

## Decisions

- **Order:** First in the list (before Hedz Psikologi), as the newest work.
- **Description:** Functional with a brief tech mention, since the automation
  pipeline (Hermes + WAHA) is the interesting part of the project.
- **Tags:** `Hermes`, `WAHA`, `Automation` — Hedz and EIRA have no tags, but tags
  are shown for older entries and add signal here.
- **Image:** Portrait channel screenshot kept as-is; images are not rendered on
  the current homepage, so aspect ratio does not matter.

## Out of Scope

- No changes to components or layout.
- No backfilling of tags for Hedz Psikologi or EIRA.

## Verification

- `pnpm build` (runs `astro check && astro build`) succeeds.
- WhatsApp Channel Automation renders first (numbered 01) in the work list with
  its tags, linking to the channel URL.
