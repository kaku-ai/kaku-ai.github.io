# Test Coverage Analysis

## Current State

This repository has **zero test coverage**. There are no test files, no testing frameworks, no CI/CD pipelines, and no `package.json` to manage tooling. The codebase is a static HTML/CSS site (5 HTML pages, 1 CSS file, ~1,558 lines total) served via GitHub Pages.

---

## Recommended Testing Areas

### 1. HTML Validation

**Priority: High**

None of the 5 HTML files are validated against the W3C HTML specification. Invalid HTML can cause inconsistent rendering across browsers and assistive technologies.

**What to test:**
- All pages pass the [W3C Nu HTML Checker](https://validator.w3.org/nu/)
- Proper nesting of elements
- No duplicate IDs
- Required attributes present on all elements

**Tool:** [html-validate](https://html-validate.org/) or [htmlhint](https://htmlhint.com/) as a local linter, runnable via npm.

---

### 2. CSS Validation

**Priority: Medium**

`style.css` (765 lines) uses modern CSS features (custom properties, `backdrop-filter`, `accent-color`) that should be validated for syntax correctness and checked for browser compatibility.

**What to test:**
- Valid CSS syntax across the entire stylesheet
- No undefined or misspelled custom properties
- Vendor-prefixed properties (`-webkit-backdrop-filter`, `-webkit-background-clip`, `-webkit-text-fill-color`) have standard equivalents where applicable

**Tool:** [stylelint](https://stylelint.io/) with standard config.

---

### 3. Accessibility (a11y)

**Priority: High**

The site has basic semantic HTML (header, nav, section, footer) and one `aria-label` on the nav toggle, but there are gaps:

- **`opt-in.html`**: Form inputs lack `required` attributes, and there are no `aria-describedby` associations between inputs and the disclaimer text. Screen readers will not communicate that fields are mandatory.
- **All pages**: The mobile nav toggle (`button.nav__toggle`) has an `aria-label` but no JavaScript to handle `aria-expanded` state, meaning it is non-functional for all users.
- **Color contrast**: The `--text-lighter: #94A3B8` color used for footer copy and disclaimers on a `--bg-light: #F8FAFC` background may fail WCAG AA contrast ratios (4.5:1 for normal text).
- **`index.html`**: The gradient text on `.gradient-text` using `-webkit-text-fill-color: transparent` may report as invisible to some accessibility checkers.

**What to test:**
- All pages meet WCAG 2.1 AA standards
- All form inputs have associated labels (present) and required/aria attributes
- Color contrast ratios pass for all text/background combinations
- Interactive elements are keyboard-navigable
- Page landmark structure is correct

**Tool:** [axe-core](https://github.com/dequelabs/axe-core) via [pa11y](https://pa11y.org/) or Playwright's accessibility testing.

---

### 4. Link Integrity

**Priority: High**

The site contains internal cross-page links and external links to third-party privacy policies. Broken links erode trust, especially on legal compliance pages.

**Specific links to verify:**
- Internal: `index.html`, `privacy.html`, `terms.html`, `sms-terms.html`, `opt-in.html` (all cross-referenced from every footer)
- External: `supabase.com/privacy`, `anthropic.com/privacy`, `twilio.com/legal/privacy` (in `privacy.html`)
- Mailto: `support@kaku.app` (appears in every footer and multiple legal pages)
- Anchor links: `#features`, `#how-it-works` (in `index.html`)

**What to test:**
- All internal links resolve to existing files
- All external links return HTTP 200
- All anchor links (`#id`) reference existing element IDs
- No orphan pages (pages that exist but aren't linked from anywhere)

**Tool:** [linkinator](https://github.com/JustinBeckwith/linkinator) or [broken-link-checker](https://github.com/stevenvachon/broken-link-checker).

---

### 5. Responsive Layout

**Priority: High**

`style.css` has three breakpoints (`1024px`, `768px`, `480px`) with significant layout changes (grid columns collapse, nav hides, footer stacks). These are untested.

**What to test:**
- `features__grid` renders 3 columns > 1024px, 2 columns at 768-1024px, 1 column < 768px
- `steps` grid renders 4 columns > 1024px, 2 columns at 768-1024px, 1 column < 768px
- Navigation links are hidden and hamburger toggle is visible below 768px
- Hero buttons stack vertically below 480px
- Footer stacks to column layout below 768px
- `optin-card` form remains usable at all viewport widths

**Tool:** [Playwright](https://playwright.dev/) with viewport emulation for visual regression and layout assertions.

---

### 6. Content Correctness

**Priority: High**

There is a **branding error** in the codebase:

- **`terms.html:121`** references "PLANLY" instead of "Kaku" in the Limitation of Liability section: *"IN NO EVENT SHALL **PLANLY**, ITS OFFICERS..."*. This is likely a copy-paste artifact from a template and is legally significant since it names the wrong entity.

**What to test:**
- No occurrences of "Planly" or other placeholder/template brand names across all pages
- Consistent use of "Kaku" as the brand name
- "Last updated" dates are consistent across legal pages
- Legal pages contain all required TCPA/SMS compliance language (STOP, HELP, message frequency, data rates)
- Footer content is identical across all pages

**Tool:** Custom grep-based assertions or a simple Node.js script.

---

### 7. SEO and Meta Tags

**Priority: Medium**

Each page has `<title>` and `<meta name="description">` tags, which is good. However, there's no structured data, no Open Graph tags, and no canonical URLs.

**What to test:**
- Every page has a unique `<title>` and `<meta name="description">`
- `<html lang="en">` is present on all pages
- Heading hierarchy is correct (single `<h1>` per page, no skipped levels)
- Favicon is present and valid

**Tool:** [html-validate](https://html-validate.org/) with SEO rules, or a custom Playwright script.

---

### 8. Form Validation (opt-in.html)

**Priority: Medium**

The opt-in form at `opt-in.html:49-79` has `action="#"` (non-functional) and inputs with no `required` attributes. There is no client-side validation and no JavaScript form handler.

**What to test:**
- All form fields (`name`, `email`, `phone`, `sms_consent` checkbox) have the `required` attribute
- `type="email"` and `type="tel"` inputs enforce proper format via browser validation
- The SMS consent checkbox must be checked before submission
- Form submission is handled (currently non-functional)
- Placeholder text doesn't serve as the only field label (labels are present, so this is fine)

**Tool:** Playwright form interaction tests.

---

### 9. Performance / Lighthouse

**Priority: Low**

The site is lightweight (no JavaScript, single CSS file, Google Fonts as only external dependency), so performance should be good. Testing confirms the baseline and catches regressions.

**What to test:**
- Lighthouse performance score >= 90
- Lighthouse accessibility score >= 90
- No render-blocking resources besides the single CSS file
- Google Fonts loaded with `display=swap` (present)
- Font preconnect hints present (present)

**Tool:** Lighthouse CI or [unlighthouse](https://unlighthouse.dev/).

---

## Bugs Found During Analysis

| Severity | File | Line | Issue |
|----------|------|------|-------|
| **High** | `terms.html` | 121 | Brand name "PLANLY" used instead of "Kaku" in Limitation of Liability |
| **Medium** | `opt-in.html` | 49-78 | Form has no `required` attributes on inputs and checkbox |
| **Medium** | All pages | - | Mobile nav toggle has no JavaScript handler; hamburger button is non-functional |
| **Low** | `style.css` | 84-86 | `backdrop-filter` has no fallback for browsers without support |

---

## Suggested Implementation Plan

### Phase 1: Add tooling and CI

1. Initialize `package.json`
2. Add `htmlhint` for HTML linting
3. Add `stylelint` for CSS linting
4. Add `pa11y` for accessibility checks
5. Add `linkinator` for link checking
6. Add a GitHub Actions workflow to run all checks on push/PR

### Phase 2: Add browser-based tests

1. Add Playwright
2. Write responsive layout tests at each breakpoint
3. Write content correctness assertions (brand name consistency, required legal text)
4. Write form validation tests for `opt-in.html`

### Phase 3: Regression and monitoring

1. Add Lighthouse CI to the GitHub Actions workflow
2. Add visual regression testing via Playwright screenshots
3. Add scheduled link checking (external links can break over time)
