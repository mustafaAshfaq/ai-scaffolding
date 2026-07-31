# OverGoods Modernization Migration Plan

## Document Purpose

This plan defines the modernization approach for the OverGoods warehouse-operations application. It is based on the current architecture blueprint, routing map, and identified business priorities.

## Business Drivers

1. Improve warehouse-operator experience.
2. Replace legacy ASP.NET WebForms and .NET Framework 4.6.2 technology.
3. Deliver a React user interface and .NET 10 backend.
4. Preserve operational continuity during migration.
5. Reduce reliance on fragile local COM-based printing over time.

## Current-State Summary

### Application Architecture

- Legacy ASP.NET WebForms application hosted in IIS.
- Approximately 100 `.aspx` pages with code-behind.
- Physical file-path routing; there is no custom route table.
- Primary navigation flow:

  ```text
  Azure SSO / authentication
    → login.aspx
    → default.aspx (subsidiary selection)
    → mainmenu.aspx
  ```text

- Shared C# and VB.NET libraries contain inventory, returns, user, warehouse, printing, product lookup, and integration logic.
- Legacy ASMX endpoints:
  - `OGSService.asmx`
  - `ProductCodeProcessor.asmx`
- SQL Server with stored-procedure-centric data access.
- Primary database: `Overgoods`.
- Additional database: `ARES_SEARCH`; its ownership and usage require documentation.
- Azure AD OAuth/OpenID/MSAL authentication combined with SQL-backed ASP.NET Session state.
- Integrations include Google Cloud Storage, external tracking/eTT/SCS/partner APIs, ActiveMQ, file outputs, and local printing.
- Existing local print path includes a WCF/browser print host and legacy COM components.
- Automated tests are not evident in the main legacy solution.

### Priority Business Domains

1. Inventory
2. Returns
3. Product lookup and classification
4. User and warehouse administration

### Immediate Warehouse-User Pain Points

1. Photo capture and management
2. Ticket printing

### Operator Equipment

- Desktop browsers
- Mobile/handheld devices
- Barcode scanners
- Cameras
- Label printers

---

# Target Architecture

## Architectural Principles

1. Use a phased strangler migration, not a big-bang replacement.
2. Build by business workflow and domain, not by legacy pages or technical layers.
3. Use a modular .NET 10 backend before introducing microservices.
4. Keep SQL Server and critical stored procedures initially; wrap them behind tested adapters.
5. Make all new APIs stateless.
6. Do not expose SQL Server or stored procedures directly to React.
7. Use feature flags and controlled rollout for every migrated workflow.
8. Keep the legacy COM print path only as a temporary compatibility boundary.
9. Use shared print queues as the preferred modern printing path where supported.
10. Add automated tests, observability, security, auditability, and rollback before production cutover.

## Target Logical Architecture

```text
React Warehouse Application
  ├─ Inventory workspace
  ├─ Returns workspace
  ├─ Product lookup and classification workspace
  └─ Administration workspace
              |
              v
.NET 10 Backend / API
  ├─ Identity and authorization
  ├─ Inventory module
  ├─ Media / photo module
  ├─ Print-job module
  ├─ Returns module
  ├─ Product classification module
  ├─ User and warehouse administration module
  ├─ Reporting/export module
  └─ Integration orchestration
              |
              v
Infrastructure Adapters and Workers
  ├─ SQL Server stored-procedure adapters
  ├─ Azure AD adapter
  ├─ Google Cloud Storage adapter
  ├─ External API adapters
  ├─ ActiveMQ/outbox worker
  ├─ Warehouse alignment file worker
  ├─ Shared print queue adapter
  └─ Legacy COM print compatibility gateway
```text

## React Route Model

The new application should follow user tasks rather than mirror legacy WebForms page names.

