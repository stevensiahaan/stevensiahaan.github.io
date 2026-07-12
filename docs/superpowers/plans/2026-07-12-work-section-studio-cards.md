# Work Section — Studio Cards Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the Work section's flat `.work-list` text rows with the `.work-grid` card grid from the "Portfolio v2 — Studio Cards" design, where each card is fronted by a device-framed screenshot.

**Architecture:** A new `work-card.astro` component owns one card and is the only file that knows what a phone bezel or side shot is. It adapts to however many images a project has (0, 1, or 2–3). Per-project card data (`frame`, `images`, `tags`) moves out of hardcoded lookup maps in `index.astro` and into `projects.json`, so adding a project stays a pure JSON edit.

**Tech Stack:** Astro 4 (`.astro` components, scoped `<style>`), plain CSS using the existing design tokens from `src/layouts/main.astro`. Biome for lint/format. No test framework in this repo.

**Spec:** `docs/superpowers/specs/2026-07-12-work-section-studio-cards-design.md`

---

## Verification Approach — read this before Task 1

This repo has **no test runner**. `pnpm build` runs `astro check && astro build`, which typechecks and emits static HTML to `dist/`. Adding Vitest to unit-test one presentational component would be disproportionate.

So the red/green loop asserts against **the built HTML in `dist/index.html`**. This is a real executable check — it catches wrong card counts, a missing EIRA media area, the wrong bezel on a project, and broken image paths.

The assertion script lives in the **scratchpad, not the repo**, so it leaves no maintenance burden:

```
/private/tmp/claude-501/-Users-katrihutabarat-Projects-stevensiahaan-github-io/baf0f5db-f01d-4937-9a40-be3d6db7c9b4/scratchpad/verify-work.mjs
```

Task 1 writes it. Tasks 2–4 run it. It is never committed.

---

## File Structure

| File | Responsibility |
| --- | --- |
| `src/components/work-card.astro` | **New.** Renders exactly one work card: media area (bezel + optional side shots) and body (num, name, desc, tags, arrow). Owns all `.work*`, `.phone`, `.screen`, `.side-shots` CSS. |
| `src/collections/projects.json` | **Modify.** Gains `frame`, `images`, `tags` per project. Becomes the single source of card data. |
| `src/pages/index.astro` | **Modify.** Work section renders `.work-grid` + `<WorkCard>`. Loses the `workTags`/`workBlurbs` maps and all `.work-*` list CSS. |

---

## Task 1: Write the verification script (red)

This task builds the failing check first. It must fail, because the card grid does not exist yet.

**Files:**
- Create: `<scratchpad>/verify-work.mjs` (scratchpad only — do NOT add to the repo)

- [ ] **Step 1: Write the assertion script**

Write to `/private/tmp/claude-501/-Users-katrihutabarat-Projects-stevensiahaan-github-io/baf0f5db-f01d-4937-9a40-be3d6db7c9b4/scratchpad/verify-work.mjs`:

```javascript
// Asserts the built Work section against the Studio Cards spec.
// Usage: pnpm build && node <scratchpad>/verify-work.mjs
import { readFileSync, existsSync } from "node:fs";
import { join } from "node:path";

const ROOT = "/Users/katrihutabarat/Projects/stevensiahaan.github.io";
const html = readFileSync(join(ROOT, "dist/index.html"), "utf8");

const failures = [];
const check = (label, actual, expected) => {
  if (actual !== expected) {
    failures.push(`${label}: expected ${expected}, got ${actual}`);
  }
};
const count = (re) => (html.match(re) ?? []).length;

// Structure
check("work cards", count(/<article[^>]*class="[^"]*\bwork\b/g), 8);
check("work-grid container", count(/class="[^"]*\bwork-grid\b/g), 1);
check("old work-list removed", count(/class="[^"]*\bwork-list\b/g), 0);

// Media areas: 7 projects have an image, EIRA has none.
check("media areas", count(/class="[^"]*\bwork-media\b/g), 7);

// Bezels: phone = Wellsy, Skul.id, Andal, Baboo.
// screen = Runchise, Hedz, WhatsApp. (EIRA is `wide` but imageless, so no bezel.)
check("phone bezels", count(/class="[^"]*\bphone\b/g), 4);
check("screen bezels", count(/class="[^"]*\bscreen\b/g), 3);

// No project has side shots yet (all have exactly 1 image).
check("side-shot rails", count(/class="[^"]*\bside-shots\b/g), 0);

// Andal Linkage and Baboo have url "#" -> rendered as <span>, not <a>.
check("linked names", count(/<a[^>]*class="[^"]*\bwork-name\b/g), 6);
check("unlinked names", count(/<span[^>]*class="[^"]*\bwork-name\b/g), 2);

// Every referenced project image must exist on disk in dist/.
for (const [, src] of html.matchAll(/<img[^>]*src="(\/assets\/images\/projects\/[^"]+)"/g)) {
  if (!existsSync(join(ROOT, "dist", src))) {
    failures.push(`missing image asset: ${src}`);
  }
}

// EIRA must render, but with no media area and no broken image.
if (!html.includes("EIRA")) failures.push("EIRA card is missing entirely");
if (html.includes("eira.png")) failures.push("EIRA still references the nonexistent eira.png");

if (failures.length > 0) {
  console.error("FAIL\n" + failures.map((f) => `  - ${f}`).join("\n"));
  process.exit(1);
}
console.log("PASS — Work section matches spec");
```

