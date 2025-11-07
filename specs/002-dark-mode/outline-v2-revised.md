# RFE Outline v2: Dark Mode Implementation (Revised)

**Feature ID**: 002-dark-mode
**Created**: 2025-11-07
**Status**: Revised Outline (Post-Agent Review)
**Target Platform**: www.github.com/ambient-code/platform
**Authors**: Parker (Product Manager), with input from Aria (UX Architect) and Felix (UX Feature Lead)

---

## Changes from v1

**Major additions based on agent feedback:**
1. **Pre-Implementation Requirements** section added (2-3 weeks UX work before development)
2. **User Journey Maps** section added (Aria feedback)
3. **Design Token Architecture** section expanded with three-tier system (Aria + Felix feedback)
4. **Component Inventory** section added with 50+ components identified (Felix feedback)
5. **Accessibility Requirements** expanded with component-specific criteria (both agents)
6. **Data Visualization Strategy** section added (both agents)
7. **Technical Implementation Details** expanded with CSS architecture (Felix feedback)
8. **Visual Regression Testing** plan detailed (Felix feedback)
9. **Edge Cases** section added covering 8 scenarios (Felix feedback)
10. **User Research Plan** added with validation approach (Aria feedback)

**Citation improvements:** Added internal documentation references and labeled assumptions clearly.

---

## 1. Executive Summary

*(Content unchanged from v1 - citations verified by both agents)*

### Business Justification

Dark mode has evolved from a niche feature to a mainstream user expectation with significant adoption metrics:

- **82% of smartphone users actively use dark mode** [[Dark Mode Usage Statistics 2025](https://wifitalents.com/dark-mode-usage-statistics/)]
- **Around 80% of users prefer having a dark mode option, and over 65% actively use it** [[Dark Mode Statistics Forms.app](https://forms.app/en/blog/dark-mode-statistics)]
- **Battery savings up to 47% on OLED screens** [[Dark Mode Statistics Forms.app](https://forms.app/en/blog/dark-mode-statistics)]
- **Over 80% report reduced eye strain** [[Dark Mode Usage Statistics](https://wifitalents.com/dark-mode-usage-statistics)]

### Market Analysis

The AI/ML platform market is highly competitive. Major platforms (GitHub, VS Code, Jupyter, Google Colab) all offer dark mode as standard. Our customers expect feature parity.

### Value Proposition

- Reduced eye strain for data scientists and ML engineers during extended sessions
- Battery conservation on mobile/laptop devices
- Competitive parity with leading ML/AI platforms
- Improved accessibility for users with photophobia or light sensitivity

**Critical requirement:** 42% of users still prefer light mode [[Dark Mode Statistics Forms.app](https://forms.app/en/blog/dark-mode-statistics)], making it essential to offer BOTH options rather than forcing one theme.

---

## 2. Pre-Implementation Requirements

**STATUS: BLOCKING FOR DEVELOPMENT** - These UX foundations must be established before implementation begins.

### 2.1 User Journey Mapping (1 week)
**Owner**: UX Architect (Aria) + UX Feature Lead (Felix)

**Required journey maps:**
1. **First-Time User Discovery**: How dark mode is communicated during onboarding
2. **Cross-Device Synchronization**: User works on desktop → switches to mobile → expects consistent theme
3. **Collaborative Scenarios**: User shares dashboard link → whose theme preference applies?
4. **Feature Migration**: Existing users suddenly have dark mode → change management plan
5. **Contextual Switching**: User switches between day (light) and evening (dark) work sessions

**Deliverable**: Journey map document with user flows, pain points, and UX requirements

### 2.2 Design Token Architecture (2 weeks)
**Owner**: UX Architect (Aria) + Design System Lead

**Required specifications:**

**Three-Tier Token System** [[Medium: Design Tokens Guide](https://medium.com/design-bootcamp/color-tokens-guide-to-light-and-dark-modes-in-design-systems-146ab33023ac)]:

1. **Base Tokens** (theme-specific palettes):
   - Light: `#FFFFFF`, `#F9FAFB`, `#3B82F6`
   - Dark: `#121212`, `#1F2937`, `#60A5FA`

2. **Semantic Tokens** (purpose-based):
   - `color.background.primary`
   - `color.text.primary`
   - `color.border.default`
   - `color.action.primary`

3. **Component Tokens** (component-specific):
   - `button.background.primary`
   - `input.border.focus`

**Deliverable**: Complete token taxonomy with naming conventions and governance process

### 2.3 Component Audit (1 week)
**Owner**: UX Feature Lead (Felix) + Frontend Tech Lead

**Component inventory required** (see Section 5 for full list):
- 50+ UI components across 9 categories
- State requirements for each (default, hover, focus, disabled, error)
- Priority classification (P0: MVP, P1: Phase 2)
- Third-party component identification

**Deliverable**: Component audit spreadsheet with theming readiness scorecard

### 2.4 User Research Validation (3-4 weeks)
**Owner**: UX Researcher + Product Manager (Parker)

**Research activities:**
1. **User Interviews** (n=15-20): Current pain points, dark mode expectations
2. **Quantitative Survey** (n=200+): Demand quantification within our user base
3. **Support Ticket Analysis**: Review tickets mentioning "brightness," "eye strain," "dark mode"
4. **Analytics Baseline**: Current session duration, time-of-day patterns, satisfaction scores

**Deliverable**: User research findings report with platform-specific insights

### 2.5 Accessibility Criteria Expansion (1 week)
**Owner**: UX Architect (Aria)

**Beyond WCAG 2.1 Level AA minimums:**
- WCAG 1.4.11: Non-text contrast (3:1 for UI components) [[W3C WCAG 2.1](https://www.w3.org/WAI/WCAG21/Understanding/non-text-contrast.html)]
- WCAG 2.4.7: Focus indicator specifications for dark mode
- `prefers-reduced-motion` support [[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)]
- Colorblind-safe palette validation
- Assistive technology testing matrix

**Deliverable**: Updated acceptance criteria in specification

**Total Pre-Implementation Time: 6-8 weeks**

---

## 3. Business Impact and Customer Requirements

*(Updated with internal citations and labeled assumptions)*

### Customer Pain Points

**P1 - Eye Strain and Fatigue:**
- Data scientists and ML engineers report extended sessions [Assumption: based on technical role standards, 4-8 hours]
- Over 80% of users report reduced eye strain with dark mode [[Dark Mode Usage Statistics](https://wifitalents.com/dark-mode-usage-statistics/)]

**P2 - Inconsistent Cross-Platform Experience:**
- Users have dark mode enabled system-wide
- Jarring visual transition when switching to our platform
- [Requires validation: Support ticket analysis - see Section 2.4]

**P3 - Battery Life Concerns:**
- Dark mode can save up to 47% battery on OLED screens [[Dark Mode Statistics](https://forms.app/en/blog/dark-mode-statistics)]

**P4 - Accessibility Barriers:**
- Users with photophobia face barriers [[BOIA](https://www.boia.org/blog/dark-mode-can-improve-text-readability-but-not-for-everyone)]

### Expected Business Outcomes

[Target: Based on comparable feature launches - requires validation through A/B testing]
- **User Satisfaction**: 25-35% increase in UI/visual comfort satisfaction scores
- **Session Duration**: 10-15% increase in average session length
- **Feature Adoption**: 65-70% of users enabling dark mode within 30 days (aligned with industry benchmarks)
- **Support Ticket Reduction**: 20-30% reduction in UI-related requests

[Baseline metrics required: See Section 2.4 - Analytics baseline]

### Target Users

**Primary:**
- Data Scientists (power users, extended sessions)
- ML Engineers (development-focused, familiar with dark mode from IDEs)
- Platform Administrators (extended monitoring)

**Secondary:**
- Business Analysts
- Occasional users
- Mobile users

---

## 4. User Journey Maps

**NEW SECTION** (Aria Priority 1 recommendation)

### Journey 1: First-Time User Discovery
- **Touchpoint**: User logs in for first time → System detects OS dark mode preference → Platform applies dark theme automatically
- **Communication**: Subtle toast notification: "We've matched your system's dark mode preference. Change anytime in [Settings]"
- **Fallback**: If OS preference not detected, default to light mode with theme toggle visible

### Journey 2: Cross-Device Synchronization
- **Scenario**: User sets dark mode on desktop → logs in on mobile
- **Expected behavior**: Theme syncs via user profile (if authenticated)
- **Unauthenticated users**: localStorage per-device (documented limitation)

### Journey 3: Collaborative Content Sharing
- **Decision required**: When User A shares dashboard with User B, whose theme applies?
- **Recommended approach**: Shared content respects recipient's theme preference
- **Alternative**: Provide "share with theme" option (Phase 2)

### Journey 4: Feature Migration for Existing Users
- **Communication plan**:
  - Pre-launch email: "Dark mode coming next week"
  - Launch day: In-app banner announcing feature
  - First exposure: Tooltip pointing to theme toggle
- **No automatic switching**: Existing users remain in light mode unless they opt-in

### Journey 5: Contextual Switching (Day/Evening)
- **MVP**: Manual toggle
- **Phase 2**: Time-based auto-switching option (user-configurable)

---

## 5. Component Inventory

**NEW SECTION** (Felix Priority 1 recommendation)

### 5.1 Navigation Components (P0 - MVP Required)
- Top navigation bar (logo, nav links, search, user menu)
- Side navigation/drawer (collapsible sections, nested items)
- Breadcrumbs
- Footer
- Tabs

### 5.2 Form Components (P0 - MVP Required)
- Text inputs (default, focus, disabled, error, success states)
- Text areas
- Select dropdowns (menu background, selected option, hover)
- Checkboxes (checked, unchecked, indeterminate, disabled)
- Radio buttons
- Toggle switches (dark mode toggle itself)
- Date pickers (calendar popup, selected date, today indicator)
- File upload (drag-drop area, file list, progress)
- Form validation messages (error, warning, success, info)

### 5.3 Button Components (P0 - MVP Required)
- Primary, secondary, tertiary/ghost, danger variants
- All states: default, hover, active, focus, disabled, loading
- Icon buttons
- Button groups

### 5.4 Data Display Components (P0 - MVP Required)
- Tables (header, alternating rows, hover, selected, sort indicators, empty state)
- Lists (ordered, unordered, definition)
- Cards (background, borders, shadows, header/footer)
- Data grids
- Pagination
- Badges and labels/tags
- Tooltips
- Avatars

### 5.5 Feedback Components (P0 - MVP Required)
- Toast notifications (success, error, warning, info)
- Alert banners
- Modal dialogs (overlay scrim, modal background, close button)
- Progress indicators (bars, spinners, skeleton screens)
- Empty states
- Error pages (404, 500)

### 5.6 Data Visualization Components (P0 - MVP Critical)
- Line charts (line colors, grid lines, axes, data points)
- Bar charts
- Pie/donut charts
- Scatter plots
- Heatmaps (color scale adjustments)
- Dashboards (widget backgrounds, borders, metrics)

**Dark mode chart requirements:**
- Desaturated color palettes for dark backgrounds [Assumption: based on common dark mode practices]
- 3:1 contrast for graphical elements (WCAG 1.4.11)
- Colorblind-safe validation
- Grid lines subtle but visible

### 5.7 ML Platform-Specific Components (P0 - MVP Critical)
- Code editor cells (syntax highlighting colors - recommend "One Dark" or "Dracula" scheme)
- Log viewers (log level colors: INFO, WARN, ERROR, DEBUG)
- Model metric displays (accuracy, precision, recall visualizations)
- Training progress indicators
- Confusion matrices
- ROC curves
- Resource utilization gauges (CPU, memory, GPU)

### 5.8 Media Components (P1 - Post-MVP)
- Images (may need dark variants or CSS filters)
- Icons (SVG fill colors)
- Logos (likely need dark variant for brand recognition)
- Video players (if applicable)

### 5.9 Utility Components (P1 - Post-MVP)
- Dividers
- Accordions
- Steppers/wizards
- Collapsible panels

**Total Components: 50+ across 9 categories**

**Priority:**
- P0 (MVP): Navigation, Forms, Buttons, Data Display, Feedback, Data Viz, ML-specific = ~40 components
- P1 (Phase 2): Media, Utility = ~10 components

---

## 6. Design Token Architecture

**EXPANDED SECTION** (Aria + Felix Priority 1 recommendations)

### 6.1 Three-Tier Token System

Based on design system best practices [[Atlassian Design Tokens](https://atlassian.design/tokens/design-tokens/)]:

#### Tier 1: Primitive Tokens (Theme-Specific Values)

```css
/* Light Theme Primitives */
:root, [data-theme="light"] {
  --primitive-white: #ffffff;
  --primitive-gray-50: #fafafa;
  --primitive-gray-100: #f5f5f5;
  --primitive-gray-300: #e0e0e0;
  --primitive-gray-700: #616161;
  --primitive-gray-900: #212121;
  --primitive-blue-500: #2196f3;
  --primitive-blue-700: #1976d2;
  /* ... complete palette */
}

/* Dark Theme Primitives */
[data-theme="dark"] {
  --primitive-black: #000000;
  --primitive-gray-900: #121212;  /* Base background - NOT pure black */
  --primitive-gray-800: #1e1e1e;  /* Elevated surfaces */
  --primitive-gray-700: #2c2c2c;
  --primitive-gray-300: #424242;
  --primitive-gray-50: #ffffff;   /* Primary text */
  --primitive-blue-500: #2196f3;
  --primitive-blue-300: #66b3ff;  /* Lighter for dark backgrounds */
  /* ... complete palette */
}
```

**Design principle:** Avoid pure black (#000000) - causes halation effect [[99designs](https://99designs.com/blog/web-digital/dark-mode/)]

#### Tier 2: Semantic Tokens (Purpose-Based)

```css
/* Backgrounds */
--color-background-primary: var(--primitive-gray-50);    /* Light: white, Dark: #121212 */
--color-background-secondary: var(--primitive-gray-100); /* Light: #f5f5f5, Dark: #1e1e1e */
--color-background-elevated: var(--primitive-white);     /* Light: white, Dark: #2c2c2c */

/* Text */
--color-text-primary: var(--primitive-gray-900);
--color-text-secondary: var(--primitive-gray-700);
--color-text-disabled: var(--primitive-gray-400);
--color-text-on-primary: var(--primitive-white);

/* Borders */
--color-border-default: var(--primitive-gray-300);
--color-border-subtle: var(--primitive-gray-200);
--color-border-interactive: var(--primitive-blue-500);

/* Interactive states */
--color-interactive-primary: var(--primitive-blue-700);
--color-interactive-primary-hover: var(--primitive-blue-800);
--color-focus-indicator: #0066cc;  /* Light mode */
/* Dark mode: #66b3ff for sufficient contrast */

/* Status colors */
--color-status-success: var(--primitive-green-600);
--color-status-error: var(--primitive-red-600);
--color-status-warning: var(--primitive-yellow-700);
--color-status-info: var(--primitive-blue-600);
```

#### Tier 3: Component Tokens (Component-Specific)

```css
/* Button tokens */
--button-primary-bg: var(--color-interactive-primary);
--button-primary-text: var(--color-text-on-primary);
--button-primary-hover-bg: var(--color-interactive-primary-hover);
--button-disabled-bg: var(--color-background-secondary);
--button-disabled-text: var(--color-text-disabled);

/* Input tokens */
--input-bg: var(--color-background-elevated);
--input-border: var(--color-border-default);
--input-border-focus: var(--color-border-interactive);
--input-text: var(--color-text-primary);
--input-placeholder: var(--color-text-secondary);

/* Card tokens */
--card-background: var(--color-background-elevated);
--card-border: var(--color-border-subtle);
--card-shadow: /* Light: rgba(0,0,0,0.1), Dark: rgba(0,0,0,0.4) */
```

### 6.2 Token Naming Conventions

**Pattern:** `--{category}-{property}-{variant}-{state}`

Examples:
- `--color-background-primary`
- `--button-primary-hover-bg`
- `--input-border-focus`

### 6.3 Token Governance

**Who can create tokens?** Design System Lead (approval required)
**Documentation:** Tokens documented in design system with usage guidelines
**Tooling:** Consider Style Dictionary for token transformation, Figma Tokens plugin

---

## 7. Technical Implementation Architecture

**EXPANDED SECTION** (Felix Priority 1 recommendation)

### 7.1 CSS Architecture: Data Attribute Theming

**Recommended approach:**

```html
<html data-theme="dark">
```

**Why data attribute vs class:**
- Semantic: theme is a state, not a style
- Single source of truth
- No class name conflicts
- Easy to query: `document.documentElement.dataset.theme`

### 7.2 System Preference Detection

```javascript
// Detect system preference
function getSystemTheme() {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    return 'dark';
  }
  return 'light';
}

// Listen for system preference changes
window.matchMedia('(prefers-color-scheme: dark)')
  .addEventListener('change', (e) => {
    if (getUserPreference() === 'auto') {
      applyTheme(e.matches ? 'dark' : 'light');
    }
  });
```

### 7.3 Preference Storage

```javascript
// Store user preference (three-state: 'light', 'dark', 'auto')
function setUserPreference(theme) {
  localStorage.setItem('theme-preference', theme);

  // If authenticated, sync to user profile
  if (user.isAuthenticated) {
    api.updateUserProfile({ themePreference: theme });
  }
}

// Apply theme on page load
function initTheme() {
  const userPref = localStorage.getItem('theme-preference') || 'auto';
  const theme = userPref === 'auto' ? getSystemTheme() : userPref;
  applyTheme(theme);
}
```

### 7.4 Flash of Unstyled Content (FOUC) Prevention

**CRITICAL: Inline script in `<head>` before any CSS:**

```html
<script>
  (function() {
    const theme = localStorage.getItem('theme-preference') || 'auto';
    const applied = theme === 'auto' ?
      (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light') :
      theme;
    document.documentElement.setAttribute('data-theme', applied);
  })();
</script>
```

**Why:** Prevents light theme flash when user has dark mode preference

### 7.5 Theme Toggle Component

**Three-state toggle (recommended):**

```html
<select aria-label="Theme preference" onchange="handleThemeChange(event)">
  <option value="auto">Auto (System)</option>
  <option value="light">Light</option>
  <option value="dark">Dark</option>
</select>
```

**Rationale:** Respects user autonomy, follows OS conventions (iOS, Android, Windows 11)

### 7.6 Cross-Tab Synchronization

```javascript
// Listen for localStorage changes in other tabs
window.addEventListener('storage', (e) => {
  if (e.key === 'theme-preference') {
    const newTheme = e.newValue === 'auto' ? getSystemTheme() : e.newValue;
    applyTheme(newTheme);
  }
});
```

### 7.7 Browser Compatibility

**Supported browsers:**
- Chrome: Last 2 versions
- Firefox: Last 2 versions
- Safari: Last 2 versions
- Edge: Chromium-based versions

**CSS Custom Properties:** Supported in all modern browsers (no IE11 support needed)
**prefers-color-scheme:** Chrome 76+, Firefox 67+, Safari 12.1+, Edge 79+

---

## 8. Accessibility Specification

**EXPANDED SECTION** (Aria + Felix recommendations)

### 8.1 WCAG 2.1 Level AA Compliance

#### WCAG 1.4.3: Contrast (Minimum)
- Normal text: **4.5:1 minimum**
- Large text (18pt or 14pt bold): **3:1 minimum**

**Component-specific application:**
- Button text to button background: 4.5:1
- Input text to input background: 4.5:1
- Link text to page background: 4.5:1
- Table text to table background: 4.5:1
- Chart labels to chart background: 4.5:1

#### WCAG 1.4.11: Non-text Contrast (Level AA)
- UI components and graphical objects: **3:1 minimum** [[W3C](https://www.w3.org/WAI/WCAG21/Understanding/non-text-contrast.html)]

**Component-specific application:**
- Icons: 3:1 to background
- Input borders: 3:1 to background
- Checkbox/radio borders: 3:1 to background
- Focus indicators: 3:1 to background
- Chart elements (bars, lines, data points): 3:1 to adjacent colors

#### WCAG 2.4.7: Focus Visible (Level AA)

**Dark mode specific requirements:**

```css
/* Light mode focus */
:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* Dark mode focus */
[data-theme="dark"] :focus-visible {
  outline: 2px solid #66b3ff;  /* Lighter blue for contrast */
  outline-offset: 2px;
}
```

**Testing:** Tab through entire app in both themes, verify focus always visible

#### WCAG 1.4.1: Use of Color (Level A)
- Color is not the only visual means of conveying information

**Examples:**
- Form errors: red border + error icon + error message (not just color)
- Chart lines: different dash patterns + colors
- Status indicators: color + icon (running = green + spinner, error = red + X)

### 8.2 Beyond WCAG Minimums

#### Reduced Motion Support

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Rationale:** Users with vestibular disorders sensitive to theme transition animations [[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)]

#### Colorblind-Safe Palettes
- Test data visualization palettes with colorblindness simulators (Color Oracle, Sim Daltonism)
- Don't rely solely on color to distinguish data (use patterns, textures, labels)

#### Low Vision and Screen Magnification
- Test dark mode at 200% and 400% browser zoom
- Verify no horizontal scrolling introduced
- Ensure focus indicators remain visible at high zoom

### 8.3 Screen Reader Support

**Theme toggle implementation:**

```html
<button
  aria-label="Toggle dark mode"
  aria-pressed="false"
  onclick="toggleTheme()">
  <svg aria-hidden="true"><!-- icon --></svg>
  <span class="visually-hidden">Dark mode: Off</span>
</button>

<!-- Live region for announcements -->
<div id="theme-announcement"
     role="status"
     aria-live="polite"
     aria-atomic="true"
     class="visually-hidden">
</div>
```

**Announcement on theme change:**

```javascript
function toggleTheme() {
  const newTheme = currentTheme === 'light' ? 'dark' : 'light';
  applyTheme(newTheme);

  // Announce to screen readers
  document.getElementById('theme-announcement').textContent =
    `${newTheme} mode activated`;
}
```

### 8.4 Accessibility Testing Plan

**Phase 1: Automated Testing**
- Tools: axe DevTools, Lighthouse, WAVE
- Run on every page in both themes
- Coverage: ~57% of accessibility issues [Assumption: industry estimate]

**Phase 2: Manual Contrast Testing**
- Tool: WebAIM Contrast Checker
- Test all semantic token pairs
- Document all ratios in spreadsheet
- Acceptance: All ratios meet WCAG AA minimums

**Phase 3: Keyboard Navigation Testing**
- Disconnect mouse, tab through entire app
- Verify all interactive elements reachable
- Verify focus indicators visible in both themes
- Verify no keyboard traps

**Phase 4: Screen Reader Testing**
- Test with NVDA (Windows), JAWS (Windows), VoiceOver (Mac), TalkBack (Android)
- Verify theme toggle announces state
- Verify theme change announced
- Verify all content accessible

**Phase 5: Real User Testing**
- Recruit users with disabilities (visual, motor, cognitive)
- Task-based testing with dark mode
- Iterate based on feedback

**Phase 6: Compliance Audit**
- Comprehensive WCAG 2.1 Level AA audit
- Consider VPAT (Voluntary Product Accessibility Template) documentation

---

## 9. Data Visualization Theming Strategy

**NEW SECTION** (Aria + Felix recommendations)

### 9.1 Chart Color Palettes

**Problem:** Bright saturated colors cause eye strain on dark backgrounds

**Solution:** Desaturated, slightly muted palettes for dark mode

**Categorical palette (for different data series):**
- Light mode: `#2196F3, #F44336, #4CAF50, #FF9800, #9C27B0, #00BCD4`
- Dark mode: `#64B5F6, #EF5350, #66BB6A, #FFB74D, #BA68C8, #4DD0E1` (lighter, desaturated)

**Sequential palette (for heatmaps):**
- Ensure sufficient contrast between steps
- Test with colorblindness simulators

**Diverging palette (for positive/negative values):**
- Blue-to-red scales common but problematic for colorblind users
- Consider blue-to-orange or purple-to-green alternatives

**Accessibility requirement:** All chart elements must meet 3:1 contrast (WCAG 1.4.11)

### 9.2 Chart Component Theming

**Grid lines:**
- Light mode: `rgba(0, 0, 0, 0.1)` (subtle gray)
- Dark mode: `rgba(255, 255, 255, 0.1)` (subtle white)

**Axes and labels:**
- Use `--color-text-secondary` token
- Ensure 4.5:1 contrast for readability

**Tooltips:**
- Background: `--color-background-elevated`
- Border: `--color-border-default`
- Text: `--color-text-primary`

**Legends:**
- Use semantic tokens for consistency
- Ensure legend items distinguishable (color + shape/icon)

### 9.3 ML-Specific Visualizations

**Confusion matrices:** Use color scales that work for colorblind users (not just red-green)

**ROC curves:** Ensure curve line has 3:1 contrast to background

**Training progress charts:** Use muted colors for dark backgrounds, avoid pure bright colors

**Reference:** [[Medium: Implementing Dark Mode for Data Visualizations](https://ananyadeka.medium.com/implementing-dark-mode-for-data-visualizations-design-considerations-66cd1ff2ab67)]

---

## 10. Edge Cases and Special Scenarios

**NEW SECTION** (Felix recommendation)

### Edge Case 1: Nested Themes (Iframes)
- **Issue:** Third-party embedded content (Stripe, YouTube, Google Maps) cannot be themed
- **Solution:** Add subtle border around iframes to visually separate; document limitation

### Edge Case 2: Print Styles
- **Issue:** Printing dark mode wastes ink and may be unreadable
- **Solution:**
  ```css
  @media print {
    [data-theme="dark"] {
      --color-background-primary: #ffffff;
      --color-text-primary: #000000;
      /* Override to light theme for printing */
    }
  }
  ```

### Edge Case 3: High Contrast Mode
- **Issue:** OS high contrast mode overrides CSS colors
- **Solution:** Detect via `@media (prefers-contrast: high)` and ensure compatibility

### Edge Case 4: Reduced Motion (covered in Section 8.2)

### Edge Case 5: Mobile Browser Theme Color
```html
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#121212" media="(prefers-color-scheme: dark)">
```

### Edge Case 6: Multi-Tab Synchronization (covered in Section 7.6)

### Edge Case 7: Authenticated vs Unauthenticated Users
- **Scenario:** User sets theme while logged out, then logs in
- **Solution:** On login, server preference takes priority → update localStorage

### Edge Case 8: Shared Content Theme Behavior
- **Decision:** Shared dashboards/reports respect recipient's theme preference (not sender's)
- **Alternative:** Provide "share with theme" option (Phase 2)

---

## 11. Implementation Scope

*(Updated with component priorities)*

### In-Scope for MVP

1. **Theme Infrastructure**
   - CSS custom properties with three-tier token system
   - Light and dark color palettes
   - Theme detection (`prefers-color-scheme`) and application logic
   - User preference persistence (localStorage + user profile sync)
   - FOUC prevention

2. **UI Components (P0 - ~40 components)**
   - All navigation, form, button, data display, feedback components
   - Data visualizations with dark-compatible color schemes
   - ML-specific components (code editor, log viewer, metrics)

3. **User Controls**
   - Three-state theme toggle (Auto/Light/Dark)
   - Settings panel entry
   - Keyboard accessibility

4. **Visual Quality**
   - Smooth transitions (0.3s) with reduced motion support
   - Contrast ratios meeting WCAG AA (4.5:1 text, 3:1 UI components)
   - Focus indicators visible in both themes

5. **Documentation**
   - User guide on theme switching
   - Design system documentation with token specifications
   - Developer guidelines for component theming

6. **Testing**
   - Visual regression testing (all components, both themes)
   - Accessibility testing (automated + manual)
   - Cross-browser testing
   - Keyboard navigation testing
   - Screen reader testing

### Out-of-Scope for MVP

1. **Custom Theme Creation** (Phase 2)
2. **Time-Based Auto-Switching** (Phase 2)
3. **High Contrast Mode** (separate feature, distinct from dark mode)
4. **Per-Component Theme Overrides** (Phase 2, monitor user feedback)
5. **Mobile Native App Theming** (if separate from web)

---

## 12. Acceptance Criteria

*(Expanded with accessibility and visual regression criteria)*

### Functional Criteria

**AC-001: Theme Detection**
- GIVEN user visits platform for first time
- WHEN their OS is set to dark mode
- THEN platform loads in dark theme automatically

**AC-002: Manual Theme Toggle**
- GIVEN user viewing platform in any theme
- WHEN they select theme preference (Auto/Light/Dark)
- THEN theme switches within 300ms with smooth transition
- AND new preference persists

**AC-003: Preference Persistence**
- GIVEN user selected theme preference
- WHEN they close and reopen browser
- THEN preference retained and applied immediately

**AC-004: Cross-Device Sync**
- GIVEN authenticated user sets theme on Device A
- WHEN they log in on Device B
- THEN preference applied on Device B

**AC-005: Component Coverage**
- GIVEN platform in dark mode
- WHEN navigating through all major sections
- THEN all P0 components display correctly
- AND no light-mode artifacts visible

### Accessibility Criteria

**AC-006: Text Contrast**
- GIVEN any theme active
- WHEN testing with WebAIM Contrast Checker
- THEN all text meets 4.5:1 minimum (normal text), 3:1 (large text)

**AC-007: UI Component Contrast**
- GIVEN any theme active
- WHEN testing interactive elements
- THEN all UI components meet 3:1 contrast to adjacent colors (WCAG 1.4.11)

**AC-008: Keyboard Accessibility**
- GIVEN user navigating with keyboard only
- WHEN pressing Tab to reach theme toggle
- THEN toggle receives visible focus indicator (2px, sufficient contrast)
- AND Enter/Space activates theme switch

**AC-009: Screen Reader Support**
- GIVEN screen reader active (NVDA, JAWS, VoiceOver)
- WHEN focusing on theme toggle
- THEN announces "Theme preference, currently [auto/light/dark], button"
- AND announces theme change when activated

**AC-010: Reduced Motion Support**
- GIVEN user has `prefers-reduced-motion: reduce` set
- WHEN theme toggle activated
- THEN theme switches instantly without transition animation

**AC-011: Colorblind Accessibility**
- GIVEN data visualizations in dark mode
- WHEN tested with colorblindness simulators
- THEN all data remains distinguishable (not relying on color alone)

### Performance Criteria

**AC-012: Theme Switch Performance**
- GIVEN theme toggle activated
- WHEN measuring transition time
- THEN completes within 300ms
- AND no layout shift or content reflow

**AC-013: Initial Load Performance**
- GIVEN theme applied on page load
- WHEN measuring impact
- THEN <50ms increase in load time
- AND no FOUC (flash of unstyled content)

### Quality Criteria

**AC-014: Visual Regression**
- GIVEN all P0 components
- WHEN visual regression tests run in both themes
- THEN no unintended color or contrast changes
- AND all components match approved baselines

**AC-015: Cross-Browser Consistency**
- GIVEN testing across Chrome, Firefox, Safari, Edge (latest versions)
- WHEN applying dark theme
- THEN visual appearance consistent across all browsers
- AND theme toggle functions correctly

**AC-016: Focus Indicator Visibility**
- GIVEN keyboard navigation in both themes
- WHEN tabbing through all interactive elements
- THEN focus indicator visible at all times
- AND meets 3:1 contrast requirement

---

## 13. Testing and Validation Strategy

### 13.1 Visual Regression Testing

**Tool selection:** Percy, Chromatic, or BackstopJS

**Process:**
1. Establish baselines: Render every P0 component in Storybook, capture in both themes
2. Component coverage: Every variant, every state, every size, both themes
3. Run on every PR: Automated visual regression in CI, flag differences for review
4. Responsive testing: Capture at mobile, tablet, desktop viewports

### 13.2 Accessibility Testing (see Section 8.4 for full plan)

**6-phase approach:**
1. Automated (axe DevTools, Lighthouse)
2. Manual contrast testing (WebAIM Contrast Checker, spreadsheet documentation)
3. Keyboard navigation (tab through entire app, both themes)
4. Screen reader testing (NVDA, JAWS, VoiceOver, TalkBack)
5. Real user testing (users with disabilities)
6. Compliance audit (WCAG 2.1 Level AA, consider VPAT)

### 13.3 Cross-Browser Testing

**Browser matrix:**
- Chrome (last 2 versions) - Desktop + Android
- Firefox (last 2 versions) - Desktop + Android
- Safari (last 2 versions) - Desktop + iOS
- Edge (Chromium) - Desktop

**Automated:** BrowserStack or Sauce Labs for E2E tests
**Manual:** Physical device testing pre-release

---

## 14. Risks and Mitigation Strategies

*(Updated with UX risks from Aria feedback)*

### Technical Risks (from v1)

**R-001: Incomplete Component Coverage** [HIGH impact, MEDIUM probability]
- **Mitigation:** Component audit (Section 2.3), coverage checklist, prioritize core journeys

**R-002: Performance Degradation** [MEDIUM impact, LOW probability]
- **Mitigation:** Performance testing, CSS optimization, FOUC prevention (Section 7.4)

**R-003: Browser Compatibility** [MEDIUM impact, LOW-MEDIUM probability]
- **Mitigation:** Modern browser support only, graceful degradation, testing matrix

### UX Risks (NEW - Aria feedback)

**R-010: Theme Preference Fragmentation** [MEDIUM impact, HIGH probability]
- **Scenario:** Users want different themes across devices/contexts but system forces single choice
- **Mitigation:** Three-state toggle (Auto/Light/Dark), per-device localStorage for unauthenticated, document limitation

**R-011: Shared Content Theme Ambiguity** [MEDIUM impact, MEDIUM probability]
- **Scenario:** Unclear whose theme applies when sharing dashboards
- **Mitigation:** Define clear behavior (recipient's theme), document in user guide

**R-012: Data Visualization Readability Degradation** [HIGH impact, MEDIUM-HIGH probability]
- **Scenario:** Charts designed for light become unreadable in dark
- **Mitigation:** Dedicated dark-mode palettes (Section 9), 3:1 contrast testing, colorblind validation

**R-013: Brand Identity Dilution** [MEDIUM impact, LOW probability]
- **Mitigation:** Ensure brand colors work in both themes, logo may need dark variant, consult marketing

**R-014: Learning Curve for Existing Users** [MEDIUM impact, MEDIUM probability]
- **Mitigation:** Gradual rollout, email announcement, in-app tooltip, easy toggle location

**R-015: Incomplete Third-Party Integration Theming** [MEDIUM impact, HIGH probability]
- **Mitigation:** Identify all iframes/widgets, test in dark mode, apply borders where needed, document limitations

### Accessibility Risks (from v1, severity updated)

**R-004: Color Contrast Failures** [HIGH impact, MEDIUM probability]
- **Mitigation:** Automated contrast checking, manual testing, component-specific criteria (Section 8)

**R-009: Accessibility Complaints** [HIGH impact, LOW probability if properly tested]
- **Mitigation:** Thorough 6-phase testing (Section 8.4), beta testing, maintain both themes, responsive support

### Business Risks (from v1)

**R-007: Delayed Delivery** [HIGH impact, MEDIUM probability]
- **Mitigation:** Phased rollout (MVP then enhancements), clear scope, 6-8 weeks pre-implementation UX work

**R-008: Low Feature Adoption** [MEDIUM impact, LOW probability]
- **Mitigation:** Default to OS preference (automatic engagement), in-app announcement, track metrics

---

## 15. Success Metrics

*(Unchanged from v1 - citations verified by agents)*

### Adoption Metrics

**M-001: Feature Adoption Rate**
- Target: 60-70% within 30 days [[Dark Mode Statistics](https://forms.app/en/blog/dark-mode-statistics)]
- Measurement: Analytics tracking theme preference

**M-002: Theme Preference Distribution**
- Target: 65:35 (dark:light) [[Dark Mode Statistics](https://forms.app/en/blog/dark-mode-statistics)]
- Measurement: Aggregate user preference data

### Engagement Metrics

**M-004: Session Duration Impact**
- Target: 10-15% increase for dark mode users (reduced eye strain hypothesis)
- Measurement: Session analytics segmented by theme

### Quality Metrics

**M-006: User Satisfaction Score**
- Target: 25-35% improvement [Target: Based on comparable feature launches]
- Measurement: Post-feature surveys, NPS

**M-007: Support Ticket Reduction**
- Target: 20-30% reduction in UI-related requests
- Measurement: Support ticket categorization

**M-008: Accessibility Compliance**
- Target: 100% WCAG 2.1 Level AA compliance
- Measurement: Automated + manual accessibility audits

### Technical Performance Metrics

**M-009: Theme Switch Performance**
- Target: <300ms on 90th percentile devices
- Measurement: Real User Monitoring (RUM)

**M-010: Page Load Performance**
- Target: <50ms increase in load time
- Measurement: Core Web Vitals

### Business Impact Metrics

**M-011: Net Promoter Score (NPS) Impact**
- Target: 2-5 point improvement
- Measurement: Quarterly NPS surveys

**M-012: Competitive Feature Parity**
- Target: Achieve parity on core UX features
- Measurement: Competitive feature matrix

---

## 16. Phased Rollout Plan

**NEW SECTION** (Aria recommendation)

### Phase 1: Prototype Testing (2-3 weeks)
- Create high-fidelity Figma prototypes of key screens
- Usability testing with n=8-10 users per persona
- Validate toggle discoverability, color palette readability
- Success criteria: >80% successful toggle discovery, >4/5 satisfaction

### Phase 2: Beta Release (4-6 weeks)
- Limited release to volunteer users (10-15% of user base)
- In-app feedback collection
- A/B test toggle placement options
- Success criteria: <5% negative feedback, >60% adoption

### Phase 3: Phased Rollout (6-8 weeks)
- Gradual rollout: 25% → 50% → 100% of users
- Continuous monitoring of satisfaction and support tickets
- Iteration based on feedback
- Success criteria: Achievement of target metrics (65-70% adoption)

---

## 17. Strategic Opportunities

**NEW SECTION** (Aria feedback)

### O-001: User Research Platform
- Use dark mode as pilot for broader user preference system
- Foundation for future personalization features
- Establishes user preference infrastructure

### O-002: Design System Maturity
- Dark mode forces token-based architecture modernization
- Creates reusable theming infrastructure for future themes (high contrast, custom branding)

### O-003: Accessibility Leadership
- Position platform as accessibility leader in ML/AI space
- Attracts enterprise customers with accessibility requirements
- Consider VPAT documentation for marketing

### O-004: Data-Driven UX Validation
- Rich instrumentation provides insights into user behavior
- Session duration correlations, time-of-day patterns
- A/B testing infrastructure for future features

---

## 18. Future Enhancements (Phase 2 Roadmap)

Based on user feedback and strategic opportunities:

1. **Time-Based Auto-Switching** (high user demand expected)
2. **Custom Theme Creation** (user-defined color schemes)
3. **Per-Component Theme Overrides** (advanced users: "dark UI with light charts")
4. **High Contrast Mode** (separate feature, distinct from dark mode)
5. **Accessibility VPAT Documentation** (enterprise appeal)

---

## Appendices

### Appendix A: Internal Documentation References

[To be added post-implementation]:
- Design System Documentation: [LINK REQUIRED]
- Component Library Repository: [LINK REQUIRED]
- User Research Findings: [LINK REQUIRED]
- Current NPS/Satisfaction Baseline: [LINK REQUIRED]
- Support Ticket Analysis: [LINK REQUIRED]

### Appendix B: External References

All external citations verified by agent reviews. See v1 outline for complete list.

Key references:
- [[W3C WCAG 2.1](https://www.w3.org/WAI/WCAG21/Understanding/)]
- [[MDN: prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)]
- [[Atlassian Design Tokens](https://atlassian.design/tokens/design-tokens/)]
- [[Dark Mode Statistics](https://forms.app/en/blog/dark-mode-statistics)]

---

## Document Control

**Version**: v2.0 (Revised with agent feedback)
**Last Updated**: 2025-11-07
**Authors**: Parker (PM), Aria (UX Architect), Felix (UX Feature Lead)

**Next Steps:**

1. **Stakeholder Review** (1 week):
   - Present revised outline to leadership
   - Approve 6-8 week pre-implementation UX work
   - Approve budget for user research and testing tools

2. **Pre-Implementation Phase** (6-8 weeks):
   - Execute user research (Section 2.4)
   - Create journey maps (Section 2.1)
   - Define design tokens (Section 2.2)
   - Complete component audit (Section 2.3)
   - Expand accessibility criteria (Section 2.5)
   - Create visual mockups and prototypes

3. **Specification Refinement** (2 weeks):
   - Incorporate research findings into spec
   - Create detailed component specifications
   - Define implementation architecture
   - Obtain final approvals

4. **Implementation Phase** (TBD based on component count):
   - Effort estimates after component audit complete
   - Phased development approach
   - Beta release → Phased rollout

**Approval Status:**
- [x] PM Review (Parker) - v2 complete
- [x] UX Architect Review (Aria) - Major revisions addressed
- [x] UX Feature Lead Review (Felix) - Component details added
- [ ] Technical Architect Review - Pending
- [ ] Stakeholder Approval - Pending

---

**This revised outline addresses all Priority 1 (MUST HAVE) recommendations from both agents. Implementation should be BLOCKED until pre-implementation requirements (Section 2) are completed.**
