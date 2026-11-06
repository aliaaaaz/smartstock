# SMARTSTOCK AI
## Functional Requirements Specification (FRS) — V1.0

**Date:** November 6, 2026  
**Status:** Draft for stakeholder review  
**Basis:** Strategic Architecture Update V1.2  
**Product direction:** Standalone inventory and business management platform with optional, verified integrations

---

## 1. Purpose and approval boundary

This document translates the approved strategic direction into observable, testable product requirements. It defines the first standalone release's users, workflows, rules, permissions, validations, failure behavior, and acceptance criteria.

This is a requirements baseline, not an implementation authorization. Major implementation should begin after the business workflows and acceptance criteria in this document have been reviewed. The Windows target, hardware architecture, first-store workflow details, and Logica integration capabilities remain unverified and are recorded as dependencies or open questions below.

### 1.1 Scope of the first usable release

The first usable standalone release is expected to provide:

- Store and user setup, authentication, role-based permissions, and audit history.
- Product catalog, identifiers/barcodes, categories, units, and supplier records.
- Purchase requests/orders, receiving, stock movements, balances, counts, adjustments, and returns.
- Basic invoice/document records and purchase cost history.
- Basic operational reports and recoverable local backups.
- An internal data model and application workflows that do not require POS/ERP connectivity, internet access, or AI services for core operations.

OCR, AI-based matching or recommendations, demand forecasting, multi-store operation, cloud hosting, and a first vendor-specific connector are later capabilities unless explicitly promoted after review. A manual or file-based workflow may be used without describing it as a verified POS integration.

### 1.2 Out of scope for this baseline

- Replacing, modifying, or writing to the existing POS/ERP.
- Assuming any API, database access, or synchronization capability for Logica or another vendor.
- Unattended AI changes to official stock, purchasing, or financial records.
- Payroll, accounting ledger, tax filing, e-commerce, or customer relationship management unless added through approved requirements.
- A committed Windows version, CPU architecture, installer technology, or multi-user network deployment before compatibility discovery.

## 2. Terms

| Term | Meaning |
|---|---|
| Store | An isolated business data boundary for users, products, suppliers, and transactions. |
| Stock movement | An immutable, auditable event that changes quantity at a stock location. |
| Stock balance | A derived quantity for a product at a location, reconcilable to movements. |
| External mapping | A link between an internal SmartStock record ID and an external system's identifier. |
| Draft | A record that can be edited and has not yet been approved or posted. |
| Posted | A confirmed business record whose effects are reflected in the authoritative ledger. |
| Connector | An optional adapter that declares and implements specific verified external operations. |
| Offline-capable core | Essential local workflows remain available without internet, AI, or external integration services. |

## 3. Actors and roles

Permissions are assigned to users within a store. The role names below are defaults; an authorized administrator may configure permissions if that capability is included in the chosen release. A person may hold more than one role. The system must deny access by default when permission is absent.

| Actor / default role | Responsibilities |
|---|---|
| Store owner / administrator | Configure the store, users, roles, locations, settings, backups, and integration access; review audit records. |
| Manager / approver | Review and approve controlled purchasing, stock adjustments, returns, and other configured actions. |
| Buyer | Maintain supplier information and prepare purchase requests/orders. |
| Receiver / stock operator | Record deliveries, counts, transfers, and permitted stock events. |
| Catalog operator | Maintain products, categories, units, barcodes, and supplier product references. |
| Finance / invoice operator | Record and review invoices, costs, taxes, discounts, and supporting documents. |
| Analyst / read-only user | View permitted operational records and reports without changing them. |
| Integration administrator | Configure, test, run, and disconnect optional connectors; review import/export results. |
| System process | Run approved background tasks such as imports or backups under a recorded initiating user or system identity. |

## 4. Priority and release notation

- **P0 — Required foundation:** Needed for a safe, independently usable first release.
- **P1 — Essential operations:** Needed for the first standalone inventory and purchasing release.
- **P2 — Later capability:** Valuable, but may follow the first usable release.
- **Release target:** **Foundation**, **Standalone operations**, or **Later**.

