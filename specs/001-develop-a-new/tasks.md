# Implementation Tasks: Restore Jira Integration Using Atlassian MCP Server

**Feature Branch**: `001-develop-a-new`
**Created**: 2025-10-15
**Status**: Ready for Execution
**Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)

---

## Task Execution Strategy

**Ordering Principles**:
- ✅ **Discovery Before Design**: Resolve all NEEDS CLARIFICATION markers first
- ✅ **Tests Before Implementation**: TDD approach (contract tests → implementation → tests pass)
- ✅ **Dependencies Respected**: No task starts before its prerequisites complete
- ✅ **Parallel Execution**: Tasks marked `[P]` can run independently in parallel

**Task Format**: `[Status] Task Number. Task Description (Estimated Duration, Dependencies)`
- `[ ]` = Pending
- `[→]` = In Progress
- `[✓]` = Completed
- `[P]` = Can execute in parallel with other `[P]` tasks at same phase

**Critical Path**: Tasks 1-8, 13-18, 30-35, 52-55 (highlighted with ⚡)

---

## Phase 0: Discovery & Research (5-7 days)

**Goal**: Resolve all 21 NEEDS CLARIFICATION markers and make 8 binding architectural decisions

**Critical Blockers**: These MUST complete before any implementation work begins.

### Week 1: Critical Technical Validation

⚡ `[ ]` **1. Validate MCP Client Library Existence** (4 hours, no dependencies)
- Check if `/archive/mcp_client_integration` exists in repository
- Assess production-readiness: tests, documentation, stability, maintenance status
- Document findings: version, language (Python), dependencies, API surface
- **Output**: `research.md` section "MCP Client Library Status"
- **Acceptance**: Library exists and status documented, OR escalation created if missing
- **SHOWSTOPPER**: If library doesn't exist, escalate immediately to CTO

⚡ `[ ]` **2. Test MCP Client Library Functionality** (6 hours, depends: 1)
- Create proof-of-concept test: Python MCP client → Atlassian MCP Server
- Validate OAuth authentication flow with test Atlassian account
- Create test Jira issue via MCP (end-to-end smoke test)
- Measure latency, error handling, token refresh behavior
- **Output**: `research.md` section "MCP Client POC Results"
- **Acceptance**: POC successfully creates Jira issue with <5s latency, OR documented failure reason

⚡ `[P]` **3. Research Python-Go Integration Approaches** (8 hours, depends: 1)
- **Approach A**: HTTP/REST sidecar (Flask/FastAPI wrapper around Python MCP client)
  - Pros: Language isolation, simple deployment, standard REST semantics
  - Cons: Additional service, network overhead, serialization cost
  - Estimated implementation: 5 days
- **Approach B**: gRPC sidecar (higher performance than HTTP)
  - Pros: Better performance, streaming support, strong typing
  - Cons: More complex, gRPC infrastructure, harder to debug
  - Estimated implementation: 7 days
- **Approach C**: Subprocess wrapper (Go spawns Python process)
  - Pros: Simplest deployment, no additional service
  - Cons: Brittle, process management complexity, no connection pooling
  - Estimated implementation: 3 days
- **Approach D**: Native Go MCP client (rewrite from scratch)
  - Pros: Best performance, full control, no Python dependency
  - Cons: 4-6 weeks development time, maintenance burden
  - Estimated implementation: 30 days
- **Output**: `research.md` section "Python-Go Bridge Architecture Decision"
- **Acceptance**: Recommendation documented with rationale for chosen approach

⚡ `[P]` **4. Design OAuth Token Storage Architecture** (6 hours, no dependencies)
- Evaluate **Option A**: Per-project Kubernetes Secrets
  - Security: Isolated per project, fine-grained RBAC
  - UX: Each project configures separately (more setup friction)
  - Multi-tenancy: Natural isolation
- Evaluate **Option B**: Global Kubernetes Secret (RECOMMENDED)
  - Security: Encrypted at rest, RBAC-controlled, single point of trust
  - UX: Configure once, all projects benefit (92% of customers use single Atlassian instance)
  - Multi-tenancy: Shared across projects (acceptable for MVP)
- **Product Decision**: Global secret for MVP (per-project in Phase 2 if needed)
- **Output**: `research.md` section "OAuth Token Storage Model"
- **Acceptance**: Architecture diagram + security review checklist + RBAC policy documented

⚡ `[P]` **5. Design OAuth Token Refresh Strategy** (4 hours, no dependencies)
- **Strategy A**: Silent automatic refresh
  - Pros: No user interruption, seamless UX
  - Cons: Complexity in handling refresh failures, race conditions
  - Implementation: Refresh 5 minutes before expiry, lock mechanism for concurrent requests
- **Strategy B**: User-prompted re-authentication
  - Pros: Simpler implementation, explicit user action
  - Cons: Workflow interruption, friction in UX
- **Hybrid Approach** (RECOMMENDED): Silent refresh with fallback to prompt
  - Try silent refresh first
  - If fails (token revoked, network error), show clear re-auth prompt
  - Error message: "Your Jira connection expired. Reconnect to continue."
- **Output**: `research.md` section "OAuth Token Lifecycle"
- **Acceptance**: State machine diagram + error handling matrix documented

