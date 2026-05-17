# Requirements Document

## Introduction

The CV Download Section is a new section on the stevensiahaan.github.io personal portfolio site that lets visitors obtain a copy of Steven Siahaan's curriculum vitae (CV). The CV file is hosted externally on Google Drive at the URL `https://drive.google.com/file/d/1NxScqC2Rmtg1fqzbdhCtLR3XTApAR2CU/view?usp=sharing`. The feature surfaces a clearly labelled call-to-action that resolves to a direct download (or external preview) of the hosted file, integrated with the existing Astro-based design system (numbered section eyebrow, serif headings, monospace metadata, existing button styles).

## Glossary

- **Site**: The Astro static site published at `stevensiahaan.github.io`.
- **CV_Section**: The new section rendered on the home page (`src/pages/index.astro`) that contains the CV introduction copy and the CV_Download_Link.
- **CV_Download_Link**: The interactive anchor element inside the CV_Section that the visitor activates to obtain the CV.
- **CV_Source_URL**: The original Google Drive viewer URL `https://drive.google.com/file/d/1NxScqC2Rmtg1fqzbdhCtLR3XTApAR2CU/view?usp=sharing`.
- **CV_Direct_Download_URL**: The Google Drive direct-download form `https://drive.google.com/uc?export=download&id=1NxScqC2Rmtg1fqzbdhCtLR3XTApAR2CU` derived from the file ID in CV_Source_URL.
- **Visitor**: Any unauthenticated user of the Site, on desktop or mobile.
- **Navigation_Menu**: The site-wide menu defined in `src/collections/menu.json` and rendered by `src/components/header.astro`.
- **Design_System**: The existing visual language of the Site (CSS variables `--serif`, `--mono`, `--ink`, `--accent`, `--rule`, `--surface`; section pattern with numbered eyebrow, section-title, and `.frame` container; `.btn` styles).

## Requirements

### Requirement 1: Render the CV section on the home page

**User Story:** As a Visitor, I want a dedicated CV section on the home page, so that I can find and download Steven's CV without leaving the site.

#### Acceptance Criteria

1. THE Site SHALL render the CV_Section as a distinct `<section>` element on the home page (`/`).
2. THE CV_Section SHALL include a numbered section eyebrow, a section title, an introductory paragraph, and the CV_Download_Link, following the same structural pattern used by the About, Work, Capabilities, Experience, and Contact sections.
3. THE CV_Section SHALL have an HTML `id` attribute set to `cv` so that it can be linked to via the URL fragment `/#cv`.
4. THE CV_Section SHALL be positioned between the Experience section and the Contact section on the home page.

### Requirement 2: Provide a working CV download action

**User Story:** As a Visitor, I want a clearly labelled action to download the CV, so that I can save a local copy to review or share.

#### Acceptance Criteria

1. THE CV_Section SHALL contain exactly one CV_Download_Link as its primary call-to-action.
2. THE CV_Download_Link SHALL have its `href` attribute set to the CV_Direct_Download_URL.
3. WHEN a Visitor activates the CV_Download_Link, THE Site SHALL navigate the browser to the CV_Direct_Download_URL so that the browser initiates a file download or Google Drive preview.
4. THE CV_Download_Link SHALL open in a new browser tab by setting `target="_blank"` and SHALL include `rel="noopener"` to protect the originating context.
5. THE CV_Download_Link SHALL contain visible text that identifies the action as a CV download (for example, "Download CV").

### Requirement 3: Apply consistent visual design

**User Story:** As a Visitor, I want the CV section to look like the rest of the site, so that the experience feels cohesive.

#### Acceptance Criteria

1. THE CV_Section SHALL use the Design_System tokens (CSS variables `--serif`, `--mono`, `--ink`, `--ink-soft`, `--ink-mute`, `--rule`, `--surface`, `--accent`) for typography, color, and spacing.
2. THE CV_Section SHALL wrap its content in the existing `.frame` container so that horizontal alignment matches sibling sections.
3. THE CV_Download_Link SHALL adopt the existing `.btn` styling used by other primary call-to-action buttons on the home page.
4. THE section eyebrow inside the CV_Section SHALL use the next sequential two-digit number after the Experience section's eyebrow, formatted as `NN · CV` (for example, `05 · CV`).
5. WHERE the section eyebrow numbers in subsequent sections (such as Contact) follow CV_Section, THE Site SHALL renumber those eyebrows so that the sequence remains consecutive without gaps or duplicates.

### Requirement 4: Accessibility and semantics

**User Story:** As a Visitor using assistive technology or keyboard navigation, I want the CV download to be reachable and clearly described, so that I can use it without a mouse or visual cues.

#### Acceptance Criteria

1. THE CV_Download_Link SHALL be a native `<a>` element with a non-empty `href` attribute so that it is reachable through keyboard tab order.
2. THE CV_Download_Link SHALL include an accessible name that announces both the action and the file type, for example by setting `aria-label="Download CV (opens in new tab)"` or by including equivalent visible text.
3. THE CV_Section SHALL have a heading element (`<h2>`) that participates in the home page's heading outline at the same level as other section titles.
4. THE CV_Download_Link SHALL preserve the focus outline provided by the existing `.btn` styles so that keyboard focus is visible.

### Requirement 5: Responsive layout

**User Story:** As a Visitor on a mobile device, I want the CV section to display correctly, so that I can download the CV from a phone.

#### Acceptance Criteria

1. WHILE the viewport width is at most 720px, THE CV_Section SHALL render its eyebrow, title, body copy, and CV_Download_Link in a single column without horizontal scrolling.
2. WHILE the viewport width is greater than 720px, THE CV_Section SHALL use the same two-column section-head layout (eyebrow column plus title column) as the other sections on the home page.
3. THE CV_Download_Link SHALL remain fully visible and tappable, with a minimum touch target height of 40px, on viewports as narrow as 320px.

### Requirement 6: Configurable CV source

**User Story:** As the site owner, I want the CV source URL to live in one place, so that I can update the link to a new CV without hunting through markup.

#### Acceptance Criteria

1. THE Site SHALL store the CV_Source_URL (or its file ID) in a single source location, either as a constant in the page frontmatter, in `src/collections/menu.json`-style JSON data, or in a dedicated collections file.
2. WHEN the site owner changes the stored CV source value, THE Site SHALL render the updated CV_Direct_Download_URL on the next build without further code changes.
3. IF the stored CV source value is empty or missing, THEN THE Site SHALL fail the build with a descriptive error rather than rendering a broken link.

### Requirement 7: Build and deployment compatibility

**User Story:** As the site owner, I want the new section to ship through the existing GitHub Pages pipeline, so that no infrastructure changes are needed.

#### Acceptance Criteria

1. THE CV_Section SHALL be implemented entirely with the existing Astro toolchain (no new runtime dependencies beyond what is already in `package.json`).
2. WHEN `pnpm build` is executed, THE Site SHALL produce a static `dist/` output that includes the rendered CV_Section.
3. THE CV_Section source files SHALL pass the project's existing Biome lint and format checks.