```text
/app/inventory
/app/inventory/receive
/app/inventory/search
/app/inventory/items/:itemId
/app/inventory/items/:itemId/photos
/app/inventory/items/:itemId/activity
/app/inventory/disposition
/app/inventory/quality-audit
/app/inventory/overages
/app/returns
/app/returns/:returnId
/app/product-lookup
/app/classification
/app/admin/users
/app/admin/roles
/app/admin/warehouses
/app/admin/locations
```text

---

# Migration Strategy

## Coexistence Model

The legacy WebForms application and the modern React/.NET 10 application will operate in parallel while workflows are migrated.

For every workflow:

1. Document current behavior and dependencies.
2. Add characterization tests for critical legacy behavior.
3. Build the React/.NET 10 replacement.
4. Release it behind a feature flag.
5. Pilot with selected users, roles, warehouses, or subsidiaries.
6. Measure performance, usability, reliability, and support outcomes.
7. Retain the legacy route during an agreed rollback period.
8. Retire the legacy route only after business and operational approval.

## Deployment Approach

Initially deploy the .NET 10 backend as a modular application, or a small set of tightly related deployables.

Separate modules into independent services only when there is a demonstrated need for:

- independent scaling
- independent release schedules
- clear data ownership
- operational isolation
- distinct reliability requirements
- separate team ownership

---

# Phased Migration Plan

## Phase 0 — Governance, Discovery, and Baseline

### Objectives

- Establish agreed product, technical, security, data, and operational decisions.
- Identify the first workflow to migrate.
- Establish measurable user-experience and operational baselines.

### Activities

#### Workflow Inventory

For Inventory, Returns, Product Lookup/Classification, and Administration, document:

- user roles
- warehouse and subsidiary context
- workflow trigger and expected outcome
- happy path and exception paths
- current WebForms pages and redirects
- ASMX calls
- shared-library methods
- stored procedures
- external APIs and file/queue integrations
- reporting/export dependencies
- photo and print dependencies
- audit and compliance requirements
- business and technical owners

#### Warehouse-User Research

Observe operators performing real work, focusing on:

- barcode-scanner usage
- desktop versus mobile/handheld usage
- camera/photo capture process
- number of page transitions per task
- manual data re-entry
- printing delays and failed-print recovery
- printer/queue selection
- network/connectivity limitations
- common workarounds and support issues

#### Required Architecture Decisions

Create and approve Architecture Decision Records for:

1. React and .NET 10 target architecture.
2. Modular domain architecture versus microservices.
3. Authentication and authorization approach.
4. Coexistence, routing, feature-flag, rollback, and release model.
5. SQL Server and stored-procedure transition approach.
6. Photo storage and Google Cloud Storage integration approach.
7. Print queue target architecture and COM compatibility strategy.
8. VB.NET migration strategy: port, wrap temporarily, or retain in a time-bound compatibility component.
9. Observability, logging, alerting, and audit standards.
10. Data ownership model, including `ARES_SEARCH`.

### Deliverables

- Approved modernization charter
- Workflow inventory
- Dependency map
- Integration inventory
- Stored-procedure/database map
- Print and photo discovery report
- UX and operational baseline
- Target architecture decisions
- Release governance model

### Exit Criteria

- The first migration slice is approved.
- Product, technical, operational, security, and data owners are identified.
- Coexistence and rollback approach is approved.
- Domain boundaries and API contracts are required before new implementation begins.

---

## Phase 1 — Modern Platform Foundation

### Objectives

Build the secure and observable React/.NET 10 platform needed for safe workflow migration.

### React Foundation

- Application shell and navigation.
- Responsive desktop and mobile layouts.
- Warehouse-friendly keyboard-first interactions.
- Barcode-scanner focus and input handling.
- Shared user, warehouse, and subsidiary context.
- Accessibility and clear error/loading states.
- Feature-flag support.

### .NET 10 Backend Foundation

- .NET 10 API application.
- OpenAPI documentation.
- API versioning and consistent error contracts.
- Azure AD authentication.
- Capability-based authorization.
- Explicit warehouse/subsidiary context.
- Stateless APIs; no new dependency on ASP.NET Session.
- Correlation IDs and audit events.
- Shared validation and error-handling standards.