- [ ] **Step 2: Run it to verify it fails**

```bash
cd /Users/katrihutabarat/Projects/stevensiahaan.github.io
pnpm build && node "/private/tmp/claude-501/-Users-katrihutabarat-Projects-stevensiahaan-github-io/baf0f5db-f01d-4937-9a40-be3d6db7c9b4/scratchpad/verify-work.mjs"
```

Expected: **FAIL**, listing `work cards: expected 8, got 0`, `work-grid container: expected 1, got 0`, and `old work-list removed: expected 0, got 1`. This confirms the script is actually looking at the current, unmodified page.

Nothing to commit — the script is scratchpad-only.

---

## Task 2: Enrich `projects.json` with card data

**Files:**
- Modify: `src/collections/projects.json`

- [ ] **Step 1: Rewrite the file with `frame`, `images`, and `tags`**

Replace the entire contents of `src/collections/projects.json`. Note that EIRA's `image` becomes an **empty** `images` array — `/assets/images/projects/eira.png` does not exist on disk and was a live broken reference.

```json
[
	{
		"name": "WhatsApp Channel Automation",
		"description": "An automated WhatsApp channel delivering daily AI news and updates — content sourced, summarized, and published end-to-end with Hermes and WAHA.",
		"url": "https://whatsapp.com/channel/0029Vb5wchg60eBWja1uj83K",
		"frame": "wide",
		"images": ["/assets/images/projects/whatsapp-ai-channel.png"],
		"tags": ["Hermes", "WAHA", "Automation"]
	},
	{
		"name": "Hedz Psikologi",
		"description": "An online psychology assessment platform for intelligence, personality, and mental-health tests — from the Intelligence Structure Test to depression, anxiety, and stress screening — guided by professional psychologists.",
		"url": "https://hedzpsikologi.com/",
		"frame": "wide",
		"images": ["/assets/images/projects/hedz.png"],
		"tags": ["Web", "Psychology", "Assessment"]
	},
	{
		"name": "EIRA",
		"description": "Internal generative AI platform for Mitsubishi Group: conversational chat, RAG-powered regulation search, image generation, automated meeting minutes, and document translation.",
		"url": "https://eira.bsi.co.id/",
		"frame": "wide",
		"images": [],
		"tags": ["GenAI", "RAG", "Azure"]
	},
	{
		"name": "Wellsy",
		"description": "Digitises gym body composition, tracks progress, and surfaces AI-powered insights.",
		"url": "https://wellsy.site/",
		"frame": "phone",
		"images": ["/assets/images/projects/wellsy.png"],
		"tags": ["Flutter", "AI", "Health"]
	},
	{
		"name": "Runchise POS",
		"description": "Cross-platform point-of-sale for hundreds of franchises — Android, iOS, Windows.",
		"url": "https://play.google.com/store/apps/details?id=com.runchise.pos",
		"frame": "wide",
		"images": ["/assets/images/projects/runchise.jpg"],
		"tags": ["Flutter", "POS", "Bluetooth"]
	},
	{
		"name": "Skul.id",
		"description": "A school platform for attendance, grades, schedules & virtual classrooms.",
		"url": "https://play.google.com/store/apps/details?id=co.id.telkomsel",
		"frame": "phone",
		"images": ["/assets/images/projects/skulid.webp"],
		"tags": ["Flutter", "EdTech", "BLoC"]
	},
	{
		"name": "Andal Linkage",
		"description": "Mobile HRIS for location-based attendance, leave, permits and medical claims.",
		"url": "#",
		"frame": "phone",
		"images": ["/assets/images/projects/andal_linkage.webp"],
		"tags": ["Flutter", "HRIS", "Geo"]
	},
	{
		"name": "Baboo",
		"description": "A creative platform for reading, writing, and selling original stories & comics.",
		"url": "#",
		"frame": "phone",
		"images": ["/assets/images/projects/baboo.png"],
		"tags": ["Android", "Firebase", "Payments"]
	}
]
```