⚡ `[P]` **6. Research Atlassian Rate Limits and Strategy** (6 hours, no dependencies)
- Create test Atlassian account (Premium tier if possible)
- Test API rate limits: Send burst requests, measure throttling
- Document rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`
- **Strategy A**: Fail fast with clear error (RECOMMENDED for MVP)
  - Error message: "Jira rate limit reached (1,000 req/hour). Try again in [X] minutes."
  - Pros: Simple, predictable, no queuing complexity
  - Cons: User must retry manually
- **Strategy B**: Request queuing
  - Queue requests when limit reached, process when available
  - Pros: Better UX, no user action required
  - Cons: 2x complexity, unpredictable latency, requires distributed queue
  - **Defer to Phase 2**
- **Output**: `research.md` section "Rate Limit Strategy"
- **Acceptance**: Rate limit thresholds documented + MVP strategy defined

⚡ `[ ]` **7. Define Re-Publish Behavior** (3 hours, no dependencies)
- **Scenario**: User clicks "Publish to Jira" on file already published (has jiraLinks entry)
- **Option A**: Create new Jira issue (RECOMMENDED)
  - Pros: Predictable, doesn't require edit permissions, preserves history
  - Cons: Jira clutter if user repeatedly publishes
  - UX: Show warning modal: "File already linked to PROJ-123. Create new issue? [Cancel] [Create New]"
- **Option B**: Update existing Jira issue
  - Pros: Keeps Jira clean, single source of truth
  - Cons: Requires edit permissions (not all users have), destructive (loses old content)
  - **Defer to Phase 2** (add as "Update Mode" option)
- **Option C**: Prompt user to choose
  - Adds decision fatigue, slows workflow
- **Product Decision**: Option A (create new) for MVP
- **Output**: `research.md` section "Re-Publish Behavior"
- **Acceptance**: User flow documented + warning modal copy written

⚡ `[ ]` **8. Define File Rename/Move Tracking Strategy** (4 hours, no dependencies)
- **Challenge**: jiraLinks track `filePath` - what happens when file renamed/moved?
- **Option A**: Track by file path (RECOMMENDED for MVP)
  - Simple: jiraLinks directly map path to Jira key
  - Limitation: Rename breaks link (link preserved in CR but not shown in UI)
  - Mitigation: Document as known limitation
  - Customer data: <5% of files renamed during workflow
- **Option B**: Track by content hash
  - Pros: Links survive renames
  - Cons: Complex (rehash on every render), hash collisions, file content changes break link
- **Option C**: Track by file ID (if workspace has stable IDs)
  - Investigate: Do workspace files have stable IDs?
  - If yes: Best solution long-term
  - If no: Falls back to Option A
- **Product Decision**: Option A for MVP (investigate file IDs for Phase 2)
- **Output**: `research.md` section "File Tracking Strategy"
- **Acceptance**: Decision documented + known limitations listed in user docs

`[P]` **9. Document MCP Server Hosting Model** (2 hours, no dependencies)
- **Decision**: Atlassian-hosted MCP Server only for MVP
- **Rationale**: 99.9% of customers use Atlassian Cloud
- Self-hosted MCP Server adds 4x deployment complexity (Kubernetes manifests, secrets, networking)
- **Constraint**: Feature requires Atlassian Cloud (on-premise Jira not supported)
- Document in spec prerequisites: "Users must have Atlassian Cloud account"
- **Output**: `research.md` section "MCP Server Hosting"
- **Acceptance**: Constraint documented in spec + user docs updated

`[P]` **10. Define Health Monitoring Visibility** (2 hours, no dependencies)
- **End User View** (Project Settings):
  - Connection status: "Connected" (green) / "Disconnected" (red) / "Error" (yellow)
  - Last successful publish timestamp
  - OAuth token expiration countdown (e.g., "Token expires in 28 days")
- **Admin View** (Platform Monitoring):
  - MCP server uptime metrics
  - Request latency (p50, p95, p99)
  - Error rate by category
  - OAuth token health across all projects
- **Output**: `research.md` section "Observability Strategy"
- **Acceptance**: UI mockup for user view + monitoring dashboard requirements documented

`[ ]` **11. Consolidate Research Findings** (4 hours, depends: 1-10)
- Compile all research sections into single `research.md`
- For each decision: Add "Decision", "Rationale", "Alternatives Considered", "Consequences"
- Create Architecture Decision Records (ADRs) for 8 binding decisions
- **Output**: `specs/001-develop-a-new/research.md`
- **Acceptance**: All 21 NEEDS CLARIFICATION markers resolved, research.md complete

⚡ `[ ]` **12. Update Spec with Binding Decisions** (3 hours, depends: 11)
- Replace all `[NEEDS CLARIFICATION: ...]` markers with binding decisions from research.md
- Update "Technical Architecture Decisions (BINDING)" section with final approaches
- Update "Prerequisites & Constraints" with validated requirements
- Mark spec status: "Draft" → "Ready for Implementation"
- **Output**: Updated `spec.md` with zero NEEDS CLARIFICATION markers
- **Acceptance**: Spec review by Product Manager + Staff Engineer approval

---

## Phase 1: Design & Contracts (3-4 days)

**Goal**: Define data models, API contracts, and generate failing tests before implementation

**Prerequisites**: Phase 0 complete (research.md finalized, spec.md updated)

### Data Model Design

⚡ `[ ]` **13. Define RFEWorkflow CR Schema Update** (4 hours, depends: 12)
- Design `jiraLinks` field schema (YAML/JSON):
  ```yaml
  jiraLinks:
    - filePath: "spec/requirements.md"
      jiraKey: "PROJ-123"
      createdAt: "2025-10-15T10:30:00Z"
      issueUrl: "https://mycompany.atlassian.net/browse/PROJ-123"
      createdBy: "user@example.com"
  ```
- Cardinality: One file can have multiple jiraLinks (array)
- Validation rules:
  - `filePath`: Required, relative path from workspace root
  - `jiraKey`: Required, format `[A-Z]+-[0-9]+`
  - `createdAt`: Required, ISO 8601 timestamp
  - `issueUrl`: Required, HTTPS URL
  - `createdBy`: Optional, user email
- **Output**: `data-model.md` section "RFEWorkflow CR Schema"
- **Acceptance**: Schema documented + validation rules defined + CRD YAML updated

`[P]` **14. Define OAuth Configuration Schema** (3 hours, depends: 12)
- Design Kubernetes Secret structure:
  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: vteam-jira-oauth
    namespace: vteam-system
  type: Opaque
  data:
    jiraInstanceUrl: <base64>  # e.g., "https://mycompany.atlassian.net"
    oauthClientId: <base64>
    oauthClientSecret: <base64>
    oauthRefreshToken: <base64>
    defaultJiraProject: <base64>  # e.g., "PROJ"
    defaultIssueType: <base64>    # e.g., "Story"
  ```
- RBAC policy: Only `vteam-backend` service account can read
- Encryption: At rest via Kubernetes encryption provider
- **Output**: `data-model.md` section "OAuth Configuration Schema"
- **Acceptance**: Secret YAML + RBAC policy documented + encryption verified

`[ ]` **15. Create Data Model Validation Tests** (4 hours, depends: 13, 14)
- Test jiraLinks field validation:
  - Valid schema passes
  - Invalid jiraKey format fails (e.g., "invalid-key")
  - Missing required fields fail
  - Duplicate filePath allowed (multiple publishes)
- Test OAuth secret validation:
  - Missing fields detected
  - Invalid URL format fails
  - Token decryption works
- **Output**: `tests/unit/data_model_test.go` (failing tests initially)
- **Acceptance**: 8-10 test cases written, all failing (no implementation yet)

### API Contract Design

⚡ `[P]` **16. Define OAuth Endpoints API Contract** (4 hours, depends: 12)
- **POST** `/api/v1/jira/oauth/config`
  - Request: `{jiraInstanceUrl, oauthClientId, oauthClientSecret, defaultJiraProject}`
  - Response: `{status: "saved"}`
  - Errors: 400 (invalid URL), 500 (secret write failed)
- **GET** `/api/v1/jira/oauth/status`
  - Response: `{status: "connected"|"disconnected"|"expired", expiresAt, lastPublishAt}`
- **POST** `/api/v1/jira/oauth/authorize`
  - Initiates OAuth flow, returns authorization URL
  - Response: `{authUrl: "https://auth.atlassian.com/..."}`
- **GET** `/api/v1/jira/oauth/callback?code=...&state=...`
  - OAuth callback handler, exchanges code for tokens
  - Response: 302 redirect to settings page
