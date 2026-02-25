---
title: "feat: Build 5 Analytic Tests Knowledge Library Piece"
type: feat
date: 2026-02-25
---

# Build "5 Analytic Tests" Knowledge Library Piece

## Overview

Create an interactive HTML presentation and print-ready PDF companion to the existing Data Treatment Process artifact. The piece walks prospects through 5 analytic test scenarios that demonstrate Third Horizon's data quality, treatment methodology, and credibility in healthcare price transparency. Each test follows a "trap hook + test proof" narrative: name the common data challenge, pose a provocative question, show how Third Horizon addresses it, and provide an illustrative data snapshot as evidence.

## Problem Statement / Motivation

The BD team needs consistent, client-focused sales enablement materials to support prospect conversations and show-and-tells. The existing Data Treatment Process artifact explains *how* the data pipeline works, but there is no companion piece that demonstrates *what the data actually proves* — the kind of "show me the receipts" credibility that fast-tracks prospect decisions. The 5 analytic tests are already documented internally (Analytic Test Responses document + supporting workbook), but they exist as dense technical write-ups, not presentation-ready materials. Peter at MMA also needs a copy for MMA's own knowledge library.

## Proposed Solution

A single self-contained HTML file (`five-analytic-tests.html`) that functions as:
1. **Interactive slide presentation** — keyboard/click navigation, one test per slide, for live screen shares
2. **Print/PDF export** — `Cmd+P` produces a clean multi-page document for leave-behinds

### Content Architecture: 7 Slides

| # | Slide | Content |
|---|-------|---------|
| 0 | **Title** | Headline, subtitle, brief setup paragraph, Third Horizon attribution |
| 1 | **Test A: The Modifier Trap** | Radiology — professional vs. technical component separation |
| 2 | **Test B: The Site of Service Trap** | Orthopedics — same surgeon, different facility setting |
| 3 | **Test C: The Ghost Rate Check** | Pathology/Psychiatry — filtering impossible service-provider combinations |
| 4 | **Test D: The Inpatient Reality Check** | DRG — real facility case rates from payer data |
| 5 | **Test E: The Outpatient Case Rate Check** | Base rate vs. bundled case rate distinction |
| 6 | **Closing / CTA** | Summary takeaway, next steps prompt, contact information |

### Per-Test Slide Structure (Slides 1-5)

Each test slide contains 5 content sections in a consistent layout:

```
+----------------------------------------------------------+
|  THE MODIFIER TRAP                            Slide 1/5  |
|  Can your data vendor actually separate the               |
|  professional and technical components?                   |
|----------------------------------------------------------|
|                          |                                |
|  THE CHALLENGE           |  HOW WE ADDRESS IT             |
|  [2-3 sentences on the   |  [2-3 sentences on Third      |
|   industry data problem]  |   Horizon's approach]         |
|                          |                                |
|----------------------------------------------------------|
|  THE EVIDENCE                                             |
|  [Mini data table: 3-7 representative rows from workbook] |
|  Source: [attribution] | Data as of [quarter/year]        |
|----------------------------------------------------------|
|  WHY IT MATTERS                                           |
|  [1-2 sentence implication for brokers/employers]         |
|----------------------------------------------------------|
|  [> See full data] (toggle, hidden until populated)       |
+----------------------------------------------------------+
```

### Illustrative Data Snapshots Per Test

Drawn from the Analytic Test Sample Data workbook. Each snapshot shows 3-7 representative rows:

**Test A — The Modifier Trap:**
Show 3-4 providers (1 hospital, 2-3 radiologists) with their no-modifier, -26, and -TC rates side by side. Point: modifiers are separated and rates differ meaningfully.

**Test B — The Site of Service Trap:**
Show 2-3 surgeons with rates across POS 21/22 (Hospital) and POS 24 (ASC). Point: place of service is captured per the CMS schema, enabling facility-type comparison.

**Test C — The Ghost Rate Check:**
Show the split: pathologists/psychiatrists have 99213 rates (rare but real billing) but zero CABG rates (true ghost rate eliminated). Point: claims-based validation filters impossible combinations without discarding legitimate rare services.