### Platform Engineering

- CI/CD pipelines.
- Unit, integration, contract, and end-to-end test execution.
- Dependency and security scanning.
- Centralized configuration and secret management.
- Structured logs, metrics, traces, and dashboards.
- Health checks and owned alerts.
- Database migration/versioning process.
- Feature flags and controlled rollout controls.

### Exit Criteria

- React and .NET 10 applications deploy to non-production environments.
- Azure AD authentication and authorization work end to end.
- Logs, metrics, traces, health checks, and alerts are available.
- Feature flags can be targeted by warehouse, subsidiary, role, and user.
- New production code passes required automated quality and security gates.

---

## Phase 2 — First Production Slice: Inventory Photos and Ticket Printing

### Objective

Address the highest-priority warehouse-user pain points while building reusable capabilities for Inventory and Returns.

## 2.1 Photo Capture and Management

### Legacy Scope

Analyze and map:

- `AttachPhoto.aspx`
- `InventoryPhoto.aspx`
- `photos.aspx`
- `Image.aspx`
- `ItemDetails.aspx`
- Existing Google Cloud Storage helper usage
- Related SQL tables and stored procedures

### Modern Workflow

```text
Scan barcode or search for item
  → View item summary
  → Capture or upload photo
  → Confirm photo/item association
  → Upload with visible progress
  → Review saved photo
  → Continue inventory task or request ticket print
```text

### .NET 10 Media Module

Responsibilities:

- Validate operator authorization and warehouse context.
- Validate file type, size, integrity, and upload safety.
- Associate photos with inventory items.
- Store photo metadata and audit events.
- Use a Google Cloud Storage adapter.
- Generate secure, short-lived photo retrieval URLs.
- Support upload retry and clear failure handling.
- Preserve read access to legacy photos during coexistence.

### React Photo Experience

- Mobile camera capture where browser/device support allows.
- Desktop file upload and camera support where available.
- Photo preview before submission.
- Upload progress and failure recovery.
- Barcode-first item selection.
- Clear display of item, warehouse, and photo status.

## 2.2 Ticket Printing

### Legacy Scope

Analyze and map:

- `PrintTickets.aspx`
- `PickTicketHistory.aspx`
- Related ticket/letter print pages where applicable
- Existing local WCF print host
- Legacy COM component responsibilities
- Label templates and required data fields
- Printer models and queue configuration
- Reprint, audit, and error-recovery procedures

### Target Print Architecture

```text
React UI
   |
   | Create and monitor print job
   v
.NET 10 Print Job API
   |
   ├─ Shared print queue adapter
   |      └─ Network printer / label-print queue
   |
   └─ Legacy COM compatibility gateway
          └─ Existing local WCF/COM print implementation
```text

### Print Job API Responsibilities

- Validate user, item, ticket type, and authorization.
- Create a unique, auditable print job.
- Validate ticket data and template version.
- Select an approved destination.
- Track print status:
  - `Queued`
  - `Sent`
  - `Printed`
  - `Failed`
  - `Retrying`
  - `Cancelled`
- Support controlled reprints.
- Record user, timestamp, destination, item, document, and reprint reason.
- Return clear status to the React application.

### Migration Approach

1. Introduce the Print Job API.
2. Prefer shared print-queue integration for supported printers and labels.
3. Keep the COM/WCF route as a temporary fallback.
4. Migrate printers, queues, and locations incrementally.
5. Retire the COM route only after all required printing scenarios are proven.

### Phase 2 Exit Criteria

- Pilot operators can scan/select an item in React.
- Operators can capture/upload and view item photos.
- Operators can submit ticket print jobs.
- Queue printing works for the pilot printer/template.
- COM fallback remains available where required.
- Failed photo uploads and print jobs are visible and recoverable.
- All actions are auditable.
- Legacy workflow remains available for rollback.

---

## Phase 3 — Inventory Workspace

### Objective