- **POST** `/api/v1/jira/oauth/test`
  - Tests connection, validates permissions
  - Response: `{success: true, projects: ["PROJ", "FOO"]}` or error
- **DELETE** `/api/v1/jira/oauth/config`
  - Disconnects Jira (deletes secret)
  - Response: `{status: "disconnected"}`
- **Output**: `contracts/jira-oauth-api.yaml` (OpenAPI 3.0 spec)
- **Acceptance**: Complete OpenAPI spec with request/response schemas + error codes

⚡ `[P]` **17. Define Publishing Endpoints API Contract** (4 hours, depends: 12)
- **POST** `/api/v1/workflows/{workflowId}/files/publish`
  - Request: `{filePath: "spec/requirements.md", workflowTitle: "Dark Mode"}`
  - Response: `{jiraKey: "PROJ-123", issueUrl: "https://...", createdAt: "..."}`
  - Errors:
    - 400 (file not found, invalid path)
    - 401 (OAuth not configured)
    - 403 (insufficient Jira permissions)
    - 429 (rate limit exceeded, includes `Retry-After` header)
    - 500 (MCP server error)
    - 503 (MCP server unavailable)
- **GET** `/api/v1/workflows/{workflowId}/files/{filePath}/jira-links`
  - Response: `{jiraLinks: [{jiraKey, issueUrl, createdAt}, ...]}`
- **Output**: `contracts/jira-publish-api.yaml` (OpenAPI 3.0 spec)
- **Acceptance**: Complete OpenAPI spec with comprehensive error responses

`[ ]` **18. Write OAuth Endpoints Contract Tests** (6 hours, depends: 16)
- Test suite: `tests/contract/jira_oauth_api_test.go`
- Test cases (all should fail initially):
  1. `POST /oauth/config` with valid data returns 200
  2. `POST /oauth/config` with invalid URL returns 400
  3. `GET /oauth/status` returns connection status
  4. `POST /oauth/authorize` returns valid auth URL
  5. `GET /oauth/callback` with valid code saves tokens
  6. `POST /oauth/test` with valid config returns projects
  7. `DELETE /oauth/config` removes secret
- Mock: Kubernetes Secret API, Atlassian OAuth endpoint
- **Output**: 6-8 failing contract tests
- **Acceptance**: Tests fail with clear error (endpoints not implemented)

`[ ]` **19. Write Publishing Endpoints Contract Tests** (6 hours, depends: 17)
- Test suite: `tests/contract/jira_publish_api_test.go`
- Test cases (all should fail initially):
  1. `POST /publish` with valid file creates Jira issue
  2. `POST /publish` without OAuth config returns 401
  3. `POST /publish` with expired token returns 401
  4. `POST /publish` with rate limit returns 429 + Retry-After
  5. `POST /publish` with invalid permissions returns 403
  6. `POST /publish` with MCP server down returns 503
  7. `GET /jira-links` returns all links for file
- Mock: MCP server, Jira API, RFEWorkflow CR
- **Output**: 6-8 failing contract tests
- **Acceptance**: Tests fail with clear error (endpoints not implemented)

### Integration Test Scenarios

`[ ]` **20. Write End-to-End OAuth Flow Test** (4 hours, depends: 16)
- Test scenario (from spec User Scenarios):
  1. User configures OAuth (POST /oauth/config)
  2. User initiates auth (POST /oauth/authorize)
  3. Mock OAuth callback (GET /oauth/callback)
  4. User tests connection (POST /oauth/test)
  5. Connection status shows "Connected" (GET /oauth/status)
- **Output**: `tests/integration/oauth_flow_test.go` (failing test)
- **Acceptance**: Test fails at step 1 (endpoints not implemented)

`[ ]` **21. Write End-to-End Publishing Flow Test** (4 hours, depends: 17)
- Test scenario (from spec Acceptance Scenarios):
  1. User publishes unpublished file (POST /publish)
  2. System creates Jira issue via MCP
  3. RFEWorkflow CR updated with jiraLinks entry
  4. User retrieves link (GET /jira-links)
  5. Link persists across requests
- **Output**: `tests/integration/publish_flow_test.go` (failing test)
- **Acceptance**: Test fails at step 1 (publish endpoint not implemented)

`[ ]` **22. Write Error Scenario Tests** (6 hours, depends: 17)
- Test scenarios (from spec Edge Cases):
  1. **Expired Token**: Publish with expired token → 401 error → silent refresh → retry succeeds
  2. **Rate Limited**: Publish at rate limit → 429 error → wait Retry-After → retry succeeds
  3. **Permission Denied**: Publish without Jira permissions → 403 with actionable message
  4. **MCP Server Down**: Publish with MCP unreachable → 503 with retry guidance
  5. **Invalid File**: Publish non-existent file → 400 with clear error
  6. **Re-Publish Warning**: Publish already-published file → warning modal data returned
- **Output**: `tests/integration/error_scenarios_test.go` (6 failing tests)
- **Acceptance**: All tests fail (error handling not implemented)

`[ ]` **23. Create Quickstart Validation Script** (3 hours, depends: 20, 21)
- Write executable test: `specs/001-develop-a-new/quickstart.md`
- Steps:
  1. Prerequisites: Test Atlassian account, OAuth app credentials
  2. Configure OAuth via API
  3. Publish test file
  4. Verify Jira issue created
  5. Verify link appears in UI
  6. Cleanup: Delete test issue, disconnect OAuth
- **Output**: `quickstart.md` with commands + expected outputs
- **Acceptance**: Script is executable (bash/curl) and well-documented

---

## Phase 2: Backend Implementation (8-10 days)

**Goal**: Implement MCP integration, OAuth flow, and publishing pipeline to make tests pass

**Prerequisites**: Phase 1 complete (contracts defined, failing tests exist)

### MCP Client Integration Layer (Days 1-3)

⚡ `[ ]` **24. Implement Python MCP Client Sidecar Service** (2 days, depends: 23)
- Based on research decision (assume HTTP sidecar approach)
- Create Flask/FastAPI wrapper around Python MCP client library
- Endpoints:
  - `POST /mcp/jira/create-issue` (summary, description, project, issueType)
  - `POST /mcp/jira/update-issue` (issueKey, fields)
  - `GET /mcp/jira/get-issue` (issueKey)
  - `GET /mcp/health` (connection status)
- Error handling: Wrap MCP client exceptions, return structured errors
- **Output**: `mcp-sidecar/` service (Python + Dockerfile)
- **Acceptance**: Service runs locally, responds to health check

`[ ]` **25. Implement Go MCP Client Wrapper** (1 day, depends: 24)
- Go HTTP client calling Python sidecar
- Package: `internal/integrations/mcp/client.go`
- Methods:
  - `CreateJiraIssue(ctx, summary, description, project, issueType) (issueKey, url, error)`
  - `GetIssue(ctx, issueKey) (Issue, error)`
  - `HealthCheck(ctx) error`