Each requirement has a stable ID. Acceptance criteria are measurable pass conditions; they are not a substitute for implementation tests and operational validation.

## 5. Functional requirements

### 5.1 Store, access, and audit

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-001 | P0 / Foundation | The system shall create and configure a store with a name, default locale, currency, time zone, and at least one stock location. | A newly configured store can be reopened with its settings intact; required fields cannot be omitted. |
| FR-002 | P0 / Foundation | The system shall authenticate named users and provide a controlled initial administrator setup. | Unauthenticated users cannot access protected business records; the initial setup cannot silently create a publicly accessible administrator account. |
| FR-003 | P0 / Foundation | The system shall enforce permissions for viewing, creating, editing, approving, posting, exporting, and administering records. | For each protected action, an allowed role succeeds and a role without permission is denied without changing data. |
| FR-004 | P0 / Foundation | The system shall record audit events for sign-in/security events and material create, edit, approval, posting, export, and administrative actions. | An authorized reviewer can identify actor, action, record, time, and outcome; business users cannot alter or erase audit events through normal workflows. |
| FR-005 | P0 / Foundation | The system shall isolate each store's business records and access rules. | A user assigned to one store cannot view or change another store's records through the interface or application API. |
| FR-006 | P0 / Foundation | The system shall allow an administrator to disable a user and revoke that user's future access. | A disabled user's next protected request is denied; prior audit history remains attributable to that user. |

### 5.2 Product catalog and suppliers

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-010 | P1 / Standalone operations | Authorized users shall create, view, update, archive, and search products using an internal SmartStock identity. | A product remains identifiable after edits; products referenced by posted transactions cannot be hard-deleted through ordinary workflows. |
| FR-011 | P1 / Standalone operations | A product shall support a name, optional description, category, base unit, active status, and configurable reorder threshold. | Required fields and valid references are enforced; archived products remain visible in historical transactions. |
| FR-012 | P1 / Standalone operations | A product shall support one or more identifiers, including barcodes, with identifier type and uniqueness rules within the store. | Duplicate active identifiers are rejected or routed to an explicit resolution workflow; lookup by identifier returns the correct product or a clear no-match result. |
| FR-013 | P1 / Standalone operations | Authorized users shall manage categories, units of measure, and explicit unit conversions. | A conversion must identify source and target units and a positive conversion factor; unsupported or ambiguous conversions are not silently applied. |
| FR-014 | P1 / Standalone operations | Authorized users shall create, edit, search, and archive supplier profiles. | A supplier has an internal identity and required display name; archival preserves purchase and invoice history. |
| FR-015 | P1 / Standalone operations | The system shall record supplier-specific product identifiers and supplier catalog details separately from the product's internal identity. | The same product can have references from multiple suppliers; external or supplier IDs do not become the SmartStock primary identity. |
| FR-016 | P2 / Later | The system should provide duplicate-product candidates based on identifiers and configurable similarity signals. | Candidates are presented for human review; the system does not merge or rewrite official product records automatically. |

### 5.3 Purchasing and receiving

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-020 | P1 / Standalone operations | An authorized buyer shall create and edit a purchase request with supplier, products, quantities, units, and optional expected prices. | Invalid products, suppliers, units, or non-positive quantities cannot be submitted; edits are auditable. |
| FR-021 | P1 / Standalone operations | An authorized approver shall approve or reject a purchase request according to configured approval permissions. | The decision, actor, time, and optional reason are recorded; rejected requests cannot be received as approved orders. |
| FR-022 | P1 / Standalone operations | An authorized buyer shall create a purchase order from an approved request or directly when permission allows. | The order has a unique internal number, supplier, lines, quantities, units, and lifecycle status; creation does not change stock. |
| FR-023 | P1 / Standalone operations | The system shall support partial and complete goods receiving against a purchase order. | Each receipt records actual quantities, location, date, and operator; cumulative received quantity is visible and cannot be accidentally counted twice. |
| FR-024 | P1 / Standalone operations | Posting a goods receipt shall create corresponding stock ledger movements in the same business transaction. | If posting fails, neither the receipt nor its stock effects are partially committed; a successful receipt appears in the balance and ledger exactly once. |
| FR-025 | P1 / Standalone operations | The system shall record discrepancies between ordered and received goods, including overage, shortage, damage, or substitution. | Discrepancies are visible for review and do not alter ordered quantities without an authorized, audited edit. |
| FR-026 | P2 / Later | The system should support supplier returns linked to receipts or purchases. | A posted return records reason, quantities, location, approval where required, and a reversing/return stock movement. |