Replace fragmented inventory WebForms pages with a complete, operator-centered inventory workspace.

### Candidate Legacy Scope

- `LTRReceive.aspx`
- `ListInventory.aspx`
- `DispositionInventory.aspx`
- `InventoryActivity.aspx`
- `QualityAudit.aspx`
- `OverageDataEntry.aspx`
- `HotListCandidate.aspx`
- `AttachPhoto.aspx`
- `PrintTickets.aspx`
- `ItemDetails.aspx`
- `InventoryPhoto.aspx`

### Suggested Delivery Increments

1. Item search/listing and item details.
2. Photos and ticket printing.
3. Item activity/history.
4. Receiving/LTR workflow.
5. Disposition workflow.
6. Quality-audit workflow.
7. Overage data entry.
8. Hot-list workflow.

### Backend Requirements

The Inventory module should provide:

- explicit commands for state-changing operations
- query APIs for search, lists, details, and history
- centralized validation and business rules
- stored-procedure adapters
- audit history
- concurrency control
- database integration tests
- API contract tests

### Phase 3 Exit Criteria

- Priority inventory workflows operate fully in React/.NET 10.
- Required roles and warehouse context are enforced.
- Photo and print capabilities are integrated.
- Inventory accuracy and operational outcomes meet agreed targets.
- Equivalent legacy routes are retired only after accepted cutover.

---

## Phase 4 — Product Lookup and Classification

### Objective

Replace legacy product lookup and classification with a modern, observable, scanner-friendly workflow.

### Legacy Scope

- `ProductCodeProcessor.asmx`
- `UpcLookup.aspx`
- `knowngoods.aspx`
- `unknownovergoods.aspx`
- `unknowngoods.aspx`
- `CarrierKnowngoods.aspx`
- Product lookup configuration and process sequencing
- Related shared libraries and stored procedures

### Required Discovery

External dependencies are currently unknown. Before implementation, document:

- UPC/NDC/product-data sources
- eTT, SCS, carrier, or other external calls
- source priority and fallback sequence
- request/response contracts
- manual overrides
- classification rules
- approval and audit rules
- latency and outage behavior
- current error handling

### Target Experience

- Barcode/scan-first lookup.
- Clear results and source information.
- Classification confidence or status where applicable.
- Authorized operator override.
- Explicit classification reason and audit event.
- Clear response when an external lookup source is unavailable.
- Reusable API for Inventory and Returns workflows.

### Phase 4 Exit Criteria

- Legacy behavior is characterized and tested.
- New .NET 10 API is observable and resilient.
- React workflow improves lookup speed and reduces rework.
- Legacy ASMX route can be retired for migrated users.

---

## Phase 5 — Returns

### Objective

Modernize returns using the reusable Inventory, Photo, Print, and Product Classification capabilities.

### Discovery Scope

- `ReturnsLibrary`
- `UpdateReturnInfo.aspx`
- `Address.aspx`
- Customer return reports
- Item status and disposition rules
- Tracking and external update dependencies
- Customer and return data rules

### Target Capability

- Return search and details.
- Return intake and updates.
- Address/customer data management.
- Return status transitions.
- Item photo support where required.
- Ticket/letter printing where required.
- External tracking updates with retries and recovery.
- Audit history for all changes.

### Phase 5 Exit Criteria

- A complete priority returns journey is live in React/.NET 10.
- Integration failures are visible, recoverable, and monitored.
- Required print/photo actions are supported.
- Legacy routes for the migrated returns journey are retired.

---

## Phase 6 — User and Warehouse Administration

### Objective

Replace legacy page-level administration with a modern capability-based model.

### Legacy Scope

User management:

- `usermanagement/default.aspx`
- `usermanagement/NewUser.aspx`
- `usermanagement/selectuser.aspx`
- `usermanagement/deleteuser.aspx`
- `usermanagement/pagepermissions.aspx`

Warehouse administration:

- `WarehouseAdd.aspx`
- `UpdateLocations.aspx`
- `WarehouseAlignment.aspx`