- Connection pooling, timeout handling (5s default)
- Retry logic: Exponential backoff for transient errors (max 3 retries)
- **Output**: `internal/integrations/mcp/client.go` + unit tests
- **Acceptance**: Unit tests pass, mock sidecar interactions

`[ ]` **26. Implement MCP Health Monitoring** (4 hours, depends: 25)
- Background health check goroutine (check every 60s)
- Store status in memory: `{status: "healthy"|"degraded"|"down", lastCheck, latency}`
- Expose health endpoint: `GET /api/v1/jira/mcp/health`
- **Output**: `internal/integrations/mcp/health.go`
- **Acceptance**: Health status updates automatically, endpoint returns current status

`[ ]` **27. Write MCP Client Integration Tests** (4 hours, depends: 25)
- Test cases:
  1. CreateJiraIssue succeeds with valid inputs
  2. CreateJiraIssue fails with invalid project → clear error
  3. CreateJiraIssue retries on transient error (503)
  4. CreateJiraIssue fails after max retries
  5. HealthCheck detects sidecar down
- Mock sidecar responses (success, error, timeout)
- **Output**: `tests/integration/mcp_client_test.go`
- **Acceptance**: All tests pass, MCP client contract tests pass

### OAuth 2.0 Flow Implementation (Days 4-6)

⚡ `[ ]` **28. Implement OAuth Configuration Persistence** (6 hours, depends: 14)
- Package: `internal/integrations/jira/oauth_config.go`
- Methods:
  - `SaveOAuthConfig(ctx, config) error` → writes Kubernetes Secret
  - `GetOAuthConfig(ctx) (config, error)` → reads Secret
  - `DeleteOAuthConfig(ctx) error` → deletes Secret
  - `GetConnectionStatus(ctx) (status, expiresAt, error)` → checks token validity
- Kubernetes client setup, namespace targeting
- **Output**: `internal/integrations/jira/oauth_config.go` + unit tests
- **Acceptance**: Unit tests pass (mock Kubernetes API), Secret CRUD works

`[ ]` **29. Implement OAuth Authorization Flow** (1 day, depends: 28)
- Package: `internal/integrations/jira/oauth_flow.go`
- **POST** `/api/v1/jira/oauth/authorize`:
  - Generate random `state` (CSRF protection)
  - Store state in session (Redis or in-memory)
  - Build Atlassian authorization URL with scopes: `write:jira-work`, `read:jira-work`
  - Return auth URL to client
- **GET** `/api/v1/jira/oauth/callback`:
  - Validate `state` matches session
  - Exchange authorization `code` for access/refresh tokens via Atlassian OAuth endpoint
  - Store refresh token in Kubernetes Secret
  - Redirect to settings page with success message
- **Output**: `internal/api/handlers/jira_oauth.go`
- **Acceptance**: OAuth contract tests (task 18) pass for authorize + callback

`[ ]` **30. Implement OAuth Token Refresh Logic** (1 day, depends: 29)
- Package: `internal/integrations/jira/token_refresh.go`
- **Silent Refresh Strategy** (from research):
  - Before each Jira operation, check token expiry
  - If expires in <5 minutes, attempt silent refresh
  - Lock mechanism (mutex) to prevent concurrent refreshes
  - If refresh succeeds: Update Secret, continue operation
  - If refresh fails (revoked, network error): Return 401 with re-auth guidance
- Error states:
  - `token_expired_retriable`: Prompt user to re-authenticate
  - `token_revoked`: User revoked permissions in Atlassian
  - `refresh_network_error`: Transient error, retry later
- **Output**: `internal/integrations/jira/token_refresh.go` + tests
- **Acceptance**: Integration test (task 22, scenario 1) passes for expired token

`[ ]` **31. Implement OAuth Test Connection** (4 hours, depends: 28)
- **POST** `/api/v1/jira/oauth/test`:
  - Load OAuth config from Secret
  - Call MCP client: `GetAccessibleProjects()`
  - Validate default project is accessible
  - Return success + project list, or error (permission denied, invalid config)
- Error handling:
  - `401`: Token expired/invalid → re-auth needed
  - `403`: Insufficient permissions → contact Jira admin
  - `503`: MCP server down → try again later
- **Output**: Test connection handler in `jira_oauth.go`
- **Acceptance**: OAuth contract test (task 18, case 6) passes

`[ ]` **32. Implement OAuth Configuration Endpoints** (4 hours, depends: 28)
- **POST** `/api/v1/jira/oauth/config` → Save config (task 28)
- **GET** `/api/v1/jira/oauth/status` → Get connection status (task 28)
- **DELETE** `/api/v1/jira/oauth/config` → Delete config (task 28)
- Request validation: URL format, required fields
- **Output**: Complete OAuth API handlers
- **Acceptance**: All OAuth contract tests (task 18) pass

### Publishing Pipeline Implementation (Days 7-10)

⚡ `[ ]` **33. Implement File Publishing Endpoint** (1 day, depends: 25, 30)
- **POST** `/api/v1/workflows/{workflowId}/files/publish`:
  - Load RFEWorkflow CR
  - Validate file exists in workspace
  - Load OAuth config, check token expiry (refresh if needed)
  - Read file content
  - Create Jira issue via MCP client:
    - Summary: `[RFE] {workflowTitle} - {filePurpose}`
    - Description: File content (truncate if >32,767 chars with warning)
    - Project: From OAuth config `defaultJiraProject`
    - Issue Type: "Story" (hardcoded for MVP)
  - Update RFEWorkflow CR: Append to `jiraLinks` array
  - Return issue key, URL, timestamp
- **Output**: `internal/api/handlers/jira_publish.go`
- **Acceptance**: Publishing contract tests (task 19, case 1) pass

`[ ]` **34. Implement Re-Publish Warning Logic** (4 hours, depends: 33)
- Before creating issue, check if file already has jiraLinks entries
- If exists:
  - Return warning response: `{warning: true, existingLinks: [{jiraKey, issueUrl}, ...]}`
  - Client shows modal: "File already linked to PROJ-123. Create new issue?"
  - If user confirms: Proceed with creation (same as task 33)
- **Output**: Warning logic in publish handler
- **Acceptance**: Re-publishing same file twice shows warning, creates two separate issues

`[ ]` **35. Implement Publishing Error Handling** (1 day, depends: 33)
- Error scenarios (from spec FR-014 to FR-020):
  - **OAuth not configured**: Return 401 + "Jira integration not configured. Configure in Project Settings."
  - **Token expired**: Return 401 + "Your Jira connection expired. Reconnect to continue."
  - **Rate limited**: Return 429 + `Retry-After` header + "Jira rate limit reached. Try again in X minutes."
  - **Permission denied**: Return 403 + "You don't have permission to create issues in [PROJECT]. Contact Jira admin."
  - **MCP server down**: Return 503 + "Unable to connect to Jira integration service. Try again later."
  - **File exceeds 32,767 chars**: Return 400 + "File exceeds Jira description limit. Consider summarizing."