**Test D — The Inpatient Reality Check:**
Show 3-5 academic medical centers with MS-DRG 470 rate ranges and carrier coverage. Point: real DRG-level negotiated rates exist and show meaningful variation across payers.

**Test E — The Outpatient Case Rate Check:**
Show 2-3 facilities with base negotiated rate vs. calculated case rate, with flags for "likely case rate" and "insufficient data." Point: we distinguish and flag rate types rather than presenting them as equivalent.

## Technical Approach

### Architecture

Single self-contained HTML file following the established pattern in this repo (`network-navigator-water-map.html`):
- All CSS inline in `<style>` block
- All JS inline in `<script>` block
- No external dependencies except optional Google Fonts (with system font fallback)
- CSS custom properties for easy re-theming when Starset brand is applied

### Data Model

```javascript
/** @typedef {Object} Test
 * @property {string} id          - kebab-case slug (e.g., "modifier-trap")
 * @property {string} letter      - test letter ("A"-"E")
 * @property {string} trapName    - display name (e.g., "The Modifier Trap")
 * @property {string} hookQ       - provocative question
 * @property {string} challenge   - industry data problem (2-3 sentences)
 * @property {string} approach    - Third Horizon's approach (2-3 sentences)
 * @property {string} implication - why it matters to brokers/employers (1-2 sentences)
 * @property {Array}  evidence    - array of row objects for the data snapshot table
 * @property {Array}  columns     - column definitions for the evidence table
 * @property {string} source      - data source attribution
 * @property {string} dataDate    - data freshness label (e.g., "Q4 2025")
 * @property {number} order       - 1..5
 */
```

### Navigation System

- **Keyboard:** Left/Right arrow keys advance slides; clamp at boundaries (no wrapping)
- **Click:** Slide indicator dots/numbers in a fixed nav bar
- **Hash routing:** `#slide=modifier-trap`, `#slide=site-of-service-trap`, etc. Falls back to title slide on invalid hash
- **Transitions:** CSS opacity/transform transition (180ms ease, matching existing artifact). Use View Transition API where supported, with graceful fallback
- **No pin/lock:** Unlike the water treatment diagram (which uses pin to prevent hover-triggered changes), the slide model is intentional-navigation-only — pin is unnecessary

### Print/PDF Strategy

```css
@media print {
  /* Show all slides, one per page */
  .slide { display: block; break-after: page; }

  /* Hide interactive-only elements */
  .slide-nav, .toggle-detail, .skip-link { display: none; }

  /* Omit expandable detail sections (empty in v1) */
  .expandable-section { display: none; }

  /* Force data tables to stay on one page */
  .evidence-table { break-inside: avoid; }

  /* Print header/footer via @page */
  @page { size: letter; margin: 0.6in; }
}
```

### Accessibility

Match the patterns established in `network-navigator-water-map.html`:
- Skip link to slide content
- `aria-roledescription="slide"` on each section
- `aria-live="polite"` region for slide change announcements
- `aria-hidden="true"` on non-active slides
- `focus-visible` outlines (3px solid, 2px offset)
- `prefers-reduced-motion: reduce` media query
- Semantic `<table>` markup with `<caption>`, `<th scope>` for all evidence tables
- Screen-reader-only utility class for announcement text

### CSS Design System (Skeleton)

Start with the existing blue palette from the water treatment artifact as a neutral skeleton. CSS variables make re-theming to Starset brand a single-pass edit:

```css
:root {
  --primary: #1E40AF;
  --secondary: #3B82F6;
  --accent: #F59E0B;
  --bg: #F8FAFC;
  --text: #1E3A8A;
  --surface: #FFFFFF;
  --radius: 16px;
  --transition: 180ms ease;
  /* Swap these when Starset brand is applied */
}
```

Typography: System font stack for offline reliability (`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`), with Google Fonts Fira Sans/Code as progressive enhancement if online.

## Implementation Phases

### Phase 1: HTML Skeleton + Navigation (Core Structure)

