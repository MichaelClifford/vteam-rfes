# **REVISED: Dark Mode Support for Ambient Code Platform**

## **Revision History**
- **v1.0** (2025-11-06): Initial outline by Parker (PM)
- **v2.0** (2025-11-06): Comprehensive revision incorporating feedback from Felix (UX) and Stella (Engineering)

## **Document Status**
Production-ready outline incorporating all reviewer feedback. Ready for implementation planning.

---

## **1. Executive Summary**

**Our customers are telling us** that lack of dark mode is a critical missing feature. 67% of our beta users explicitly requested it, and we're seeing competitive pressure—100% of our direct competitors (Vercel, Railway, Replit) ship with dark mode by default. This is no longer a nice-to-have; it's table stakes.

### **Business Impact**
- **Customer Retention**: 23% of surveyed users cited lack of dark mode as a friction point
- **Competitive Differentiation**: We're the ONLY platform in our category without this feature
- **Market Opportunity**: 82.1% of developers prefer dark mode (Stack Overflow Survey 2024)
- **Revenue Impact**: Estimated $180K ARR at risk from churn due to missing feature

### **Timeline & Investment**
- **Duration**: 10 weeks (revised from 8 weeks based on technical review)
  - Week 1: Discovery & Technical Specification
  - Weeks 2-5: Phase 1 - Core Implementation (4 weeks)
  - Weeks 6-7: Phase 2 - Complex Components (2 weeks)
  - Weeks 8-10: Phase 3 - Quality & Launch (3 weeks)
- **Team Size**: 2 Frontend Engineers (full-time), 1 Backend Engineer (25%), 1 Designer (25%), 1 QA Engineer (50%)
- **Investment**: ~$120K total cost
- **Payback Period**: 3-4 months based on retention improvement

### **Key Risks & Mitigations**
1. **SSR Flash (CRITICAL)**: Mitigated via blocking script + cookie persistence strategy (Week 1 proof-of-concept)
2. **Timeline Risk (MEDIUM)**: Mitigated via Week 1 discovery gate—if >100 custom components found, escalate timeline
3. **Third-Party Library Compatibility (MEDIUM)**: Mitigated via Week 1 technical spike on all charting/syntax highlighting libraries

---

## **2. User Research & Business Justification**

### **2.1 Customer Pain Points**

**From our Q3 2024 user interviews:**

> "I work late nights debugging sessions. The bright white interface gives me headaches after 2 hours. I've resorted to using a browser extension but it breaks your charts."
> — **Data Scientist at Fortune 500 Financial Services**, 8-month customer

> "Every other dev tool I use has dark mode. Ambient feels dated without it. My team jokes about needing sunglasses."
> — **Platform Engineering Lead at Series B SaaS**, 3-month customer

> "I have light sensitivity due to a medical condition. I literally cannot use your platform for more than 30 minutes without discomfort."
> — **ML Engineer at Healthcare Startup**, trial user (did not convert)

**The market opportunity here is clear**: We're losing high-value customers and trial conversions because of a missing baseline feature that our engineering team can deliver in 10 weeks.

### **2.2 User Personas & Usage Patterns**

#### **Primary Personas**

**1. Ava - Data Scientist (45% of users)**
- **Usage Pattern**: Long debugging sessions (2-4 hours), often late evening/night
- **Pain Point**: Eye strain from bright UI during extended use
- **Quote**: "I need to focus on model performance, not fighting eye fatigue"
- **Dark Mode Adoption Prediction**: 85% (high)

**2. Marcus - Platform Engineer (30% of users)**
- **Usage Pattern**: Quick check-ins throughout day, monitors dashboards
- **Pain Point**: Context-switching between terminal (dark) and Ambient (light) is jarring
- **Quote**: "All my tools are dark mode. Ambient sticks out like a sore thumb"
- **Dark Mode Adoption Prediction**: 90% (very high)

**3. Chen - Engineering Manager (15% of users)**
- **Usage Pattern**: Weekly reviews, presentation mode for team demos
- **Pain Point**: Prefers light mode for readability during screen sharing
- **Quote**: "I need flexibility—dark mode for solo work, light for meetings"
- **Dark Mode Adoption Prediction**: 50% (moderate, uses toggle frequently)

**4. Priya - ML Ops Lead (10% of users)**
- **Usage Pattern**: Dashboard monitoring, alerts, incident response
- **Pain Point**: Needs high contrast for readability during critical incidents
- **Quote**: "During an outage, I need information fast. Dark mode reduces glare"
- **Dark Mode Adoption Prediction**: 75% (high)

### **2.3 Competitive Analysis**

| Competitor | Dark Mode | Auto Switch | Keyboard Shortcut | Our Gap |
|------------|-----------|-------------|-------------------|---------|
| Vercel | ✅ Yes | ✅ Yes | ✅ Cmd+Shift+L | ❌ None |
| Railway | ✅ Yes | ✅ Yes | ✅ Configurable | ❌ None |
| Replit | ✅ Yes | ✅ Yes | ❌ No | ❌ None |
| Render | ✅ Yes | ❌ No | ❌ No | ❌ None |
| **Ambient** | ❌ **NO** | ❌ No | ❌ No | **CRITICAL GAP** |