### Target Authorization Model

Examples of capabilities:

```text
inventory.read
inventory.receive
inventory.disposition
inventory.photo.capture
inventory.ticket.print
returns.read
returns.update
product.lookup
product.classification.override
warehouse.manage
warehouse.location.manage
user.manage
role.manage
report.export
```text

### Security Requirements

- Rationalize roles before migration.
- Implement audit logging for user, role, warehouse, and location changes.
- Define controls, approvals, and audit records for any manager override behavior.
- Use Azure AD and capability checks for privileged functions.
- Fail closed for privileged actions when identity/token validation fails.

### Phase 6 Exit Criteria

- User, role, warehouse, and location administration operates in React/.NET 10.
- Permissions are capability-based and auditable.
- Legacy page permission management is retired.

---

## Phase 7 — Integrations, Reporting, and Legacy Retirement

## Integration Modernization

| Legacy Integration | Target Direction |
|---|---|
| ActiveMQ / CIpe publishing | Outbox-backed worker, idempotent publishing, retries, queue monitoring, re-drive capability |
| Tracking/claims updates | .NET 10 worker with timeout, retry, circuit breaker, and recovery process |
| Warehouse alignment file generation | Scheduled worker with artifact tracking, re-run ability, and stale-output alerts |
| Google Cloud Storage | Encapsulated storage adapter and secure access model |
| Local COM printing | Temporary compatibility gateway until print queues/mobile alternatives cover requirements |

## Reporting Strategy

Categorize reports before migration:

1. Operational real-time reports
2. Compliance/audit reports
3. Management reports
4. Low-use or obsolete reports

Migrate reports after their owning operational domain. Preserve required CSV/RTF exports while avoiding duplicate report logic in both platforms.

## Legacy Retirement Criteria

Retire a legacy page, ASMX endpoint, library method, stored procedure, or COM path only when:

- all relevant users are migrated
- replacement behavior meets acceptance criteria
- reports and exports remain available
- audit and support requirements are met
- dependencies are removed or redirected
- the rollback period has ended
- operational ownership approves retirement
- code, configurations, secrets, and infrastructure are removed

---

# Data Strategy

## Initial Data Approach

Retain SQL Server and existing stored procedures initially.

The primary goal is to replace unsupported UI and application technology safely. Database modernization should be incremental and driven by business value, ownership, and testability.

## Data Rules

- React must not access SQL Server directly.
- .NET 10 modules access data through domain-owned adapters.
- Stored procedures are wrapped behind explicit interfaces.
- Critical stored procedures receive integration tests.
- Schema changes are versioned and automated.
- Do not introduce new shared-database writes without data-ownership approval.
- Avoid distributed transactions across modules/services.
- Use idempotency and outbox/event patterns for asynchronous cross-boundary workflows.

## Required Data Ownership Work

Create an ownership matrix for:

- inventory
- returns
- user and role data
- warehouse and location data
- photo metadata
- print-job history
- classification rules/results
- reports
- `ARES_SEARCH`

---

# Security, Reliability, and Observability

## Security

- Azure AD remains the identity provider.
- New APIs use modern .NET authentication and authorization middleware.
- New APIs are stateless and do not rely on ASP.NET Session.
- Warehouse/subsidiary context is explicit and validated.
- Secrets are centrally managed and rotated.
- File uploads are validated and access-controlled.
- Photos use secure, short-lived access URLs.
- Privileged changes, classifications, and reprints are audited.
- External integrations and print gateways are threat-modeled.

## Reliability

- Bounded timeouts and retries for SQL Server and external calls.
- Circuit breakers for unreliable external APIs.
- Dead-letter/re-drive support for asynchronous work.
- Clear operator recovery for photo and print failures.
- Health checks for APIs, workers, queues, storage, and print gateways.
- Runbooks for print, photo, integration, and queue failures.

## Observability

Use structured logs and correlation IDs across React, APIs, workers, and integrations.

Track:

- photo upload success/failure rate and duration
- print-job success/failure/retry rate and duration
- reprint rate
- lookup latency/failure rate
- inventory workflow completion time
- external API error rate
- ActiveMQ/worker queue backlog
- database timeouts and errors

Provide dashboards, alerts, and named operational owners.

---

# Testing Strategy

## Legacy Characterization Testing

Before replacing a workflow, capture current behavior for:

- validation rules
- business rules
- stored-procedure results
- permissions
- error cases
- external contracts
- print outputs where practical
- reports/exports where required

## New-System Testing

| Test Type | Purpose |
|---|---|
| Unit tests | Business rules, validation, permissions, classification logic |
| Integration tests | SQL stored procedures, GCS, queues, print adapters, external APIs |
| Contract tests | API contracts and external dependency compatibility |
| End-to-end tests | Scan → photo → print, inventory changes, returns workflows |
| Performance tests | Search, lookup, upload, and print-job creation |
| Security tests | Authentication, authorization, upload security, secrets, audit controls |
| User acceptance tests | Real warehouse devices, printers, queues, and operating conditions |

All new production functionality must pass automated tests in CI.

---

# Release and Rollback Model

## Rollout Controls

Release by:

- warehouse
- subsidiary
- user role
- named pilot user
- feature flag

## Cutover Process

1. Baseline current metrics.
2. Enable workflow for pilot users.
3. Monitor task time, errors, photo failures, print failures, and feedback.
4. Expand in controlled cohorts.
5. Retain the legacy workflow during the rollback period.
6. Disable the feature flag if acceptance thresholds are breached.
7. Retire the legacy workflow after business and operational sign-off.

## Rollback Triggers

Define per-release thresholds, including:

- excessive print failures
- excessive photo upload failures
- inventory data mismatch
- slower operator completion time
- critical integration failure
- authorization/security defect
- unexpected support volume

---

# Success Measures

## Warehouse Experience

- Reduced time to capture and associate item photos.
- Reduced time to request and receive ticket printing.
- Fewer screens and page transitions per warehouse task.
- Less manual data entry and re-entry.
- Improved first-attempt photo-upload success.
- Improved first-attempt ticket-print success.
- Reduced print/photo support tickets.
- Positive pilot-user feedback.

## Technical Modernization

- Increasing proportion of workflows running on React/.NET 10.
- Decreasing WebForms and ASMX usage.
- Decreasing session-dependent workflow usage.
- Automated test coverage for migrated workflows.
- Feature-flagged release and rollback support.
- Structured observability for every migrated capability.
- Progressive reduction of COM/local-print dependency.

## Initial Operational Targets

These must be confirmed with production telemetry:

- Interactive inventory/search operations: p95 under 2 seconds.
- Print-job request acknowledgement: p95 under 2 seconds.
- Operator-critical workflow availability: 99.9% monthly target.
- Photo upload performance: defined based on image size and warehouse network conditions.
- Photo and print failures: visible, recoverable, and auditable.

---

# Immediate Next Actions

1. Approve **Inventory Photos and Ticket Printing** as the first migration slice.
2. Identify a pilot warehouse, pilot roles, and pilot users.
3. Conduct warehouse observation sessions on desktop and mobile/handheld devices.
4. Inventory all photo dependencies:
   - pages
   - SQL tables/stored procedures
   - GCS storage paths and metadata
   - roles and audit rules
5. Inventory all print dependencies:
   - ticket types
   - label templates
   - printer models
   - queues
   - COM responsibilities
   - WCF/local print host behavior
   - reprint and failure procedures
6. Confirm the handheld-printing model:
   - mobile printer
   - handheld device submitting to printer
   - shared queue selected from handheld
7. Create and approve Phase 0 architecture decisions.
8. Build the React/.NET 10 platform foundation.
9. Create the detailed backlog for the first flow:

   ```text
   Scan item
     → view item
     → capture/upload photo
     → create ticket print job
     → monitor/retry print
   ```text