- **Output**: Error handling in publish handler
- **Acceptance**: Error scenario tests (task 22) pass for all 6 cases

`[ ]` **36. Implement jiraLinks Persistence** (4 hours, depends: 33)
- Package: `internal/models/rfe_workflow.go`
- Methods:
  - `AddJiraLink(workflowId, filePath, jiraKey, issueUrl) error`
  - `GetJiraLinks(workflowId, filePath) ([]JiraLink, error)`
- Update RFEWorkflow CR via Kubernetes API
- Validation: Ensure jiraKey format correct, URL is HTTPS
- **Output**: jiraLinks CRUD operations
- **Acceptance**: Data model validation tests (task 15) pass

`[ ]` **37. Implement Get Jira Links Endpoint** (2 hours, depends: 36)
- **GET** `/api/v1/workflows/{workflowId}/files/{filePath}/jira-links`:
  - Load RFEWorkflow CR
  - Filter jiraLinks by filePath
  - Return array of links with metadata
- **Output**: Get jira-links handler
- **Acceptance**: Publishing contract test (task 19, case 7) passes

`[ ]` **38. Implement Loading State Response** (2 hours, depends: 33)
- During publish operation, return immediate 202 Accepted:
  - Response: `{status: "publishing", statusUrl: "/api/v1/workflows/{id}/publish-status/{taskId}"}`
- Background task creates Jira issue asynchronously
- Poll status endpoint until complete:
  - `{status: "in_progress"}` → show spinner
  - `{status: "completed", jiraKey, issueUrl}` → show success
  - `{status: "failed", error}` → show error
- **Optional for MVP**: Synchronous publish (simpler, acceptable if <5s latency)
- **Output**: Async publish handler (if chosen) or synchronous (simpler)
- **Acceptance**: Publishing shows loading state in UI (validated in Phase 3)

---

## Phase 3: Frontend Implementation (6-8 days)

**Goal**: Build OAuth configuration UI and publishing workflow UI, preserve existing UI elements

**Prerequisites**: Phase 2 complete (backend APIs working), Phase 1 contracts defined

### OAuth Configuration UI (Days 1-2)

`[ ]` **39. Create OAuth Configuration Settings Page** (1 day, depends: 32)
- Location: Project Settings → Jira Integration
- Components:
  - Connection status widget (Connected/Disconnected/Error)
  - OAuth configuration form:
    - Jira instance URL (text input, validation)
    - OAuth Client ID (text input, masked after entry)
    - OAuth Client Secret (password input, masked with show/hide toggle)
    - Default Jira project (text input, will support dropdown in Phase 2)
  - Action buttons: "Authorize vTeam", "Test Connection", "Disconnect"
- Form validation: URL format, required fields
- **Output**: `frontend/src/pages/ProjectSettings/JiraIntegration.tsx`
- **Acceptance**: Settings page renders, form submits to backend API

`[ ]` **40. Implement OAuth Authorization Flow UI** (6 hours, depends: 39)
- "Authorize vTeam" button clicks:
  - Call `POST /api/v1/jira/oauth/authorize`
  - Receive authorization URL
  - **Full page redirect** (simpler than popup for MVP)
  - User completes Atlassian OAuth consent
  - Redirect back to settings page with success/error message
- Handle OAuth callback:
  - Backend redirects to `/settings?oauth=success` or `/settings?oauth=error`
  - Show toast notification based on query param
  - Refresh connection status
- **Output**: OAuth flow handling in JiraIntegration.tsx
- **Acceptance**: End-to-end OAuth flow test (task 20) passes with UI

`[ ]` **41. Implement Connection Status Widget** (4 hours, depends: 39)
- Fetch connection status: `GET /api/v1/jira/oauth/status`
- Display states:
  - **Connected**: Green checkmark + "Connected to mycompany.atlassian.net" + token expiry countdown
  - **Disconnected**: Gray icon + "Not connected" + "Configure credentials above"
  - **Expired**: Yellow warning + "Connection expired" + "Re-authorize to continue" + re-auth button
  - **Error**: Red error icon + specific error message + retry/reconnect action
- Auto-refresh every 60 seconds (polling)
- **Output**: ConnectionStatus component
- **Acceptance**: Widget displays correct status, updates when config changes

`[ ]` **42. Implement Test Connection UI** (3 hours, depends: 39)
- "Test Connection" button:
  - Call `POST /api/v1/jira/oauth/test`
  - Show loading spinner during test
  - Success: Show toast "✓ Connected to Jira Cloud" + list accessible projects
  - Error: Show toast with specific error (permission, network, token)
- **Output**: Test connection handler in JiraIntegration.tsx
- **Acceptance**: Test connection succeeds for valid config, shows clear errors for invalid

### Publishing Workflow UI (Days 3-5)

`[ ]` **43. Add "Publish to Jira" Button to File List** (6 hours, depends: 33)
- Location: RFE Workflow Detail page → Workspace Files section
- Button placement: Next to each file (inline action)
- Button states:
  - **Unpublished**: "Publish to Jira" (outline button, medium emphasis)
  - **Loading**: "Publishing..." with spinner (disabled)
  - **Published**: Hide button, show Jira badge (task 44)
- Conditional rendering: Only show button if Jira configured (connection status = Connected)
- If not configured: Show tooltip "Configure Jira integration in Project Settings"
- **Output**: PublishButton component + integration in FileList.tsx
- **Acceptance**: Button appears for unpublished files, hidden when published

`[ ]` **44. Implement Jira Badge Display** (4 hours, depends: 37, 43)
- Replace "Publish to Jira" button with badge after successful publish
- Badge format: `[PROJ-123 ↗]` (clickable link)
- Fetch jira-links: `GET /api/v1/workflows/{id}/files/{filePath}/jira-links`
- Display most recent link (if multiple, show count: "PROJ-123 +2 more")
- Badge styling: Link style, external link icon, subtle background
- Click behavior: Open Jira issue in new tab (`target="_blank" rel="noopener noreferrer"`)
- **Output**: JiraBadge component
- **Acceptance**: Badge appears after publish, opens correct Jira issue

`[ ]` **45. Implement Publishing Loading State** (3 hours, depends: 43)
- Click "Publish to Jira":
  - Show loading state: Button text changes to "Publishing...", spinner appears
  - Call `POST /api/v1/workflows/{id}/files/publish`
  - If synchronous response (5s): Wait for response
  - If async (202 Accepted): Poll status endpoint every 1s
- Progress phases:
  - "Connecting to Jira..." (0-1s)
  - "Creating issue..." (1-4s)
  - "Saving link..." (4-5s)
- **Output**: Loading state in PublishButton component
- **Acceptance**: Loading indicators show during publish, accurate status messages

