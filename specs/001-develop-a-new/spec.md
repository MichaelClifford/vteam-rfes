# Feature Specification: Restore Jira Integration Using Atlassian MCP Server

**Feature Branch**: `001-develop-a-new`
**Created**: 2025-10-15
**Status**: Draft
**Input**: User description: "Develop a new feature based on rfe.md"

## Execution Flow (main)
```
1. Parse RFE requirements from rfe.md
   → Feature: Restore Jira integration using Atlassian MCP Server
2. Extract key concepts from description
   → Actors: Product Managers, Administrators, Engineers
   → Actions: Configure OAuth, publish files to Jira, track links
   → Data: OAuth tokens, Jira links, workspace files
   → Constraints: OAuth complexity, rate limits, Python-Go integration
3. Ambiguities identified and marked:
   → MCP client language bridge (CRITICAL)
   → OAuth configuration model (per-project vs global)
   → Token lifecycle management strategy
   → Rate limiting user experience
   → Data model for jiraLinks field
4. User scenarios defined (see section below)
5. Functional requirements generated from RFE + agent analysis
6. Key entities identified (JiraLink, OAuth configuration)
7. Review checklist: NEEDS CLARIFICATION markers present
```

---

## ⚡ Quick Guidelines
- ✅ Focus on WHAT users need and WHY
- ❌ Avoid HOW to implement (no tech stack, APIs, code structure)
- 👥 Written for business stakeholders, not developers

### Section Requirements
- **Mandatory sections**: Must be completed for every feature
- **Optional sections**: Include only when relevant to the feature
- When a section doesn't apply, remove it entirely (don't leave as "N/A")

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story

**Context**: vTeam previously had custom Jira integration (API token-based) that allowed users to publish RFE workflow artifacts to Jira. This feature migrates to the more secure and maintainable Atlassian MCP Server with OAuth 2.0 authentication.

**Personas**:
- **Product Manager (Primary)**: Creates RFE workflows, publishes specifications to Jira for tracking
- **Administrator (Secondary)**: Configures integration settings, monitors health
- **Engineer (Tertiary)**: Views Jira links from published specs

**Main Success Scenario**: Product Manager publishes an RFE specification to Jira

1. Product Manager creates a new RFE workflow titled "Dark Mode Support"
2. AI agent generates a specification document at `spec/requirements.md` in the workspace
3. Product Manager navigates to Project Settings → Jira Integration
4. System displays connection status: "Not Connected"
5. Product Manager clicks "Connect to Jira" and follows OAuth setup flow:
   - Enters Jira instance URL (e.g., mycompany.atlassian.net)
   - Enters Jira project key (e.g., PROJ)
   - Clicks "Authorize" → OAuth popup opens
   - Completes Atlassian OAuth consent flow in popup window
   - Popup closes automatically, connection status updates to "Connected"
6. Product Manager selects default issue type (Story) and project
7. Product Manager tests connection successfully
8. Product Manager navigates to RFE workflow detail page
9. Product Manager clicks "Publish to Jira" button next to `spec/requirements.md`
10. System shows loading state: "Publishing to Jira..."
11. System creates Jira issue with summary "[RFE] Dark Mode Support - Specification"
12. System receives issue key "PROJ-123" from Atlassian MCP Server
13. System updates RFEWorkflow CR with jiraLinks entry: `{filePath: "spec/requirements.md", jiraKey: "PROJ-123", createdAt: "2025-10-15T10:30:00Z", issueUrl: "https://mycompany.atlassian.net/browse/PROJ-123"}`
14. UI displays success toast: "✓ Published to Jira: PROJ-123"
15. UI shows clickable Jira badge next to file: `[PROJ-123 ↗]`
16. Product Manager clicks the badge → Jira issue opens in new tab
17. Product Manager verifies issue contains spec content in description

### Acceptance Scenarios

1. **Given** Jira integration is not configured, **When** user clicks "Publish to Jira" on any workspace file, **Then** system displays error message "Jira integration not configured. Please configure in Project Settings" with actionable link to settings

