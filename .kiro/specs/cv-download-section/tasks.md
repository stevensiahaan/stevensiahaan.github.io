# Implementation Plan: CV Download Section

## Overview

Add a configurable CV download section between Experience and Contact on the home page. The implementation is fully static: a new JSON collection (`src/collections/cv.json`) holds the Drive file ID and copy, the home page imports and validates that data in its Astro frontmatter, and the rendered `<section id="cv">` reuses the existing `.section` / `.frame` / `.section-head` / `.btn` design tokens. Verification leans on the project's existing toolchain — `pnpm check` (Biome) and `pnpm build` (`astro check && astro build`) — which is also the GitHub Pages deploy gate. Per the design's Testing Strategy, no test framework is introduced and PBT is intentionally not used (configuration validation + UI rendering, no Correctness Properties section in the design).

## Tasks

- [x] 1. Set up CV collection data
  - [x] 1.1 Create `src/collections/cv.json` with the documented schema
    - Fields: `fileId` (`"1NxScqC2Rmtg1fqzbdhCtLR3XTApAR2CU"`), `sourceUrl` (the original Drive viewer URL), `label` (`"Download CV"`), `ariaLabel` (`"Download CV (opens in new tab)"`), `description` (one-paragraph blurb introducing the CV).
    - JSON shape mirrors `menu.json` / `experiences.json` so Astro auto-types the import in the page frontmatter.
    - _Requirements: 6.1, 6.2_

- [x] 2. Wire and render the CV section in `src/pages/index.astro`
  - [x] 2.1 Add CV import, missing-`fileId` guard, and download URL helper to the frontmatter
    - Import `cv` from `"../collections/cv.json"` alongside the existing `projects` / `experiences` imports.
    - Throw `Error("CV fileId is missing in src/collections/cv.json. Set it to the Google Drive file ID before building.")` when `cv.fileId` is missing, empty, or whitespace-only — Astro will surface the message during `astro check && astro build` and fail the deploy.
    - Define `buildDriveDownloadUrl(fileId: string): string` returning `` `https://drive.google.com/uc?export=download&id=${fileId}` `` and assign `cvDownloadUrl = buildDriveDownloadUrl(cv.fileId)`.
    - _Requirements: 6.1, 6.2, 6.3_

  - [x] 2.2 Insert `<section id="cv">` markup between the Experience and Contact sections
    - Use `<section class="section" id="cv">` wrapping a `<div class="frame">` with a `.section-head` (`<span class="section-eyebrow">05 · CV</span>`, `<h2 class="section-title">`) and a `<div class="cv-grid">` containing the description paragraph and the download button.
    - Render the button as `<a class="btn" href={cvDownloadUrl} target="_blank" rel="noopener" aria-label={cv.ariaLabel}>{cv.label} <span class="arrow">→</span></a>` so it inherits the existing `.btn` focus outline and touch target.
    - Render the description paragraph from `cv.description` inside `<p class="cv-blurb">`.
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.1, 2.2, 2.3, 2.4, 2.5, 3.2, 3.3, 4.1, 4.2, 4.3, 4.4, 5.3_

  - [x] 2.3 Renumber the Contact section eyebrow
    - Change `05 · Contact` to `06 · Contact` in the existing `<section class="contact" id="contact">` block so the eyebrow sequence stays consecutive (`01` About → `02` Work → `03` Capabilities → `04` Experience → `05` CV → `06` Contact).
    - _Requirements: 3.4, 3.5_

  - [x] 2.4 Append scoped CV layout CSS to the existing `<style>` block
    - Add `.cv-grid` (`display: grid; grid-template-columns: 1.4fr auto; gap: 64px; align-items: center;`).
    - Add `.cv-blurb` (`font-size: 17px; line-height: 1.65; color: var(--ink-soft); max-width: 56ch; margin: 0;`).
    - Add `@media (max-width: 900px) { .cv-grid { grid-template-columns: 1fr; gap: 32px; align-items: start; } }`.
    - Use only existing CSS variables (`--serif`, `--mono`, `--ink`, `--ink-soft`, `--ink-mute`, `--rule`, `--surface`, `--accent`); introduce no new tokens.
    - _Requirements: 3.1, 5.1, 5.2_

  - [x] 2.5 Verify the build-time validation path
    - Temporarily set `cv.fileId` to `""` in `src/collections/cv.json`, run `pnpm build`, and confirm it fails with the descriptive "CV fileId is missing…" error from task 2.1; then restore the original `fileId` value.
    - _Requirements: 6.3_

- [x] 3. Run lint and build gates
  - [x] 3.1 Run `pnpm check` (Biome) and resolve any findings on the new and modified files
    - Targets `src/collections/cv.json` and `src/pages/index.astro`.
    - _Requirements: 7.3_

  - [x] 3.2 Run `pnpm build` and confirm the produced `dist/index.html` includes the rendered CV section with the derived `href` and the renumbered Contact eyebrow
    - Inspect `dist/index.html` for `id="cv"`, the `https://drive.google.com/uc?export=download&id=…` href, and `06 · Contact`.
    - _Requirements: 7.1, 7.2_

- [x] 4. Final checkpoint
  - Ensure all build and lint gates pass, ask the user if questions arise.

## Notes

- Sub-tasks marked with `*` are optional and can be skipped without blocking the feature; 2.5 exercises the build-failure path rather than the happy path.
- Per the design's Testing Strategy, this feature deliberately ships without a test framework. `pnpm check` and `pnpm build` are the verification gates and double as the GitHub Pages deploy gate in `.github/workflows/deploy.yml`.
- No Property-Based Tests are listed because the design has no Correctness Properties section (the surface area is configuration validation plus static UI rendering, and the only pure helper is a single template literal). See the design's "PBT applicability assessment" for the full rationale.
- Sub-tasks 2.1, 2.2, 2.3, and 2.4 all modify `src/pages/index.astro` and are scheduled into separate waves so file writes do not conflict.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["2.1"] },
    { "id": 2, "tasks": ["2.2"] },
    { "id": 3, "tasks": ["2.3"] },
    { "id": 4, "tasks": ["2.4"] },
    { "id": 5, "tasks": ["2.5"] },
    { "id": 6, "tasks": ["3.1", "3.2"] }
  ]
}
```