`[ ]` **46. Implement Publishing Success Feedback** (2 hours, depends: 45)
- On successful publish:
  - Show success toast (5s, dismissible): "✓ Published to Jira: PROJ-123"
  - Replace button with Jira badge (task 44)
  - Update file list (re-fetch or optimistic update)
- **Output**: Success handling in PublishButton component
- **Acceptance**: Success toast appears, badge visible immediately

`[ ]` **47. Implement Publishing Error Handling** (6 hours, depends: 45)
- Error display strategy (from spec):
  - **Not configured (401)**: Toast (persistent) + link to settings
  - **Token expired (401)**: Banner (persistent) + "Reconnect" button
  - **Permission denied (403)**: Modal (blocking) + "Contact admin" guidance
  - **Rate limited (429)**: Toast (auto-dismiss) + countdown "Retry in 47 minutes"
  - **MCP server down (503)**: Toast (5s) + "Retry" button
  - **File too large (400)**: Modal + guidance to summarize
- Error messages match backend responses (FR-014 to FR-020)
- **Output**: Error handling in PublishButton component
- **Acceptance**: All error scenarios (task 22) display correct UI feedback

### Re-Publish Warning UI (Day 5)

`[ ]` **48. Implement Re-Publish Warning Modal** (4 hours, depends: 34, 47)
- If file already has jiraLinks, backend returns `{warning: true, existingLinks}`
- Show modal:
  - Title: "File Already Published"
  - Body: "This file is already linked to PROJ-123. Publishing again will create a new Jira issue."
  - Actions: [Cancel] [Create New Issue]
- If user cancels: Close modal, no action
- If user confirms: Call publish endpoint again with `force=true` param
- **Output**: RePublishWarningModal component
- **Acceptance**: Warning shows on re-publish attempt, creates new issue on confirm

### Actions Menu for Published Files (Day 6)

`[ ]` **49. Add Three-Dot Actions Menu to Published Files** (4 hours, depends: 44)
- For files with Jira badge, add three-dot menu icon (⋮)
- Menu options:
  - "View in Jira" → Opens issue in new tab
  - "Re-publish" → Shows warning modal (task 48)
  - "Unlink" → Removes jiraLinks entry (not implemented in MVP, show "Coming soon")
- Menu positioning: Dropdown, right-aligned
- **Output**: FileActionsMenu component
- **Acceptance**: Menu appears for published files, "View" and "Re-publish" work

### Accessibility & Polish (Days 7-8)

`[ ]` **50. Implement ARIA Live Regions for Status Updates** (3 hours, depends: 45, 46, 47)
- Add `role="status"` or `role="alert"` to dynamic status messages
- Screen reader announces:
  - "Publishing to Jira" (when loading starts)
  - "Published successfully to Jira issue PROJ-123" (on success)
  - "Error: Jira integration not configured" (on error)
- Live region for connection status changes in settings
- **Output**: Accessibility attributes in PublishButton, JiraBadge, ConnectionStatus
- **Acceptance**: Screen reader testing validates announcements

`[ ]` **51. Ensure Keyboard Navigation for All Interactive Elements** (4 hours, depends: 39-49)
- Tab order: OAuth form fields → buttons → file list → publish buttons → Jira badges
- Focus indicators: Visible outline on all interactive elements
- Focus management: Return focus to trigger button after modal close
- Enter/Space key support: Activate buttons and links
- Escape key: Close modals and menus
- **Output**: Keyboard event handlers + CSS focus styles
- **Acceptance**: Full workflow navigable with keyboard only

`[ ]` **52. Validate WCAG 2.1 AA Compliance** (6 hours, depends: 39-51)
- Color contrast: 4.5:1 for text, 3:1 for interactive elements
- Text sizing: All text resizable to 200% without loss of function
- Interactive target size: Minimum 44×44 pixels for buttons
- External link indication: Visually-hidden text or ARIA label for Jira badges
- Error identification: Errors announced to screen readers
- **Output**: Accessibility audit report + fixes
- **Acceptance**: WAVE, axe DevTools pass with zero violations

---

## Phase 4: Security & Operations (4-5 days, can overlap with Phase 2-3)

**Goal**: Security review, operational readiness, monitoring setup

**Prerequisites**: Backend implementation (Phase 2) in progress or complete

### Security Review

`[P]` **53. Conduct OAuth Token Storage Security Audit** (1 day, depends: 28)
- Review Kubernetes Secret encryption at rest
- Validate RBAC policy: Only `vteam-backend` service account has read access
- Test unauthorized access: Non-privileged user cannot read Secret
- Review token rotation policy: How often refresh tokens are renewed
- Validate TLS 1.2+ for Atlassian OAuth communication
- **Output**: Security audit report with findings + remediation
- **Acceptance**: Zero critical findings, medium findings have mitigation plan

`[P]` **54. Penetration Testing of OAuth Flow** (1 day, depends: 29)
- Test CSRF protection: Validate `state` parameter prevents CSRF
- Test authorization code interception: Ensure code can't be reused
- Test redirect URI validation: Prevent open redirect vulnerabilities
- Test token leakage: Ensure tokens not logged or exposed in responses
- **Output**: Penetration test report + fixes
- **Acceptance**: Zero high-severity findings

### Observability & Monitoring

`[P]` **55. Implement MCP Server Health Metrics** (6 hours, depends: 26)
- Prometheus metrics:
  - `jira_mcp_requests_total` (counter, labels: method, status)
  - `jira_mcp_request_duration_seconds` (histogram, labels: method)
  - `jira_mcp_connection_status` (gauge, values: 0=down, 1=up)
  - `jira_oauth_token_expiry_seconds` (gauge, time until expiry)
- Expose metrics endpoint: `GET /metrics`
- **Output**: Metrics in `internal/integrations/mcp/metrics.go`
- **Acceptance**: Metrics visible in Prometheus, graphs in Grafana

`[P]` **56. Create Operational Runbook** (1 day, depends: 26, 30)
- Runbook sections:
  1. **MCP Server Down**: Check MCP sidecar health, restart if needed, escalate if persists >15 min
  2. **OAuth Token Expired Across All Users**: Check global secret, validate refresh token, re-authenticate if revoked
  3. **Rate Limit Spike**: Identify high-volume users, investigate automation/bots, consider queuing (Phase 2)
  4. **Permission Denied Errors**: Validate Jira user permissions, check project RBAC
  5. **Incident Response**: On-call escalation, user communication template
- **Output**: `docs/operations/jira-integration-runbook.md`
- **Acceptance**: Runbook reviewed by SRE team, added to on-call playbooks

`[P]` **57. Set Up Monitoring Dashboards** (6 hours, depends: 55)
- Grafana dashboard: "Jira Integration Health"
  - MCP server uptime (last 24h, 7d, 30d)
  - Request latency (p50, p95, p99)
  - Error rate by type (auth, permission, rate limit, network)
  - OAuth token health (expiry countdown, refresh success rate)
  - Publishing volume (issues created per day)