The `description` values for Wellsy, Runchise, Skul.id, Andal, and Baboo are the shorter card blurbs that previously lived in the `workBlurbs` map — they now live here, and the map gets deleted in Task 4.

- [ ] **Step 2: Confirm every referenced image exists**

```bash
cd /Users/katrihutabarat/Projects/stevensiahaan.github.io
node -e '
const ps = require("./src/collections/projects.json");
let bad = 0;
for (const p of ps) for (const i of p.images) {
  if (!require("fs").existsSync("public" + i)) { console.error("MISSING", i); bad++; }
}
console.log(bad ? "FAIL" : "PASS — all " + ps.length + " projects, images resolve");
process.exit(bad ? 1 : 0);'
```

Expected: `PASS — all 8 projects, images resolve`

- [ ] **Step 3: Commit**

```bash
git add src/collections/projects.json
git commit -m "feat: add frame, images, and tags to project data

Moves per-project card data out of the hardcoded workTags/workBlurbs
maps in index.astro. EIRA gets an empty images array, dropping the
reference to the nonexistent eira.png.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: Create the `work-card.astro` component

**Files:**
- Create: `src/components/work-card.astro`

This component is not yet rendered by anything — Task 4 wires it in. Creating it standalone keeps the commit focused.

- [ ] **Step 1: Write the component**

Create `src/components/work-card.astro`:

```astro
---
interface Props {
	name: string;
	description: string;
	url: string;
	frame?: "phone" | "wide";
	images?: string[];
	tags?: string[];
	index: number;
}

const {
	name,
	description,
	url,
	frame = "phone",
	images = [],
	tags = [],
	index,
} = Astro.props;

const [hero, ...sideShots] = images;
const isExternal = url.startsWith("http");
const num = String(index + 1).padStart(2, "0");

const mediaClass = [
	"work-media",
	frame === "wide" && "wide",
	sideShots.length === 0 && "solo",
]
	.filter(Boolean)
	.join(" ");
---

<article class="work">
  {
    hero && (
      <div class={mediaClass}>
        <div class={frame === "wide" ? "screen" : "phone"}>
          <img src={hero} alt={`${name} — screenshot`} loading="lazy" />
        </div>
        {sideShots.length > 0 && (
          <div class="side-shots">
            {sideShots.map((src) => (
              <img class="side-shot" src={src} alt="" loading="lazy" />
            ))}
          </div>
        )}
      </div>
    )
  }

  <div class="work-body">
    <div class="work-topline"><span class="work-num">{num}</span></div>

    <div class="work-name-row">
      {
        isExternal ? (
          <Fragment>
            <a class="work-name" href={url} target="_blank" rel="noopener">
              {name}
            </a>
            <a
              class="work-arrow"
              href={url}
              target="_blank"
              rel="noopener"
              aria-hidden="true"
              tabindex="-1"
            >
              →
            </a>
          </Fragment>
        ) : (
          <Fragment>
            <span class="work-name">{name}</span>
            <span class="work-arrow">→</span>
          </Fragment>
        )
      }
    </div>

    <p class="work-desc">{description}</p>

    <div class="work-tags">
      {tags.map((t) => <span class="tag">{t}</span>)}
    </div>
  </div>
</article>