2. **Given** Jira integration is configured and OAuth token is valid, **When** user clicks "Publish to Jira" on an unpublished file, **Then** system creates a new Jira issue, displays the issue key as a clickable link, and shows success confirmation

3. **Given** a file is already published to Jira (has jiraLinks entry), **When** user clicks "Publish to Jira" on that file again, **Then** system [NEEDS CLARIFICATION: should system create new issue, update existing issue, or show warning?]

4. **Given** OAuth token has expired, **When** user attempts to publish to Jira, **Then** system [NEEDS CLARIFICATION: silently refreshes token and continues, or prompts user to re-authenticate?]

5. **Given** Atlassian rate limit (1,000 req/hour) is reached, **When** user attempts to publish, **Then** system [NEEDS CLARIFICATION: queues request, fails immediately with retry time, or warns user before publishing?]

6. **Given** user has Jira view-only permissions, **When** user attempts to publish, **Then** system displays error "Insufficient Jira permissions to create issues in [PROJECT]. Contact your Jira administrator or choose a different project"

7. **Given** MCP server is unreachable, **When** user attempts to publish, **Then** system displays error "Jira integration service temporarily unavailable. Try again in a few minutes" with retry button

8. **Given** multiple workspace files are unpublished, **When** user selects files and clicks publish, **Then** system [NEEDS CLARIFICATION: publishes one at a time sequentially, or publishes in parallel with per-file status tracking?]

### Edge Cases

- **What happens when** a user closes the OAuth popup manually before completing authorization?
  - System should detect timeout and show message: "OAuth authorization was cancelled. Try again when ready."

- **What happens when** a Jira issue is deleted externally but vTeam still has the jiraLink?
  - [NEEDS CLARIFICATION: Should system validate links periodically, show broken link indicator, or leave stale?]

- **What happens when** a workspace file is renamed but has an existing Jira link?
  - [NEEDS CLARIFICATION: Should jiraLinks track by file path or content hash? What happens to links on rename/move?]

- **What happens when** multiple users simultaneously publish the same file?
  - [NEEDS CLARIFICATION: Should system detect conflicts, allow duplicate issues, or use locking?]

- **What happens when** OAuth token refresh fails (user revoked permissions in Atlassian)?
  - System should detect and prompt: "Jira access was revoked. Reconnect to continue publishing" with re-auth button

- **What happens when** a workspace file exceeds Jira's description field limit (32,767 characters)?
  - System should warn: "File exceeds Jira description limit. Consider splitting into multiple issues or summarizing."

- **How does system handle** Jira Cloud outages (Atlassian service down)?
  - System should detect infrastructure errors and show: "Atlassian services are experiencing issues. Check status at status.atlassian.com"

---

## Requirements *(mandatory)*

### Functional Requirements

#### Core Publishing Workflow

- **FR-001**: System MUST allow users to publish individual workspace files from RFE workflows to Jira Cloud as new issues via Atlassian MCP Server
- **FR-002**: System MUST create Jira issues with summary "[RFE] {workflow-title} - {file-purpose}", description containing file content, and assignment to the configured Jira project
- **FR-003**: System MUST display Jira issue keys as clickable external links next to published workspace files
- **FR-004**: System MUST open Jira issue links in a new browser tab with proper security attributes (rel="noopener noreferrer")
- **FR-005**: System MUST track published Jira issues in the RFEWorkflow Custom Resource jiraLinks field with file path, issue key, creation timestamp, and issue URL

#### OAuth Authentication & Configuration

- **FR-006**: System MUST authenticate to Atlassian Cloud using OAuth 2.0 (replacing previous API token method)
- **FR-007**: System MUST provide OAuth configuration interface in Project Settings allowing users to enter Jira instance URL, OAuth client credentials, and Jira project key
- **FR-008**: System MUST initiate OAuth consent flow in a popup window (with fallback to full redirect if popup blocked) when users click "Authorize"
- **FR-009**: System MUST display connection status (Connected/Disconnected/Expired) in Project Settings and RFE workflow pages
- **FR-010**: System MUST provide "Test Connection" capability in settings that validates OAuth credentials, MCP server connectivity, and Jira project accessibility
- **FR-011**: System MUST store OAuth tokens [NEEDS CLARIFICATION: per-project in individual secrets or globally in a shared secret?]
- **FR-012**: System MUST handle OAuth token expiration by [NEEDS CLARIFICATION: automatic silent refresh, or prompting user to re-authenticate?]
- **FR-013**: System MUST handle OAuth token revocation (user removes permissions in Atlassian) by displaying actionable error message prompting re-authentication