- **Output**: Grafana dashboard JSON + import instructions
- **Acceptance**: Dashboard deployed to production Grafana

`[P]` **58. Configure Alerting Rules** (4 hours, depends: 55)
- Alerts:
  - **MCP Server Down**: Trigger if `jira_mcp_connection_status == 0` for >5 minutes
  - **High Error Rate**: Trigger if error rate >10% for >15 minutes
  - **Token Expiring Soon**: Trigger if `jira_oauth_token_expiry_seconds < 7 days`
  - **Rate Limit Approaching**: Trigger if >800 requests in past hour (80% of 1,000 limit)
- Notification channels: PagerDuty (critical), Slack (warning)
- **Output**: Prometheus alerting rules YAML
- **Acceptance**: Test alerts fire correctly, on-call receives notifications

---

## Phase 5: Documentation & Enablement (5-6 days)

**Goal**: Comprehensive documentation for users, admins, and developers

**Prerequisites**: Implementation complete (Phases 2-3), operations setup (Phase 4)

### Admin Documentation

`[ ]` **59. Write Admin Setup Guide** (1 day, depends: 29, 39)
- Sections:
  1. **Prerequisites**: Atlassian Cloud account, OAuth app creation access
  2. **Create OAuth App in Atlassian Developer Console** (step-by-step with screenshots):
     - Navigate to https://developer.atlassian.com
     - Create new OAuth 2.0 app
     - Configure redirect URI: `https://vteam.example.com/api/v1/jira/oauth/callback`
     - Set scopes: `write:jira-work`, `read:jira-work`
     - Copy Client ID and Client Secret
  3. **Configure vTeam**: Enter credentials in Project Settings
  4. **Test Connection**: Validate setup works
  5. **Troubleshooting**: Common issues (redirect URI mismatch, missing scopes, network errors)
- **Output**: `docs/admin/jira-integration-setup.md` + screenshots
- **Acceptance**: Admin can follow guide and complete setup in <10 minutes

`[ ]` **60. Write Migration Guide** (6 hours, depends: 59)
- Audience: Admins migrating from old custom Jira integration
- Sections:
  1. **What Changed**: API tokens → OAuth, REST API → MCP Server
  2. **Migration Steps**:
     - Export existing Jira links (if migration needed)
     - Remove old Jira configuration
     - Configure new OAuth integration (link to task 59 guide)
     - Test publishing
  3. **Known Limitations**: Links from old integration not auto-migrated (manual re-publish if needed)
  4. **Rollback Plan**: If issues, how to revert to old integration (if still available)
- **Output**: `docs/admin/jira-migration-guide.md`
- **Acceptance**: Migration steps validated with test instance

### User Documentation

`[ ]` **61. Write User Guide for Publishing Workflow** (1 day, depends: 43-48)
- Sections:
  1. **Overview**: What Jira integration does, when to use it
  2. **Publishing a File**:
     - Navigate to RFE workflow
     - Click "Publish to Jira" next to file
     - Wait for confirmation
     - Click Jira badge to view issue
  3. **Re-Publishing**: What happens when publishing file again (warning modal)
  4. **Viewing Jira Issues**: How to navigate from vTeam to Jira
  5. **Troubleshooting**:
     - "Jira integration not configured" → Link to admin
     - "Connection expired" → Re-authenticate
     - "Permission denied" → Contact Jira admin
     - "Rate limit" → Wait and retry
  6. **FAQ**: Common questions
- **Output**: `docs/user/jira-integration-guide.md` + screenshots
- **Acceptance**: User can complete first publish by following guide

`[ ]` **62. Create Video Walkthrough** (1 day, optional, depends: 61)
- Target audience: Non-technical Product Managers
- Video content:
  1. OAuth setup (admin perspective) - 3 minutes
  2. Publishing workflow (user perspective) - 2 minutes
  3. Error recovery (common issues) - 2 minutes
