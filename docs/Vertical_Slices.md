# OverGoods (OGS) — Vertical Slices from the Monolith

 

Companion to `ogs-routes.md`. This document identifies **vertical slices** (feature-cohesive units) that can be extracted from the legacy WebForms monolith (`Legacy/DI_Overgoods/OGSRoot/Overgoods/`) into the target service architecture (`api-service/`, `business-service/`, `data-service/`, `retry-service/`, `ui-service/`).

 

The slice analysis is backed by nine per-area analyses stored in the session workspace (`files/slice-*.md`) covering: auth/menu, data-entry, search, user-management, warehouse-ops, tools, reports, web-services, and standalone pages.

 

---

 

## 1. UI inventory (what exists today)

 

### Pages — 98 `.aspx` total

- Overgoods root: **66** (login, hubs, data entry, warehouse ops, tools, standalone).

- `Overgoods/reports/`: **27**.

- `Overgoods/usermanagement/`: **5**.

 

### User controls — 7 `.ascx`

- Root: `loginname.ascx`, `MenuOptions.ascx`.

- `usercontrols/`: `OgsCategories.ascx`, `UCLookupResults.ascx`, `cbostatesusctl.ascx`.

- `usermanagement/usercontrols/`: `sectionpagelist.ascx`, `newpassword.ascx`.

 

### Web services — 2 `.asmx`

- `OGSService.asmx` — 9 WebMethods used across ~10 pages for AJAX lookups (categories, SLICs, districts, users-in-SLIC, location, item detail, return detail, hot-list detail, maturity date).

- `ProductCodeProcessor.asmx` — 2 WebMethods for UPC/product-code lookup used during data entry.

 

### Class libraries (`OGSLib/Libraries/`)

VB: `AresLibrary` (OGSLibrary namespace), `CommonLibrary`, `WarehouseLibrary`, `PSCScanner`.

C#: `clsUPSShipAPI`, `eTTInterface`, `InventoryLibrary`, `PrintDataSetsLibrary`, `ReturnsLibrary`, `UserLibrary`, `HotListLibrary`.

Ancillary: `HIPWeb`/`HIPWeb64` (label print), `OGSPrintControls`, `Semantics3`, `eTTOutput`, `xpldexport`.

 

### Standalone service projects

`OGSNewTrackingService`, `OvergoodsToCipe` (Common/Console/Service), `CitibankAccountsExtract`, `OGSWarehouseAlignment`, `OGSModernBrowserPrintService`.

 

---

 

## 2. Cross-cutting concerns (must be resolved before slicing)

 

| Concern | Today | Implication |

|---------|-------|-------------|

| **AuthN** | Azure AD via MSAL, plus CEC signed-request and eTT inbound POST handled in `login.aspx.cs`. | Externalize to a single identity gateway; each service becomes a resource server. |

| **AuthZ** | Page-level via `User.AccessAllowed(pageId)` backed by section/page grant list (`usp_ARES_SectionPageList`, `usp_USER_GetUserPermissions`). Enforced in every page's `Page_Load`. | Replace with role/scope claims + policy-based auth. Preserve the section/page grant list as a migration bridge. |

| **Session state** | Heavy `Session[]` usage (`SessionManager.CurrentUser`, `SubsidiaryId`, `CurrentCSV`, `CurrentRTF`, `CurrentEditUser`, `eTTSearchRequest/Response`). | New UI must be stateless; move ephemeral state to the client or a short-lived cache. eTT request/response should become an idempotent operation with a server-side correlation ID. |

| **Photo storage** | Google Cloud Storage via `App_Code/Tools.cs` (`UploadFileToGCP`, `Image.aspx` streams). | Photos already GCS-backed — easy to expose behind a media service. |

| **Printing** | Two paths: (1) client-side scripts (Dymo, `CreatePrintScript`, RTF letters via `SessionManager.CurrentRTF`); (2) `OGSModernBrowserPrintService` (REST) for incident/shipping/DYMO/Zebra. `PickTickets` does NOT currently call `OGSModernBrowserPrintService`. | Consolidate on the modern print service; retire client-side print scripts. |