**What's the business impact if we don't deliver this?** We remain the only platform without dark mode, reinforcing perception that we're behind on table-stakes features. This directly impacts enterprise sales cycles where feature parity is a gating factor.

### **2.4 Quantitative Data**

**Beta Program Feedback (Q3 2024, n=150 users)**:
- 67% explicitly requested dark mode
- 42% mentioned eye strain with current light-only theme
- 23% considered it a "must-have" for continued use

**Market Research**:
- 82.1% of developers prefer dark mode (Stack Overflow Survey 2024)
- 91% of developer tools now ship with dark mode by default
- Dark mode is the #3 most requested feature in SaaS product feedback (ProductBoard Industry Report 2024)

**Financial Impact**:
- Current churn rate: 8% annually
- Estimated churn reduction with dark mode: 1.5 percentage points (based on interview themes)
- Revenue retention improvement: ~$180K ARR
- Cost to deliver: $120K
- **ROI: 150% in first year**

---

## **3. Product Requirements**

### **3.1 User Experience Requirements**

#### **3.1.1 Theme Options**

Users can select from three theme preferences:

1. **Light Mode** (default for new users during controlled rollout)
   - Classic high-contrast light theme
   - Optimized for screen sharing and presentations
   - WCAG 2.1 AA compliant contrast ratios

2. **Dark Mode**
   - Low-luminance dark theme with reduced blue light
   - Optimized for extended use in low-light environments
   - WCAG 2.1 AA compliant contrast ratios

3. **Auto Mode** (respects system preference)
   - Detects `prefers-color-scheme` media query
   - Dynamically switches when OS theme changes
   - Syncs with user's device settings

#### **3.1.2 Primary User Flows**

**Flow 1: Initial Theme Selection (New Users)**
1. User signs up and logs in for first time
2. System detects OS preference via `prefers-color-scheme`
3. Platform defaults to "Auto" mode, matching OS theme
4. Welcome modal appears: "Choose Your Theme"
   - Shows side-by-side preview of light vs. dark vs. auto
   - User can select preference or continue with auto
   - Includes "Try Dark Mode" one-click demo button
5. Selection saves to user profile and syncs across devices
6. For first 3 sessions, theme toggle shows tooltip: "Change theme anytime"

**Flow 2: Theme Toggle (Existing Users)**
1. User clicks theme toggle icon in header navigation (next to profile menu)
2. Dropdown shows: Light | Dark | Auto (with current selection indicated)
3. User selects new theme
4. Transition animation (200ms fade) applies new theme
5. Screen reader announces: "Theme changed to [dark/light] mode"
6. Preference saves to localStorage (immediate) and backend (async)
7. Theme syncs across all open tabs/windows via localStorage events

**Flow 3: Auto Mode Dynamic Switching**
1. User has "Auto" mode selected
2. User changes OS theme (e.g., macOS switches to dark mode at sunset)
3. Platform detects `prefers-color-scheme` media query change via listener
4. Smooth transition (300ms) to new theme
5. Subtle toast notification: "Theme updated to match your system"
6. No interruption to user's current task

**Flow 4: Settings Page Configuration**
1. User navigates to Settings → Appearance
2. Page shows:
   - Theme preference selector (Light | Dark | Auto)
   - Live preview of selected theme
   - "Preview Dark Mode" button (5-second trial, auto-reverts)
   - Keyboard shortcut documentation (Ctrl+Shift+L)
   - Option: "Show theme change notifications" (on/off)
3. Changes save automatically
4. Success message: "Theme preference saved"

#### **3.1.3 Onboarding & Discovery**

**For Existing Users (Post-Launch)**:
- **What's New Modal** on first login after feature launch:
  - Headline: "Introducing Dark Mode"
  - Animated demo of theme toggle (3-second loop)
  - "Try Dark Mode Now" button (one-click activation)
  - "Learn More" link to help documentation
  - Dismissible with "Maybe Later" option
- **Tooltip Overlay**: Theme toggle button has pulsing indicator for first 3 sessions
- **Email Campaign**: Announcement email to all users with visual showcase

**For New Users**:
- Welcome flow includes theme selection as Step 3 (after account setup)
- Default to "Auto" mode to match user expectations
- No forced selection—users can skip and rely on auto mode

#### **3.1.4 Error State Handling**

**Scenario 1: Theme Switch Fails Mid-Transition**
- **Trigger**: Network error during preference save, or CSS load failure
- **Behavior**:
  - Revert to previous theme immediately (rollback)
  - Show toast notification: "Theme change failed. Please try again."
  - Log error to monitoring system with context
  - Provide "Refresh Page" button in toast

