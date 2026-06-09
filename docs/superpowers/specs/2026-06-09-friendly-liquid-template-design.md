# Design: friendly.liquid — Styled ePI Viewer Template

**Date:** 2026-06-09  
**Status:** Approved

---

## Overview

Add a new Liquid template, `apps/visualiser/templates/friendly.liquid`, that renders FHIR ePI Composition resources with a clean, accessible visual style. It joins the existing templates (`epi.liquid`, `epi2.liquid`, `gh.liquid`, etc.) and is selectable from the Stylesheet dropdown in `visualiser/index.html`.

The template is fully self-contained: all CSS is embedded in a `<style>` block at the top of the template output. No new CSS files, no changes to `index.html`'s `<head>`.

---

## Visual Design

**Palette:** Green (`#16a34a` primary, `#f0fdf4` backgrounds, `#bbf7d0` borders, `#dcfce7` badge fills)  
**Typography:** `system-ui, -apple-system, sans-serif`; body text at 14px / 1.75 line-height  
**Radius:** 12px for accordion cards, 20px for pills, 8px for controls  
**Shadows:** Subtle (`0 1px 3px rgba(0,0,0,0.05)`), elevated on open item (`0 2px 8px rgba(22,163,74,0.12)`)

---

## Template Structure

The template output (injected into `<div id="result">`) has three sections:

### 1. Global `<style>` block

Scopes styles to the whole page, not just `#result`, so it can restyle the existing Bootstrap controls already rendered by `index.html`. Key rules:

| Selector | Purpose |
|---|---|
| `body` | Background `#f9fafb`, system font |
| `.container.mt-2` | The existing Bootstrap wrapper in `index.html` — restyled to look like a controls bar (white bg, bottom border, flex row) |
| `.container.mt-2 .btn`, `.dropdown-menu` | Replace Bootstrap button look with rounded, minimal selects |
| `.product-header` | Green bar below controls |
| `.accordion-item`, `.accordion-header`, `.accordion-body` | All accordion chrome |
| `.section-pill` | Green pill badge for section number/title prefix |
| `.chevron` | Animated SVG chevron (rotates 180° when open) |

CSS is vanilla — no preprocessor, no variables (for maximum browser compatibility with the no-build environment).

### 2. Product header bar

```
┌─────────────────────────────────────────┐
│ 💊  Metformin 500mg film-coated tablets │  ← Composition.title
│     Electronic Product Information      │
└─────────────────────────────────────────┘
```

- Background: `#16a34a`
- Product name from `{{ composition.title }}`
- Subtitle is the fixed string "Electronic Product Information"
- Pill icon (💊) is decorative and hardcoded

### 3. Accordion sections

Iterates `composition.section`. For each section that has subsections (`sec.section`), renders each subsection as an accordion item. Sections without subsections are rendered directly as accordion items.

**Accordion item anatomy:**

```
┌─────────────────────────────────────────┐  ← border-radius: 12px
│  [ 4.1 ]  Therapeutic indications   ▾  │  ← header (clickable)
├─────────────────────────────────────────┤  ← shown only when open
│  <section text from sec.text.div>       │  ← body
└─────────────────────────────────────────┘
```

- **Closed state:** white card, grey chevron pointing down, green pill with muted fill
- **Open state:** `#f0fdf4` header background, green chevron pointing up, solid green pill, green border and shadow on card
- **Section pill content:** the full `sec.title` string (or `sec2.title` for subsections). No numeric splitting — if the title is "4.1 Therapeutic indications", the entire string goes in the pill. FHIR section titles cannot be reliably split by prefix.

**Toggle mechanism:** Pure CSS + inline `onclick` on each header (`this.parentElement.classList.toggle('open')`). No jQuery dependency, no Bootstrap JS needed.

**Body content:** `{{ sec2.text.div }}` for subsections, `{{ sec.text.div }}` for top-level sections rendered directly. Output is raw HTML (same as existing templates). Embedded `<style>` from the FHIR narrative is accepted as-is.

---

## Interaction with `index.html`

`index.html` renders its Stylesheet/Language dropdowns before `<div id="result">`. The `<style>` block from `friendly.liquid` — injected into the DOM inside `#result` — applies globally and reskins those dropdowns to match the green design system. This is intentional: browsers accept `<style>` elements inside `<body>` and apply them page-wide.

The Liquid template receives the URL parameters (`mpid`, `language`, `baseurl`, `stylesheet`) through `index.html`'s existing JS, which fetches the Composition bundle and calls `engine.parseAndRender(ltemplate, { json })`. No changes to `index.html` are needed.

---

## Files Changed

| File | Change |
|---|---|
| `apps/visualiser/templates/friendly.liquid` | **Create** — new self-contained template |
| `apps/visualiser/index.html` | **Add** `friendly.liquid` as a dropdown option in the Stylesheet menu |

No other files are modified.

---

## Out of Scope

- Changes to `epi.liquid`, `gh.liquid`, or any existing template
- Changes to `prodbrowser.html` or the product list page
- Handling of embedded `Binary` resources / base64 images (follow the pattern in `gh.liquid` if needed later)
- Internationalisation of the "Electronic Product Information" subtitle string