| **Data access** | 100% stored procedures (`usp_INV_*`, `usp_USER_*`, `usp_REPORT_*`, `usp_eTT_*`, `usp_Synonym_*`, `usp_Citibank_*`, `usp_CIPE_*`, `usp_SLIC_*`, `usp_RETURNS_*`, `usp_AUDITS_*`). | `data-service` should expose these SPs; migrate to typed repositories over time. |

| **Reports export** | CSV/RTF via `SessionManager.CurrentCSV/CurrentRTF` written by report pages, streamed by `downloadcsv.aspx`/`downloadrtf.aspx`. | Replace with an on-demand endpoint that returns a stream, no session hop. |

| **Databases** | Two: `OVERGOODS` (transactional) and `ARES_SEARCH` (full-text index over `tAppInventoryAll`). | Search slice needs both. |

| **Missing route** | `reports.aspx` links to `datareport.aspx?reporttype=(manifestscan|originscan|deliveryscan)` but **that file does not exist**. Dead links. | Either resurrect the report or remove links in the new UI. |

 

---

 

## 3. Vertical slices

 

Each slice is a feature-cohesive extraction unit. **Order of extraction** is designed to minimize inter-slice dependencies at extraction time.

 

Difficulty legend: **S** = small (single page group, few libs), **M** = medium (multi-page, one lib, moderate procs), **L** = large (many pages, cross-lib, external integration), **XL** = extra-large (spans multiple integrations or shared runtime state).

 

### Slice 1 — Identity & Access (foundational)

**Order: 1**  •  **Difficulty: L**

 

- **Pages/controls:** `login.aspx`, `auth.aspx`, `AzureSSOLoginPage.aspx`, `default.aspx` (subsidiary picker), `ProfileNotFound.aspx`, `mainmenu.aspx`, `WarehouseMenu.aspx`, `toolsmenu.aspx`, `MasterPage.master`, `loginname.ascx`, `MenuOptions.ascx`, all pages under `usermanagement/` and its `usercontrols/`.

- **Libraries:** `UserLibrary` (User, UserManager, UserGroups), Azure/MSAL config in `CommonLibrary.Constants`.

- **DB tables/procs:** `usp_USER_*` (12 procs), `usp_ARES_SectionPageList`.

- **External:** Azure AD (OIDC), CEC signed request, eTT inbound handshake (auth handshake only — search flow lives in Slice 3).

- **Outputs a service:** identity provider + user/permission REST API.

- **Depends on:** none.

- **Blocks:** every other slice's authorization checks.

 

### Slice 2 — Inventory Core (data entry & incident lifecycle)

**Order: 2**  •  **Difficulty: XL**

 

- **Pages:** `dataentry.aspx`, `unknownovergoods.aspx`, `knowngoods.aspx`, `WweMiOrphans.aspx`, `sensitivematerial.aspx`, `AbandonedOvergoods.aspx`, `NonTransportingGoods.aspx`, `scssalvage.aspx`, `unknowngoods.aspx`, `hazmat*.aspx`, `guns*.aspx`, `Firearms.aspx`.

- **Controls:** `OgsCategories.ascx`, `UCLookupResults.ascx`, `cbostatesusctl.ascx`.

- **Web services:** `ProductCodeProcessor.asmx`, `OGSService.GetChildCategories/GetLocation/GetMaturityDate`.

- **Libraries:** `InventoryLibrary` (`clsInventory`, `InventoryManager`), `OGSLibrary.LTRLibrary`, `OGSLibrary.CRIS`, `OGSLibrary.AresLibrary.GetShipMethods/GetShipToCountries`, `Semantics3`.

- **Owned procs:** `usp_INV_AddInventory`, `usp_INV_AddInventoryPhotos`, `usp_INV_UpdateInventory`, `usp_INV_VirtualDelete`, LTR box open/create/close.

