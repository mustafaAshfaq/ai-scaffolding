# Diagram Templates

Use Mermaid in deliverables. Create during Phase 5 (Synthesis).

## Swimlane Diagram

Map actors and layers from execution traces. One swimlane per logical layer.

### Template

```mermaid
flowchart TB
    subgraph User["User / Browser"]
        U1[Click btnSearch]
    end
    subgraph WebForm["WebForm Layer"]
        W1[Sales.aspx PostBack]
        W2[btnSearch_Click]
    end
    subgraph Service["Application Services"]
        S1[SalesService.GetByDateRange]
    end
    subgraph Data["Data Access"]
        D1[SalesRepository.Query]
    end
    subgraph DB["SQL Server"]
        DB1[(usp_SalesByDate)]
    end

    U1 --> W1 --> W2 --> S1 --> D1 --> DB1
    DB1 --> D1 --> S1 --> W2 --> W1 --> U1
```

### Swimlane identification checklist

From traces, extract:

- [ ] **User** — all UI triggers
- [ ] **Presentation** — .aspx, .ascx, master pages, MVC views
- [ ] **Application** — code-behind handlers, controllers
- [ ] **Cross-cutting** — modules, filters, auth, logging
- [ ] **Domain/Services** — BLL, managers, validators
- [ ] **Data** — repositories, EF contexts, ADO.NET
- [ ] **External** — SOAP, REST, file system, SMTP, queue
- [ ] **Database** — tables, stored procs

### When to create swimlanes

| Flow type | Priority |
|-----------|----------|
| Authentication / session | High |
| Multi-step business process | High |
| Payment / submission | High |
| CRUD with validation | Medium |
| Simple read-only page | Low (skip unless migration-critical) |

## Sequence Diagram

Use for High-priority flows. Show temporal order and return messages.

### Template

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SalesPage as Sales.aspx
    participant CodeBehind as Sales.aspx.cs
    participant Service as SalesService
    participant Repo as SalesRepository
    participant DB as SQL Server

    User->>Browser: Click Search
    Browser->>SalesPage: POST (ViewState + postback)
    SalesPage->>CodeBehind: btnSearch_Click
    CodeBehind->>Service: GetByDateRange(from, to)
    Service->>Repo: Query(filter)
    Repo->>DB: EXEC usp_SalesByDate
    DB-->>Repo: Result set
    Repo-->>Service: List<Sale>
    Service-->>CodeBehind: List<Sale>
    CodeBehind->>SalesPage: Bind GridView
    SalesPage-->>Browser: HTML 200
    Browser-->>User: Rendered grid
```

### Sequence diagram catalog entry

```markdown
### SEQ-001: Sales search
- **Priority:** High
- **Entry:** Reports/Sales.aspx
- **Trigger:** btnSearch_Click
- **Actors:** User, Sales.aspx, SalesService, SalesRepository, SQL
- **Why diagram:** Core reporting path, ViewState postback, stored proc
- **Migration note:** Candidate for Razor Page + minimal API
```

## Navigation Graph

Separate from swimlanes — shows page-to-page movement only.

```mermaid
flowchart LR
    Login[Login.aspx] -->|success| Home[Default.aspx]
    Home -->|menu| Sales[Reports/Sales.aspx]
    Home -->|menu| Admin[Admin/Users.aspx]
    Sales -->|btnExport| Export[Export.ashx]
```

## Interaction Map (global)

Combine navigation + traces in one view for critical paths.

```mermaid
flowchart TD
    subgraph Auth
        L[Login.aspx] -->|forms auth| H[Default.aspx]
    end
    subgraph Reports
        H -->|nav| S[Sales.aspx]
        S -->|btnSearch| T1[SalesService]
        T1 --> DB[(Database)]
        DB --> S
    end
```

## Cross-cutting concerns diagram

For migration planning — show System.Web dependencies.

```mermaid
flowchart LR
    subgraph Legacy["ASP.NET Framework"]
        WF[WebForms]
        SW[System.Web]
        VS[ViewState]
        MOD[HttpModules]
    end
    subgraph Target[".NET Latest"]
        RP[Razor Pages / Blazor]
        MW[Middleware pipeline]
        ID[ASP.NET Core Identity]
    end
    WF -.->|replace| RP
    SW -.->|replace| MW
    VS -.->|eliminate or Redis| ID
    MOD -.->|replace| MW
```

## Diagram Pack Deliverable

Include in final output:

1. Navigation graph (site-wide)
2. 1 swimlane per High-priority domain (auth, core CRUD, integrations)
3. 1 sequence diagram per High-priority interaction trace
4. Cross-cutting migration diagram (current → target)
