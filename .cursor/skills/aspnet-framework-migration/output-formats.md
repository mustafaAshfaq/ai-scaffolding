# Output Formats

Final deliverable templates for Phase 6.

## 1. Discovery Report

```markdown
# Discovery Report: [Application Name]

## Executive summary
[2–3 sentences: app type, scale, migration complexity]

## Entry points
| Type | Path | File | Notes |
|------|------|------|-------|
| Primary | /Login.aspx | Login.aspx | Forms auth gate |

## Project inventory
[From structure-agent]

## Dependencies
[From dependency-agent — highlight HIGH risk]

## Data access
[From data-agent]

## Security model
[From auth-agent]
```

## 2. Navigation Map

```markdown
# Navigation Map

## Graph
[Mermaid flowchart from diagram-templates.md]

## Edge list
| From | To | Trigger | Auth required |
|------|-----|---------|---------------|

## Orphan pages
- [path] — reason
```

## 3. Interaction Map

```markdown
# Interaction Map

## Summary
- Pages scanned: N
- Interactions inventoried: N
- Traces completed: N

## Per-page inventories
[Tables from page-scanner-agents]

## Global cross-page flows
[Mermaid interaction map]

## Execution traces
[Structured traces from trace-agents]
```

## 4. Architecture Blueprint

```markdown
# Architecture Blueprint: [Application Name]

## Current state (as-is)
### Layers
| Layer | Components | Technology |
|-------|------------|------------|

### Integration points
| System | Protocol | Called from |

## Target state (to-be)
### Recommended stack
- Runtime: .NET [version]
- Web: [Razor Pages | Blazor | MVC | Minimal APIs]
- Auth: ASP.NET Core Identity
- Data: [EF Core | Dapper]

### Bounded contexts
| Context | Legacy modules | Target service |

## Component mapping
| Legacy | Target | Strategy | Effort |
|--------|--------|----------|--------|
| Reports/*.aspx | Reports API + SPA/Blazor | Strangler | L |

## Cross-cutting migration
| Concern | Legacy | Target |
|---------|--------|--------|
| Config | web.config | appsettings.json + options |
| DI | manual new() | built-in DI |
| Logging | log4net/NLog | ILogger + OpenTelemetry |
```

## 5. Migration Roadmap

```markdown
# Migration Roadmap

## Strategy
[Strangler fig / hybrid / big-bang — with rationale]

## Phases

### Phase 0: Foundation (weeks X–Y)
- [ ] Solution structure, CI/CD, shared libraries
- [ ] Identity parity with legacy auth
- **Exit criteria:** New host runs alongside IIS

### Phase 1: [Module name]
- [ ] Work packages from traces
- **Exit criteria:** Parity tests pass for [flows]

### Phase N: Decommission
- [ ] Remove IIS app pool, redirect DNS

## Work packages
| ID | Package | Dependencies | Traces covered | Effort |
|----|---------|--------------|----------------|--------|

## Risk register
| Risk | Impact | Mitigation |
|------|--------|------------|
| System.Web in shared lib | High | Extract adapters first |

## Testing strategy
- **Parity:** Replay traced interactions, compare response shape
- **Integration:** DB, external APIs per swimlane
- **E2E:** High-priority sequence diagram flows
```

## 6. Migration Plan (execution detail)

```markdown
# Migration Plan

## Scope
- In: [modules from navigation map]
- Out: [exclusions]

## Team assumptions
[If unknown, state assumptions]

## Per-module plan

### Module: Reports
**Legacy:** Reports/*.aspx, ReportsBLL
**Target:** Reports.Api + Reports.Web
**Flows:** SEQ-001, SEQ-002
**Tasks:**
1. Extract SalesService to .NET class library
2. Create API endpoint matching btnSearch trace
3. Build UI replacement
4. Strangler route: /reports/* → new host

## Cutover checklist
- [ ] Traffic shadowing
- [ ] Rollback path
- [ ] Data migration (if any)
```

## 7. Diagram Pack Index

```markdown
# Diagram Pack

| ID | Type | Title | Priority | File/section |
|----|------|-------|----------|--------------|
| NAV-01 | Navigation | Site map | — | Navigation Map |
| SW-01 | Swimlane | Login flow | High | § Auth |
| SEQ-01 | Sequence | Sales search | High | § Reports |
```

## Orchestrator Final Response

When presenting to the user, lead with:

1. Executive summary (complexity, recommended strategy)
2. Link to or embed diagram pack (Mermaid)
3. Migration roadmap phases
4. Top 5 risks
5. Suggested first sprint (Phase 0 + one strangler slice)

Keep detailed per-page traces in appendix or separate artifact unless the user wants full inline output.