<style>
  .work {
    display: flex;
    flex-direction: column;
    border: 1px solid var(--rule);
    border-radius: 14px;
    background: #fff;
    overflow: hidden;
    transition:
      border-color 0.2s ease,
      transform 0.2s ease,
      box-shadow 0.2s ease;
  }
  .work:hover {
    border-color: var(--ink);
    transform: translateY(-3px);
    box-shadow: 0 12px 32px rgba(14, 17, 22, 0.07);
  }
  .work:hover .work-name {
    color: var(--accent);
  }
  .work:hover .work-arrow {
    transform: translateX(6px);
    color: var(--accent);
  }

  /* ─── Media ─── */
  .work-media {
    display: grid;
    grid-template-columns: 1.1fr 1fr;
    gap: 20px;
    align-items: end;
    background: var(--surface);
    border-bottom: 1px solid var(--rule);
    padding: 32px 32px 0;
    height: 300px;
    overflow: hidden;
  }
  /* wide (dashboard / POS) framing */
  .work-media.wide {
    grid-template-columns: 1fr;
    grid-template-rows: 1fr auto;
  }
  /* solo: a single hero image, no side rail */
  .work-media.solo {
    grid-template-columns: 1fr;
    grid-template-rows: 1fr;
  }

  .phone {
    width: 100%;
    max-width: 190px;
    justify-self: center;
    height: calc(100% - 32px);
    align-self: end;
    border: 6px solid var(--ink);
    border-bottom: 0;
    border-radius: 26px 26px 0 0;
    background: #ededea;
    overflow: hidden;
    position: relative;
  }
  .screen {
    width: 100%;
    height: 100%;
    border: 5px solid var(--ink);
    border-bottom: 0;
    border-radius: 12px 12px 0 0;
    background: #ededea;
    overflow: hidden;
    position: relative;
  }
  .phone img,
  .screen img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: top center;
  }

  .side-shots {
    display: grid;
    grid-template-rows: 1fr 1fr;
    gap: 12px;
    height: calc(100% - 24px);
    align-self: end;
    padding-bottom: 24px;
  }
  .wide .side-shots {
    grid-template-rows: none;
    grid-template-columns: 1fr 1fr;
    height: 84px;
    padding-bottom: 24px;
  }
  .side-shot {
    width: 100%;
    height: 100%;
    min-height: 0;
    object-fit: cover;
    border-radius: 10px;
    border: 1px solid var(--rule-2);
  }

  /* ─── Body ─── */
  .work-body {
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 24px 28px 28px;
    flex: 1;
  }
  .work-topline {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: 16px;
  }
  .work-num {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--ink-mute);
    letter-spacing: 0.06em;
  }
  .work-name-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
  }
  .work-name {
    font-family: var(--serif);
    font-size: 28px;
    line-height: 1.05;
    transition: color 0.2s ease;
  }
  .work-desc {
    font-size: 14px;
    line-height: 1.6;
    color: var(--ink-soft);
    margin: 0;
  }
  .work-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    margin-top: auto;
    padding-top: 8px;
  }
  .work-arrow {
    font-size: 20px;
    color: var(--ink-mute);
    transition:
      transform 0.25s ease,
      color 0.2s ease;
  }

  @media (max-width: 600px) {
    .work-media {
      padding: 24px 20px 0;
      height: 240px;
      gap: 14px;
    }
    .work-body {
      padding: 20px 20px 24px;
    }
  }
</style>
```

**Note on the `.tag` class:** it is defined globally in `src/layouts/main.astro` and is intentionally NOT redefined here. Astro scopes component styles, so a local `.tag` rule would not be needed — the global one already applies.

- [ ] **Step 2: Typecheck**

```bash
cd /Users/katrihutabarat/Projects/stevensiahaan.github.io
pnpm build
```

Expected: build succeeds, 0 errors. (The component compiles even though nothing imports it yet.)

- [ ] **Step 3: Commit**

```bash
git add src/components/work-card.astro
git commit -m "feat: add work-card component for Studio Cards grid

Renders one project card with a device-framed screenshot. Adapts to
0-3 images: no images renders a text-only card, one renders the hero
bezel alone, two or three add the side-shot rail.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: Wire the grid into `index.astro` (green)

**Files:**
- Modify: `src/pages/index.astro` (frontmatter, Work section markup, `<style>` block)

- [ ] **Step 1: Import the component**

In the frontmatter of `src/pages/index.astro`, add the import alongside the existing ones:

```astro
import Layout from "../layouts/main.astro";
import WorkCard from "../components/work-card.astro";
```

- [ ] **Step 2: Delete the `workTags` and `workBlurbs` maps**

Remove these two `const` declarations from the frontmatter entirely. They are now dead — the data lives in `projects.json`.

```typescript
// DELETE both of these:
const workTags: Record<string, string[]> = { ... };
const workBlurbs: Record<string, string> = { ... };
```