- **Depends on:** Slice 1 (auth), Slice 7 (media for photo attachment), Slice 8 (print — Dymo labels).

- **Notes:** Largest slice by page count. Regional hazmat/guns pages are near-duplicates — collapse into one page + policy in the rewrite.

 

### Slice 3 — Search & eTT Integration

**Order: 3**  •  **Difficulty: L**

 

- **Pages:** `search.aspx`, `searchresults.aspx`, `AdminSearch.aspx`, `eTTPostback.aspx`, `synonymsearch.aspx`, `ViewAllKeyWord.aspx`, `UpcLookup.aspx`.

- **Libraries:** `AresLibrary` (Synonyms + lookup collections), `eTTInterface`, `eTTOutput`.

- **DBs:** `OVERGOODS` (`vARESSearch`, `usp_Synonym_*`, `usp_eTT_*`), `ARES_SEARCH` (index tables).

- **External integration:** eTT inbound `searchVoidRequest` XML, outbound `searchResponse` XML posted to `eTTURL`, `VoidSent` acknowledgement.

- **Depends on:** Slice 1.

- **Notes:** eTT is the most tightly coupled external protocol. Preserve exact XML contract during migration (`searchVoidRequest.cs`, `searchResponse.cs`) and route through `retry-service` for the async post-back.

 

### Slice 4 — Warehouse Operations

**Order: 4**  •  **Difficulty: L**

 

- **Pages:** `HotListCandidate.aspx`, `LTRReceive.aspx`, `DispositionInventory.aspx`, `AttachPhoto.aspx`, `ListInventory.aspx`, `QualityAudit.aspx`, `OverageDataEntry.aspx`, `InventoryActivity.aspx`, `PrintTickets.aspx`, `Exceptions.aspx` (audit editor), `ListerHistory.aspx`, `PickTicketHistory.aspx`, `LettersHistory.aspx`.

- **Web services used:** `OGSService.GetSLICs/GetDistricts/GetUsersInSLIC/GetLocation/ItemDetailAjax/ItemReturnDetailAjax/HotListDetailAjax`.

- **Libraries:** `WarehouseLibrary`, `InventoryLibrary.InventoryManager` (disposition/photo/remarks/shelf), `InventoryLibrary.AuditManager`, `InventoryLibrary.LetterManager`, `HotListLibrary.clsHotListInventory` (read), `PrintDataSetsLibrary.PickTicketData`, `ReturnsLibrary.PickTicketManager.CreatePrintScript`.

- **Owned procs:** `usp_INV_ReceiveInventory`, `usp_INV_SetDisposition`, `usp_INV_SetPhotoURL`, `usp_INV_*RemarkShowStatus`, `usp_INV_AddInventoryRemark`, `usp_INV_GetInventoryPhotos`, `usp_INV_GetInventoryRemarks`, `usp_INV_GetPossibleDispositionsForInventory`, `usp_INV_GetInventoryHistory`, `usp_INV_UpdatePhotosStatus`, `usp_AUDITS_*`.

- **Depends on:** Slice 1, Slice 2 (shares `InventoryLibrary`), Slice 7 (photos), Slice 8 (print).

 

### Slice 5 — Returns, Voids & Tracking

**Order: 5**  •  **Difficulty: M**

 

- **Pages:** `AddTrackingNumbers.aspx`, `ReverseVoid.aspx`, `UpdateReturnInfo.aspx`, `ItemDetails.aspx`, `Address.aspx`.

- **Libraries:** `ReturnsLibrary.ReturnsManager` (Load/Update, ReverseVoid, tracking numbers, AddKnownShipment, AddVoidTrackingNumber), `clsUPSShipAPI.VoidShipment`.

- **Owned procs:** `usp_RETURNS_*`.

- **External:** UPS XOLT (Ship, Void endpoints) via Zscaler proxy.

- **Standalone consumer:** `OGSNewTrackingService` reads UPS tracking updates.