### 5.4 Inventory and stock ledger

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-030 | P1 / Standalone operations | The system shall maintain stock by product and location using a stock movement ledger as the authoritative history. | Each posted movement has an internal ID, product, location, quantity delta, unit, event type, time, actor/source, and link to its originating record. |
| FR-031 | P1 / Standalone operations | The system shall display a stock balance derived from posted movements. | For a selected product and location, the displayed balance equals the sum of applicable posted ledger movements; pending drafts are excluded. |
| FR-032 | P1 / Standalone operations | Authorized users shall record stock transfers between locations. | A transfer creates linked outgoing and incoming movements atomically; source and destination cannot be the same location. |
| FR-033 | P1 / Standalone operations | Authorized users shall perform a physical inventory count and review the variance from the recorded balance. | A count records counted quantity, expected quantity, variance, location, time, and counter; starting a count alone does not change stock. |
| FR-034 | P1 / Standalone operations | Posting an approved count adjustment shall create an auditable stock movement. | The adjustment requires configured approval where applicable; posting is atomic and the resulting balance reconciles to the ledger. |
| FR-035 | P1 / Standalone operations | The system shall support authorized stock corrections with a reason and, where configured, approval. | Posted movements are not silently overwritten; a correction is represented by a new movement linked to the original event. |
| FR-036 | P2 / Later | The system should support batch/lot and expiry tracking where enabled for a product. | A tracked receipt captures batch and expiry information; expiry alerts are derived from recorded dates and configured thresholds. |
| FR-037 | P1 / Standalone operations | The system shall prevent duplicate posting of the same stock-affecting business event. | Repeating a submission or retry with the same internal event identity does not create a second movement or change the balance twice. |

### 5.5 Invoices, costs, and documents

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-040 | P1 / Standalone operations | Authorized users shall record an invoice with supplier, invoice number, issue date, currency, line items, and optional supporting document. | Required fields are validated; invoice lines preserve quantity, unit price, discounts, taxes, and line totals as entered or explicitly calculated. |
| FR-041 | P1 / Standalone operations | The system shall flag likely duplicate supplier invoices using supplier identity and invoice reference, with optional date/amount signals. | A duplicate candidate is shown before finalization; users can resolve it with a recorded reason where authorized. |
| FR-042 | P1 / Standalone operations | The system shall maintain purchase price history linked to received or invoiced product lines. | Each cost record identifies source document, supplier, product, quantity basis, currency, and effective date; records are not overwritten by newer prices. |
| FR-043 | P0 / Foundation | Financial quantities shall use decimal-safe precision and explicit currency representation. | Repeated calculations do not introduce binary floating-point artifacts; rounding rules are explicit at display or document boundaries. |
| FR-044 | P1 / Standalone operations | The system shall represent tax, discounts, and additional purchase costs as explicit values with their applicable basis. | Totals can be reconciled to the stored lines and adjustments; missing or ambiguous tax/cost treatment is not inferred silently. |
| FR-045 | P2 / Later | The system may calculate average cost or FIFO layers when the selected costing policy is approved. | The active policy and inputs are traceable; changing a policy does not rewrite source invoices or stock movements. |
| FR-046 | P1 / Standalone operations | The system shall store document metadata and provide authorized access to linked files. | A document is associated with an internal record, has a recorded filename/type/size and uploader, and unauthorized users cannot retrieve it. |

