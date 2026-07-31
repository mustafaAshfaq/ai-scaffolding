---
name: aspnet-framework-migration
description: Creates roadmap, migration plan, and architecture blueprint for modernizing legacy ASP.NET Framework apps to .NET latest. Orchestrates sub-agents to map entry points, navigation, page interactions, and end-to-end execution traces across WebForms/MVC. Use when migrating ASP.NET Framework, .aspx, WebForms, System.Web, or planning .NET modernization.
---

# ASP.NET Framework Migration Orchestrator

You are the **migration orchestrator**. Do not jump to migration recommendations until discovery, navigation mapping, interaction tracing, and diagram synthesis are complete.

## Deliverables

Produce these artifacts (templates in [output-formats.md](output-formats.md)):

1. **Discovery report** — project inventory, entry points, tech stack
2. **Navigation map** — how users move through the application
3. **Interaction map** — every UI action → backend execution → response
4. **Swimlane diagrams** — actors, layers, and handoffs
5. **Sequence diagram catalog** — flows worth diagramming
6. **Architecture blueprint** — target .NET architecture mapped from legacy
7. **Migration roadmap** — phased plan with risks, dependencies, and effort

## Orchestration Model

```
Phase 0: Intake → Phase 1: Discovery (parallel) → Phase 2: Navigation map
→ Phase 3: Interaction inventory (parallel per area) → Phase 4: Execution traces (parallel)
→ Phase 5: Synthesis → Phase 6: Deliverables
```

Track progress with a checklist. Update it after each phase.

## Phase 0: Intake

Confirm scope before launching sub-agents:

- [ ] Solution path(s) and project type(s): WebForms, MVC, Web API, ASHX, ASMX, WCF
- [ ] Target .NET version (default: latest LTS)
- [ ] Migration strategy preference: strangler fig, big-bang, or hybrid (if unknown, recommend strangler fig)
- [ ] Exclusions: admin tools, dead code, third-party binaries without source

## Phase 1: Discovery (parallel sub-agents)

Launch independent discovery agents via the Task tool. See [subagents.md](subagents.md) for prompts.

| Agent | Scope | Output |
|-------|-------|--------|
| `entry-point-agent` | Global.asax, Startup, default docs, routing, IIS config | Entry point catalog |
| `structure-agent` | .csproj, folders, App_Code, bin, packages.config | Project inventory |
| `dependency-agent` | NuGet, COM, GAC, System.Web, third-party DLLs | Dependency matrix |
| `data-agent` | Connection strings, EF/ADO.NET, stored procs, ORM | Data access map |
| `auth-agent` | web.config auth, membership, roles, custom filters | Security model |

**Merge** outputs into a single discovery report. Identify the **primary entry point** (first user-facing URL/page) and **secondary entry points** (APIs, handlers, scheduled jobs).

### Entry point identification rules

1. Check `Global.asax` / `Application_Start` for routing and default routes
2. Check `web.config`: `<defaultDocument>`, `<authentication>`, `<httpModules>`
3. Find `Default.aspx`, `Login.aspx`, or MVC `RouteConfig` default route
4. Trace `Response.Redirect` and `Server.Transfer` from startup/auth flows
5. Document external entry: ASHX (`*.ashx`), ASMX (`*.asmx`), WCF `.svc`, Web API controllers

## Phase 2: Navigation Map

Build a **site navigation graph** from the primary entry point.

### Algorithm

1. Start at primary entry point page/handler
2. For each page, extract navigation elements:
   - `<a href>`, `<asp:HyperLink>`, `<asp:Menu>`, `<asp:TreeView>`
   - `Response.Redirect`, `Server.Transfer`, `window.location`
   - Post-login redirects, master page menus, breadcrumbs
3. Record edges: `Source → Target` with trigger (link click, post-login, menu)
4. BFS/DFS until all reachable pages are mapped or scope limit hit
5. Flag unreachable pages (orphans) and external links

### Output format

```markdown
## Navigation Map

### Entry: /Login.aspx
- Login.aspx --[btnLogin Click]--> Default.aspx
- Default.aspx --[menu: Reports]--> Reports/Index.aspx
...

### Orphan pages
- Legacy/OldReport.aspx (no inbound links found)
```

Assign **one `page-scanner-agent` per logical area** (folder/module) when the site has 20+ pages.

## Phase 3: Interaction Inventory (parallel per area)

For **every page/WebForm** in the navigation map, inventory all user interaction elements.

### Elements to capture

| Element | ASP.NET patterns |
|---------|------------------|
| Buttons | `<asp:Button>`, `<input type="submit">`, `LinkButton`, `ImageButton` |
| Links | `<asp:HyperLink>`, `<a>`, `LinkButton` (non-postback) |
| Form submit | `<form runat="server">`, postback triggers |
| Inputs | `TextBox`, `DropDownList`, `CheckBox`, `RadioButton`, `FileUpload`, `GridView` commands |
| AJAX | `UpdatePanel`, `ScriptManager`, `PageMethods`, jQuery `$.ajax`, `WebMethod` |
| Grid actions | `GridView` RowCommand, `Repeater` ItemCommand, `ListView` |
| Client scripts | `onclick`, `__doPostBack`, custom JS handlers |