- **Depends on:** Slice 1, Slice 2.

 

### Slice 6 — Reporting & Exports

**Order: 6 (parallel with 4–5)**  •  **Difficulty: L**

 

- **Pages:** `reports.aspx` + all 27 report pages + `downloadcsv.aspx`, `downloadrtf.aspx`.

- **Libraries:** `CommonLibrary.CSV`, `WarehouseLibrary.WarehouseManager.GetWarehousesForSubsidiary` (WHPerf), `OGSLibrary.SLICInfo`.

- **DB:** Direct `SqlCommand` on OVERGOODS + `usp_REPORT_*`, `US_Known_Overgoods_Weekly_Report`.

- **Special:** `KnownOvergoodsWeeklyExtract` fills `SessionManager.CurrentCSV`; letters flow uses `CurrentRTF`.

- **Dead code:** `reports.aspx` links to non-existent `datareport.aspx?reporttype=…` — treat as backlog decision.

- **Depends on:** Slice 1 (permissions). Independent of others because reports are read-only from OVERGOODS.

- **Notes:** Reports do NOT use `PrintDataSetsLibrary`. The three parametric report types (`manifestscan`, `originscan`, `deliveryscan`) are broken today.

 

### Slice 7 — Media / Photos

**Order: 3 (parallel with Slice 3)**  •  **Difficulty: S**

 

- **Pages:** `InventoryPhoto.aspx`, `photos.aspx`, `Image.aspx`, `AttachPhoto.aspx` (also in Slice 4).

- **Helpers:** `App_Code/Tools.cs` (`UploadFileToGCP`, `StorageClient.UploadObject/DownloadObject`).

- **External:** Google Cloud Storage.

- **Depends on:** Slice 1.

- **Notes:** Already cleanly encapsulated on GCS. Good candidate for the first extraction after identity to prove out the new architecture.

 

### Slice 8 — Printing & Letters

**Order: 4 (parallel with Slice 4)**  •  **Difficulty: M**

 

- **Pages:** `PrintLetters.aspx` (RTF letter generation via `LetterManager` + `RTFComplianceLetter.CreateRTF_Entry`); `PrintTickets.aspx` (pick tickets via `PickTicketManager.CreatePrintScript` + `PrintDataSetsLibrary`); `downloadrtf.aspx`.

- **Standalone consumer:** `OGSModernBrowserPrintService` (already REST — targets DYMO/Zebra/incident/shipping/ticket).

- **Libraries:** `PrintDataSetsLibrary`, `OGSPrintControls`, `HIPWeb`/`HIPWeb64`, `InventoryLibrary.LetterManager`.

- **Depends on:** Slice 1, Slice 2, Slice 4.

- **Notes:** Consolidate all print paths onto the modern print service; retire in-page Dymo scripts and RTF-via-session download.

 

### Slice 9 — Administrative Tools

**Order: 7**  •  **Difficulty: M**

 

- **Pages:** `toolsmenu.aspx`, `HotListDataCapture.aspx`, `UpdateLocations.aspx`, `FinalStatusTool.aspx`, `WarehouseAdd.aspx`, `WarehouseAlignment.aspx`, `KnownBlockedAccounts.aspx`, `CitibankPilotMaintenance.aspx`, `DeleteOvergoods.aspx`.

- **Libraries:** `HotListLibrary.clsHotListInventory` (write ops), `InventoryLibrary.InventoryManager.SetShelfLocationForInventory`, `InventoryLibrary.InventoryManager.DeleteInventory`, `OGSLibrary.LTRLibrary.DeleteLTRBox`, `OGSLibrary.BlockedAccounts`.

- **Procs:** `usp_InsertWarehouse`, `usp_EditWarehouse`, `usp_SLIC_CreateSlicWarehouseRelationship`, `usp_SLIC_ApplyWarehouseChangeToSlicTables`, `usp_REPORT_FinalStatusDetails`, `usp_Citibank_*`.