#### Error Handling & User Feedback

- **FR-014**: System MUST display error message "Jira integration not configured. Please configure in Project Settings" with actionable link when attempting to publish without configuration
- **FR-015**: System MUST display error message "Your Jira connection expired. Reconnect to continue publishing" when OAuth token cannot be refreshed automatically
- **FR-016**: System MUST display error message "Unable to connect to Jira integration service. Please try again later" when MCP server is unreachable
- **FR-017**: System MUST display error message "You don't have permission to create issues in [PROJECT]. Contact your Jira admin or choose a different project" when user lacks Jira permissions
- **FR-018**: System MUST display error message with estimated retry time when Atlassian rate limits (1,000 req/hour for Premium) are reached
- **FR-019**: System MUST provide clear loading indicators during publishing with status messages: "Connecting to Jira...", "Creating Jira issue...", "Saving link..."
- **FR-020**: System MUST display success confirmation (toast notification) showing created issue key after successful publishing

#### Rate Limiting & Reliability

- **FR-021**: System MUST handle Atlassian MCP Server rate limits (1,000 requests/hour for Premium, moderate for Standard) by [NEEDS CLARIFICATION: queuing requests, failing fast with clear error, or warning users proactively?]
- **FR-022**: System MUST implement error handling for MCP connection failures, authentication issues, and Jira operation errors with user-actionable messages
- **FR-023**: System MUST provide health check endpoints exposing MCP server connectivity status [NEEDS CLARIFICATION: visible to end users, project admins, or platform admins only?]

#### Configuration & Settings

- **FR-024**: System MUST allow [NEEDS CLARIFICATION: per-project or global?] configuration of Atlassian OAuth credentials (client ID, client secret)
- **FR-025**: System MUST allow users to select default Jira project and issue type (Story, Task, Epic, Bug) in settings
- **FR-026**: System MUST validate Jira project key accessibility during configuration and display available projects based on user's OAuth permissions
- **FR-027**: System MUST preserve existing UI elements for Jira configuration in project settings and "Publish to Jira" functionality in RFE workflow pages

#### Data & State Management

- **FR-028**: System MUST persist jiraLinks entries in RFEWorkflow Custom Resource with schema: `{filePath: string, jiraKey: string, createdAt: timestamp, issueUrl: string}`
- **FR-029**: System MUST handle re-publishing of already-linked files by [NEEDS CLARIFICATION: creating new issue, updating existing issue, or prompting user to choose?]
- **FR-030**: System MUST handle workspace file rename/move operations by [NEEDS CLARIFICATION: tracking by path (breaks on rename) or by content hash (maintains link)?]

### Technical Architecture Decisions (BINDING)

[NEEDS CLARIFICATION: These architectural decisions must be made binding in the spec to prevent implementation inconsistencies]

- **MCP Client Integration**: System MUST integrate MCP client using [NEEDS CLARIFICATION: Go native client, Python client via HTTP/gRPC sidecar, or Python subprocess wrapper?]
  - **Critical**: The RFE references `/archive/mcp_client_integration` (Python) but backend is Go. This impedance mismatch must be resolved in the spec.
  - **Validation Required**: Confirm the MCP client library exists, is production-ready, and has been tested for this use case.

- **OAuth Token Storage Model**: System MUST store OAuth tokens [NEEDS CLARIFICATION: per-project for isolation, or globally for simplicity?]
  - **Per-project**: Higher security isolation, more complex configuration UX, separate tokens per project
  - **Global**: Simpler UX, single point of failure, shared token across projects

