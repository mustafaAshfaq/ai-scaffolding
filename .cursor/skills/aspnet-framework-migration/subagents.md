# Sub-agent Definitions

Copy prompts into Task tool calls. Replace `{{SOLUTION_PATH}}`, `{{AREA}}`, `{{PAGE}}`, `{{INTERACTION}}` before launching.

## entry-point-agent

```
You are the entry-point discovery agent for an ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}

Find ALL application entry points:
1. Global.asax / Application_Start, route registration
2. web.config: defaultDocument, authentication mode, httpModules, httpHandlers
3. Default.aspx, Login.aspx, RouteConfig, WebApiConfig
4. ASHX (*.ashx), ASMX (*.asmx), WCF (*.svc), Web API controllers
5. IIS Express / launchSettings if present
6. Scheduled jobs, timers in Application_Start

Return:
- Primary user entry (URL + file)
- Secondary entries (APIs, handlers, jobs)
- Startup pipeline order
- Auth gate: what blocks anonymous access
```

## structure-agent

```
You are the project structure agent for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}

Inventory:
- All .csproj / .vbproj: target framework, project type (Web Application, Class Library)
- Folder layout: App_Code, App_Data, bin, Areas, Controllers, Views, *.aspx locations
- Code-behind pattern: *.aspx.cs, partial classes, designer files
- Master pages, user controls (.ascx), themes, App_GlobalResources
- Shared libraries referenced

Return a project inventory table: Project | Type | Framework | Key folders | Notes
```

## dependency-agent

```
You are the dependency analysis agent for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}

Analyze:
- packages.config / PackageReference and versions
- System.Web and System.Web.* usage
- COM interop, P/Invoke, GAC references
- Third-party DLLs in bin/ without source
- Known blockers for .NET Core/ASP.NET Core (BinaryFormatter, Remoting, etc.)

Return dependency matrix: Package/DLL | Used by | .NET compat | Migration action
Flag HIGH risk items.
```

## data-agent

```
You are the data access agent for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}

Map:
- Connection strings (web.config, app.config)
- ADO.NET (SqlConnection, SqlCommand), EF / EF6, NHibernate, Dapper
- Stored procedures, inline SQL, dynamic SQL
- Transaction patterns, Unit of Work
- DataSets, DataTables usage

Return data access map: Component | Pattern | DB objects | Migration notes
```

## auth-agent

```
You are the authentication/authorization agent for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}

Document:
- web.config <authentication>, <authorization>, <membership>, <roleManager>
- Custom modules, filters, base page auth checks
- Login flow, session establishment, timeout, sliding expiration
- Role/claim checks in code (IsInRole, PrincipalPermission, custom)
- Impersonation, Windows auth

Return security model: Mechanism | Config/code location | Pages affected | .NET Identity mapping
```

## page-scanner-agent

```
You are the page interaction scanner for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}
Area/folder: {{AREA}}

For every .aspx, .ascx, .cshtml, .vbhtml in this area:

1. List navigation elements (links, menus, redirects)
2. List interaction elements (buttons, inputs, grids, file uploads)
3. For each: ID, type, trigger event, handler name (from markup or code-behind)
4. Note AJAX: UpdatePanel, ScriptManager, PageMethods, jQuery ajax calls
5. Note master page and user control composition

Return interaction inventory table per page. Do not trace execution yet.
```

## trace-agent

```
You are the execution trace agent for ASP.NET Framework migration.

Solution path: {{SOLUTION_PATH}}
Page: {{PAGE}}
Interaction: {{INTERACTION}}

Trace this user action end-to-end until HTTP response returns:

1. UI trigger (postback, AJAX, link)
2. Page lifecycle if WebForms: Init → Load → Event → PreRender → Render
3. Code-behind handler or controller action
4. Call chain: services, repositories, helpers
5. Data access, external APIs, file I/O, messaging
6. Response: redirect, JSON, HTML render, ViewState changes
7. Side effects: session, cache, cookies, logs

Return structured trace with numbered call chain. Flag unknown/unresolved calls.
Reference tracing-workflow.md patterns for WebForms postback and ViewState.
```

## diagram-agent

```
You are the diagram synthesis agent for ASP.NET Framework migration.

Input: merged navigation map, interaction map, and execution traces.

Produce:
1. Swimlane diagrams (Mermaid) for top 5 critical flows
2. Sequence diagram catalog with priority (High/Medium/Low)
3. One Mermaid sequence diagram per High-priority flow

Use actors/lanes: User, Browser, Page, Code-behind, Service, Data, DB, External.
Follow diagram-templates.md format.
```

## roadmap-agent

```
You are the migration roadmap agent for ASP.NET Framework migration.

Input: discovery report, interaction map, architecture blueprint, diagram catalog.

Produce:
1. Phased migration roadmap (strangler fig preferred)
2. Work packages with dependencies
3. Risk register from dependency-agent HIGH items
4. Testing strategy per phase (parity tests from traces)
5. Cutover criteria per module

Use output-formats.md templates. Map each legacy component to .NET latest target.
```

## Parallel Batching Strategy

| Site size | Agents | Batch |
|-----------|--------|-------|
| < 20 pages | 1 page-scanner, 1 trace | All pages |
| 20–100 pages | 1 scanner per top-level folder | 5–10 pages per trace agent |
| 100+ pages | Scanner per module + priority filter | Trace High-traffic pages first |

Priority for tracing: login, checkout, report generation, admin CRUD, integration endpoints.