Build the slide container, navigation system, and empty slide templates.

**Tasks:**
- [x] Create `five-analytic-tests.html` with document structure, CSS variables, base typography
- [x] Build 7 slide sections (title + 5 tests + CTA) with placeholder content
- [x] Implement keyboard navigation (arrow keys, clamp at boundaries)
- [x] Implement click navigation (slide indicator bar)
- [x] Implement hash routing (`#slide=<slug>`) with fallback to title
- [x] Add slide transition (opacity/transform, 180ms)
- [x] Add `@media print` rules: all slides visible, page breaks, hide nav
- [x] Add accessibility: skip link, ARIA roles, live region, focus management, reduced motion

**Files:**
- `five-analytic-tests.html` (new)

### Phase 2: Content + Data Snapshots (The Substance)

Write the narrative content and wire in the illustrative data from the workbook.

**Tasks:**
- [x] Write title slide content (headline, subtitle, setup paragraph)
- [x] Write Test A content: challenge, approach, implication, evidence table (4-5 rows from Test A worksheet)
- [x] Write Test B content: challenge, approach, implication, evidence table (4-5 rows from Test B worksheet)
- [x] Write Test C content: challenge, approach, implication, evidence table (5-6 rows showing 99213 present / 33533 absent split)
- [x] Write Test D content: challenge, approach, implication, evidence table (5 rows — one per academic medical center)
- [x] Write Test E content: challenge, approach, implication, evidence table (4-5 rows showing base vs. case rate flags)
- [x] Write CTA slide content (summary, next step, contact info placeholder)
- [x] Add source attribution and data freshness labels to each evidence table
- [ ] Review all content for client-safe language (no internal jargon, conservative data claims)

**Source files (read-only reference):**
- `/Users/topher416/Downloads/Analytic Test Responses (5).docx`
- `/Users/topher416/Downloads/Analytic Test Sample Data (5).xlsx`

### Phase 3: Expandable Detail Architecture (Future-Proofing)

Add the structural hooks for option (c) full evidence panels, without populating them yet.

**Tasks:**
- [x] Add `<details>` / `<summary>` expandable section to each test slide
- [x] Hide toggle via CSS class (`.detail-hidden { display: none }`) — toggle becomes visible when `data-has-detail="true"` is set
- [x] Style the expandable section to match card/detail-block patterns
- [x] Ensure print CSS omits expandable sections entirely
- [x] Test that hidden toggles do not affect layout or accessibility

### Phase 4: Visual Polish + Brand Application (Deferred)

Apply Starset brand identity once logo/colors are provided.

**Tasks:**
- [ ] Obtain Starset logo, color palette, and any typography guidelines
- [ ] Update CSS variables to match Starset brand
- [ ] Add logo to title slide and/or slide nav
- [ ] Add print header/footer with branding and page numbers
- [ ] Prepare MMA-branded variant if needed (CSS variable swap or separate file)
- [ ] Final visual QA across screen + print

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Single HTML file, no build tools | Matches existing repo pattern; portable for email/sharing |
| System fonts with Google Fonts as progressive enhancement | Offline reliability for local file presentation |
| No pin/lock behavior | Slide navigation is intentional (no hover preview to pin against) |
| Hide expandable toggles until populated | Prevents empty/broken UI during live presentations |
| Clamp at slide boundaries (no wrap) | Prevents jarring wrap-around during presentations |
| Hash routing with descriptive slugs | Consistent with existing `#stage=water-intake` pattern; readable URLs |
| Print omits expandable sections | They're empty in v1; print only shows confirmed content |
| Per-slide "Why It Matters" section | Matches water treatment artifact's "value to brokers/employers" pattern; prevents BD team from improvising inconsistently |

## Open Questions Requiring Input

### Critical (Blocks Content Creation)

1. **Data anonymization:** Should the evidence tables use real provider names/NPIs from the workbook, or anonymize to "Provider A / Carrier X" with real rate values? Real names are more credible but create legal/competitive exposure if the PDF circulates beyond the prospect.