### Per-element record

```markdown
### Page: Reports/Sales.aspx
| ID | Element | Type | Trigger | Handler hint |
|----|---------|------|---------|--------------|
| btnSearch | asp:Button | PostBack | Click | btnSearch_Click |
| gvSales | GridView | RowCommand | Edit | gvSales_RowCommand |
```

Use `page-scanner-agent` — one per folder/area. Run in parallel.

## Phase 4: Execution Tracing (parallel per interaction)

For each inventoried interaction, **drill down until the HTTP response is returned**. Follow [tracing-workflow.md](tracing-workflow.md).

### Trace depth (stop when response returns)

```
UI event → code-behind / controller action → page lifecycle hooks
→ business/service layer → data access → external calls (SOAP, file, email)
→ response path (ViewState, redirect, JSON, render)
```

### Per-trace record

```markdown
### Trace: Reports/Sales.aspx → btnSearch_Click
**Trigger:** PostBack, btnSearch_Click
**Call chain:**
1. Sales.aspx.cs: btnSearch_Click
2. SalesService.GetByDateRange()
3. SalesRepository.Query() → usp_SalesByDate
**Response:** GridView rebind, HTML 200
**State:** ViewState updated, Session["Filter"]
**Side effects:** None
```

Assign **one `trace-agent` per page or per module**. Parallelize independent pages. For shared services, note cross-references instead of re-tracing.

## Phase 5: Synthesis

Merge navigation map + interaction map + traces.

### 5a. Cross-page interaction map

```markdown
## Interaction Map (global)
[User] → Login.aspx (btnLogin) → [AuthModule] → Default.aspx
[User] → Default.aspx (menu Reports) → Reports/Sales.aspx
[User] → Sales.aspx (btnSearch) → [SalesService] → [SQL] → Sales.aspx (render)
```

### 5b. Swimlane identification

Identify swimlanes from traced layers. Typical lanes:

- User / Browser
- WebForm or MVC View
- Code-behind / Controller
- HTTP modules / Filters / Middleware (target)
- Application / Domain services
- Data access (ADO.NET, EF, repositories)
- Database / External APIs
- Infrastructure (logging, cache, messaging)

Document in [diagram-templates.md](diagram-templates.md) format.

### 5c. Sequence diagram catalog

Flag flows that warrant sequence diagrams:

- Login / logout / session establishment
- Multi-step wizards (ViewState-heavy)
- PostBack chains across pages
- AJAX partial updates
- File upload / export
- Payment or approval workflows
- Background job triggers from UI

Rate each: **High** (critical path, complex), **Medium**, **Low**.

## Phase 6: Deliverables

Compile final artifacts using [output-formats.md](output-formats.md):

1. **Architecture blueprint** — current vs target, bounded contexts, service boundaries
2. **Migration roadmap** — phases, strangler routes, cutover criteria
3. **Migration plan** — work packages, dependencies, risks, testing strategy
4. **Diagram pack** — swimlanes + sequence diagrams (Mermaid)

### Modernization mapping quick reference

| Legacy | .NET latest target |
|--------|-------------------|
| WebForms (.aspx) | Razor Pages, Blazor, or MVC (per fit) |
| ASHX / ASMX | Minimal APIs or controllers |
| System.Web | ASP.NET Core middleware pipeline |
| web.config | appsettings.json + DI |
| Session / ViewState | Session middleware, stateless design, or Redis |
| Membership provider | ASP.NET Core Identity |
| WCF | gRPC, REST, CoreWCF (bridge) |
| HttpModules/Handlers | Middleware |
| packages.config | PackageReference, Central Package Management |

## Parallelization Rules

- **Parallelize:** independent folders, independent page traces, dependency vs data vs auth discovery
- **Sequential:** navigation map before interaction inventory; interaction inventory before traces; traces before synthesis
- **Max concurrent:** up to 4 sub-agents at once; batch pages in groups of 5–10
- **Merge conflicts:** when two agents trace the same shared service, merge into one canonical service entry

## Sub-agent Launch Pattern

Use the Task tool with `subagent_type="explore"` for read-only discovery and tracing.

```
Task: page-scanner-agent
Prompt: [copy from subagents.md, fill {{AREA}} and {{SOLUTION_PATH}}]
readonly: true
```

For synthesis and deliverables, run orchestrator locally or use `subagent_type="planner"` for roadmap drafting.

## Quality Gates

Do not finalize until:

- [ ] Primary and secondary entry points documented
- [ ] All reachable pages in navigation map (orphans listed)
- [ ] Every page has interaction inventory
- [ ] Every High/Medium interaction has execution trace
- [ ] Swimlanes identified for major flows
- [ ] At least one sequence diagram per High-rated flow
- [ ] Migration roadmap references traced flows (not guesses)

## Additional Resources

- Sub-agent prompts: [subagents.md](subagents.md)
- Tracing methodology: [tracing-workflow.md](tracing-workflow.md)
- Diagram templates: [diagram-templates.md](diagram-templates.md)
- Output templates: [output-formats.md](output-formats.md)