### 5.6 Reports and operational alerts

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-050 | P1 / Standalone operations | Authorized users shall view current stock by product and location and inspect the movement history behind a balance. | Report filters identify their store/location and time scope; a balance can be reconciled to its ledger entries. |
| FR-051 | P1 / Standalone operations | Authorized users shall view purchase order, receipt, supplier, and purchase cost history. | Report totals and statuses agree with the underlying records for the same filter criteria. |
| FR-052 | P1 / Standalone operations | The system shall identify items at or below a configured reorder threshold. | The alert identifies product, location, current balance, and threshold; it does not create or approve a purchase order. |
| FR-053 | P2 / Later | The system may provide expiry, slow-moving, price-anomaly, supplier comparison, and demand-forecast insights. | Each insight exposes its input period/source and confidence or limitations where relevant; users decide whether to act. |

### 5.7 Integrations and imports/exports

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-060 | P0 / Foundation | The standalone application shall open and permit its core local workflows when no connector is configured or available. | Removing or failing a connector does not prevent local catalog, supplier, purchasing, inventory, or report screens from operating. |
| FR-061 | P2 / Later | Each connector shall declare its name, version, capabilities, read/write scope, formats, authentication needs, compatibility, and limitations. | The capability screen lists only implemented and configured operations; unsupported operations are clearly unavailable. |
| FR-062 | P2 / Later | External record identifiers shall be stored as mappings to internal SmartStock records. | Removing an external mapping does not delete the internal record; multiple external systems can map to the same internal record subject to uniqueness rules per connector. |
| FR-063 | P1 / Standalone operations | The system shall support a documented, staged manual CSV import for approved catalog data before records are committed. | The user can preview rows, field mappings, validation errors, and likely duplicates; invalid rows are not silently accepted. |
| FR-064 | P2 / Later | The system should support Excel import/export and additional connector types through replaceable adapters. | A connector can be added without changing core business identity rules; every operation's permissions and effects are explicit. |
| FR-065 | P2 / Later | Imports shall be repeatable and idempotent when a stable source key or import identity is available. | Reprocessing the same source does not duplicate committed records; conflicts are reported for resolution. |
| FR-066 | P2 / Later | Exports that affect an external system shall require explicit authorization and approval appropriate to the operation. | The preview names the target, records, and operation; the result and initiating/approving users are audited. |
| FR-067 | P2 / Later | A connector shall support capability inspection, connection validation, permission testing, and disconnect where applicable. | Tests report success/failure and timestamp without performing an unapproved write; disconnect preserves SmartStock records and mappings. |

### 5.8 AI and OCR assistance

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-070 | P2 / Later | OCR shall be provided through a replaceable adapter and may extract candidate invoice fields and lines. | Extracted values retain source/document references and confidence where available; users can correct them before official records are created. |
| FR-071 | P2 / Later | AI or matching services may propose product matches, duplicate candidates, price anomalies, or reorder recommendations. | Recommendations are labeled as suggestions, include available evidence, and require an authorized human action to affect official records. |
| FR-072 | P0 / Foundation | Core business workflows shall not require an AI service or internet connection where local operation is technically feasible. | With AI/network services unavailable, users can still perform essential local catalog, purchasing, receiving, inventory, and reporting workflows. |
| FR-073 | P0 / Foundation | No AI-generated recommendation shall silently create or post an inventory, purchase, or financial transaction. | Before and after a recommendation is generated, official records remain unchanged until a permitted user confirms the applicable workflow. |

### 5.9 Backup, recovery, and configuration

| ID | Priority / target | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-080 | P0 / Foundation | The system shall provide a documented backup procedure covering the database, document files, and necessary configuration. | A backup has a timestamp and can be identified as complete; backup access is restricted to authorized users. |
| FR-081 | P0 / Foundation | The system shall provide a documented restore procedure and record the outcome of a restore operation. | A restored copy can open and retrieve representative records; an unsuccessful restore reports an actionable error and does not present partial data as successful. |
| FR-082 | P0 / Foundation | Configuration shall separate environment/deployment settings from business records and secrets. | Missing or invalid required configuration produces a clear error; credentials are not written to ordinary audit messages or logs. |
| FR-083 | P0 / Foundation | Core workflows shall preserve committed data across normal application restart. | After a clean restart, committed catalog, supplier, purchase, receipt, stock, and audit records remain available. |