2. **Title slide and CTA content:** What headline, subtitle, and call-to-action should appear? Suggested defaults:
   - Title: "5 Tests Your Price Transparency Data Should Pass"
   - Subtitle: "A Third Horizon Analytics Knowledge Brief"
   - CTA: "Want to see your markets run through these tests? [Contact]"

### Important (Affects Dual-Audience Use)

3. **MMA co-branding:** Does Peter at MMA need an MMA-branded version, or will the Starset/Third Horizon version work for their library? The existing artifacts in this repo use "MMA Network Navigator" branding — is that relationship changing?

4. **Link to water treatment piece:** Should the interactive version include a link or reference to the Data Treatment Process artifact, or should they remain standalone?

## Dependencies & Risks

| Risk | Mitigation |
|------|------------|
| Data sensitivity — real rates in leave-behinds | Resolve anonymization decision before Phase 2 |
| Stale data in circulating PDFs | Include "Data as of [date]" on every evidence table |
| Starset brand not yet available | Build skeleton with neutral/blue palette; CSS variables enable single-pass re-theme |
| Expandable detail clicked before content exists | Toggle hidden by default; only shown when `data-has-detail="true"` |
| Offline font loading | System font stack as primary; Google Fonts as enhancement |
| Prospect views on mobile | Responsive grid collapses to single column; data tables scroll horizontally |

## Success Criteria

- [ ] BD team can present all 5 tests in a live screen share with keyboard navigation
- [ ] `Cmd+P` produces a clean, professional PDF with one test per page
- [ ] Each test slide clearly communicates challenge, solution, and evidence in under 60 seconds of narration
- [ ] Peter at MMA receives a file he can include in the MMA knowledge library
- [ ] Starset brand can be applied by updating CSS variables only (no structural HTML changes)
- [ ] All accessibility patterns from the water treatment artifact are maintained

## References

### Internal
- Existing artifact: `/Users/topher416/tic-water/network-navigator-water-map.html` (design system, interaction patterns)
- Source content: `/Users/topher416/Downloads/Analytic Test Responses (5).docx`
- Source data: `/Users/topher416/Downloads/Analytic Test Sample Data (5).xlsx`
- Brainstorm: `docs/brainstorms/2026-02-25-five-analytic-tests-knowledge-library-brainstorm.md`

### Content Summary (from source documents)

**Test A — The Modifier Trap (Radiology):**
CPT 70450, NY, Aetna. Data shows all modifiers (-26, -TC, global) and multiple POS are available for all providers. Some NPI/modifier combos have non-unique rates (multiple TINs or POS differentiation). Key point: modifiers are properly separated.

**Test B — The Site of Service Trap (Orthopedics):**
CPT 27447, MI, BCBS/Cigna. Top 5 ortho surgeons across POS 21, 22, 24. Rates are the same across facility-based POS (expected — Medicare MPFS shows this too). Only 1 of 5 providers has Cigna rates. Key point: POS is captured per CMS schema.

**Test C — The Ghost Rate Roster Check:**
CPT 99213 + 33533, CA, all plans. Top 10 pathologists + 10 psychiatrists. CABG (33533) rates do not exist for these providers — filtered by claims-based validation. 99213 rates exist for pathologists (rare but real — 65% of psychiatrists bill it). Key point: true ghost rates eliminated, rare-but-legitimate services retained, competent querying is essential.

**Test D — The Inpatient Reality DRG Check:**
MS-DRG 470, 5 academic medical centers (NYP Columbia, Yale New Haven, UCI, NYU Langone Tisch, Duke). Negotiated rates present across multiple carriers. Wide range: ~$13.8K to ~$97.9K. Key point: real DRG-level facility case rates exist in the data.

**Test E — The Outpatient Case Rate Check:**
CPT 27447, IN, IU Health / MHP / Community Hospital North. Shows base negotiated rate and calculated case rate (summing co-occurring J-codes at 60%+ episode frequency). Flags: "likely case rate," "insufficient data to calculate." Key point: base vs. bundled rates are distinguished and transparently flagged.
