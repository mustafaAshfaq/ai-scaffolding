# Execution Tracing Workflow

Drill from UI action to HTTP response. Apply to WebForms, MVC, and hybrid apps.

## WebForms PostBack Trace

### 1. Identify postback source

- `__EVENTTARGET` / `__EVENTARGUMENT` hidden fields
- Button `name` attribute with `$` (e.g. `ctl00$btnSave`)
- `AutoPostBack="true"` on dropdowns/checkboxes
- `LinkButton` → javascript `__doPostBack()`

### 2. Map control to handler

```csharp
// Markup: <asp:Button ID="btnSave" OnClick="btnSave_Click" />
// → Sales.aspx.cs: protected void btnSave_Click(object sender, EventArgs e)
```

- `GridView` → `OnRowCommand`, `OnRowEditing`, `OnRowDeleting`
- `Repeater` / `ListView` → `ItemCommand`
- Page → `Page_Load`, override `OnInit`, `PreRender`

### 3. Page lifecycle (when relevant)

```
Init → LoadViewState → LoadPostData → Load → Control events → PreRender → SaveViewState → Render
```

Note ViewState-dependent logic in `Page_Load` (`if (!IsPostBack)`).

### 4. Follow the call chain

| Pattern | Where to look |
|---------|---------------|
| Direct call | `SalesService.Save()` in code-behind |
| Business layer | `*BLL`, `*Service`, `*Manager` classes |
| Data layer | `*DAL`, `*Repository`, `*DataAccess` |
| Inline SQL | `SqlCommand` in code-behind (flag for extraction) |
| Web service | `new ServiceClient()`, `WebReference` |
| Config | `ConfigurationManager.AppSettings` |

### 5. Response path

- **Stay on page:** rebind grids, show labels, UpdatePanel refresh
- **Redirect:** `Response.Redirect`, `Server.Transfer`
- **AJAX:** `PageMethods`, `WebMethod`, handler JSON, UpdatePanel partial HTML
- **File:** `Response.BinaryWrite`, `TransmitFile`

### 6. State and side effects

- Session keys read/written
- ViewState changes
- Application cache
- Cookies set
- Logging / audit calls
- Fire-and-forget threads (flag as migration risk)

## MVC / Web API Trace

```
Route → Controller.Action → [Filters] → Service → Repository → DB
→ ActionResult (View, JsonResult, RedirectToAction)
```

Check `FilterConfig`, `Authorize`, custom `ActionFilterAttribute`.

## ASHX / ASMX / WCF Trace

```
HTTP request → handler ProcessRequest / WebMethod / OperationContract
→ implementation → dependencies → response stream
```

## AJAX Trace

1. Find JS file or inline script initiating call
2. Resolve URL: `PageMethods.X`, `*.asmx/X`, `*.ashx`, `/api/`
3. Trace server handler
4. Trace success/error callbacks in JS

## Unresolved Handler Resolution

If handler unknown:

1. Search code-behind for control ID
2. Search `*.designer.cs` for control declarations
3. Grep for `Handles` (VB) or wired events in `OnInit`
4. Check base page class (`Inherits="SiteMasterPage"`)
5. Check master page for shared handlers

Mark as `UNRESOLVED` if still unknown; do not invent calls.

## Trace Output Schema

```yaml
trace_id: Sales_btnSearch
page: Reports/Sales.aspx
element: btnSearch (asp:Button)
trigger: PostBack Click
handler: btnSearch_Click
lifecycle: Load → btnSearch_Click → PreRender → Render
call_chain:
  - file: Sales.aspx.cs
    method: btnSearch_Click
  - file: SalesService.cs
    method: GetByDateRange
  - file: SalesRepository.cs
    method: Query
  - db: usp_SalesByDate
response:
  type: HTML
  status: 200
  action: GridView.DataBind
state:
  session: [FilterDate]
  viewstate: updated
side_effects: []
risk_flags: []
```

## Stop Condition

Stop tracing when:

- HTTP response is committed (render, redirect, JSON returned)
- OR external fire-and-forget with no impact on response (note and stop)

Do not trace into framework internals (`System.Web.UI` source).