## 6. Business rules

1. **Internal identity:** Every SmartStock product, supplier, user, store, location, and business transaction has an internal identity. External identifiers are mappings or source references.
2. **Store boundary:** Every business record belongs to a store; access is scoped to the user's authorized store membership.
3. **Ledger authority:** Posted stock movements are the authoritative record of stock change. Balances are derived or reconciled projections.
4. **Atomic posting:** A stock-affecting event and its ledger movements either commit together or do not commit.
5. **No silent edits to history:** Posted financial and stock records are corrected with linked, auditable events or permitted reversal workflows, not invisible overwrites.
6. **Approval separation:** Where an approval is required, the system records requester and approver. A user may not approve their own action when the store policy prohibits self-approval.
7. **Human control:** AI/OCR outputs are candidate data. They become official only after permitted review and explicit confirmation.
8. **Idempotency:** Repeating a committed import or posting request with the same stable identity must not duplicate business effects.
9. **Decimal financials:** Money and rates use decimal-safe representations, currency codes, and defined rounding behavior.
10. **Unit integrity:** Quantities are stored with units. Conversions occur only through an explicit valid conversion path.
11. **Archival:** Records referenced by posted transactions are archived/inactivated rather than deleted through ordinary user operations.
12. **Integration truthfulness:** A connector exposes only verified, configured capabilities. “Universal” describes extensibility, not guaranteed compatibility.
13. **No external write by implication:** Import, sync, and export directions and effects are explicit; external write actions require authorization and configured approval.
14. **Offline behavior:** External service outages do not corrupt local records or block supported standalone workflows. Queued integration work must show status and recoverable errors.

## 7. Data validation requirements

- Required names, references, dates, and statuses must be present before a record can be submitted or posted.
- Quantities and conversion factors must be positive where a line represents an amount received/ordered; signed deltas are permitted only in correctly typed ledger events.
- A posted receipt cannot cause the related order's received quantity to exceed its ordered amount without an explicit over-receipt permission and recorded reason.
- Product identifiers must follow store-level uniqueness rules; collisions must be surfaced to the user.
- Invoice line arithmetic, document totals, tax, discounts, and currency precision must be validated using documented rounding rules.
- Dates and timestamps must be stored consistently and displayed in the store's configured time zone.
- A movement must reference an active or historically valid product and location, a supported unit, an event type, an actor/source, and an originating record when applicable.
- Invalid imports must return row-level reasons and must not silently coerce unrecognized columns, units, currencies, or identifiers.
- File uploads must enforce configurable file type and size limits and must be stored with record-level authorization.
- Validation failures must identify the field or row and a correction path without exposing secrets or internal stack traces.

## 8. Error handling and recovery

| Situation | Required behavior |
|---|---|
| Permission denied | Reject the operation, leave data unchanged, and record a security-relevant audit event where appropriate. |
| Validation failure | Return field/row-level errors; do not partially post the business transaction. |
| Database or transaction failure | Roll back the full operation; provide a safe retry path and a correlation/reference ID for support. |
| Duplicate submission/retry | Return the existing outcome or a duplicate notice; do not repeat stock or financial effects. |
| Import row conflict | Preserve staging data, explain the conflict, and allow authorized correction or exclusion before commit. |
| Connector unavailable | Keep local workflows available; mark the connector/job unavailable or failed and retain retryable work without claiming success. |
| AI/OCR unavailable | Allow manual entry and review; do not block invoice, purchase, or stock workflows. |
| Insufficient or negative stock | Apply the store's explicit negative-stock policy; when not configured, block the operation and explain the quantity/location conflict. |
| Restore/backup failure | Report failure clearly, preserve the prior usable data state, and record the operation outcome. |
| Unexpected application error | Show a non-sensitive message and reference ID; retain diagnostic detail only in access-controlled logs. |

## 9. Permission matrix baseline

This is a minimum suggested permission baseline. Store policy may grant finer permissions. Approval actions must remain distinct from preparation/posting where required.

