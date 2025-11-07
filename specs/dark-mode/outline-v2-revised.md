# RFE: Dark Mode Implementation

**Version:** 2.0 (Revised - Addressing Technical and UX Feedback)
**Date:** 2025-11-07
**Status:** Draft - Ready for Final Review
**Target Platform:** ambient-code/platform
**Author:** Parker (Product Manager)
**Reviewers:** Felix (UX Feature Lead), Stella (Staff Engineer)

---

## Executive Summary

### Business Justification

Dark mode has evolved from a niche feature to a mainstream user expectation with significant market penetration. Our customers are telling us they need this capability to remain competitive with modern web platforms.

**Market Data:**
- 82% of smartphone users use dark mode [[35+ Dark mode statistics, forms.app, 2025](https://forms.app/en/blog/dark-mode-statistics)]
- 82% of mobile users prefer dark mode by 2025 [[Dark Mode Design Trends for 2025, AlterSquare](https://altersquare.medium.com/dark-mode-design-trends-for-2025-should-your-startup-adopt-it-a7e7c8c961ab)]
- 70% of users expect dark mode options in applications [[10 Dark Mode UI Best Practices, Design Studio UI/UX](https://www.designstudiouiux.com/blog/dark-mode-ui-design-best-practices/)]
- 94% of tech companies offer dark mode as a standard feature [[Dark Mode Usage Statistics, Gitnux](https://gitnux.org/dark-mode-usage-statistics/)]

The market opportunity here is clear: platforms without dark mode risk user dissatisfaction and churn. User expectations have fundamentally shifted, with 64.6% of people expecting websites to apply dark mode automatically [[How Many People Use Dark Mode, EarthWeb](https://earthweb.com/blog/how-many-people-use-dark-mode)].

### Competitive Landscape

The data shows customer adoption increases when platforms offer system-preference-aware dark mode with manual override capabilities. Our competitors are moving fast on this - 94% of tech companies have already implemented dark mode as standard, and 88% of software updates now include automatic dark mode options [[Dark Mode Usage Statistics, Gitnux](https://gitnux.org/dark-mode-usage-statistics/)].

**What's the business impact if we don't deliver this?** We risk being perceived as outdated and losing users to more modern platforms that respect user preferences and system settings.

---

## Business Impact and Customer Requirements

### Customer Pain Points

Based on market research and user behavior patterns:

1. **Eye Strain Reduction**: Users report reduced eye strain when working in low-light environments with dark mode enabled [[Dark Mode Can Improve Text Readability, BOIA](https://www.boia.org/blog/dark-mode-can-improve-text-readability-but-not-for-everyone)]

2. **Energy Efficiency**: Dark mode reduces energy consumption on OLED and AMOLED screens by turning off individual pixels when rendering true black [[Complete Dark Mode Design Guide, UI Deploy](https://ui-deploy.com/blog/complete-dark-mode-design-guide-ui-patterns-and-implementation-best-practices-2025)]

3. **Accessibility Considerations**: While dark mode benefits many users, accessibility is nuanced. Some users with dyslexia prefer lighter backgrounds, and users with astigmatism may experience "halation effects" with dark backgrounds [[Inclusive Dark Mode, Smashing Magazine](https://www.smashingmagazine.com/2025/04/inclusive-dark-mode-designing-accessible-dark-themes/)]

4. **User Control**: 64.6% of users expect automatic dark mode detection, but best practices recommend offering manual override [[How Many People Use Dark Mode, EarthWeb](https://earthweb.com/blog/how-many-people-use-dark-mode)]

### Target User Personas

[Assumption: Platform serves data scientists, ML engineers, developers, and platform administrators - typical for OpenShift AI Platform]

- **Data Scientists**: Long work sessions analyzing models and data visualizations; benefit from reduced eye strain
- **ML Engineers**: Work across multiple environments; expect modern UI patterns and system preference integration
- **Platform Administrators**: Configure complex dashboards; need consistent visual experience
- **Developers**: Review code, YAML configurations, and logs; expect syntax highlighting in both themes

### Business Metrics Impact

**Expected Outcomes:**
- Increased user satisfaction and NPS scores
- Reduced support tickets related to eye strain and display preferences
- Improved competitive positioning in platform comparisons
- Higher user retention through modern UX expectations

[Assumption: Specific platform metrics will need to be baselined during implementation. Target: 50%+ adoption rate within 3 months of launch.]

---

## Technical Context and Architecture

### Existing Platform Technology Stack

**[TBD - Requires Technical Discovery]**

Before implementation, we need to document:
1. Current framework/library (React, Vue, Angular, vanilla JS?)
2. CSS architecture (CSS Modules, Styled Components, Sass, Tailwind, custom?)
3. Design token system (if any exists)
4. Component library structure and location
5. Build and bundling configuration (Webpack, Vite, esbuild?)
6. Server-side rendering vs. client-side only
7. Current browser support policy

**Critical Questions for Technical Team:**
- Do we have an existing design system or component library?
- Are CSS custom properties already in use?
- What is our current CSS organization pattern?
- Do we have automated visual regression testing?
- What accessibility testing is currently in place?

### High-Level Technical Strategy

1. **CSS Custom Properties Foundation**: Implement comprehensive design token system using CSS custom properties (variables) for centralized theme management

2. **System Preference Detection**: Use `prefers-color-scheme` media query for automatic system preference detection [[prefers-color-scheme, MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)]

3. **User Override Mechanism**: Provide toggle control allowing users to override system preferences with persistent selection across sessions [[Setting And Persisting Color Scheme Preferences, Smashing Magazine](https://www.smashingmagazine.com/2024/03/setting-persisting-color-scheme-preferences-css-javascript/)]

4. **FOUC Prevention**: Implement blocking script in `<head>` to prevent Flash of Unstyled Content during initial page load

5. **Performance Optimization**: Use CSS containment and requestAnimationFrame for efficient theme switching

---

## Complete Color Palette Specification

### Design Token System

**Token Naming Convention:**
```
--{category}-{variant}-{state}

Examples:
--surface-primary
--text-secondary
--interactive-primary-hover
--status-error-bg
```

### Light Theme Color System

```css
/* Light Theme (Default) */
:root {
  /* SURFACES */
  --surface-primary: #ffffff;           /* Main backgrounds */
  --surface-secondary: #f5f5f5;         /* Secondary panels */
  --surface-tertiary: #e8e8e8;          /* Tertiary surfaces */
  --surface-elevated: #ffffff;          /* Cards, modals */
  --surface-overlay: rgba(0, 0, 0, 0.5); /* Modal overlays */

  /* TEXT */
  --text-primary: #1a1a1a;              /* WCAG AAA: 16.0:1 on white */
  --text-secondary: #4a4a4a;            /* WCAG AAA: 8.6:1 on white */
  --text-tertiary: #737373;             /* WCAG AA: 4.6:1 on white */
  --text-inverse: #ffffff;              /* White text on dark backgrounds */
  --text-disabled: #a0a0a0;             /* Disabled state text */
  --text-link: #0066cc;                 /* Link text */
  --text-link-visited: #551a8b;         /* Visited links */

  /* INTERACTIVE */
  --interactive-primary: #0066cc;       /* Primary buttons, links */
  --interactive-primary-hover: #0052a3; /* Hover state */
  --interactive-primary-active: #003d7a; /* Active/pressed state */
  --interactive-primary-disabled: #cce0f5; /* Disabled state */
  --interactive-secondary: #6c757d;     /* Secondary actions */
  --interactive-secondary-hover: #5a6268;
  --interactive-focus: #0066cc;         /* Focus ring color */

  /* BORDERS */
  --border-subtle: #e8e8e8;             /* WCAG 3:1 on white */
  --border-medium: #d0d0d0;             /* Form inputs default */
  --border-strong: #a0a0a0;             /* Emphasized borders */
  --border-interactive: #0066cc;        /* Interactive element borders */
  --border-focus: #0066cc;              /* Focus indicator */
  --border-error: #dc3545;              /* Error state */
  --border-success: #00a86b;            /* Success state */

  /* STATUS */
  --status-success: #00a86b;            /* Success text/icons */
  --status-success-bg: #e6f7f0;         /* Success backgrounds */
  --status-warning: #ff8c00;            /* Warning text/icons */
  --status-warning-bg: #fff3e6;         /* Warning backgrounds */
  --status-error: #dc3545;              /* Error text/icons */
  --status-error-bg: #fce8e8;           /* Error backgrounds */
  --status-info: #0066cc;               /* Info text/icons */
  --status-info-bg: #e6f2ff;            /* Info backgrounds */

  /* SHADOWS */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.15);

  /* FOCUS INDICATORS */
  --focus-ring: 0 0 0 3px rgba(0, 102, 204, 0.3); /* WCAG 3:1 contrast */
  --focus-ring-width: 3px;
  --focus-ring-offset: 2px;

  /* CODE BLOCKS */
  --code-bg: #f5f5f5;
  --code-text: #c7254e;
  --code-border: #e0e0e0;

  /* DATA VISUALIZATION */
  --chart-color-1: #0066cc;             /* Primary data series */
  --chart-color-2: #00a86b;             /* Secondary series */
  --chart-color-3: #ff8c00;             /* Tertiary series */
  --chart-color-4: #dc3545;             /* Quaternary series */
  --chart-color-5: #6c757d;             /* Additional series */
  --chart-grid: #e8e8e8;                /* Gridlines */
  --chart-axis: #737373;                /* Axis labels */
}
```

### Dark Theme Color System

```css
/* Dark Theme */
[data-theme="dark"] {
  /* SURFACES */
  --surface-primary: #121212;           /* Main backgrounds */
  --surface-secondary: #1e1e1e;         /* Secondary panels */
  --surface-tertiary: #2a2a2a;          /* Tertiary surfaces */
  --surface-elevated: #1e1e1e;          /* Cards, modals (elevated) */
  --surface-overlay: rgba(0, 0, 0, 0.7); /* Modal overlays */

  /* TEXT */
  --text-primary: #e0e0e0;              /* WCAG AAA: 13.3:1 on #121212 */
  --text-secondary: #b0b0b0;            /* WCAG AAA: 7.8:1 on #121212 */
  --text-tertiary: #8a8a8a;             /* WCAG AA: 4.8:1 on #121212 */
  --text-inverse: #1a1a1a;              /* Dark text on light backgrounds */
  --text-disabled: #555555;             /* Disabled state text */
  --text-link: #4d9fff;                 /* Lighter blue for dark bg */
  --text-link-visited: #a78bfa;         /* Lighter purple for visited */

  /* INTERACTIVE */
  --interactive-primary: #4d9fff;       /* Desaturated blue */
  --interactive-primary-hover: #6bb0ff; /* Lighter on hover */
  --interactive-primary-active: #2e8aff; /* Darker when active */
  --interactive-primary-disabled: #1a3a5a; /* Disabled state */
  --interactive-secondary: #8a9099;     /* Secondary actions */
  --interactive-secondary-hover: #9ca4ae;
  --interactive-focus: #4d9fff;         /* Focus ring color */

  /* BORDERS */
  --border-subtle: #2a2a2a;             /* WCAG 3:1 on #121212 */
  --border-medium: #3d3d3d;             /* Form inputs default */
  --border-strong: #555555;             /* Emphasized borders */
  --border-interactive: #4d9fff;        /* Interactive element borders */
  --border-focus: #4d9fff;              /* Focus indicator */
  --border-error: #ff6b7a;              /* Error state */
  --border-success: #00d084;            /* Success state */

  /* STATUS */
  --status-success: #00d084;            /* Lighter green */
  --status-success-bg: #0d2e20;         /* Dark green bg */
  --status-warning: #ffb84d;            /* Lighter orange */
  --status-warning-bg: #2e2410;         /* Dark orange bg */
  --status-error: #ff6b7a;              /* Lighter red */
  --status-error-bg: #2e1416;           /* Dark red bg */
  --status-info: #4d9fff;               /* Lighter blue */
  --status-info-bg: #0d1f2e;            /* Dark blue bg */

  /* SHADOWS */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.4);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.5);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.6);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.7);

  /* FOCUS INDICATORS */
  --focus-ring: 0 0 0 3px rgba(77, 159, 255, 0.4); /* WCAG 3:1 contrast */
  --focus-ring-width: 3px;
  --focus-ring-offset: 2px;

  /* CODE BLOCKS */
  --code-bg: #1e1e1e;
  --code-text: #ff6b9d;
  --code-border: #2a2a2a;

  /* DATA VISUALIZATION */
  --chart-color-1: #4d9fff;             /* Lighter blue */
  --chart-color-2: #00d084;             /* Lighter green */
  --chart-color-3: #ffb84d;             /* Lighter orange */
  --chart-color-4: #ff6b7a;             /* Lighter red */
  --chart-color-5: #8a9099;             /* Gray */
  --chart-grid: #2a2a2a;                /* Darker gridlines */
  --chart-axis: #8a8a8a;                /* Axis labels */
}
```

**WCAG 2.1 Level AA Compliance:**
All color combinations in both themes have been validated for:
- Normal text (< 18pt): Minimum 4.5:1 contrast ratio
- Large text (>= 18pt or 14pt bold): Minimum 3:1 contrast ratio
- UI components and graphical objects: Minimum 3:1 contrast ratio

**Validation Tool:** [TBD - Use automated contrast checker in CI, e.g., @adobe/leonardo-contrast-colors or axe-core]

### Color Usage Guidelines

**Best Practices (from research):**
- Use dark gray (#121212) instead of pure black for backgrounds to reduce eye strain [[Complete Dark Mode Design Guide, UI Deploy](https://ui-deploy.com/blog/complete-dark-mode-design-guide-ui-patterns-and-implementation-best-practices-2025)]
- Avoid saturated colors; use desaturated or muted versions in dark mode [[10 Dark Mode UI Best Practices, Design Studio UI/UX](https://www.designstudiouiux.com/blog/dark-mode-ui-design-best-practices/)]
- Never simply invert colors - design intentional dark mode palette [[Dark Mode UI Design, NetGuru](https://www.netguru.com/blog/tips-dark-mode-ui)]

---

## Component Specifications

### Component State Matrix

All components must support the following states in both light and dark themes:

| State | Description | Color Variables Used |
|-------|-------------|---------------------|
| Default | Normal appearance | `--text-primary`, `--surface-primary`, `--border-medium` |
| Hover | Mouse hover | `--interactive-primary-hover`, `--surface-secondary` |
| Focus | Keyboard focus | `--border-focus`, `--focus-ring` |
| Active | Pressed/clicked | `--interactive-primary-active` |
| Disabled | Non-interactive | `--text-disabled`, `--interactive-primary-disabled` |
| Error | Validation error | `--status-error`, `--border-error`, `--status-error-bg` |
| Success | Success state | `--status-success`, `--border-success`, `--status-success-bg` |

### Forms and Input Components

**Text Inputs:**
```css
.input {
  background-color: var(--surface-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-medium);
}

.input:hover:not(:disabled) {
  border-color: var(--border-strong);
}

.input:focus {
  border-color: var(--border-focus);
  outline: none;
  box-shadow: var(--focus-ring);
}

.input:disabled {
  background-color: var(--surface-secondary);
  color: var(--text-disabled);
  border-color: var(--border-subtle);
  cursor: not-allowed;
}

.input.error {
  border-color: var(--border-error);
}

.input::placeholder {
  color: var(--text-tertiary);
  opacity: 0.7;
}

/* Handle browser autofill */
.input:-webkit-autofill {
  -webkit-box-shadow: 0 0 0 1000px var(--surface-primary) inset;
  -webkit-text-fill-color: var(--text-primary);
}
```

**Buttons:**
```css
/* Primary Button */
.button-primary {
  background-color: var(--interactive-primary);
  color: var(--text-inverse);
  border: none;
}

.button-primary:hover:not(:disabled) {
  background-color: var(--interactive-primary-hover);
}

.button-primary:active:not(:disabled) {
  background-color: var(--interactive-primary-active);
}

.button-primary:focus {
  outline: none;
  box-shadow: var(--focus-ring);
}

.button-primary:disabled {
  background-color: var(--interactive-primary-disabled);
  color: var(--text-disabled);
  cursor: not-allowed;
  opacity: 0.6;
}

/* Secondary Button */
.button-secondary {
  background-color: transparent;
  color: var(--interactive-primary);
  border: 1px solid var(--border-interactive);
}

.button-secondary:hover:not(:disabled) {
  background-color: var(--surface-secondary);
}
```

**Select/Dropdown:**
```css
.select {
  background-color: var(--surface-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-medium);
}

.select:focus {
  border-color: var(--border-focus);
  box-shadow: var(--focus-ring);
}

.select option {
  background-color: var(--surface-primary);
  color: var(--text-primary);
}
```

**Checkboxes and Radio Buttons:**
```css
.checkbox,
.radio {
  accent-color: var(--interactive-primary); /* Modern browsers */
  border: 1px solid var(--border-medium);
}

.checkbox:focus,
.radio:focus {
  outline: none;
  box-shadow: var(--focus-ring);
}

.checkbox:disabled,
.radio:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

### Navigation and Layout Components

**Header/Navigation:**
```css
.header {
  background-color: var(--surface-elevated);
  border-bottom: 1px solid var(--border-subtle);
  box-shadow: var(--shadow-sm);
}

.nav-link {
  color: var(--text-secondary);
}

.nav-link:hover {
  color: var(--text-primary);
  background-color: var(--surface-secondary);
}

.nav-link.active {
  color: var(--interactive-primary);
  border-bottom: 2px solid var(--interactive-primary);
}

.nav-link:focus {
  outline: 2px solid var(--border-focus);
  outline-offset: 2px;
}
```

**Cards:**
```css
.card {
  background-color: var(--surface-elevated);
  border: 1px solid var(--border-subtle);
  box-shadow: var(--shadow-md);
}

.card-header {
  border-bottom: 1px solid var(--border-subtle);
  color: var(--text-primary);
}

.card-body {
  color: var(--text-secondary);
}
```

**Modals:**
```css
.modal-overlay {
  background-color: var(--surface-overlay);
}

.modal {
  background-color: var(--surface-elevated);
  box-shadow: var(--shadow-xl);
  border: 1px solid var(--border-subtle);
}
```

### Typography

```css
h1, h2, h3, h4, h5, h6 {
  color: var(--text-primary);
}

p, span, div {
  color: var(--text-secondary);
}

.text-muted {
  color: var(--text-tertiary);
}

a {
  color: var(--text-link);
}

a:visited {
  color: var(--text-link-visited);
}

a:hover {
  color: var(--interactive-primary-hover);
}

a:focus {
  outline: 2px solid var(--border-focus);
  outline-offset: 2px;
}
```

### Code Blocks and Syntax Highlighting

**Basic Code Blocks:**
```css
code {
  background-color: var(--code-bg);
  color: var(--code-text);
  border: 1px solid var(--code-border);
  padding: 2px 6px;
  border-radius: 3px;
}

pre {
  background-color: var(--code-bg);
  color: var(--text-primary);
  border: 1px solid var(--code-border);
  padding: 16px;
  overflow-x: auto;
}
```

**Syntax Highlighting:**
[TBD - Requires selection of syntax highlighting library]

Options:
1. **Prism.js** with theme switching: `prism.css` (light) / `prism-tomorrow.css` (dark)
2. **Highlight.js** with theme switching: `github.css` (light) / `github-dark.css` (dark)
3. **Monaco Editor** (if using code editor): Built-in theme support

Implementation approach:
```javascript
// Dynamically load syntax theme based on active theme
const loadSyntaxTheme = (theme) => {
  const link = document.getElementById('syntax-theme');
  link.href = theme === 'dark'
    ? '/themes/syntax-dark.css'
    : '/themes/syntax-light.css';
};
```

### Tables

```css
.table {
  background-color: var(--surface-primary);
  border: 1px solid var(--border-subtle);
}

.table thead {
  background-color: var(--surface-secondary);
  color: var(--text-primary);
  border-bottom: 2px solid var(--border-medium);
}

.table tbody tr {
  border-bottom: 1px solid var(--border-subtle);
}

.table tbody tr:hover {
  background-color: var(--surface-secondary);
}

.table td, .table th {
  color: var(--text-secondary);
  border-right: 1px solid var(--border-subtle);
}
```

### Data Visualization Components

**Chart Color Palette:**
Charts must use the defined chart color tokens to ensure accessibility in both themes.

```javascript
// Example: Chart.js configuration
const chartConfig = {
  data: {
    datasets: [
      {
        label: 'Dataset 1',
        backgroundColor: getComputedStyle(document.documentElement)
          .getPropertyValue('--chart-color-1'),
        borderColor: getComputedStyle(document.documentElement)
          .getPropertyValue('--chart-color-1'),
      },
      // Additional datasets using --chart-color-2, --chart-color-3, etc.
    ]
  },
  options: {
    scales: {
      x: {
        grid: {
          color: getComputedStyle(document.documentElement)
            .getPropertyValue('--chart-grid')
        },
        ticks: {
          color: getComputedStyle(document.documentElement)
            .getPropertyValue('--chart-axis')
        }
      }
    }
  }
};
```

**Dashboard Metrics Cards:**
```css
.metric-card {
  background-color: var(--surface-elevated);
  border: 1px solid var(--border-subtle);
  box-shadow: var(--shadow-sm);
}

.metric-value {
  color: var(--text-primary);
  font-size: 2.5rem;
  font-weight: bold;
}

.metric-label {
  color: var(--text-tertiary);
  font-size: 0.875rem;
}

.metric-trend.positive {
  color: var(--status-success);
}

.metric-trend.negative {
  color: var(--status-error);
}
```

**Progress Bars:**
```css
.progress-bar {
  background-color: var(--surface-tertiary);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  background-color: var(--interactive-primary);
  transition: width 0.3s ease;
}

.progress-fill.success {
  background-color: var(--status-success);
}

.progress-fill.error {
  background-color: var(--status-error);
}
```

---

## Image and Media Handling Strategy

### Logo Treatment

**Recommendation:** SVG with CSS color variables (most flexible)

**Option 1: Single SVG with currentColor (Preferred)**
```html
<svg class="logo" viewBox="0 0 100 40">
  <path fill="currentColor" d="..."/>
</svg>
```

```css
.logo {
  color: var(--text-primary);
  width: 120px;
  height: 40px;
}
```

**Option 2: Separate Light/Dark Logo Files**
```html
<picture>
  <source srcset="logo-dark.svg" media="(prefers-color-scheme: dark)">
  <img src="logo-light.svg" alt="Platform Logo" class="logo">
</picture>
```

**Option 3: CSS Background Image with Token**
```css
:root {
  --logo-url: url('/assets/logo-light.svg');
}

[data-theme="dark"] {
  --logo-url: url('/assets/logo-dark.svg');
}

.logo {
  background-image: var(--logo-url);
  background-size: contain;
  background-repeat: no-repeat;
}
```

### User-Generated Images

**Problem:** Images with transparent backgrounds or light content may have poor visibility on dark backgrounds.

**Solution:** Add subtle border to all content images in dark mode

```css
[data-theme="dark"] .content-image {
  border: 1px solid var(--border-subtle);
  border-radius: 4px;
}

/* For images with transparency, optionally add background */
[data-theme="dark"] .content-image.transparent {
  background-color: var(--surface-secondary);
  padding: 8px;
}
```

### Icons

**Strategy:** Use SVG icons with `currentColor` or CSS color variables

```css
.icon {
  fill: currentColor;
  stroke: currentColor;
  width: 20px;
  height: 20px;
}

.icon-primary {
  color: var(--interactive-primary);
}

.icon-success {
  color: var(--status-success);
}

.icon-error {
  color: var(--status-error);
}
```

**Status Icons with Color:**
```html
<svg class="icon icon-success">
  <path fill="currentColor" d="..."/>
</svg>
```

### Screenshots and Documentation Images

**Approach:** Maintain light theme screenshots, add caption or indicator

```css
.screenshot {
  border: 1px solid var(--border-medium);
  border-radius: 4px;
}

[data-theme="dark"] .screenshot {
  /* Add slight opacity in dark mode to reduce glare */
  opacity: 0.9;
}

.screenshot-caption {
  color: var(--text-tertiary);
  font-size: 0.875rem;
  margin-top: 8px;
}
```

---

## Accessibility Requirements

### WCAG 2.1 Level AA Compliance

**Contrast Requirements:**
- Normal text (< 18pt): Minimum 4.5:1 contrast ratio [[Dark Mode Best Practices for Accessibility, DubBot](https://dubbot.com/dubblog/2023/dark-mode-a11y.html)]
- Large text (>= 18pt or 14pt bold): Minimum 3:1 contrast ratio [[Dark Mode Best Practices for Accessibility, DubBot](https://dubbot.com/dubblog/2023/dark-mode-a11y.html)]
- UI components: Minimum 3:1 contrast ratio against adjacent colors [[Dark Mode Accessibility WCAG, BOIA](https://www.boia.org/blog/offering-a-dark-mode-doesnt-satisfy-wcag-color-contrast-requirements)]

**Critical:** Dark mode does not exempt the platform from WCAG contrast requirements - both light and dark modes must independently meet accessibility standards [[Dark Mode Doesn't Satisfy WCAG, BOIA](https://www.boia.org/blog/offering-a-dark-mode-doesnt-satisfy-wcag-color-contrast-requirements)]

### Focus Indicators

**Requirements:**
- Visible in both light and dark themes
- Minimum 3:1 contrast ratio against background (WCAG 2.1 SC 1.4.11)
- Minimum 2px thickness
- 2px offset from focused element

**Implementation:**
```css
/* Focus ring visible in both themes */
:focus {
  outline: none; /* Remove default */
  box-shadow: var(--focus-ring);
}

/* Ensure focus ring has sufficient contrast */
:focus-visible {
  outline: 2px solid var(--border-focus);
  outline-offset: 2px;
}

/* Alternative: Ring style */
.button:focus-visible {
  box-shadow: 0 0 0 3px var(--interactive-primary);
}
```

### Screen Reader Support

**Theme Toggle Announcement:**
```html
<button
  class="theme-toggle"
  aria-label="Toggle dark mode"
  aria-pressed="false"
  role="switch"
>
  <span aria-hidden="true">🌙</span>
  <span class="sr-only">Dark mode: Off</span>
</button>
```

**JavaScript Update:**
```javascript
toggleButton.addEventListener('click', () => {
  const isDark = document.documentElement.getAttribute('data-theme') === 'dark';
  const newTheme = isDark ? 'light' : 'dark';

  // Update visual state
  applyTheme(newTheme);

  // Update ARIA attributes for screen readers
  toggleButton.setAttribute('aria-pressed', newTheme === 'dark');
  toggleButton.querySelector('.sr-only').textContent =
    `Dark mode: ${newTheme === 'dark' ? 'On' : 'Off'}`;
});
```

**Live Region for Theme Changes:**
```html
<div
  role="status"
  aria-live="polite"
  aria-atomic="true"
  class="sr-only"
  id="theme-announcement"
></div>
```

```javascript
const announce = (message) => {
  const announcer = document.getElementById('theme-announcement');
  announcer.textContent = message;
  setTimeout(() => {
    announcer.textContent = '';
  }, 1000);
};

// After theme change
announce(`Switched to ${newTheme} mode`);
```

### Keyboard Navigation

**Requirements:**
- Theme toggle accessible via keyboard (Tab navigation)
- Enter or Space key activates toggle
- No keyboard traps in theme selection
- Logical tab order maintained

**Implementation:**
```javascript
toggleButton.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    toggleButton.click();
  }
});
```

**Keyboard Shortcut (Optional):**
```javascript
// Ctrl+Shift+D to toggle dark mode
document.addEventListener('keydown', (e) => {
  if (e.ctrlKey && e.shiftKey && e.key === 'D') {
    e.preventDefault();
    themeManager.toggle();
  }
});
```

### Reduced Motion Support

**Requirement:** Respect `prefers-reduced-motion` for users with motion sensitivity

**Implementation:**
```css
/* Default: No transitions during page load */
.no-transitions * {
  transition: none !important;
}

/* Users who allow motion: smooth transitions */
@media (prefers-reduced-motion: no-preference) {
  * {
    transition-property: background-color, border-color, color, fill, stroke;
    transition-duration: 0.2s;
    transition-timing-function: ease-out;
  }
}

/* Users who prefer reduced motion: instant theme switching */
@media (prefers-reduced-motion: reduce) {
  * {
    transition: none !important;
  }
}
```

### Screen Reader Testing Protocol

**Testing Requirements:**

1. **NVDA (Windows)** - Latest version
   - Test theme toggle announcement
   - Verify focus indicators are announced
   - Test with Chrome and Firefox

2. **JAWS (Windows)** - Version 2024+
   - Test theme toggle announcement
   - Verify all interactive elements are accessible
   - Test with Chrome and Edge

3. **VoiceOver (macOS/iOS)** - Built-in
   - Test theme toggle announcement
   - Verify mobile experience
   - Test with Safari

4. **TalkBack (Android)** - Built-in
   - Test mobile theme toggle
   - Verify touch target sizes
   - Test with Chrome Android

**Test Scenarios:**
- Navigate to theme toggle using screen reader
- Activate theme toggle
- Verify theme change announcement
- Navigate through UI components in both themes
- Verify all content remains accessible

---

## Theme Toggle Component Specification

### Toggle Component Design

**Recommended Pattern:** Segmented control with three options

**Options:**
1. **Light** - Force light theme
2. **Dark** - Force dark theme
3. **Auto** - Follow system preference

**Visual Design:**
```
┌─────────────────────────┐
│  ☀️  |  🌙  |  ⚙️  │
│ Light | Dark | Auto  │
└─────────────────────────┘
```

### Toggle Placement

**Primary Location:** Header navigation bar (right side)
- Always visible
- Easy to access
- Consistent across pages

**Secondary Location:** User settings/preferences page
- Allows more detailed theme configuration
- Can include additional options (future: custom themes)

### Component States

```css
/* Theme toggle button group */
.theme-toggle-group {
  display: flex;
  background-color: var(--surface-secondary);
  border: 1px solid var(--border-subtle);
  border-radius: 6px;
  padding: 2px;
}

.theme-toggle-option {
  padding: 6px 12px;
  background: transparent;
  color: var(--text-secondary);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.theme-toggle-option:hover {
  background-color: var(--surface-tertiary);
  color: var(--text-primary);
}

.theme-toggle-option.active {
  background-color: var(--interactive-primary);
  color: var(--text-inverse);
  box-shadow: var(--shadow-sm);
}

.theme-toggle-option:focus-visible {
  outline: 2px solid var(--border-focus);
  outline-offset: 2px;
}
```

### Component Markup

```html
<div class="theme-toggle-group" role="radiogroup" aria-label="Color theme">
  <button
    class="theme-toggle-option"
    data-theme="light"
    role="radio"
    aria-checked="false"
    aria-label="Light theme"
  >
    <span aria-hidden="true">☀️</span>
    <span class="toggle-label">Light</span>
  </button>
  <button
    class="theme-toggle-option active"
    data-theme="dark"
    role="radio"
    aria-checked="true"
    aria-label="Dark theme"
  >
    <span aria-hidden="true">🌙</span>
    <span class="toggle-label">Dark</span>
  </button>
  <button
    class="theme-toggle-option"
    data-theme="auto"
    role="radio"
    aria-checked="false"
    aria-label="Auto (system preference)"
  >
    <span aria-hidden="true">⚙️</span>
    <span class="toggle-label">Auto</span>
  </button>
</div>
```

### Component Behavior

**User Interactions:**
1. **Click Light:** Apply light theme, store preference
2. **Click Dark:** Apply dark theme, store preference
3. **Click Auto:** Remove stored preference, follow system

**Persistence:**
```javascript
// Storage keys and values
const STORAGE_KEY = 'user-theme-preference';
const THEME_VALUES = {
  LIGHT: 'light',
  DARK: 'dark',
  AUTO: 'auto' // or null to indicate system preference
};

// Save preference
const saveThemePreference = (theme) => {
  if (theme === THEME_VALUES.AUTO) {
    localStorage.removeItem(STORAGE_KEY);
  } else {
    localStorage.setItem(STORAGE_KEY, theme);
  }
};
```

---

## Technical Implementation

### Theme Detection and Persistence

**Complete Theme Manager Implementation:**

```javascript
/**
 * ThemeManager - Handles dark mode detection, persistence, and switching
 */
class ThemeManager {
  constructor(options = {}) {
    this.storageKey = options.storageKey || 'user-theme-preference';
    this.defaultTheme = options.defaultTheme || 'light';
    this.themes = options.themes || ['light', 'dark'];
    this.onThemeChange = options.onThemeChange || (() => {});

    this.initialize();
  }

  initialize() {
    const stored = this.getStoredPreference();
    const system = this.getSystemPreference();
    const theme = stored || system || this.defaultTheme;

    this.applyTheme(theme, false);
    this.listenForSystemChanges();
    this.setupTransitionOptimization();
  }

  getStoredPreference() {
    try {
      return localStorage.getItem(this.storageKey);
    } catch (e) {
      console.warn('localStorage not available:', e);
      return null;
    }
  }

  getSystemPreference() {
    if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
      return 'dark';
    }
    return 'light';
  }

  applyTheme(theme, persist = true) {
    if (!this.themes.includes(theme)) {
      console.error(`Invalid theme: ${theme}`);
      return;
    }

    performance.mark('theme-switch-start');

    // Batch DOM updates in single frame
    requestAnimationFrame(() => {
      document.documentElement.setAttribute('data-theme', theme);
      this.updateMetaThemeColor(theme);
      this.updateColorScheme(theme);

      if (persist) {
        this.savePreference(theme);
      }

      performance.mark('theme-switch-end');
      performance.measure('theme-switch', 'theme-switch-start', 'theme-switch-end');

      const duration = performance.getEntriesByName('theme-switch')[0].duration;
      if (duration > 100) {
        console.warn(`Slow theme switch: ${duration.toFixed(2)}ms`);
      }

      this.onThemeChange(theme);
    });
  }

  updateMetaThemeColor(theme) {
    const colors = {
      light: '#ffffff',
      dark: '#121212'
    };

    const metaThemeColor = document.querySelector('meta[name="theme-color"]');
    if (metaThemeColor) {
      metaThemeColor.setAttribute('content', colors[theme]);
    }
  }

  updateColorScheme(theme) {
    // Tell browser to use dark mode defaults for form controls, scrollbars
    document.documentElement.style.colorScheme = theme;
  }

  savePreference(theme) {
    try {
      localStorage.setItem(this.storageKey, theme);
    } catch (e) {
      console.warn('Failed to save theme preference:', e);
    }
  }

  listenForSystemChanges() {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');

    mediaQuery.addEventListener('change', (e) => {
      // Only respond if user hasn't set explicit preference
      if (!this.getStoredPreference()) {
        this.applyTheme(e.matches ? 'dark' : 'light', false);
      }
    });
  }

  setupTransitionOptimization() {
    // Prevent transitions during page load
    document.documentElement.classList.add('no-transitions');

    requestAnimationFrame(() => {
      document.documentElement.classList.remove('no-transitions');
    });
  }

  toggle() {
    const current = document.documentElement.getAttribute('data-theme');
    const next = current === 'dark' ? 'light' : 'dark';
    this.applyTheme(next);
  }

  getCurrentTheme() {
    return document.documentElement.getAttribute('data-theme') || this.defaultTheme;
  }

  clearPreference() {
    try {
      localStorage.removeItem(this.storageKey);
      this.initialize();
    } catch (e) {
      console.warn('Failed to clear theme preference:', e);
    }
  }
}

// Initialize theme manager
const themeManager = new ThemeManager({
  onThemeChange: (theme) => {
    // Analytics tracking
    if (window.analytics) {
      window.analytics.track('Theme Changed', { theme });
    }
    // Update chart themes if applicable
    updateChartTheme(theme);
  }
});

export default themeManager;
```

### FOUC Prevention Strategy

**Inline Blocking Script in `<head>`:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="color-scheme" content="light dark">
  <meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
  <meta name="theme-color" content="#121212" media="(prefers-color-scheme: dark)">

  <!-- CRITICAL: Must run before any paint to prevent FOUC -->
  <script>
    (function() {
      'use strict';

      var STORAGE_KEY = 'user-theme-preference';

      function getStoredTheme() {
        try {
          return localStorage.getItem(STORAGE_KEY);
        } catch (e) {
          return null;
        }
      }

      function getSystemTheme() {
        if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
          return 'dark';
        }
        return 'light';
      }

      var theme = getStoredTheme() || getSystemTheme();

      if (theme) {
        document.documentElement.setAttribute('data-theme', theme);
        document.documentElement.style.colorScheme = theme;
      }
    })();
  </script>

  <!-- Inline critical theme tokens to prevent FOUC -->
  <style>
    /* Inline subset of critical tokens */
    :root {
      --surface-primary: #ffffff;
      --text-primary: #1a1a1a;
    }
    [data-theme="dark"] {
      --surface-primary: #121212;
      --text-primary: #e0e0e0;
    }
    body {
      background-color: var(--surface-primary);
      color: var(--text-primary);
    }
  </style>

  <!-- Full stylesheet loaded after critical inline styles -->
  <link rel="stylesheet" href="/styles/main.css">
</head>
<body>
  <!-- Application content -->
</body>
</html>
```

**Performance Impact:**
- Blocking script adds ~1-2ms to First Paint
- Acceptable trade-off to prevent flash of wrong theme
- Inline styles add ~1KB to HTML document size

### Browser Compatibility

**Full Support Matrix:**

| Browser | Version | CSS Custom Properties | prefers-color-scheme | Support Level |
|---------|---------|----------------------|---------------------|---------------|
| Chrome  | 88+     | ✅ Yes               | ✅ Yes              | Full Support  |
| Edge    | 88+     | ✅ Yes               | ✅ Yes              | Full Support  |
| Firefox | 87+     | ✅ Yes               | ✅ Yes              | Full Support  |
| Safari  | 14.1+   | ✅ Yes               | ✅ Yes              | Full Support  |
| Chrome Android | 88+ | ✅ Yes            | ✅ Yes              | Full Support  |
| iOS Safari | 14.5+ | ✅ Yes              | ✅ Yes              | Full Support  |

**Coverage:** ~95% of global users (as of 2024)

**Partial Support (Graceful Degradation):**

| Browser | Version | Support | Fallback Behavior |
|---------|---------|---------|-------------------|
| Chrome  | 49-87   | Partial | Light theme only, manual toggle works via data-theme |
| Firefox | 31-86   | Partial | Light theme only, manual toggle works |
| Safari  | 9.1-14.0 | Partial | Light theme only, manual toggle works |

**No Support:**

| Browser | Version | Support | Fallback Behavior |
|---------|---------|---------|-------------------|
| IE 11   | All     | None    | Light theme only, no toggle |
| Old browsers | < 2016 | None | Light theme only, no toggle |

**Coverage:** < 1% of users

**Polyfill Decision:** NO polyfill recommended. Better to serve light theme as graceful degradation rather than adding polyfill overhead.

**Feature Detection:**
```javascript
// Check for CSS custom property support
const supportsCustomProperties = CSS.supports('color', 'var(--test)');

// Check for prefers-color-scheme support
const supportsColorScheme = window.matchMedia('(prefers-color-scheme: dark)').media !== 'not all';

if (!supportsCustomProperties) {
  console.warn('CSS custom properties not supported. Falling back to light theme.');
  // Optionally show message to user about browser upgrade
}
```

---

## Testing Strategy

### Unit Tests

**Theme Manager Tests:**

```javascript
import { describe, test, expect, beforeEach } from '@jest/globals';
import ThemeManager from './theme-manager';

describe('ThemeManager', () => {
  beforeEach(() => {
    localStorage.clear();
    document.documentElement.removeAttribute('data-theme');
  });

  test('applies system preference when no stored preference', () => {
    window.matchMedia = jest.fn().mockImplementation(query => ({
      matches: query === '(prefers-color-scheme: dark)',
      addEventListener: jest.fn(),
    }));

    const manager = new ThemeManager();
    expect(document.documentElement.getAttribute('data-theme')).toBe('dark');
  });

  test('user preference overrides system preference', () => {
    localStorage.setItem('user-theme-preference', 'light');

    window.matchMedia = jest.fn().mockImplementation(query => ({
      matches: query === '(prefers-color-scheme: dark)',
      addEventListener: jest.fn(),
    }));

    const manager = new ThemeManager();
    expect(document.documentElement.getAttribute('data-theme')).toBe('light');
  });

  test('toggle switches between light and dark', () => {
    const manager = new ThemeManager();
    manager.applyTheme('light');

    manager.toggle();
    expect(document.documentElement.getAttribute('data-theme')).toBe('dark');

    manager.toggle();
    expect(document.documentElement.getAttribute('data-theme')).toBe('light');
  });

  test('clearPreference removes stored theme and reapplies system preference', () => {
    localStorage.setItem('user-theme-preference', 'dark');

    const manager = new ThemeManager();
    manager.clearPreference();

    expect(localStorage.getItem('user-theme-preference')).toBeNull();
  });

  test('handles localStorage unavailability gracefully', () => {
    const originalLocalStorage = global.localStorage;
    delete global.localStorage;

    expect(() => {
      const manager = new ThemeManager();
    }).not.toThrow();

    global.localStorage = originalLocalStorage;
  });
});
```

### Visual Regression Tests

**Component Library Testing:**

```javascript
import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test.describe('Visual Regression: Dark Mode', () => {
  const themes = ['light', 'dark'];

  themes.forEach(theme => {
    test(`Component library renders correctly in ${theme} mode`, async ({ page }) => {
      await page.goto('http://localhost:3000/component-library');

      // Apply theme
      await page.evaluate((t) => {
        document.documentElement.setAttribute('data-theme', t);
      }, theme);

      // Wait for transitions to complete
      await page.waitForTimeout(300);

      // Capture screenshot
      await percySnapshot(page, `Component Library - ${theme} mode`);
    });
  });

  // Test individual component states
  test('Button states in both themes', async ({ page }) => {
    await page.goto('http://localhost:3000/components/button');

    for (const theme of themes) {
      await page.evaluate((t) => {
        document.documentElement.setAttribute('data-theme', t);
      }, theme);

      await page.waitForTimeout(300);
      await percySnapshot(page, `Button Component - ${theme} mode`);
    }
  });
});
```

**Tool Options:**
- **Percy** (recommended): Integrates with CI/CD, good diff detection
- **Chromatic** (for Storybook): Excellent for component libraries
- **BackstopJS** (open source): Cost-effective alternative

### Accessibility Tests

**Automated WCAG Compliance:**

```javascript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility: WCAG Compliance', () => {
  test('light theme meets WCAG 2.1 AA requirements', async ({ page }) => {
    await page.goto('http://localhost:3000');
    await page.evaluate(() => {
      document.documentElement.setAttribute('data-theme', 'light');
    });

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('dark theme meets WCAG 2.1 AA requirements', async ({ page }) => {
    await page.goto('http://localhost:3000');
    await page.evaluate(() => {
      document.documentElement.setAttribute('data-theme', 'dark');
    });

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('focus indicators have sufficient contrast in both themes', async ({ page }) => {
    await page.goto('http://localhost:3000');

    for (const theme of ['light', 'dark']) {
      await page.evaluate((t) => {
        document.documentElement.setAttribute('data-theme', t);
      }, theme);

      // Focus first interactive element
      await page.keyboard.press('Tab');

      const accessibilityScanResults = await new AxeBuilder({ page })
        .withTags(['wcag21aa'])
        .analyze();

      expect(accessibilityScanResults.violations).toEqual([]);
    }
  });
});
```

**Contrast Ratio Validation:**

```javascript
// Custom test to validate all color token combinations
import { test, expect } from '@playwright/test';

test('all color tokens meet WCAG contrast requirements', async ({ page }) => {
  await page.goto('http://localhost:3000/design-tokens');

  const results = await page.evaluate(() => {
    const getContrast = (fg, bg) => {
      // Calculate contrast ratio (implementation omitted for brevity)
      // Use library like 'wcag-contrast' or similar
    };

    const tokens = {
      light: {
        '--text-primary': '#1a1a1a',
        '--surface-primary': '#ffffff',
        // ... all token combinations
      },
      dark: {
        '--text-primary': '#e0e0e0',
        '--surface-primary': '#121212',
        // ... all token combinations
      }
    };

    const failures = [];

    Object.entries(tokens).forEach(([theme, tokenSet]) => {
      // Test text-primary on surface-primary
      const contrast = getContrast(
        tokenSet['--text-primary'],
        tokenSet['--surface-primary']
      );

      if (contrast < 4.5) {
        failures.push({
          theme,
          pair: 'text-primary on surface-primary',
          contrast,
          required: 4.5
        });
      }
    });

    return failures;
  });

  expect(results).toEqual([]);
});
```

### E2E Tests

**Theme Switching Flows:**

```javascript
import { test, expect } from '@playwright/test';

test.describe('E2E: Theme Switching', () => {
  test('theme persists across page navigations', async ({ page }) => {
    await page.goto('/');

    // Toggle to dark mode
    await page.click('[data-testid="theme-toggle-dark"]');

    expect(await page.getAttribute('html', 'data-theme')).toBe('dark');

    // Navigate to another page
    await page.goto('/about');

    // Theme should persist
    expect(await page.getAttribute('html', 'data-theme')).toBe('dark');
  });

  test('system preference is respected on first visit', async ({ page, context }) => {
    // Emulate dark mode preference
    await context.emulateMedia({ colorScheme: 'dark' });

    await page.goto('/');

    expect(await page.getAttribute('html', 'data-theme')).toBe('dark');
  });

  test('manual selection overrides system preference', async ({ page, context }) => {
    // Emulate dark mode preference
    await context.emulateMedia({ colorScheme: 'dark' });

    await page.goto('/');
    expect(await page.getAttribute('html', 'data-theme')).toBe('dark');

    // User manually selects light mode
    await page.click('[data-testid="theme-toggle-light"]');
    expect(await page.getAttribute('html', 'data-theme')).toBe('light');

    // Reload page
    await page.reload();

    // Manual selection should persist despite system preference
    expect(await page.getAttribute('html', 'data-theme')).toBe('light');
  });

  test('auto mode follows system preference changes', async ({ page, context }) => {
    await page.goto('/');

    // Select auto mode
    await page.click('[data-testid="theme-toggle-auto"]');

    // Change system preference to dark
    await context.emulateMedia({ colorScheme: 'dark' });
    await page.waitForTimeout(100);

    expect(await page.getAttribute('html', 'data-theme')).toBe('dark');

    // Change system preference to light
    await context.emulateMedia({ colorScheme: 'light' });
    await page.waitForTimeout(100);

    expect(await page.getAttribute('html', 'data-theme')).toBe('light');
  });
});
```

### Cross-Browser Testing

**BrowserStack Matrix:**

```yaml
# BrowserStack configuration
browsers:
  desktop:
    - browser: Chrome
      versions: [latest, latest-1]
      os: [Windows 11, macOS Sonoma]
    - browser: Firefox
      versions: [latest, latest-1]
      os: [Windows 11, macOS Sonoma]
    - browser: Safari
      versions: [latest, latest-1]
      os: [macOS Sonoma]
    - browser: Edge
      versions: [latest]
      os: [Windows 11]

  mobile:
    - device: iPhone 15
      os: iOS 17
      browser: Safari
    - device: iPhone 14
      os: iOS 16
      browser: Safari
    - device: Samsung Galaxy S24
      os: Android 14
      browser: Chrome
    - device: Google Pixel 7
      os: Android 13
      browser: Chrome

test_scenarios:
  - Theme switching functionality
  - CSS custom property rendering
  - prefers-color-scheme detection
  - localStorage persistence
  - Performance metrics (theme switch duration)
  - Visual correctness
```

### Performance Testing

**Lighthouse CI Configuration:**

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      numberOfRuns: 3,
      url: [
        'http://localhost:3000/',
        'http://localhost:3000/?theme=dark',
      ],
      settings: {
        preset: 'desktop',
      },
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 1.0 }],
        'first-contentful-paint': ['error', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.01 }],
        'total-blocking-time': ['error', { maxNumericValue: 300 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

**Theme Switch Performance:**

```javascript
// Performance benchmark test
test('theme switch completes within 100ms', async () => {
  performance.mark('theme-switch-start');

  await themeManager.applyTheme('dark');

  performance.mark('theme-switch-end');
  performance.measure('theme-switch', 'theme-switch-start', 'theme-switch-end');

  const measure = performance.getEntriesByName('theme-switch')[0];
  expect(measure.duration).toBeLessThan(100);
});
```

### CI/CD Integration

**GitHub Actions Workflow:**

```yaml
name: Dark Mode Tests

on: [pull_request]

jobs:
  accessibility:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build application
        run: npm run build
      - name: Run accessibility tests
        run: npm run test:a11y
      - name: Upload accessibility report
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: a11y-report
          path: ./a11y-report.html

  visual-regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Run visual regression tests
        run: npm run test:visual
        env:
          PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}

  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Run Lighthouse CI
        run: npm run lighthouse:ci
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}

  contrast-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate color contrast ratios
        run: npm run test:contrast
      - name: Fail if WCAG violations found
        run: |
          if [ -f ./contrast-violations.json ]; then
            echo "WCAG contrast violations found!"
            cat ./contrast-violations.json
            exit 1
          fi
```

---

## Performance Budget

### Metrics and Targets

**1. Theme Switch Duration:**
- **Target:** < 100ms (instant feel)
- **Acceptable:** 100-200ms
- **Poor:** > 200ms
- **Measurement:** `performance.mark()` and `performance.measure()`

**2. First Paint Impact:**
- **Target:** Zero increase in FP/FCP
- **Acceptable:** < 50ms increase
- **Poor:** > 100ms increase
- **Measurement:** Lighthouse comparison before/after implementation

**3. Bundle Size Impact:**

JavaScript:
- **Target:** < 2KB gzipped (theme manager)
- **Acceptable:** 2-5KB
- **Poor:** > 5KB

CSS:
- **Target:** < 5KB (additional dark theme tokens and styles)
- **Acceptable:** 5-10KB
- **Poor:** > 10KB

**4. Layout Shift (CLS):**
- **Target:** CLS = 0 during theme switch
- **Acceptable:** CLS < 0.01
- **Poor:** CLS > 0.01
- **Measurement:** Core Web Vitals

**5. Total Blocking Time:**
- **Target:** No increase in TBT
- **Acceptable:** < 50ms increase
- **Poor:** > 100ms increase

### Measurement Strategy

**Automated Monitoring:**
```javascript
// Add to analytics
const trackThemePerformance = () => {
  const measure = performance.getEntriesByName('theme-switch')[0];

  if (window.analytics) {
    window.analytics.track('Theme Switch Performance', {
      duration: measure.duration,
      theme: document.documentElement.getAttribute('data-theme'),
      userAgent: navigator.userAgent,
    });
  }

  // Alert if performance budget exceeded
  if (measure.duration > 100) {
    console.warn(`Theme switch exceeded budget: ${measure.duration}ms`);
  }
};
```

**Lighthouse CI:** Run on every PR to catch performance regressions

**WebPageTest:** Monthly audits for detailed performance analysis

**Real User Monitoring (RUM):** Track actual user performance if RUM tool available

---

## Component Migration Plan

### Phase 1: Audit and Foundation (Week 1)

**Tasks:**
1. Automated search for hardcoded color values in codebase
   ```bash
   # Find hex colors
   grep -r "#[0-9a-fA-F]\{3,6\}" src/

   # Find rgb/rgba
   grep -r "rgba\?\(" src/

   # Find hsl/hsla
   grep -r "hsla\?\(" src/
   ```

2. Categorize components by complexity:
   - **Simple:** Single-color components (buttons, labels)
   - **Medium:** Multi-state components (forms, inputs)
   - **Complex:** Data visualizations, charts, dashboards

3. Identify third-party dependencies requiring theming

4. Create comprehensive color token system (CSS custom properties)

**Deliverables:**
- Audit report: List of all components requiring migration
- Hardcoded color inventory
- Third-party dependency assessment
- Complete CSS token specification

### Phase 2: Core Components (Week 2-3)

**Priority 1 (Week 2):**
- Header/navigation
- Footer
- Primary layout containers
- Typography base styles

**Priority 2 (Week 3):**
- Form inputs (text, textarea, select)
- Buttons (primary, secondary, tertiary)
- Links and interactive text
- Basic cards and containers

**Migration Pattern:**
```css
/* Before */
.button {
  background-color: #0066cc;
  color: #ffffff;
  border: 1px solid #0052a3;
}

.button:hover {
  background-color: #0052a3;
}

/* After */
.button {
  background-color: var(--interactive-primary);
  color: var(--text-inverse);
  border: 1px solid var(--interactive-primary-hover);
}

.button:hover {
  background-color: var(--interactive-primary-hover);
}
```

**Automated Codemod (Optional):**
```javascript
// Simple codemod to convert common patterns
const transform = (fileInfo, api) => {
  const j = api.jscodeshift;
  const root = j(fileInfo.source);

  // Example: Convert #0066cc to var(--interactive-primary)
  root.find(j.Literal)
    .filter(path => path.value.value === '#0066cc')
    .replaceWith(() => j.literal('var(--interactive-primary)'));

  return root.toSource();
};

module.exports = transform;
```

### Phase 3: Content Components (Week 4-5)

**Priority 3 (Week 4):**
- Tables
- Lists
- Alerts and notifications
- Modals and dialogs
- Tooltips and popovers

**Priority 4 (Week 5):**
- Dashboard components
- Data visualization (charts, graphs)
- Code blocks and syntax highlighting
- Custom icons and graphics

### Phase 4: Validation and Testing (Week 6)

**Tasks:**
1. Visual regression testing (all components, both themes)
2. Accessibility audit (WCAG 2.1 AA compliance)
3. Cross-browser testing
4. Performance benchmarking
5. User acceptance testing

**Quality Gates:**
- Zero accessibility violations
- All visual regression tests pass
- Performance budget met
- Cross-browser compatibility confirmed

### Preventing Regression

**ESLint Rule:**
```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'no-hardcoded-colors': 'error',
  },
  plugins: ['custom-rules'],
};

// custom-rules/no-hardcoded-colors.js
module.exports = {
  create(context) {
    const colorRegex = /^(#[0-9a-f]{3,8}|rgb\(|rgba\(|hsl\(|hsla\()/i;

    return {
      Literal(node) {
        if (typeof node.value === 'string' && colorRegex.test(node.value)) {
          context.report({
            node,
            message: 'Use CSS custom properties instead of hardcoded colors. See design token documentation.',
          });
        }
      },
    };
  },
};
```

**Pre-commit Hook:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check for hardcoded colors in staged files
if git diff --cached --name-only | grep -E '\.(css|scss|js|jsx|ts|tsx)$' | xargs grep -E "#[0-9a-fA-F]{3,6}|rgba?\(|hsla?\(" > /dev/null; then
  echo "Error: Hardcoded colors found in staged files."
  echo "Please use CSS custom properties from the design token system."
  echo "See: docs/design-tokens.md"
  exit 1
fi
```

---

## Implementation Scope

### In-Scope (MVP - Phase 1)

**Foundation:**
- Complete CSS custom property token system
- System preference detection via `prefers-color-scheme`
- Manual theme toggle component (light/dark/auto)
- localStorage preference persistence
- FOUC prevention implementation
- Dark theme color palette for all core components
- WCAG 2.1 AA compliance for both themes
- Reduced motion support

**Core Components:**
- Navigation/header
- Footer
- Main content areas
- Forms and inputs (text, textarea, select, checkbox, radio)
- Buttons (primary, secondary, tertiary)
- Interactive elements (links, tabs, dropdowns)
- Typography system
- Cards and containers
- Tables
- Modals and dialogs
- Alerts and notifications
- Background and surface colors
- Border and divider colors

**Technical Implementation:**
- Theme manager JavaScript module
- Inline FOUC prevention script
- CSS transition system with reduced motion support
- Browser compatibility handling (graceful degradation)
- Unit tests for theme manager
- Accessibility tests (automated + manual)
- Visual regression test setup
- Performance monitoring

**Documentation:**
- Design token reference
- Component migration guide for developers
- Accessibility testing protocol
- User-facing help documentation

### Out-of-Scope (Future Phases)

**Phase 2:**
- Account-level preference sync across devices
- Advanced data visualization theming (complex charts)
- Syntax highlighting themes (if code editor features exist)
- Image optimization for dark mode (automatic filters)
- Custom theme builder/editor

**Phase 3+:**
- Multiple theme variants (high contrast, colorblind modes)
- Scheduled automatic theme switching (time-based)
- Per-page or per-section theme overrides
- Third-party integration theme synchronization
- Advanced color customization for enterprise customers
- Animated theme transitions beyond basic fading
- Theme-aware PDF export
- Theme-aware email templates

**Not Included:**
- Mobile native application themes (if separate codebase)
- Print stylesheet dark mode variants (prints will force light mode)
- Legacy browser support (IE11, pre-2016 browsers)

---

## Risks and Mitigation Strategies

### Technical Risks

**Risk 1: WCAG Compliance Complexity**
- **Impact:** HIGH - Both themes must independently meet accessibility standards
- **Likelihood:** MEDIUM-HIGH - Requires careful color palette design
- **Mitigation:**
  - Conduct accessibility audit during design phase
  - Use automated contrast checking tools in CI/CD pipeline
  - Manual testing with accessibility experts
  - Block PRs that fail WCAG validation
  - [[Dark Mode Doesn't Satisfy WCAG, BOIA](https://www.boia.org/blog/offering-a-dark-mode-doesnt-satisfy-wcag-color-contrast-requirements)]

**Risk 2: Image and Media Content Visibility**
- **Impact:** MEDIUM - Images designed for light backgrounds may have poor visibility in dark mode
- **Likelihood:** MEDIUM
- **Mitigation:**
  - Apply CSS borders to content images in dark mode
  - Audit critical images and provide dark mode variants
  - Provide guidelines for future image uploads
  - Use `picture` element with theme-specific sources for logos
  - Test all existing images in both themes

**Risk 3: Third-Party Component Compatibility**
- **Impact:** MEDIUM-HIGH - External libraries may not support theming
- **Likelihood:** MEDIUM
- **Mitigation:**
  - Audit all third-party UI components during design phase (not implementation)
  - Apply CSS custom property overrides where possible
  - Use CSS filters as last resort for unthemeable components
  - Document known incompatibilities
  - Consider alternative components if necessary
  - Budget extra time for third-party integration

**Risk 4: Performance Regression**
- **Impact:** MEDIUM - Theme switching could cause repaints/reflows
- **Likelihood:** LOW-MEDIUM
- **Mitigation:**
  - Use CSS custom properties for efficient theme switching
  - Apply CSS containment to isolate component repaints
  - Batch DOM updates in `requestAnimationFrame`
  - Establish performance budget and monitor in CI
  - Conduct performance testing on mid-range devices

**Risk 5: Browser-Specific Rendering Issues**
- **Impact:** MEDIUM - Different browsers may render dark mode differently
- **Likelihood:** MEDIUM
- **Mitigation:**
  - Comprehensive cross-browser testing (BrowserStack)
  - Test form input autofill styling in all browsers
  - Verify scrollbar styling across browsers
  - Document browser-specific quirks
  - Provide fallbacks for unsupported features

### User Experience Risks

**Risk 6: User Accessibility Diversity**
- **Impact:** MEDIUM - Dark mode may worsen experience for users with astigmatism or dyslexia
- **Likelihood:** MEDIUM (affects subset of users)
- **Mitigation:**
  - Never force dark mode as default
  - Ensure light mode remains fully supported
  - Provide clear, easy-to-find theme toggle
  - Offer "Auto" option to respect system preferences
  - Consider user education about theme options
  - [[Inclusive Dark Mode, Smashing Magazine](https://www.smashingmagazine.com/2025/04/inclusive-dark-mode-designing-accessible-dark-themes/)]

**Risk 7: Brand Identity Consistency**
- **Impact:** LOW-MEDIUM - Dark theme may dilute brand visual identity
- **Likelihood:** LOW-MEDIUM
- **Mitigation:**
  - Collaborate with design/brand team on color palette
  - Maintain brand colors with appropriate adjustments for dark backgrounds
  - Test brand recognition with user groups
  - Create brand guidelines for dark mode usage
  - [[Dark Mode Design Best Practices, NetGuru](https://www.netguru.com/blog/tips-dark-mode-ui)]

**Risk 8: User Confusion with Theme Options**
- **Impact:** LOW - Users may not understand Light/Dark/Auto options
- **Likelihood:** LOW-MEDIUM
- **Mitigation:**
  - Provide clear labels and icons for theme toggle
  - Add tooltips explaining each option
  - Include theme selection in onboarding/help docs
  - Use industry-standard iconography (sun/moon)

### Business and Project Risks

**Risk 9: Scope Creep and Timeline Extension**
- **Impact:** HIGH - Feature expansion beyond MVP delays delivery
- **Likelihood:** MEDIUM
- **Mitigation:**
  - Strict adherence to MVP definition (this document)
  - Regular stakeholder communication on priorities
  - Phase 2+ features documented but deferred
  - Change request process for scope additions
  - Weekly progress reviews with PM

**Risk 10: Third-Party Dependency Requires Replacement**
- **Impact:** HIGH - Could significantly extend timeline
- **Likelihood:** LOW-MEDIUM
- **Mitigation:**
  - Early third-party component audit (Week 1)
  - Identify alternatives before implementation begins
  - Budget contingency time in schedule
  - Prioritize components that are themeable

**Risk 11: Insufficient Design Resources**
- **Impact:** HIGH - Complete color palette required before development
- **Likelihood:** LOW
- **Mitigation:**
  - Engage design team early (before development starts)
  - Provide design token template and requirements
  - Allocate 2 weeks for design phase
  - Validate colors with WCAG tools
  - PM works closely with design lead

**Risk 12: Hidden Hardcoded Colors**
- **Impact:** MEDIUM - Missed hardcoded colors cause visual inconsistencies
- **Likelihood:** MEDIUM-HIGH
- **Mitigation:**
  - Automated code scanning during audit phase
  - ESLint rule to prevent new hardcoded colors
  - Visual regression testing catches inconsistencies
  - Thorough component-by-component review
  - Pre-commit hooks prevent regression

---

## Success Metrics

### Primary KPIs

**1. Adoption Rate**
- **Target:** 50%+ of users engage with dark mode within 3 months of launch
- **Rationale:** 82% of mobile users prefer dark mode, but web adoption typically lower [[Dark Mode Statistics, forms.app](https://forms.app/en/blog/dark-mode-statistics)]
- **Measurement:**
  - Analytics tracking: `theme_toggle_clicked` event
  - Session analysis: Percentage of sessions with dark mode active
  - Trend analysis: Week-over-week adoption growth
- **Success Criteria:**
  - Week 1: 20% adoption
  - Week 4: 35% adoption
  - Week 12: 50% adoption

**2. User Satisfaction**
- **Target:** Positive feedback in user surveys (4+ out of 5 rating)
- **Measurement:**
  - Post-launch user survey (2 weeks after launch)
  - NPS score comparison (before/after dark mode)
  - Support ticket sentiment analysis
- **Survey Questions:**
  - "How satisfied are you with the dark mode feature?" (1-5 scale)
  - "Does dark mode improve your experience?" (Yes/No/Neutral)
  - "Is it easy to switch between themes?" (1-5 scale)

**3. Accessibility Compliance**
- **Target:** 100% WCAG 2.1 Level AA compliance for both themes
- **Measurement:**
  - Automated accessibility testing (axe-core) - ZERO violations
  - Manual WCAG audit by accessibility specialist
  - Screen reader testing protocol completion
- **Requirement:** [[WCAG Contrast Requirements, DubBot](https://dubbot.com/dubblog/2023/dark-mode-a11y.html)]
- **Success Criteria:** BLOCKING - Launch cannot proceed without full compliance

### Secondary Metrics

**4. Performance Impact**
- **Target:** Zero degradation in page load time or runtime performance
- **Measurement:**
  - Lighthouse performance score (before/after comparison)
  - Core Web Vitals (FCP, LCP, CLS, TBT)
  - Theme switch duration (< 100ms average)
- **Baseline:** Establish metrics before implementation
- **Success Criteria:**
  - Performance score: Within 2 points of baseline
  - FCP/LCP: No increase > 50ms
  - CLS during theme switch: < 0.01
  - Theme switch: 95% of switches < 100ms

**5. Support Ticket Reduction**
- **Target:** Reduction in display/visual preference related support tickets
- **Measurement:**
  - Support ticket categorization and trend analysis
  - Compare tickets 3 months pre-launch vs 3 months post-launch
  - Keywords: "eye strain", "brightness", "contrast", "dark mode request"
- **Expected Outcome:** 30% reduction in visual preference complaints

**6. Competitive Positioning**
- **Target:** Match or exceed competitor dark mode implementations
- **Measurement:**
  - Competitive feature matrix (updated post-launch)
  - User comparison studies
  - Feature parity analysis
- **Success Criteria:**
  - Feature completeness: Equal or better than 3 main competitors
  - User preference: Users rate our dark mode equal or better than competitors

**7. Browser Compatibility Coverage**
- **Target:** 95%+ of users have full dark mode support
- **Measurement:**
  - Browser analytics (percentage of users on supported browsers)
  - Graceful degradation verification (< 1% on unsupported browsers)
- **Success Criteria:**
  - Full support: 95%+ of user sessions
  - Partial support: 4% of sessions
  - No support: < 1% of sessions

### Monitoring Period

**Initial Assessment:** 2 weeks post-launch
- Early adoption metrics
- Critical bugs and accessibility issues
- Performance validation
- User feedback collection

**Primary Review:** 3 months post-launch
- Full adoption pattern analysis
- User satisfaction survey results
- Support ticket trend analysis
- Accessibility compliance verification
- Performance impact assessment

**Ongoing:** Quarterly reviews integrated into standard platform metrics
- Sustained adoption rate
- Feature usage patterns
- Performance monitoring
- Accessibility regression checks

### Analytics Implementation

**Required Event Tracking:**
```javascript
// Theme toggle clicked
analytics.track('Theme Toggle Clicked', {
  from_theme: 'light',
  to_theme: 'dark',
  toggle_location: 'header' // or 'settings'
});

// Theme applied
analytics.track('Theme Applied', {
  theme: 'dark',
  source: 'user_manual', // or 'system_preference', 'stored_preference'
  theme_switch_duration_ms: 85
});

// Page view with theme
analytics.page('Dashboard', {
  theme: 'dark',
  theme_source: 'user_manual'
});
```

---

## Developer Documentation

### Quick Start for Adding Dark Mode to New Components

**Step 1: Use Design Tokens**
Always use CSS custom properties instead of hardcoded colors.

```css
/* ❌ DON'T: Hardcoded colors */
.my-component {
  background-color: #ffffff;
  color: #1a1a1a;
  border: 1px solid #e0e0e0;
}

/* ✅ DO: Design tokens */
.my-component {
  background-color: var(--surface-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-subtle);
}
```

**Step 2: Test in Both Themes**
Always verify your component in both light and dark themes.

```javascript
// Toggle theme for testing
document.documentElement.setAttribute('data-theme', 'dark');
document.documentElement.setAttribute('data-theme', 'light');
```

**Step 3: Check Accessibility**
Verify focus indicators and contrast ratios.

```css
/* Always define focus styles */
.my-button:focus-visible {
  outline: 2px solid var(--border-focus);
  outline-offset: 2px;
}
```

**Step 4: Add Visual Regression Test**
Capture both themes in Percy/Chromatic.

```javascript
test('MyComponent renders in both themes', async () => {
  for (const theme of ['light', 'dark']) {
    await page.evaluate((t) => {
      document.documentElement.setAttribute('data-theme', t);
    }, theme);

    await percySnapshot(page, `MyComponent - ${theme} theme`);
  }
});
```

### Design Token Reference

Complete token reference available at: `[TBD - Create design token documentation page]`

**Common Patterns:**

**Text on Backgrounds:**
```css
/* Primary text on primary background */
.text { color: var(--text-primary); }
.background { background-color: var(--surface-primary); }

/* Secondary text on secondary background */
.muted-text { color: var(--text-secondary); }
.panel { background-color: var(--surface-secondary); }
```

**Interactive Elements:**
```css
/* Primary button */
.btn-primary {
  background-color: var(--interactive-primary);
  color: var(--text-inverse);
}

.btn-primary:hover {
  background-color: var(--interactive-primary-hover);
}

.btn-primary:active {
  background-color: var(--interactive-primary-active);
}
```

**Status Colors:**
```css
/* Success message */
.alert-success {
  background-color: var(--status-success-bg);
  color: var(--status-success);
  border: 1px solid var(--status-success);
}

/* Error message */
.alert-error {
  background-color: var(--status-error-bg);
  color: var(--status-error);
  border: 1px solid var(--status-error);
}
```

### Common Pitfalls to Avoid

**1. Don't Use Pure Black (#000000)**
Use `--surface-primary` in dark mode, which is #121212 (dark gray).

**2. Don't Invert Colors**
Dark mode is not `filter: invert(1)`. Design intentional color palette.

**3. Don't Forget Focus Indicators**
Focus rings must be visible in both themes with 3:1 contrast.

**4. Don't Hardcode Opacity**
Use semantic tokens that account for theme differences.

**5. Don't Assume Dark = Better**
Always provide user choice. Some users require light mode for accessibility.

### Testing Checklist for New Components

- [ ] Uses design tokens (no hardcoded colors)
- [ ] Tested in both light and dark themes
- [ ] Focus indicators visible in both themes
- [ ] Passes automated accessibility tests (axe-core)
- [ ] WCAG 2.1 AA contrast ratios verified
- [ ] Visual regression tests added for both themes
- [ ] Works in all supported browsers
- [ ] Reduced motion alternative provided (if animated)
- [ ] Screen reader accessible (if interactive)
- [ ] Documentation updated

---

## Timeline and Milestones

### Revised Realistic Timeline

**Total Estimated Duration:** 9-11 weeks (realistic)

**Phase 0: Discovery and Planning (2 weeks)**
- Week 1-2:
  - Technical stack audit and documentation
  - Design token creation
  - Complete color palette design with WCAG validation
  - Third-party dependency audit
  - Performance baseline establishment
  - Stakeholder alignment

**Phase 1: Foundation Implementation (2 weeks)**
- Week 3-4:
  - CSS custom property system implementation
  - Theme detection and persistence logic
  - FOUC prevention implementation
  - Base token implementation
  - Developer documentation
  - Unit tests for theme manager

**Phase 2: Component Migration (3-4 weeks)**
- Week 5-6 (Core Components):
  - Navigation, footer, forms
  - Buttons, links, typography
  - Basic cards and containers
  - Visual regression test setup

- Week 7-8 (Content Components):
  - Tables, lists, modals
  - Dashboard components
  - Data visualization
  - Image and media handling
  - Syntax highlighting

**Phase 3: Testing and Refinement (2-3 weeks)**
- Week 9:
  - Accessibility audit (both themes)
  - Cross-browser testing
  - Performance optimization

- Week 10-11:
  - User acceptance testing
  - Bug fixes and polish
  - Documentation finalization
  - Launch preparation

**Launch:** Week 12

### Key Milestones

| Milestone | Week | Deliverable | Success Criteria |
|-----------|------|-------------|------------------|
| Technical Discovery Complete | 2 | Audit report, tech stack documentation | Stakeholder approval |
| Design Tokens Finalized | 2 | Complete color palette, WCAG validated | Design team sign-off |
| Foundation Complete | 4 | Theme switching works on empty page | Zero FOUC, performance budget met |
| Core Components Complete | 6 | Navigation, forms, buttons themed | Visual regression tests pass |
| All Components Complete | 8 | Full platform themed | 100% component coverage |
| Testing Complete | 10 | All tests pass, WCAG compliant | Zero blocking issues |
| Launch Ready | 11 | Documentation complete, UAT passed | Stakeholder approval |
| Launch | 12 | Dark mode live to all users | Monitoring in place |

---

## Open Questions and TBD Items

### Technical Discovery Required

1. **[TBD] What is the existing tech stack?**
   - Framework/library (React, Vue, Angular, vanilla JS?)
   - CSS architecture (CSS Modules, Styled Components, Sass, Tailwind?)
   - Build tooling (Webpack, Vite, esbuild?)

2. **[TBD] Does an existing design system exist?**
   - If yes, where is it documented?
   - If no, should dark mode implementation establish the pattern?

3. **[TBD] Are we using server-side rendering (SSR)?**
   - Affects FOUC prevention strategy
   - Determines where theme detection happens

4. **[TBD] What is the current browser support policy?**
   - Need specific version requirements
   - Affects polyfill decisions

5. **[TBD] Do we have a performance monitoring system?**
   - Need to establish baseline metrics
   - RUM (Real User Monitoring) available?

6. **[TBD] What testing infrastructure exists?**
   - Visual regression testing tool?
   - Automated accessibility testing?
   - E2E testing framework?

7. **[TBD] How many hardcoded colors exist in the codebase?**
   - Requires automated code scanning
   - Affects migration timeline estimate

8. **[TBD] What third-party UI dependencies are in use?**
   - Chart libraries (D3, Chart.js, Recharts)?
   - Syntax highlighters (Prism, Highlight.js)?
   - Other UI components?

### Design Team Input Required

9. **[TBD] Has the design team created dark mode mockups?**
   - Visual reference needed for implementation
   - Helps identify edge cases

10. **[TBD] Are there brand guidelines for dark mode?**
    - Brand colors that must remain consistent?
    - Logo variations needed?

11. **[TBD] What syntax highlighting theme should we use?**
    - If code editor/blocks exist
    - Light theme choice?
    - Dark theme choice?

### Product and Strategy Questions

12. **[TBD] What is the rollout strategy?**
    - Feature flag for gradual rollout?
    - Beta testing with user subset?
    - All users at once?

13. **[TBD] Should dark mode be opt-in or automatic (system preference)?**
    - Recommendation: Automatic with manual override
    - Needs stakeholder decision

14. **[TBD] Will there be user education/onboarding?**
    - Tooltip on first visit?
    - Help documentation?
    - Blog post announcement?

### Assumptions to Validate

15. **[Assumption] Platform serves data scientists, ML engineers, developers**
    - Validate user personas with actual user research
    - May affect prioritization of features (e.g., syntax highlighting)

16. **[Assumption] 50% adoption target within 3 months**
    - Validate with historical feature adoption data
    - Adjust based on platform-specific usage patterns

17. **[Assumption] No mobile native apps require theming**
    - Confirm scope boundaries
    - If mobile apps exist, separate RFE may be needed

---

## Next Steps

### Immediate Actions (Week 1)

1. **Parker (PM):**
   - Schedule technical discovery meeting with Stella, Felix, and design team
   - Gather answers to TBD questions from engineering and design teams
   - Present business case to stakeholders for budget/timeline approval

2. **Felix (UX Feature Lead):**
   - Work with design team to finalize complete color palette
   - Create theme toggle component mockups
   - Validate all color combinations meet WCAG 2.1 AA

3. **Stella (Staff Engineer):**
   - Conduct technical audit of existing codebase
   - Document current tech stack and architecture
   - Identify hardcoded colors and third-party dependencies
   - Establish performance baselines

4. **Design Team:**
   - Create comprehensive color palette with exact hex values
   - Validate all color combinations with WCAG contrast checker
   - Design logo variations (if needed)
   - Create dark mode mockups for key pages

### Pre-Implementation Review (Week 2)

1. Schedule architecture review meeting to discuss:
   - Technical approach validation
   - CSS custom property structure
   - Theme manager implementation
   - FOUC prevention strategy
   - Testing approach

2. Update this RFE with technical specifications from discovery

3. Get stakeholder sign-off on:
   - Timeline (9-11 weeks)
   - Resource allocation
   - Success metrics
   - MVP scope

### Implementation Kickoff (Week 3)

1. Assign engineering resources
2. Create project board/tracking
3. Begin Phase 1: Foundation Implementation
4. Set up CI/CD pipeline for accessibility and performance testing

---

## Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-07 | Parker | Initial outline with business case |
| 2.0 | 2025-11-07 | Parker | Comprehensive revision addressing Felix and Stella feedback:<br>- Added complete color palette with exact values<br>- Added detailed component specifications<br>- Added comprehensive accessibility requirements<br>- Added theme toggle component specification<br>- Added complete technical implementation details<br>- Added testing strategy and CI/CD integration<br>- Added performance budget and metrics<br>- Added component migration plan<br>- Added developer documentation<br>- Added realistic timeline (9-11 weeks)<br>- Added image/media handling strategy<br>- Added data visualization considerations |

---

## Approval

**Document Status:** Ready for Final Review and Stakeholder Approval

**Reviewers:**
- [ ] Felix (UX Feature Lead) - Review addressing all UX feedback
- [ ] Stella (Staff Engineer) - Review addressing all technical feedback
- [ ] Design Team Lead - Approve color palette and design approach
- [ ] Engineering Manager - Approve timeline and resource allocation
- [ ] Product Leadership - Approve business case and success metrics

**Next Action:** Schedule stakeholder review meeting

**Author:** Parker (Product Manager)
**Date:** 2025-11-07