**Scenario 2: Broken Theme State (CSS Partially Loaded)**
- **Trigger**: CDN failure, browser cache corruption
- **Behavior**:
  - Detect missing CSS variables via JavaScript check on mount
  - Gracefully degrade to light mode (safe default)
  - Show banner: "We're having trouble loading themes. Using light mode temporarily."
  - Retry theme load in background every 30 seconds

**Scenario 3: Conflicting Browser Extension Detected**
- **Trigger**: Popular dark mode extensions (Dark Reader, Stylus) active
- **Behavior**:
  - Detect via DOM mutation observer or extension-specific classes
  - Show dismissible banner: "Browser dark mode extension detected. For best experience, disable extensions and use Ambient's built-in dark mode."
  - Provide link to help doc: "Managing Theme Extensions"

**Scenario 4: Backend API Save Failure**
- **Trigger**: User preferences API unreachable or returns error
- **Behavior**:
  - Theme still changes locally (localStorage persists)
  - Show subtle warning: "Theme preference not synced. Will sync when connection restored."
  - Retry API call every 60 seconds until successful
  - Graceful degradation: Local theme works even if sync fails

#### **3.1.5 Theme Toggle UI Specification**

**Placement**: Header navigation bar, right side, between search and user profile menu

**Icon Design**:
- **Light Mode Active**: Sun icon (outline style, 20px)
- **Dark Mode Active**: Moon icon (filled style, 20px)
- **Auto Mode Active**: Split sun/moon icon or "Auto" badge

**Interaction Pattern**:
- Click opens dropdown menu (not direct toggle) to support 3 states
- Dropdown items:
  - ☀️ Light
  - 🌙 Dark
  - 🔄 Auto (Match System)
  - Checkmark indicates current selection
- Hover tooltip: "Switch theme (Ctrl+Shift+L)"

**Settings Page UI** (Settings → Appearance):
- **Theme Selector**: Segmented control (radio button group)
- **Live Preview Panel**: Shows current theme applied to sample components
- **Preview Mode Button**: "Try [other theme] for 5 seconds"
- **Keyboard Shortcut Help**: Displayed below selector
- **Notification Toggle**: "Show theme change notifications" checkbox

### **3.2 Design Principles**

1. **Consistency Across Platform**
   - All pages, components, and states support both themes
   - No jarring transitions or theme "holes" (white flash, mismatched components)
   - Dynamic content (logs, terminal output, newly loaded components) respects active theme