- **Output**: Video hosted on YouTube/Vimeo + embedded in docs
- **Acceptance**: Video <10 minutes, covers end-to-end flow
- **Optional for MVP**: Defer to Phase 2 if resource-constrained (LOW priority clarification #19)

### Developer Documentation

`[ ]` **63. Update Backend API Documentation** (6 hours, depends: 16, 17)
- Document all new endpoints:
  - OAuth endpoints (config, authorize, callback, test, status, disconnect)
  - Publishing endpoints (publish file, get jira-links)
- Include OpenAPI spec from tasks 16-17
- Add example requests/responses with curl
- Document error codes and messages
- **Output**: `docs/api/jira-integration-api.md` + OpenAPI spec in `docs/api/openapi.yaml`
- **Acceptance**: Developers can integrate with API using docs alone

`[ ]` **64. Document Data Model Changes** (3 hours, depends: 13)
- Document RFEWorkflow CR schema update:
  - `jiraLinks` field structure
  - Validation rules
  - Example YAML
- Document OAuth Secret schema
- **Output**: `docs/architecture/jira-data-model.md`
- **Acceptance**: Developers understand CR changes for future features

`[ ]` **65. Update Architecture Diagrams** (6 hours, depends: 24, 29, 33)
- Create architecture diagram showing:
  - Frontend → Go Backend → Python MCP Sidecar → Atlassian MCP Server → Jira Cloud
  - OAuth flow sequence diagram
  - Publishing flow sequence diagram
  - Data model diagram (RFEWorkflow CR + jiraLinks)
- **Output**: `docs/architecture/jira-integration-architecture.md` + diagrams (Mermaid or PlantUML)
- **Acceptance**: New engineers can understand architecture from diagrams

### Troubleshooting & Support

`[ ]` **66. Write Troubleshooting Guide** (1 day, depends: 35, 47)
- Common issues:
  1. **OAuth setup fails**: Check redirect URI, scopes, client credentials
  2. **Token expires repeatedly**: Check refresh token validity, network connectivity
  3. **Permission denied**: Validate Jira user permissions, project access
  4. **Rate limit errors**: Explain Atlassian limits, suggest throttling
  5. **MCP server unreachable**: Check sidecar health, network policies, firewall rules
  6. **File too large**: Guidance on summarizing or splitting specs
  7. **Jira issue not created**: Check MCP logs, Atlassian status page
- For each issue: Symptom → Diagnosis → Resolution → Escalation path
- **Output**: `docs/troubleshooting/jira-integration.md`
- **Acceptance**: Support team can resolve 80% of issues using guide

`[ ]` **67. Create Support Ticket Templates** (2 hours, depends: 66)
- Templates for common issues:
  - OAuth configuration assistance
  - Token expiration investigation
  - Permission troubleshooting
  - MCP server connectivity debugging
- Include: Expected info from user, diagnostic commands, log locations
- **Output**: Support ticket templates in Zendesk/Jira Service Desk
- **Acceptance**: Support team uses templates for Jira integration tickets

### Quickstart Validation

`[ ]` **68. Execute Quickstart End-to-End Test** (4 hours, depends: 23, all implementation)
- Run quickstart.md script (from task 23) against production-like environment
- Validate:
  1. OAuth configuration succeeds
  2. File publishes to Jira
  3. Issue created with correct content
  4. Link appears in vTeam UI
  5. Link opens correct Jira issue
  6. Re-publish shows warning
  7. Error scenarios display correct messages
- **Output**: Quickstart test report (pass/fail + screenshots)
- **Acceptance**: All quickstart steps pass, no manual intervention needed

---

## Phase 6: Final Validation & Deployment (3-4 days)

**Goal**: Production readiness validation, deployment, post-launch monitoring

**Prerequisites**: All phases complete, documentation published

### Pre-Production Testing

`[ ]` **69. Run Full Regression Test Suite** (1 day, depends: 68)
- Execute all tests:
  - Unit tests (tasks 15, 25, 27, 28, 30, 36)
  - Contract tests (tasks 18, 19)
  - Integration tests (tasks 20, 21, 22, 27)
  - End-to-end tests (tasks 20, 21, 68)
  - Accessibility tests (task 52)
- Validate test coverage: >80% for new code
- **Output**: Test report with pass/fail counts + coverage metrics
- **Acceptance**: All tests pass, coverage >80%

`[ ]` **70. Conduct UAT with Product Managers** (1 day, depends: 69)
- Recruit 3-5 Product Managers for user acceptance testing
- Test scenarios:
  1. First-time OAuth setup
  2. Publish single file to Jira
  3. Re-publish same file (warning modal)
  4. Navigate to Jira issue from badge
  5. Recover from token expiration
  6. Publish file without permissions (error handling)
- Collect feedback: Usability, clarity of errors, performance
- **Output**: UAT report with findings + action items
- **Acceptance**: >8/10 average satisfaction score, zero critical issues

`[ ]` **71. Performance & Load Testing** (1 day, depends: 69)
- Test scenarios:
  - Concurrent publishing: 10 users publish simultaneously → <5s latency
  - Rate limit handling: Approach 1,000 req/hour → graceful degradation
  - OAuth token refresh under load: 100 requests during token refresh → no failures
  - MCP server failure: Sidecar down → clear error messages, no crashes
- **Output**: Performance test report (latency percentiles, error rates)
- **Acceptance**: P95 latency <5s, error rate <1%, no crashes under load

### Deployment

`[ ]` **72. Deploy MCP Sidecar Service** (4 hours, depends: 71)
- Build Docker image for Python MCP sidecar (task 24)
- Deploy to Kubernetes: Deployment + Service + ConfigMap
- Configure resource limits: CPU, memory
- Health checks: liveness probe, readiness probe
- **Output**: Kubernetes manifests in `k8s/mcp-sidecar/`
- **Acceptance**: Sidecar pods running, health checks passing

`[ ]` **73. Deploy Backend API Changes** (4 hours, depends: 72)
- Deploy Go backend with new OAuth + publishing endpoints
- Update RFEWorkflow CRD (add jiraLinks field)
- Create Kubernetes Secret (empty, will be populated by users)
- Configure RBAC policy for Secret access
- **Output**: Backend deployment + CRD update + Secret + RBAC
- **Acceptance**: Backend pods running, API endpoints responding

`[ ]` **74. Deploy Frontend UI Changes** (3 hours, depends: 73)
- Build frontend with OAuth settings page + publishing UI
- Deploy to CDN/static hosting
- Update environment config: Backend API URLs
- **Output**: Frontend deployment
- **Acceptance**: UI loads, OAuth settings accessible, publish button visible

`[ ]` **75. Smoke Test in Production** (2 hours, depends: 74)
- Test with real Atlassian account:
  1. Configure OAuth in production vTeam
  2. Publish test file
  3. Verify Jira issue created
  4. Verify link appears and works
  5. Test error scenario (invalid config)
- **Output**: Production smoke test report
- **Acceptance**: End-to-end flow works in production

### Post-Launch

`[ ]` **76. Monitor Key Metrics for 48 Hours** (2 days, depends: 75)
- Monitor (from task 57):
  - MCP server uptime
  - Publishing success rate
  - OAuth token refresh success rate
  - Error rate by type
  - User adoption (% of PMs configuring Jira)
- Respond to alerts, triage issues
- **Output**: Post-launch monitoring report
- **Acceptance**: Uptime >99%, success rate >95%, no critical incidents

`[ ]` **77. Conduct Retrospective** (2 hours, depends: 76)
- Team retrospective:
  - What went well?
  - What didn't go well?
  - What did we learn?
  - Action items for next feature
- Document lessons learned
- **Output**: Retrospective notes + action items
- **Acceptance**: Retrospective completed, action items assigned

---

## Summary

**Total Tasks**: 77 tasks across 6 phases

**Estimated Timeline**:
- **Phase 0** (Discovery): 5-7 days (CRITICAL PATH)
- **Phase 1** (Design): 3-4 days (CRITICAL PATH)
- **Phase 2** (Backend): 8-10 days (CRITICAL PATH)
- **Phase 3** (Frontend): 6-8 days (overlaps with Phase 2)
- **Phase 4** (Security): 4-5 days (overlaps with Phases 2-3)
- **Phase 5** (Documentation): 5-6 days (after implementation)
- **Phase 6** (Validation): 3-4 days (CRITICAL PATH)

**Critical Path Duration**: 19-25 days (4-5 weeks)
**Total Calendar Duration**: 25-35 days (5-7 weeks with parallelization)

**Key Milestones**:
- ✅ Week 1 End: Research complete, all NEEDS CLARIFICATION resolved
- ✅ Week 2 End: Contracts defined, failing tests exist
- ✅ Week 3-4 End: Backend + Frontend implementation complete
- ✅ Week 5 End: Documentation complete, UAT passed
- ✅ Week 6 End: Production deployment, monitoring stable

**Critical Success Factors**:
1. **Task 1-2**: MCP client library exists and works (SHOWSTOPPER if missing)
2. **Task 3**: Python-Go bridge decision made by Day 4
3. **Task 12**: Spec updated with zero NEEDS CLARIFICATION before Phase 1
4. **Task 18-19**: Contract tests define clear API before implementation
5. **Task 75**: Production smoke test passes before launch announcement

**Risk Mitigation**:
- Tasks 1-12 (Discovery) MUST complete before any implementation → prevents rework
- Contract tests (18-19) written before implementation → prevents API drift
- Integration tests (20-22) validate scenarios from spec → ensures requirements met
- Parallel execution where possible → reduces calendar time
- Post-launch monitoring (76) → early detection of production issues

**Next Steps**:
1. Review this task list with team
2. Assign owners to Phase 0 tasks (1-12)
3. Begin execution starting with task 1 (MCP client validation)
4. Daily standups to track progress and unblock
5. Update task statuses in this document as work progresses

---

*Generated from spec.md on 2025-10-15. This task list is ready for execution.*