- **Standalone counterparts:** `OGSWarehouseAlignment` (batch equivalent of `WarehouseAlignment.aspx`); `CitibankAccountsExtract` (export equivalent of `CitibankPilotMaintenance.aspx`).

- **Depends on:** Slice 1, Slice 2 (for delete/shelf), Slice 4 (indirect).

 

### Slice 10 — Integrations (CIPE, Citibank, Tracking, Retry)

**Order: 8**  •  **Difficulty: L**

 

- **Standalone projects:** `OvergoodsToCipe` (Common/Console/Service), `CitibankAccountsExtract`, `OGSNewTrackingService`.

- **Procs:** `usp_CIPE_GetTraceUserLogs`, `usp_CIPE_GetInventory`, `usp_CIPE_GetLtrBoxes`, `usp_CIPE_UpdateEventLog`, `usp_CIPE_UpdateLastSuccessfulSend`; `usp_Citibank_SelectAccounts`.

- **External:** ActiveMQ (CIPE), Citibank flat-file drop, UPS tracking.

- **Target service:** primarily `retry-service` (async, resilient outbound).

- **Depends on:** Slice 2 (inventory events), Slice 5 (returns/voids), Slice 9 (Citibank maintenance).

 

### Slice 11 — Ancillary / Diagnostic (retire or replace)

**Order: last**  •  **Difficulty: S**

 

- `WebServiceTest.aspx` — dev-only test harness; do not port.

- `ErrorPage.aspx`, `ErrorInvalidInput.aspx` — replace with framework error middleware in new UI.

- `xpldexport` library — evaluate; likely retire.

- The dead `datareport.aspx?reporttype=…` links — remove from new UI unless business confirms need.

 

---

 

## 4. Recommended extraction order (summary)

 

1. **Slice 1 — Identity & Access** (foundation)

2. **Slice 7 — Media / Photos** (small, clean; proves architecture)

3. **Slice 6 — Reporting & Exports** (read-only; low blast radius; can proceed while other slices are in flight)

4. **Slice 3 — Search & eTT** (external contract; isolate early)

5. **Slice 2 — Inventory Core** (largest; the beating heart)

6. **Slice 4 — Warehouse Operations** (depends on 2)

7. **Slice 5 — Returns/Voids/Tracking** (depends on 2)

8. **Slice 8 — Printing & Letters** (depends on 2, 4)

9. **Slice 9 — Administrative Tools**

10. **Slice 10 — Integrations (retry-service)**

11. **Slice 11 — Cleanup**

 

---

 

## 5. Suggested mapping to target services

 

| Target service | Owns |

|----------------|------|

| `api-service` | Public REST endpoints for each slice; authorization at the gateway boundary. |

| `business-service` | Domain logic re-hosted from `InventoryLibrary`, `ReturnsLibrary`, `HotListLibrary`, `AresLibrary`, `UserLibrary`, `WarehouseLibrary`, `eTTInterface`. |

| `data-service` | Stored-proc facade + repository layer over `OVERGOODS` and `ARES_SEARCH`. |

| `retry-service` | Async outbound to UPS XOLT, eTT postback, CIPE (ActiveMQ), Citibank drop, tracking pull. |

| `ui-service` | React/Vite replacement for all `.aspx`/`.ascx`. |

 

---

 

## 6. Open questions

 

1. Are `datareport.aspx?reporttype=manifestscan|originscan|deliveryscan` reports still required? (File missing from repo.)

2. Should regional hazmat/guns pages remain separate URLs or collapse into a single localized page?

3. `OGSModernBrowserPrintService` — is it already deployed and used, or still pilot? (Pick-ticket flow does not call it today.)

4. Which pages should be dropped in the modernization vs. one-to-one ported? (`WebServiceTest.aspx` should not be ported.)

5. `Session["CurrentCSV"]`-based CSV export is single-user, single-tab safe only — confirm no known user complaints before designing the replacement.

 

 