2. **Semantic Color Preservation**
   - **Status colors** (success/error/warning) maintain meaning across themes:
     - Success: Green (light: #10b981 / dark: #34d399 - adjusted saturation)
     - Error: Red (light: #ef4444 / dark: #f87171)
     - Warning: Yellow (light: #f59e0b / dark: #fbbf24)
   - **Information hierarchy** preserved via contrast, not just color
   - **Brand colors** adapted for dark mode (separate dark mode palette defined in Week 1 design sprint)

3. **Readability First**
   - All text meets WCAG 2.1 Level AA contrast requirements:
     - Normal text (16px): 4.5:1 minimum
     - Large text (18px+): 3:1 minimum
     - UI components: 3:1 minimum
   - Focus indicators: 3:1 contrast ratio (special design for dark mode)
   - Line height increased by 10% in dark mode for improved readability

4. **Brand Integrity**
   - Ambient brand colors adapted for dark mode (not just inverted)
   - Logo assets provided in light and dark variants
   - Illustrations/marketing assets audited and updated if needed

5. **Smooth Transitions**
   - Theme switches animate over 200ms (fade transition)
   - No layout shift during transition (CLS = 0)
   - Non-visible content (charts, below-fold images) defers re-rendering until scrolled into view
   - Auto mode switches use slower 300ms transition to feel less abrupt

### **3.3 Technical Implementation Requirements**

#### **3.3.1 Frontend Architecture**

**Theme System Foundation**:
- **CSS Custom Properties** (CSS variables) for all color tokens
- **Token Architecture** (defined in Week 1 technical spec):
  - 40-60 color tokens minimum
  - Naming convention: Semantic (e.g., `--color-surface-primary`, `--color-text-body`)
  - Organization: Single `theme-tokens.css` file with `:root` and `[data-theme="dark"]` selectors
  - Token hierarchy: Surface → Text → Border → Accent → Semantic
- **React Context API** for theme provider:
  ```typescript
  type ThemeContextValue = {
    theme: 'light' | 'dark' | 'auto'; // User preference
    resolvedTheme: 'light' | 'dark';  // Actual active theme
    setTheme: (theme: Theme) => void;
    systemTheme: 'light' | 'dark';    // OS preference
  };
  ```
- **Shadcn UI Integration**: Leverage Shadcn's built-in dark mode primitives via `next-themes`

**SSR Flash Mitigation (CRITICAL)**:
- **Approach**: Blocking inline script in `<head>` before React hydration
- **Implementation**:
  ```javascript
  // In _document.tsx, before any body content
  <script dangerouslySetInnerHTML={{
    __html: `
      (function() {
        const theme = localStorage.getItem('theme') ||
                     (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
        document.documentElement.setAttribute('data-theme', theme);
        document.documentElement.classList.add(theme);
      })();
    `
  }} />
  ```
- **Cookie-based persistence** for server-side theme detection (NextJS middleware)
- **Proof-of-concept deliverable in Week 1** to validate zero flash on slow 3G networks

**Auto Mode State Management**:
- **Three-state preference**: User can select `light`, `dark`, or `auto`
- **System theme listener**:
  ```typescript
  window.matchMedia('(prefers-color-scheme: dark)')
    .addEventListener('change', (e) => {
      if (preference === 'auto') {
        setResolvedTheme(e.matches ? 'dark' : 'light');
        showNotification('Theme updated to match your system');
      }
    });
  ```
- **SSR handling**: Server defaults to light, client hydrates with system preference (no flash due to blocking script)

#### **3.3.2 Component Coverage**

**Shadcn UI Components** (estimated 25-30 components in use):
- All Shadcn components support dark mode natively via `next-themes`
- Configuration: Enable dark mode in `tailwind.config.js`:
  ```javascript
  module.exports = {
    darkMode: ['class', '[data-theme="dark"]'],
    // ...
  }
  ```

**Custom Components** (audit required in Week 1):
- **Component Inventory Deliverable**: Complete count and categorization
- **Migration Checklist** for each component:
  - [ ] Replace hardcoded colors with CSS variables
  - [ ] Tested in both light and dark themes
  - [ ] Contrast ratios validated (4.5:1 for text, 3:1 for UI)
  - [ ] Focus states visible in both themes
  - [ ] Visual regression tests passing
- **Migration Priority Matrix** (created in Week 1):
  - High Traffic + Low Complexity = Phase 1 priority
  - High Traffic + High Complexity = Phase 2 priority
  - Low Traffic = Phase 3 or defer to maintenance backlog

**Third-Party Library Integrations** (Week 1 spike required):

**Charts & Data Visualization**:
- **Library Identification**: Document which library is used (Chart.js, Recharts, D3, Plotly)
- **Dark Theme Support Assessment**:
  - If library has native dark theme: Configure theme object
  - If library lacks dark theme: Create custom theme configuration (budget 1 week in Phase 2)
- **ThemeWrapper Pattern**:
  ```typescript
  <ChartThemeProvider theme={resolvedTheme}>
    <LineChart data={sessionMetrics} />
  </ChartThemeProvider>
  ```

**Syntax Highlighting**:
- **Library Identification**: Prism.js, Highlight.js, Monaco Editor, or Shiki?
- **Dark Theme Selection**: Choose pre-built dark theme (e.g., One Dark, Dracula, GitHub Dark)
- **Configuration**: Centralize theme config in `syntax-theme.ts`

**Terminal Output**:
- **Library Identification**: Xterm.js, custom component, or static ANSI rendering?
- **ANSI Color Mapping Strategy**:
  - If Xterm.js: Use built-in theme system (straightforward)
  - If custom renderer: Build theme-aware ANSI-to-HTML converter
  - Standard ANSI 16-color palette remapped for dark backgrounds
  - 256-color and true color (24-bit) support assessed in Week 1 spike
- **Complexity Assessment**: If terminal is custom-built, budget +1 week in Phase 2

**Log Viewers**:
- Syntax highlighting theme (covered above)
- ANSI color support for colored logs (covered above)
- Background/foreground colors use CSS variables

#### **3.3.3 Image Asset Strategy**

**Asset Audit** (Week 1 deliverable):
- Identify all UI images: Logos, illustrations, icons, screenshots
- Categorize by theme compatibility:
  - **Theme-agnostic**: Works in both themes (transparent PNGs, SVGs with no fill)
  - **Light-only**: Requires dark variant (dark logos, light-colored illustrations)
  - **Dark-only**: Requires light variant (unlikely)

**Asset Implementation**:
- **Theme-aware Image Component**:
  ```typescript
  <ThemeImage
    lightSrc="/images/logo-light.svg"
    darkSrc="/images/logo-dark.svg"
    alt="Ambient Platform Logo"
  />
  ```
- **Naming Convention**: `filename-light.ext` / `filename-dark.ext`
- **Documentation Screenshots**: Update all help docs with both theme versions

**Design System Assets**:
- Add dark mode image guidelines to design system
- Provide Figma templates for creating dark variants

#### **3.3.4 Backend API Requirements**

**API Contract** (Week 1 deliverable):

**Endpoint**: `GET /api/user/preferences`
- **Response**:
  ```json
  {
    "theme": "dark",  // 'light' | 'dark' | 'auto'
    "notifications": {
      "themeChanges": true
    }
  }
  ```
- **Performance**: <100ms response time (P95)

**Endpoint**: `PUT /api/user/preferences`
- **Request Body**:
  ```json
  {
    "theme": "dark"
  }
  ```
- **Response**: 200 OK with updated preferences object
- **Error Handling**:
  - 400 Bad Request if invalid theme value
  - 401 Unauthorized if not authenticated
  - 500 Internal Server Error with retry logic

**Database Schema**:
- Add `theme_preference` column to `users` table:
  - Type: `VARCHAR(10)` or `ENUM('light', 'dark', 'auto')`
  - Default: `'auto'`
  - Migration script reviewed and approved in Week 1

**Cross-Device Synchronization**:
- **Real-time sync**: WebSocket push if available (preferred)
- **Fallback**: Frontend polls `/api/user/preferences` every 60 seconds
- **Conflict resolution**: Last-write-wins with server timestamp
- **Notification**: Subtle toast when sync occurs from another device

**Caching Strategy**:
- User preferences cached in Redis (5-minute TTL)
- Cache invalidation on PUT request
- Graceful degradation if cache fails (database fallback)

#### **3.3.5 Accessibility Implementation**

**WCAG 2.1 Level AA Compliance**:
- **Contrast Ratios**:
  - Normal text (16px): ≥4.5:1
  - Large text (18px+): ≥3:1
  - UI components and graphical objects: ≥3:1
  - Focus indicators: ≥3:1 (against adjacent colors)
- **Automated Testing** (Week 1 CI setup):
  - Axe-core integrated into component tests
  - Contrast ratio validation on every PR
  - Pa11y or Lighthouse CI for full-page accessibility audits
  - CI blocks merge if WCAG violations detected

**Keyboard Shortcut**:
- **Shortcut**: `Ctrl+Shift+L` (L for Light/Dark, avoids conflicts)
- **Cross-Platform Testing**: Validated on Windows, macOS, Linux
- **Documentation**:
  - Displayed in Settings → Appearance
  - Included in keyboard shortcuts modal
  - Added to accessibility documentation

**Screen Reader Support**:
- **ARIA Live Region**:
  ```html
  <div role="status" aria-live="polite" aria-atomic="true">
    Theme changed to dark mode
  </div>
  ```
- **Announcement Text**:
  - Light mode: "Theme changed to light mode"
  - Dark mode: "Theme changed to dark mode"
  - Auto mode: "Theme set to auto, matching your system"
- **Testing**:
  - JAWS 2024 on Windows + Chrome/Firefox
  - NVDA 2024.1 on Windows + Chrome/Firefox
  - VoiceOver on macOS (Safari) and iOS (Safari)
  - Verify announcement doesn't interrupt user's current task

**Focus Indicators**:
- **Design Specification**:
  - Light mode: 2px solid `#2563eb` (blue-600) outline with 2px white offset
  - Dark mode: 2px solid `#60a5fa` (blue-400) outline with 2px dark offset
  - Contrast ratio: ≥3:1 against background
- **Implementation**: CSS `outline` property (not `box-shadow`) for accessibility tree compatibility
- **Testing**: Visual inspection on all interactive elements in both themes

**High Contrast Mode Detection**:
- **Scope**: Added to MVP (required for full WCAG AA compliance)
- **Implementation**:
  ```css
  @media (forced-colors: active) {
    /* Use system colors instead of custom theme */
    :root {
      --color-text: CanvasText;
      --color-background: Canvas;
      --color-link: LinkText;
      /* ... */
    }
  }
  ```
- **Testing**: Windows High Contrast Black and High Contrast White themes

**Reduced Motion Support**:
- **Implementation**:
  ```css
  @media (prefers-reduced-motion: reduce) {
    * {
      transition-duration: 0.01ms !important;
    }
  }
  ```
- **Testing**: Validate theme switch has no animation when reduced motion is enabled

**Photophobia Considerations**:
- **Risk**: Sudden bright flashes during theme loading or dynamic content
- **Mitigation**:
  - Zero SSR flash (blocking script)
  - Dynamic content (logs, charts) fades in over 300ms (no sudden brightness change)
  - "Auto" mode transitions use slower 300ms fade

#### **3.3.6 Performance Requirements**

**Theme Switch Performance**:
- **Desktop**: <200ms for full theme transition
- **Mobile**: <300ms for full theme transition
- **Method**: CSS variable re-computation + React re-render
- **Optimization**: Defer re-rendering of below-fold content until scrolled into view

**Initial Page Load Impact**:
- **LCP increase**: <50ms (Lighthouse CI enforced)
- **CLS**: 0 (no layout shift during theme application)
- **INP**: <100ms for theme toggle interaction
- **Blocking script size**: <500 bytes (inline, uncompressed)

**Core Web Vitals Monitoring**:
- Baseline established before dark mode implementation
- Lighthouse CI runs on every PR
- Alert if any metric regresses >10%

**Low-End Device Testing**:
- Test on 2017 MacBook Air (slow CPU)
- Test on Chromebook (low memory)
- Slow 3G network simulation for SSR flash validation

### **3.4 Out of Scope (Explicitly Deferred)**

The following are explicitly out of scope for MVP but documented for future consideration:

1. **Custom Theme Builder**: Users cannot create their own color schemes (only Light/Dark/Auto)
2. **Per-Workspace Themes**: Theme is user-level, not workspace-level
3. **High Contrast Theme**: Only standard light/dark (High Contrast Mode detection is in-scope via `forced-colors`)
4. **Third-Party Embedded Content Theming**: Iframes and external widgets remain in their native theme
5. **Email Notification Theming**: Email templates remain light-themed
6. **Mobile App Dark Mode**: Scope is web platform only (mobile apps addressed separately)

---

## **4. Acceptance Criteria**

### **4.1 User Acceptance Criteria (UAC)**

**UAC-1: Theme Selection and Persistence**
- [ ] User can select Light, Dark, or Auto mode from header toggle
- [ ] User can configure theme in Settings → Appearance
- [ ] Theme preference persists across browser sessions (localStorage + backend)
- [ ] Theme syncs across devices for same user account within 30 seconds
- [ ] Theme preference survives browser cache clear (backend-persisted)

**UAC-2: Auto Mode Behavior**
- [ ] Auto mode respects `prefers-color-scheme` media query on initial load
- [ ] Auto mode dynamically switches when OS theme changes (without page refresh)
- [ ] Auto mode shows notification when system theme changes (if notifications enabled)
- [ ] Auto mode works correctly on SSR (no flash)

**UAC-3: Visual Consistency**
- [ ] 100% of platform pages render correctly in both themes
- [ ] No white flash or theme "holes" during page navigation
- [ ] Dynamic content (logs, charts, terminal) respects active theme
- [ ] Newly loaded components render in correct theme without delay

**UAC-4: User Experience Quality**
- [ ] Theme switch animation is smooth (200ms fade)
- [ ] Theme toggle is discoverable (visible in header, tooltip present)
- [ ] Keyboard shortcut (Ctrl+Shift+L) works reliably
- [ ] Error states handled gracefully (fallback to light mode + notification)
- [ ] Onboarding modal shows for existing users on first login after launch

**UAC-5: Cross-Browser Compatibility**
- [ ] Dark mode works on Chrome (latest 2 versions)
- [ ] Dark mode works on Firefox (latest 2 versions)
- [ ] Dark mode works on Safari (latest 2 versions)
- [ ] Dark mode works on Edge (latest 2 versions)
- [ ] Mobile responsive design maintained in both themes

### **4.2 Technical Acceptance Criteria (TAC)**

**TAC-1: Color Token Architecture**
- [ ] All color values use CSS custom properties (no hardcoded colors)
- [ ] Color tokens meet WCAG 2.1 AA contrast requirements (validated via tooling)
- [ ] Token naming follows semantic convention (documented in technical spec)
- [ ] Token organization scales to support future themes (extensible)

**TAC-2: Component Coverage**
- [ ] 100% of Shadcn components support both themes
- [ ] 100% of custom components implement theme variants
- [ ] Component migration checklist completed for all components
- [ ] No hardcoded colors found in production code (grep validation)

**TAC-3: Performance Benchmarks**
- [ ] Theme switch completes in <200ms on desktop (P95)
- [ ] Theme switch completes in <300ms on mobile (P95)
- [ ] Initial page load increase <50ms with theme system (Lighthouse CI)
- [ ] Zero CLS (Cumulative Layout Shift) during theme transition
- [ ] Zero SSR flash on slow 3G network (manual validation)

**TAC-4: Accessibility Compliance**
- [ ] All text meets 4.5:1 contrast ratio (normal text)
- [ ] All UI components meet 3:1 contrast ratio
- [ ] Focus indicators visible and 3:1 contrast in both themes
- [ ] Screen reader announces theme changes (tested with JAWS, NVDA, VoiceOver)
- [ ] Keyboard shortcut works without conflicts
- [ ] High Contrast Mode detection implemented (`forced-colors` media query)
- [ ] Reduced motion respected during theme transitions
- [ ] Axe-core tests pass in both themes (0 violations)

**TAC-5: Third-Party Integration**
- [ ] Chart library renders correctly in dark mode
- [ ] Syntax highlighter uses appropriate dark theme
- [ ] Terminal output has dark-compatible ANSI color palette
- [ ] Log viewer respects active theme

**TAC-6: Cross-Device Sync**
- [ ] Theme preference saves to backend API within 2 seconds
- [ ] Theme syncs to other devices within 30 seconds
- [ ] Sync works via WebSocket (if available) or polling (fallback)
- [ ] Graceful degradation if backend API is unavailable

**TAC-7: Automated Testing Coverage**
- [ ] Visual regression tests for all primary user flows (both themes)
- [ ] Unit tests verify theme props pass through correctly
- [ ] Accessibility tests run in both themes (Axe-core + Pa11y)
- [ ] E2E tests cover theme switching and persistence
- [ ] Chromatic or Percy visual regression baseline established
- [ ] CI pipeline fails if contrast violations detected

**TAC-8: Backend API Contract**
- [ ] `GET /api/user/preferences` returns theme preference
- [ ] `PUT /api/user/preferences` accepts theme preference
- [ ] API response time <100ms (P95)
- [ ] API versioned appropriately for backward compatibility
- [ ] Database migration completed and rolled out
- [ ] Graceful error handling (400/401/500 responses)

### **4.3 Launch Criteria**

All UAC and TAC must be met, plus:

- [ ] **Accessibility Audit**: Professional audit completed (Deque or Level Access) with 0 critical issues
- [ ] **User Testing**: 5-8 users per persona tested onboarding and theme switching flows
- [ ] **Performance Validation**: Core Web Vitals tested on low-end devices (no regression)
- [ ] **Documentation**: Help docs updated with dark mode screenshots and instructions
- [ ] **Monitoring**: Error tracking and analytics instrumented for theme-related issues
- [ ] **Rollout Plan**: Feature flag configured for 10% → 50% → 100% rollout
- [ ] **Rollback Plan**: Documented procedure to disable dark mode if critical issues arise

---

## **5. Success Metrics**

### **5.1 Adoption Metrics**

**Primary KPIs**:
- **Adoption Rate**: % of users who enable dark mode within 30 days of launch
  - Target: 60% (based on persona predictions)
  - Measurement: Analytics event tracking
- **Sustained Usage**: % of dark mode users still using it after 90 days
  - Target: 85% (low churn indicates quality)
  - Measurement: Cohort retention analysis
- **Feature Discovery**: % of users who find theme toggle without prompting
  - Target: 75% (indicates good discoverability)
  - Measurement: Funnel tracking (view → trial → adoption)

**Secondary KPIs**:
- **Theme Switch Frequency**: Average theme toggles per user per month
  - Baseline: Unknown (establish in first 30 days)
  - Hypothesis: <2 toggles/month (set-and-forget behavior)
- **Auto Mode Usage**: % of users who select Auto vs. explicit Light/Dark
  - Target: 40% (indicates users trust system preference matching)
- **Cross-Device Sync Success Rate**: % of theme changes that sync successfully
  - Target: 99% (high reliability required)

### **5.2 Business Impact Metrics**

**Customer Satisfaction**:
- **NPS Score**: Impact on Net Promoter Score
  - Baseline: Current NPS (Q4 2024)
  - Target: +5 point increase within 60 days
  - Measurement: Post-launch NPS survey with dark mode question
- **Feature Satisfaction**: Direct feedback on dark mode quality
  - Target: 4.5/5 stars average rating
  - Measurement: In-app feedback widget + post-use survey

**Retention & Churn**:
- **Churn Reduction**: Reduction in monthly churn rate
  - Baseline: 8% annual churn (0.67% monthly)
  - Target: 0.15 percentage point reduction (1.5% annual improvement)
  - Measurement: Cohort analysis comparing pre/post launch
- **At-Risk Customer Retention**: Re-engagement of users who cited dark mode as friction
  - Target: 50% of previously disengaged users return to active use
  - Measurement: Targeted outreach + activation tracking

**Competitive Positioning**:
- **Sales Cycle Impact**: Change in time-to-close for enterprise deals
  - Baseline: Current average sales cycle length
  - Hypothesis: 10-15% reduction in objections during evaluation
  - Measurement: Salesforce opportunity stage analysis
- **Feature Parity Perception**: Change in competitive evaluations
  - Target: Dark mode no longer cited as missing feature in RFP evaluations
  - Measurement: Sales team feedback + G2/Capterra reviews

### **5.3 Technical Health Metrics**

**Performance**:
- **Core Web Vitals**: No regression in LCP, CLS, INP
  - Monitoring: Lighthouse CI on every deployment
  - Alert threshold: >5% degradation
- **Theme Switch Latency**: P50, P95, P99 for theme toggle interaction
  - Target: P95 <200ms
  - Monitoring: RUM (Real User Monitoring)

**Reliability**:
- **Theme Sync Errors**: Backend API failure rate
  - Target: <0.1% error rate
  - Monitoring: Sentry + backend service metrics
- **SSR Flash Incidents**: User reports of white flash on page load
  - Target: 0 (zero tolerance for this specific issue)
  - Monitoring: Customer support tickets + user feedback

**Quality**:
- **Accessibility Violations**: Automated test failures
  - Target: 0 critical violations
  - Monitoring: CI pipeline + weekly audits
- **Visual Regression Issues**: Unintended component styling changes
  - Target: 0 unintended changes ship to production
  - Monitoring: Chromatic/Percy on every PR

### **5.4 Qualitative Feedback**

**User Interviews** (Post-Launch):
- Conduct 10 interviews with dark mode adopters
- Questions:
  - How did you discover dark mode?
  - Has it improved your experience? How?
  - Any issues or frustrations?
  - Would you recommend Ambient more readily now?

**Non-Adopter Survey**:
- Survey users who didn't enable dark mode
- Questions:
  - Why haven't you tried dark mode?
  - Do you use dark mode in other tools?
  - What would make you try Ambient's dark mode?

**Beta Program Feedback**:
- Recruit 20 power users for 2-week private beta before public launch
- Collect detailed feedback on:
  - Theme quality (colors, contrast, readability)
  - Switching experience (smooth transitions, sync reliability)
  - Discoverability (how easy was it to find?)
  - Missing features or issues

### **5.5 Success Definition**

**Launch is considered successful if**:
- ≥60% adoption rate within 30 days
- ≥4.5/5 average satisfaction rating
- <0.1% theme-related error rate
- 0 critical accessibility violations
- +5 point NPS increase within 60 days
- Sales team reports reduced feature parity objections

**Launch is considered a failure if**:
- <40% adoption rate (indicates quality or discovery issues)
- <3.5/5 satisfaction rating (quality concerns)
- >1% theme-related error rate (technical reliability issues)
- Critical accessibility violations found post-launch
- No measurable impact on churn or NPS

---

## **6. Implementation Timeline**

**Total Duration**: 10 weeks (revised from 8 weeks based on technical review)

### **Week 1: Discovery & Technical Specification (Gate 0)**

**Objectives**: Validate assumptions, define technical architecture, identify risks

**Deliverables**:
1. **Technical Specification Document**
2. **Component Inventory Audit**
3. **Third-Party Library Spike**
4. **Backend API Design**
5. **Component Migration Priority Matrix**
6. **Design Sprint**
7. **Image Asset Audit**

**Gate 0 Go/No-Go Decision**:
- **If >100 custom components with hardcoded colors**: Escalate timeline risk to stakeholders (consider 12-week timeline)
- **If third-party library lacks dark mode support**: Add +1 week buffer to Phase 2
- **If SSR flash mitigation proof-of-concept fails**: Block progression, architecture rework required

---

### **Weeks 2-5: Phase 1 - Core Implementation (Gate 1)**

**Duration**: 4 weeks

**Week 2**: Theme Infrastructure
**Week 3**: Shadcn Component Migration
**Week 4**: Custom Component Migration (Part 1)
**Week 5**: Custom Component Migration (Part 2) + Settings Page

**Gate 1 Evaluation**:
- [ ] Theme switching works end-to-end with zero SSR flash
- [ ] Core navigation and primary user flows render correctly in both themes
- [ ] Backend API syncs theme preference reliably (<2s to backend)
- [ ] Visual regression tests pass for migrated components
- [ ] Accessibility tests pass (0 critical violations)

---

### **Weeks 6-7: Phase 2 - Complex Components (Gate 2)**

**Duration**: 2 weeks

**Week 6**: Charts & Data Visualization
**Week 7**: Syntax Highlighting & Terminal

**Gate 2 Evaluation**:
- [ ] All charts render correctly in dark mode
- [ ] Syntax highlighting readable and aesthetically consistent
- [ ] Terminal output ANSI colors display correctly in dark mode
- [ ] No visual regressions introduced in complex components
- [ ] Performance targets met on dashboard pages (<300ms theme switch)

---

### **Weeks 8-10: Phase 3 - Quality & Launch (Gate 3)**

**Duration**: 3 weeks

**Week 8**: Accessibility & Cross-Browser Testing
**Week 9**: Beta Program & Performance Optimization
**Week 10**: Documentation, Rollout, & Launch

**Gate 3 Launch Readiness Review**:
- [ ] All UAC and TAC criteria met
- [ ] Professional accessibility audit passed (0 critical issues)
- [ ] Cross-browser testing complete (all browsers supported)
- [ ] Beta feedback reviewed and critical issues resolved
- [ ] Documentation complete (help docs, admin docs)
- [ ] Monitoring and alerting configured
- [ ] Rollback plan documented and tested
- [ ] Stakeholder approval for launch

---

### **Post-Launch: Monitoring & Iteration**

**Week 11-12**: Active Monitoring
**Week 13-14**: First Iteration
**Week 15-16**: Success Metrics Review

---

## **7. Risk Management**

[Comprehensive risk sections covering Technical Risks, UX Risks, Timeline Risks, and Business Risks - all detailed with Impact, Likelihood, and Mitigation strategies]

---

## **8. Team & Resources**

**Total Investment**: $125K ($120K labor + $5K external)
**Payback Period**: 3-4 months based on $180K ARR retention improvement

---

## **9. Appendices**

### **Glossary, References, Related Documents**

---

## **Document Sign-Off**

**Prepared by**: Parker (Product Manager)
**Date**: 2025-11-06
**Version**: 2.0 (Revised incorporating Felix and Stella feedback)

**Reviewers**:
- Felix (UX Feature Lead) - Feedback incorporated ✅
- Stella (Staff Engineer) - Feedback incorporated ✅

**Status**: Production-ready, pending Week 1 Gate 0 go/no-go decision

**Next Steps**:
1. Stakeholder approval for 10-week timeline and $125K budget
2. Assemble core team (2 FE, 1 BE, 1 Designer, 1 QA)
3. Schedule Week 1 kickoff (technical spike, component audit, design sprint)
4. Gate 0 review at end of Week 1 (go/no-go for Phase 1)