- **MCP Server Hosting**: System MUST connect to [NEEDS CLARIFICATION: Atlassian-hosted MCP Server only, or support self-hosted deployment?]
  - **Hosted**: Simpler deployment, requires trust in Atlassian infrastructure, data flows through Atlassian
  - **Self-hosted**: More complex deployment, full control, data sovereignty

- **Rate Limit Strategy**: System MUST handle rate limits by [NEEDS CLARIFICATION: fail-fast with retry timing, queuing with estimated completion, or client-side throttling?]
  - **User Impact**: Affects whether publishing "just works" or requires retry/queue management

### Key Entities *(included - feature involves data)*

**JiraLink** (stored in RFEWorkflow Custom Resource)
- **filePath**: Workspace file path relative to workspace root (e.g., "spec/requirements.md")
- **jiraKey**: Jira issue key in format PROJECT-NUMBER (e.g., "PROJ-123")
- **createdAt**: ISO 8601 timestamp when link was created
- **issueUrl**: Full HTTPS URL to Jira issue for direct navigation

**Relationships**:
- One workspace file can have [NEEDS CLARIFICATION: one or multiple?] Jira links
- JiraLinks are scoped to a specific RFEWorkflow (not shared across workflows)
- JiraLinks persist even if the Jira issue is deleted externally [NEEDS CLARIFICATION: should system validate periodically?]

**OAuth Configuration** (stored in Kubernetes Secret)
- **jiraInstanceUrl**: Base URL of Atlassian Cloud instance (e.g., "https://mycompany.atlassian.net")
- **oauthClientId**: OAuth 2.0 client ID from Atlassian Developer Console
- **oauthClientSecret**: OAuth 2.0 client secret (encrypted at rest)
- **oauthRefreshToken**: Long-lived refresh token for obtaining access tokens (encrypted at rest)
- **defaultJiraProject**: Default Jira project key for issue creation (e.g., "PROJ")
- **defaultIssueType**: Default Jira issue type (Story, Task, Epic, or Bug)

**Relationships**:
- OAuth Configuration is scoped [NEEDS CLARIFICATION: per-project or globally for vTeam instance?]
- Multiple projects may share a global OAuth configuration, or each project maintains its own
- OAuth tokens have expiration and refresh lifecycle managed by the system

**Behaviors**:
- Publishing an unpublished file creates a new jiraLinks entry and new Jira issue
- Publishing an already-linked file [NEEDS CLARIFICATION: creates new issue/updates existing/prompts user?]
- Deleting a workspace file [NEEDS CLARIFICATION: preserves or removes?] its jiraLinks entries
- Renaming a workspace file [NEEDS CLARIFICATION: breaks link (if path-based) or maintains link (if hash-based)?]

---

## Out of Scope

The following items are explicitly excluded from this feature:

- **Bi-directional sync**: Jira issue updates do NOT sync back to vTeam workspace files
- **Automatic updates**: Workspace file changes do NOT automatically update linked Jira issues
- **Confluence integration**: May be considered in future RFE using same MCP pattern
- **On-premise Jira**: Atlassian MCP Server requires Atlassian Cloud; Jira Server/Data Center not supported
- **Migration of existing links**: Jira links from previous custom integration are not automatically migrated
- **Custom field mapping**: Only standard fields (summary, description, project, issue type) are populated; custom Jira fields are not supported in MVP
- **Jira webhooks**: vTeam does not receive or process Jira webhook events for issue status updates
- **Batch publishing UI**: Multiple file selection and bulk publish is post-MVP (single file at a time in MVP)
- **Issue type auto-detection**: Issue type is user-configured (default Story); no automatic detection based on file type

---

## Prerequisites & Constraints

### User Prerequisites
- Users must have Atlassian Cloud account (on-premise Jira not supported)
- Users must have permissions to create OAuth apps in Atlassian Developer Console OR admin must configure global OAuth
- Users must have "create issue" permission in the target Jira project
- Users' browsers must allow popups from vTeam domain (for OAuth flow)