| Capability | Owner/admin | Manager | Buyer | Receiver | Catalog operator | Finance | Analyst | Integration admin |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Manage store/users/roles | Yes | No | No | No | No | No | No | No |
| Read operational records | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Scoped |
| Maintain catalog/suppliers | Yes | Yes | Supplier scope | No | Yes | Read | No | No |
| Prepare purchase requests/orders | Yes | Yes | Yes | No | No | Read | No | No |
| Approve purchase/stock actions | Yes | Yes | No by default | No | No | No by default | No | No |
| Receive goods / count stock | Yes | Yes | No | Yes | No | No | No | No |
| Post stock adjustments/returns | Yes | Yes | No | Configurable | No | No | No | No |
| Manage invoices/costs | Yes | Yes | Read | No | No | Yes | Read | No |
| View reports | Yes | Yes | Scoped | Scoped | Scoped | Scoped | Yes | Scoped |
| Configure/test connectors | Yes | No by default | No | No | No | No | No | Yes |
| Approve external writes/exports | Yes | Configurable | No | No | No | No | No | Prepare only by default |
| Manage backups/restores | Yes | No by default | No | No | No | No | No | No |

All denials must be enforced in the application layer, not only by hiding interface controls.

## 10. End-to-end workflows

### WF-01: Set up a standalone store

1. Administrator creates the store and first location, sets locale/currency/time zone, and creates named users.
2. Administrator assigns roles and confirms access boundaries.
3. Catalog operator enters products, identifiers, units, and suppliers manually or through a staged import.
4. Users can close and reopen the application and continue with saved records without configuring a connector.

**Success:** The store can manage its internal catalog and records independently of external software and network services.

### WF-02: Purchase and receive goods

1. Buyer creates a purchase request with supplier and requested lines.
2. Approver reviews and approves/rejects it; decision is audited.
3. Buyer creates an order and shares/records it through the store's chosen process.
4. Receiver records full or partial delivery, actual quantities, destination, and discrepancies.
5. Authorized user posts receipt; receipt and stock movements commit atomically.
6. Users inspect updated balances, movement history, and purchase history.

**Success:** Ordered, received, and outstanding quantities are distinct; a receipt affects stock exactly once.

### WF-03: Count and adjust inventory

1. Authorized user opens a count for a location and records observed quantities.
2. System displays expected quantity and variance without changing stock.
3. Manager reviews the variance and approves where required.
4. Posting creates a linked adjustment movement and updates the derived balance.

**Success:** The count is traceable, approval is recorded, and balance reconciles to ledger.

### WF-04: Record and review supplier invoice

1. Finance operator enters supplier, invoice reference, date, currency, lines, tax/discount values, and supporting document.
2. System checks for likely duplicates and validates arithmetic and references.
3. Operator corrects discrepancies and confirms the invoice under their permission.
4. Cost history links to invoice and relevant receipt/product lines; source values remain available.

**Success:** The invoice and costs are traceable, decimal-safe, and do not silently create stock movements unless explicitly linked to a posted receipt.

### WF-05: Stage and commit a catalog import

1. Authorized user selects a supported CSV template/file and maps columns to internal fields.
2. System previews parsed rows, validation errors, duplicate candidates, and intended actions.
3. User resolves, excludes, or corrects rows and confirms import.
4. System commits valid approved rows idempotently and reports counts, conflicts, and failures.

**Success:** Re-running the same import does not duplicate records when stable source identities are present; no invalid row is silently imported.

### WF-06: Review an AI/OCR suggestion (later capability)

1. User submits an invoice/document for extraction or requests a product/reorder suggestion.
2. System presents source, extracted/candidate values, and available confidence/evidence.
3. Authorized user edits and explicitly confirms a normal business workflow.
4. Only that approved workflow can create or post official records.

**Success:** Declining or losing the AI service leaves official records unchanged and manual workflows available.

## 11. Non-functional requirements and quality gates

These requirements guide solution selection; numeric service-level targets should be set after hardware and deployment discovery.