Leave `expTags` and `expRewrites` alone — the Experience section still uses them and is out of scope.

- [ ] **Step 3: Replace the Work section markup**

Find the `<!-- ── Selected work ── -->` section. Replace the `<div class="work-list">…</div>` block (and everything inside it) with:

```astro
        <div class="work-grid">
          {
            projects.map((project, i) => (
              <WorkCard
                name={project.name}
                description={project.description}
                url={project.url}
                frame={project.frame as "phone" | "wide"}
                images={project.images}
                tags={project.tags}
                index={i}
              />
            ))
          }
        </div>
```

The surrounding `<section class="section" id="work">`, `<div class="frame">`, and `.section-head` markup stay exactly as they are.

- [ ] **Step 4: Replace the Work CSS**

In the `<style>` block, delete the entire `/* ─── Work list ─── */` group — every rule from `.work-list` through the `@media (max-width: 1100px)` block that hides `.work-desc` and `.work-tags`. All of it is superseded; the card's own styles live in `work-card.astro`.

Replace it with just the grid container:

```css
  /* ─── Work grid ─── */
  .work-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 24px;
  }
  @media (max-width: 900px) {
    .work-grid {
      grid-template-columns: 1fr;
    }
  }
```

- [ ] **Step 5: Build and run the verification script (expect green)**

```bash
cd /Users/katrihutabarat/Projects/stevensiahaan.github.io
pnpm build && node "/private/tmp/claude-501/-Users-katrihutabarat-Projects-stevensiahaan-github-io/baf0f5db-f01d-4937-9a40-be3d6db7c9b4/scratchpad/verify-work.mjs"
```

Expected: `PASS — Work section matches spec`

If it fails, the message names the exact mismatch (e.g. `phone bezels: expected 4, got 5`). Fix and re-run — do not proceed to Step 6 until it passes.

- [ ] **Step 6: Lint**

```bash
pnpm check
```

Expected: Biome reports no errors. It may reformat; if it does, include those changes in the commit.

- [ ] **Step 7: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: replace Work list with Studio Cards grid

Work section now renders a two-column card grid of WorkCard
components, each fronted by a device-framed screenshot. Removes the
name-keyed workTags/workBlurbs maps, which had silently lost EIRA and
Hedz Psikologi their tags.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 5: Drive the real page in a browser

The spec is explicit: *"Build success alone is not sufficient evidence — the page gets driven."* The `dist/` assertions prove structure; only a browser proves it *looks* right.

**Files:** none modified — this is verification.

- [ ] **Step 1: Start the preview server**

```bash
cd /Users/katrihutabarat/Projects/stevensiahaan.github.io
pnpm preview
```

Run it in the background. Note the URL it prints (typically `http://localhost:4321`).

- [ ] **Step 2: Screenshot at desktop width**

Use the `playwright-skill` (or the Playwright MCP tools) to navigate to `<url>/#work`, set viewport to **1280×900**, and screenshot the Work section.

Confirm by eye:
- Cards render **two per row**.
- Phone bezels (Wellsy, Skul.id, Andal Linkage, Baboo) are portrait, rounded at the top, bleeding off the bottom of the media area.
- Wide bezels (Runchise POS, Hedz, WhatsApp) are landscape.
- Single-image cards center their hero with **no empty gap** where a side rail would be.
- The **EIRA card renders text-only** — no media area, no broken-image icon, and it doesn't look visually broken sitting next to the image cards.

- [ ] **Step 3: Screenshot at mobile width**

Set viewport to **390×844** and screenshot again.

Confirm: the grid collapses to **one column**, and the media area shortens (240px) without the bezel overflowing its card.

- [ ] **Step 4: Confirm no failed image requests**

Check the browser's network requests for the page. Expected: **zero 404s** on `/assets/images/projects/*`.

- [ ] **Step 5: Report**

Report the outcome honestly, with the screenshots. If anything looks wrong, say so plainly rather than declaring success — a passing build and a passing script do not mean the section looks right.

---

## Done

At completion:
- The Work section is a two-column Studio Cards grid.
- `projects.json` is the single source of card data; adding a project is a JSON edit.
- The broken `eira.png` reference is gone; EIRA degrades to a text-only card.
- Dropping two more screenshots into any project's `images` array upgrades that card to the full side-rail design with **no code change**.

**Known follow-up (not in this plan):** EIRA has no screenshot. When one exists, add it to `images` and the card gains its media area automatically.