### Technical Prerequisites
- vTeam backend has network access to Atlassian MCP Server (https://api.atlassian.com)
- [NEEDS CLARIFICATION: MCP client library at `/archive/mcp_client_integration` must be production-ready, OR commitment to build Go MCP client]
- Kubernetes cluster for storing OAuth secrets securely
- TLS 1.2+ support for secure communication with Atlassian

### Constraints
- **Rate Limits**: Atlassian MCP Server enforces 1,000 requests/hour for Premium/Enterprise, moderate limits for Standard
- **Field Limits**: Jira description field limited to 32,767 characters
- **OAuth Scopes**: OAuth app requires scopes: `write:jira-work`, `read:jira-work`
- **Data Residency**: Requests flow through Atlassian-hosted MCP Server infrastructure (unless self-hosted option chosen)
- **Browser Compatibility**: OAuth popup flow requires modern browsers with popup support

---

## Acceptance Criteria

### MVP Phase 1 (Must Have)

The feature is complete when:

1. **Configuration**:
   - Users can configure Atlassian OAuth credentials in vTeam Project Settings
   - System displays connection status (Connected/Disconnected/Expired/Error) with visual indicators
   - Users can test OAuth connection and validate Jira project accessibility
   - System provides clear setup guidance for OAuth app creation in Atlassian

2. **Publishing**:
   - Users can successfully publish individual workspace files to Jira, creating new issues via Atlassian MCP Server
   - Created Jira issues contain proper summary ("[RFE] {title} - {purpose}"), description (file content), project assignment, and issue type
   - System displays clear loading indicators during publishing with status messages
   - System displays success confirmation with issue key after successful publish

3. **Link Display**:
   - Published files display clickable Jira issue keys as badges next to file names
   - Clicking Jira badge opens issue in new browser tab with proper security attributes
   - Unpublished files display "Publish to Jira" button (visible only when Jira is configured)

4. **Data Persistence**:
   - RFEWorkflow CRs properly track jiraLinks entries mapping file paths to Jira keys with timestamps and URLs
   - jiraLinks entries persist across page refreshes and user sessions
   - System maintains data integrity even if Jira issues are deleted externally

5. **Error Handling**:
   - Error messages are clear and actionable for all scenarios: missing configuration, expired authentication, connection failures, permission issues, rate limits
   - Users can retry failed operations without losing context
   - System distinguishes between transient errors (retry) and permanent errors (reconfigure)

6. **Security**:
   - OAuth tokens stored encrypted in Kubernetes Secrets
   - OAuth consent flow uses proper redirect URIs and security best practices
   - System validates Jira permissions before allowing publish operations
   - External links use `rel="noopener noreferrer"` for security

7. **Observability**:
   - MCP server health status is visible to [NEEDS CLARIFICATION: administrators only, or also end users?]
   - Failed publish operations are logged with trace IDs for support troubleshooting
   - System tracks OAuth token health and expiration timelines

8. **Documentation**:
   - Admin documentation includes OAuth setup instructions with screenshots
   - User documentation explains publishing workflow and error recovery
   - Prerequisites clearly state Atlassian Cloud requirement (no on-premise support)
   - Troubleshooting guide covers common OAuth and MCP connection issues

9. **Testing**:
   - Integration tests validate end-to-end Jira issue creation workflow
   - Error scenarios are covered by automated tests
   - OAuth flow is tested with valid, expired, and revoked tokens
   - Rate limiting behavior is validated

### Post-MVP Phase 2 (Should Have)

10. **Enhanced UX**:
    - Proactive token health monitoring with expiration warnings (7 days before expiry)
    - Batch publishing support for multiple files with per-file status tracking
    - Rich tooltips showing Jira issue details on hover
    - Publishing history/audit log for transparency

11. **Advanced Features**:
    - Custom Jira field mapping configuration
    - Issue type auto-detection based on file purpose
    - Support for self-hosted Atlassian MCP Server
    - Multi-instance support (different Atlassian Cloud accounts per project)

---

## User Experience Details

### OAuth Configuration Workflow

**Step-by-Step Setup** (for non-technical Product Managers):

1. **Prerequisites Display**: Before configuration, system shows checklist:
   - [ ] You have an Atlassian Cloud account
   - [ ] You have admin access to create OAuth apps in Atlassian Developer Console
   - [ ] You have "create issue" permission in your Jira project
   - [Setup Guide Video] [Text Instructions]

2. **OAuth App Creation Guidance**: System provides either:
   - **Option A (Recommended)**: "Use organization OAuth app" - Admin has pre-configured global OAuth, user just authorizes
   - **Option B (Advanced)**: "Create project-specific OAuth app" - User follows guided setup:
     - Link to [Atlassian Developer Console](https://developer.atlassian.com)
     - Step-by-step instructions for creating OAuth 2.0 app
     - Required scopes: `write:jira-work`, `read:jira-work`
     - Redirect URI to copy-paste: `https://vteam.example.com/api/v1/jira/oauth/callback`

3. **Credential Entry**: Users enter:
   - Jira instance URL (with validation and autocomplete for common formats)
   - OAuth Client ID (with masked display after entry)
   - OAuth Client Secret (with masked display and "Show" toggle)

4. **Authorization**: User clicks "Authorize vTeam"
   - System detects popup blocker and shows fallback message if needed
   - OAuth popup opens with Atlassian login
   - User completes consent in Atlassian (grants permissions)
   - Popup auto-closes on success
   - Parent window updates without manual refresh

5. **Validation**: After OAuth success:
   - System fetches accessible Jira projects
   - User selects default project from dropdown
   - User selects default issue type (Story, Task, Epic, Bug)
   - User clicks "Test Connection" to validate end-to-end
   - Success: "✓ Connected to Jira Cloud - mycompany.atlassian.net"

### Publishing Workflow UI

**File List Display** (in RFE Workflow Detail page):

```
Workspace Files:
├── spec/
│   └── requirements.md [PROJ-123 ↗]  [⋮ Actions]
├── architecture/
│   └── system-design.md [PROJ-124 ↗]  [⋮ Actions]
└── epics/
    └── epic-001.md [Publish to Jira]
```

**Publish Button States**:
- **Initial (unpublished)**: "Publish to Jira" button (outline style, medium emphasis)
- **Loading**: "Publishing..." with spinner (disabled state)
- **Success**: Button replaced with Jira badge "[PROJ-123 ↗]" (link style)
- **Already published**: Three-dot menu with options: "View in Jira | Re-publish | Unlink"

**Connection Status Widget** (in Project Settings):

```
┌────────────────────────────────────────────────────┐
│ ✓ Connected to Jira Cloud                         │
│   mycompany.atlassian.net • Project: PROJ         │
│   Last published: 2 hours ago                     │
│   Token expires: 28 days                          │
│   [Disconnect] [Test Connection]                  │
└────────────────────────────────────────────────────┘
```

**Error Display Patterns**:

| Error Type | Display Method | Dismissible | Action |
|-----------|---------------|-------------|--------|
| Configuration missing | Inline alert banner (persistent) | No | Link to settings |
| Token expired | Page-level banner (persistent) | No | Re-authenticate button |
| Connection failure | Toast notification (5 sec) | Yes | Retry button |
| Permission denied | Modal dialog (blocking) | Yes (close button) | Contact admin guidance |
| Rate limited | Toast with countdown (auto-dismiss) | Yes | Queue or retry later |

### Accessibility Considerations

- **ARIA live regions** for status updates during publishing (screen reader announcements)
- **Keyboard navigation** for all interactive elements (OAuth button, publish buttons, Jira links)
- **Color independence** for connection status (icon + text + color, never color alone)
- **Focus management** for OAuth popup flow (return focus to trigger button after close)
- **External link indication** for Jira badges (visually-hidden text or ARIA label: "PROJ-123 opens in new tab")
- **Error announcements** using `role="alert"` for critical errors
- **WCAG 2.1 AA compliance** for color contrast, interactive target sizes, text sizing

---

## Review & Acceptance Checklist

### Content Quality
- [ ] No implementation details (languages, frameworks, APIs) - **PARTIAL**: Some implementation language remains in requirements
- [ ] Focused on user value and business needs - **YES**: Clear user-centric scenarios
- [ ] Written for non-technical stakeholders - **MOSTLY**: Some technical jargon in OAuth sections
- [ ] All mandatory sections completed - **YES**

### Requirement Completeness
- [ ] No [NEEDS CLARIFICATION] markers remain - **NO**: 21 clarification points identified (see below)
- [ ] Requirements are testable and unambiguous - **PARTIAL**: Core requirements testable, but ambiguities in edge cases
- [ ] Success criteria are measurable - **YES**: Acceptance criteria are concrete
- [ ] Scope is clearly bounded - **YES**: Out of scope section is comprehensive
- [ ] Dependencies and assumptions identified - **PARTIAL**: MCP client library status needs validation

### Critical Ambiguities Requiring Resolution

**HIGH PRIORITY (Must resolve before planning):**

1. **MCP Client Language Bridge** (FR-006, Technical Architecture): How is Python MCP client integrated with Go backend? (Sidecar, subprocess, or native Go client?)
2. **MCP Client Library Status**: Does `/archive/mcp_client_integration` exist and is it production-ready? Or must we build Go MCP client?
3. **OAuth Token Storage Model** (FR-011): Per-project secrets or global shared secret?
4. **OAuth Token Refresh Strategy** (FR-012): Silent automatic refresh or user-prompted re-authentication?
5. **OAuth Configuration Scope** (FR-024): Per-project configuration or global admin-configured?
6. **Rate Limit Handling Strategy** (FR-021): Queue requests, fail fast with retry time, or client-side throttling?
7. **Re-publish Behavior** (FR-029, Scenario 3): Create new issue, update existing, or prompt user?
8. **MCP Server Hosting Model** (Technical Architecture): Atlassian-hosted only or support self-hosted?

**MEDIUM PRIORITY (Should clarify for better spec):**

9. **Health Monitoring Visibility** (FR-023): Visible to end users, project admins, or platform admins only?
10. **Multiple Files Publishing** (Scenario 8): Sequential one-at-a-time or parallel with per-file status?
11. **Batch Publishing Scope**: MVP or post-MVP? Affects FR scope and UX design
12. **File-Link Cardinality** (Key Entities): Can one file have multiple Jira links or only one?
13. **Stale Link Handling** (Edge Cases): Validate links periodically, show broken indicator, or leave stale?
14. **File Rename Tracking** (FR-030, Edge Cases): Track by path (breaks on rename) or content hash (maintains)?
15. **Concurrent Publishing** (Edge Cases): Conflict detection, allow duplicates, or use locking?
16. **File Size Validation** (Edge Cases): Handle 32,767 char limit proactively or reactively?
17. **OAuth Popup Timeout** (UX Details): How long before timeout? What message on manual close?
18. **Jira Permission Pre-validation**: Check permissions during config or at publish time?

**LOW PRIORITY (Can defer to planning):**

19. **OAuth App Configuration Help**: Video walkthrough, text guide, or both?
20. **Multiple Atlassian Instances**: Support different Atlassian Cloud accounts per project?
21. **Custom Fields**: Hard out-of-scope or consider for post-MVP?

---

## Execution Status

- [x] User description parsed (from rfe.md)
- [x] Key concepts extracted (OAuth, MCP, publishing, linking)
- [x] Ambiguities marked (21 clarification points identified)
- [x] User scenarios defined (primary + 8 acceptance + 8 edge cases)
- [x] Requirements generated (30 functional requirements + technical architecture section)
- [x] Entities identified (JiraLink, OAuth Configuration)
- [ ] Review checklist passed - **BLOCKED**: Spec has 21 uncertainties requiring clarification

---

## Next Steps

### Before Moving to Planning Phase

This spec requires **architectural decision-making** to resolve critical ambiguities. Recommended approach:

1. **Technical Validation** (1-2 days):
   - Verify `/archive/mcp_client_integration` library exists and assess production-readiness
   - If Python library exists: Architect the Python-Go bridge (recommend HTTP/gRPC sidecar over subprocess)
   - If library doesn't exist: Make "build Go MCP client" an explicit requirement and adjust timeline
   - Create proof-of-concept test: Go backend → MCP client → Atlassian MCP Server → Create test Jira issue

2. **Architecture Workshop** (half-day session with Product, UX, Engineering):
   - **Decision 1**: MCP client integration approach (binding decision)
   - **Decision 2**: OAuth token storage model (per-project vs global)
   - **Decision 3**: MCP server hosting (Atlassian-hosted vs self-hosted support)
   - **Decision 4**: Rate limit strategy (fail-fast vs queue vs throttle)
   - **Decision 5**: Re-publish behavior (new issue vs update vs prompt)
   - Document decisions in "Technical Architecture Decisions (BINDING)" section

3. **UX Clarification** (async review):
   - OAuth configuration flow: Finalize step-by-step UX for non-technical users
   - Error handling patterns: Confirm toast vs inline vs modal for each error type
   - Already-published state: Confirm button replacement vs action menu approach

4. **Spec Revision** (1 day):
   - Replace all [NEEDS CLARIFICATION] markers with binding decisions
   - Add validated MCP client library status to Prerequisites
   - Finalize data model schema for jiraLinks field
   - Update acceptance criteria with resolved ambiguities

5. **Stakeholder Review** (2-3 days):
   - Product Manager: Validates user scenarios and acceptance criteria
   - UX Architect: Reviews OAuth setup complexity and error UX
   - Staff Engineer: Confirms technical feasibility and MVP scope realism
   - Security: Reviews OAuth token storage and permission validation approach

### After Spec is Finalized

6. **Use `/clarify` command** to identify any remaining underspecified areas through targeted questions
7. **Use `/plan` command** to generate implementation plan with tasks
8. **Use `/analyze` command** to validate consistency across spec, plan, and tasks

---

## References

- **Source RFE**: `/workspace/sessions/agentic-session-1760541094/workspace/vteam-rfes/rfe.md`
- **Previous Jira Integration**: Commits c149a3f, 6497b4c, bae8bfc (custom REST API implementation)
- **Atlassian MCP Server Documentation**: https://developer.atlassian.com/cloud/mcp/
- **MCP Protocol Specification**: https://modelcontextprotocol.io/
- **RFEWorkflow CRD**: (path to CRD definition - to be added)
- **vTeam Architecture Documentation**: (link to architecture docs - to be added)

---

## Appendix: Agent Analysis Summary

This specification was developed with input from specialized agents:

**Aria (UX Architect)** identified:
- OAuth setup complexity as major adoption risk (3x more setup friction than API tokens)
- Need for comprehensive error taxonomy with 4 categories (configuration, authentication, infrastructure, operation)
- Multi-phase publishing feedback requirements (5 phases from initiation to success/failure)
- 8 critical UX ambiguities including OAuth flow hosting, token refresh transparency, and permission mismatch handling
- Accessibility considerations for OAuth popups, screen readers, and WCAG 2.1 AA compliance

**Felix (UX Feature Lead)** identified:
- 5 core component patterns (OAuth Connection Manager, File List with Actions, Jira Link Component, Error States, Publishing Workflow)
- Visual hierarchy recommendations for settings integration and workflow pages
- Detailed interaction design for OAuth flow (5-step sequence) and publishing (6-step sequence with micro-interactions)
- Accessibility requirements including ARIA live regions, keyboard navigation, focus management, and WCAG compliance checklist
- 18 design ambiguities across component behavior, visual treatment, interaction patterns, and edge cases

**Stella (Staff Engineer)** identified:
- **Critical risk**: Python-Go impedance mismatch with MCP client is underspecified and could be a showstopper
- MCP client library status needs validation (does `/archive/mcp_client_integration` actually exist and work?)
- 3 architectural decisions that MUST be in spec (not deferred): MCP client bridge, OAuth token model, MCP hosting approach
- MVP scope is too ambitious - recommended splitting into MVP1 (core publishing) and MVP2 (enhancements)
- 5 requirements gaps: multi-tenancy/security, idempotency, observability, data migration handling, jiraLinks data model
- Estimated **2-3 weeks of discovery work** if spec proceeds without resolving ambiguities

---

*This specification is marked as **Draft - Needs Technical Refinement**. It should not proceed to implementation planning until the 8 high-priority architectural ambiguities are resolved through technical validation and architecture workshop.*