- **Reliability:** Business transactions affecting stock or money are atomic and recoverable after application failure.
- **Data integrity:** Stock balances reconcile to ledger movements; committed records survive normal restarts; imports are idempotent where stable source identity is available.
- **Security:** Authentication, permission enforcement, store isolation, protected documents, and access-controlled diagnostics apply to all supported interfaces.
- **Usability/localization:** Interface text supports Arabic and French with an extensible localization structure; layout and date/number display follow selected locale without changing stored meaning.
- **Maintainability:** Business rules remain independent of UI, persistence implementation, AI/OCR providers, and external connectors.
- **Portability:** Deployment/runtime choices remain open until Windows version and architecture are verified; business logic must not depend on a specific installer.
- **Offline operation:** Essential local operations work without AI and internet; exact offline installation/update behavior is a deployment decision.
- **Backup and restore:** A documented backup must include all required data and files; restore must be operationally verified before pilot use.
- **Observability:** Errors have safe user-facing messages and reference IDs; diagnostic logs avoid secrets and are access-controlled.
- **Performance:** Response-time, catalog-size, and concurrent-user thresholds remain to be agreed after profiling the target store computer and expected workload.

## 12. Dependencies and assumptions

| ID | Dependency / assumption | Required resolution |
|---|---|---|
| DEP-01 | Target store computer OS version, edition, CPU architecture, memory, storage, and privileges are unknown. | Discover and document before choosing runtime packaging or installer. |
| DEP-02 | Exact Logica vendor/product/version and licensed integration methods are unknown. | Identify product/version and review vendor documentation with authorized access before any connector design. |
| DEP-03 | Store operating workflows, document formats, stock policies, and approval thresholds are not yet documented. | Validate workflows with the store owner/operators before baselining detailed rules. |
| DEP-04 | Currency, tax conventions, invoice formats, locale, and rounding policies require confirmation. | Record store-approved values before financial calculations are finalized. |
| DEP-05 | Single-device pilot is the initial target; multi-user/network use is future scope. | Confirm whether simultaneous local users are expected and verify SQLite suitability against that workload. |
| DEP-06 | Existing database architecture V1.0 was referenced but not present in the workspace at drafting time. | Reconcile entity names and constraints against that specification before schema design. |
| DEP-07 | Backup destination, retention, encryption, and restore ownership are not specified. | Agree practical local/USB backup and recovery procedure for pilot. |
| DEP-08 | CSV data sources and quality are unknown. | Collect representative authorized sample files and define import templates/mapping rules. |

## 13. Acceptance and sign-off

The FRS is ready for approval when stakeholders confirm:

1. The roles and initial permission boundaries represent store operations.
2. The purchase, receiving, count, adjustment, invoice, and import workflows reflect actual practice.
3. P0 and P1 requirements are sufficient for a first independently useful release.
4. Negative stock, self-approval, over-receipt, invoice duplicate resolution, and correction policies are explicit.
5. Currency, tax, units, locations, backup, and audit expectations are understood.
6. Open dependencies are assigned an owner and discovery step.

Approval of this document authorizes requirements baseline refinement only unless separately stated; implementation sequencing should then proceed in small milestones against the approved acceptance criteria.

| Reviewer | Role | Decision | Date | Notes |
|---|---|---|---|---|
|  | Store owner | Pending |  |  |
|  | Operations representative | Pending |  |  |
|  | Finance representative | Pending |  |  |
|  | Technical reviewer | Pending |  |  |

## 14. Open questions for review

1. What currency, tax rates/rules, and rounding conventions should the pilot use?
2. Are negative stock balances ever permitted? If so, which roles and scenarios may create them?
3. Must a purchase approver differ from its requester? What thresholds require approval?
4. Are receipt overages allowed, and what approvals/limits apply?
5. Which units, conversions, product variants, batches, and expiry tracking are needed in the pilot?
6. How should initial stock be established and verified during migration/setup?
7. What file types and maximum document sizes are practical for invoices and supporting records?
8. What backup medium, frequency, retention period, and restore operator fit the store workflow?
9. What Arabic/French language should be the initial default, and are product names commonly bilingual?
10. Which reports are required on day one, and what date ranges/exports are needed?

---

**End of FRS V1.0 draft**

