# OFBiz Cross-Project Validation

This file documents the cross-project validation of the debt visibility framework
in Apache OFBiz (Open For Business), an enterprise resource planning system.
The validation was conducted independently of the Fineract case derivation,
following a prospective protocol established before systematic search began.

**What was being validated:** Whether the two structural visibility classes derived
from Fineract — detectability failure and salience failure — also appear in an
independent codebase with different architecture, language mix, and contributor
community.

**What was not being validated:** Accountability failure. No convincing OFBiz
analogue was identified within the scoped modules. That negative result is
reported as a boundary condition of the framework, not suppressed.

**Repository examined:** https://github.com/apache/ofbiz-framework
(trunk), plus ofbiz-plugins-work and ofbiz-plugins-bare for cross-reference.

---

## Part 1: Detectability Failure (Case B equivalent)

### Prospective protocol

OFBiz modules were scoped and a systematic search procedure was defined
before any candidate was examined. The three visibility classes were defined
before OFBiz search began. Classification criteria were applied consistently.

### Three-condition definition

A service declaration was classified as a detectability failure if:

1. A service declaration exists in a servicedef XML file with a declared
   implementation class and method
2. The declared method does not exist in that class (or any class), at any
   point in the repository's full history
3. The declaration is not self-documenting its own disablement (e.g., commented
   out, carry an explicit deprecation notice)

Losing any one condition disqualifies the candidate.

### Reachability classification

Confirmed instances are further classified into three profiles:

- **Reachable phantom:** A live controller route, SOAP/REST export, service-engine
  condition action, or scheduled job can actually trigger the missing implementation.
  An outside actor can produce a runtime failure without special tooling.
- **Dormant declaration:** No invocation path found through the examined channels
  (no export, no controller wiring, no service-engine condition, no scheduler,
  no internal caller). Inert with respect to the paths examined, not a claim
  that invocation is impossible under all generic dispatch.
- **Orphaned-by-substitution:** A route or wiring point exists under the same name
  but has always pointed to a different, working service since the route was created.
  The phantom has no live path; only the name is shared.

These profiles describe different runtime risk levels. They are not interchangeable.

---

### Confirmed Instances (7 total)

#### B1 — `creditOrderPaymentPreference`

| Field | Value |
|-------|-------|
| Location | `applications/accounting/servicedef/services_paymentmethod.xml:331–332` |
| Declaration | `engine="java"`, `location="...PaymentGatewayServices"`, `invoke="creditOrderPaymentPreference"`, `auth="true"` |
| Origin commit | `0f1bbb7b0d49d1834b9ed39494a6b383d589b3b5` |
| Origin date | 2009-04-28 (Marco Risaliti, OFBIZ-2369) |
| Context | Added in same commit as working sibling `refundOrderPaymentPreference` — a designed-but-unfinished symmetric API |
| Implementation status | Never implemented in any `.java`/`.groovy` file at any point in history |
| Controller wiring | None |
| Callers | None |
| File maintenance | 56 commits, most recent 2021-07-20 |
| Plugins verification | Absent from ofbiz-plugins-work and ofbiz-plugins-bare |
| Age | ~17 years (2009–2026) |
| Reachability | **Dormant declaration** |

#### B2 — `testPermissionExistingTesting`

| Field | Value |
|-------|-------|
| Location | `framework/service/servicedef/services_test_se.xml:193–194` |
| Declaration | `engine="java"`, `location="...ServiceEngineTestPermissionServices"`, `invoke="testPermissionExistingTesting"`, `auth="true"` |
| Origin commit | `f4aab777a516b5c8c1fc7b6057b12e8d577f45d1` |
| Origin date | 2019-08-30 (Nicolas Malin, OFBIZ-7113) |
| Context | Six services declared in one commit block; five route to real methods; the sixth routes to an invoke name never written |
| Implementation status | Implementation class contains exactly two methods; no third method ever existed; pickaxe returns zero hits |
| Controller wiring | None |
| Callers | None |
| File maintenance | 24 commits, most recent 2026-06-02 |
| Plugins verification | Absent from both repositories |
| Age | ~7 years (2019–2026) |
| Reachability | **Dormant declaration** |

#### B5 — `deleteProductionRunRoutingTask`

| Field | Value |
|-------|-------|
| Location | `services_production_run.xml:222–223`, `auth="true"` |
| Origin commit | `94fe6d1cdaa176365cce77422f7ce2a8b1ad0534` |
| Origin date | 2006-07-01 (David E. Jones, initial SVN import) |
| Implementation status | Never implemented at any point in history |
| Controller wiring | `controller.xml:600` — request-map exists but invokes `deleteWorkEffort` (a working service), not the phantom. Substitution commit: `6f873aa9b5`, Jacopo Cappellato, 2007-03-28, present from the moment the request-map was created |
| UI hyperlink | `ProductionRunForms.xml:234` — targets the URI and functions end-to-end, routing to `deleteWorkEffort`, not the phantom |
| Callers of phantom specifically | None |
| Plugins verification | Absent from both repositories |
| Age | ~20 years (2006–2026) |
| Reachability | **Orphaned-by-substitution** — route has always pointed to a different, working service |

#### B6 — `deleteProductionRunComponent`

| Field | Value |
|-------|-------|
| Location | `services_production_run.xml:245–246`, `auth="true"` |
| Origin commit | `94fe6d1cdaa176365cce77422f7ce2a8b1ad0534` |
| Origin date | 2006-07-01 (same commit as B5) |
| Implementation status | Never implemented |
| Controller wiring | `controller.xml:654–659` — request-map invokes `removeWorkEffortGoodStandard`, not the phantom |
| UI hyperlink | `ProductionRunForms.xml:973` — functions via substitute |
| Callers of phantom specifically | None |
| Plugins verification | Absent from both repositories |
| Age | ~20 years (2006–2026) |
| Reachability | **Orphaned-by-substitution** — identical profile to B5 |

#### B7 — `updateLayoutImageOnly`

| Field | Value |
|-------|-------|
| Location | `applications/content/webapp/content/WEB-INF/controller.xml` (~line 1531–1536) |
| Declaration | `type="java"`, `invoke="updateLayoutImageOnly"`, `auth="true"` (direct wiring, not substitution) |
| Origin commit | `94fe6d1cdaa176365cce77422f7ce2a8b1ad0534` |
| Origin date | 2006-07-01 |
| Later touches | Five commits 2024–2026 modified surrounding file without implementing or removing the block |
| Implementation status | Never implemented in `LayoutEvents.java` or anywhere else |
| Sibling contrast | `updateLayoutImage` (line 173) and `updateLayoutSubContent` (line 556) are real, implemented methods in the same file |
| UI/template references | None — zero links to the URI anywhere |
| Failure mechanism | `JavaEventHandler.invoke()` calls `Class.getMethod(event.getInvoke(), ...)` then `Method.invoke(...)`; since the method does not exist on `LayoutEvents`, throws `NoSuchMethodException` at dispatch time (unhandled crash, distinct from Fineract's clean HTTP 400) |
| Plugins verification | Absent from both repositories |
| Age | ~20 years (2006–2026) |
| Reachability | **Reachable phantom** — live `controller.xml` entry with `auth="true"`; any plain HTTP request to `/control/updateLayoutImageOnly` hits it directly |

#### B8 — `addSeparator`

| Field | Value |
|-------|-------|
| Location | Trunk: `applications/order/webapp/ordermgr/WEB-INF/controller.xml:630–635`, `uri="addseperator"` [sic]. Ecommerce plugin: `ecommerce/webapp/ecommerce/WEB-INF/controller.xml:310–315` (different auth: `https="false" auth="false"`) |
| Origin | Both routes: `94fe6d1cdaa176365cce77422f7ce2a8b1ad0534`, 2006-07-01 |
| Ecommerce route history | Only two commits ever touch this line: origin (2006-07-01) and a zero-diff pure path rename at `80858bb5c166`, 2009-03-14 — no content change |
| Implementation status | Never implemented in `ShoppingCartEvents.java` or anywhere else |
| UI/template links | None in either trunk or ecommerce plugin |
| Failure mechanism | `JavaEventHandler.java` lines 97/99 — `k.getMethod(...)` then `m.invoke(...)`; throws `NoSuchMethodException` at dispatch (same class as B7) |
| Plugins presence | B8 is the only one of the seven present in the plugins repositories (ecommerce plugin route confirmed via `git grep HEAD`) |
| Age | ~20 years (2006–2026) |
| Reachability | **Reachable phantom, dual route, differing auth** — reachable via plain HTTP to `/control/addseperator` with no special tooling |
| Note | Trunk and plugins-repo commit hashes are repository-local and not directly comparable (different hash for same content, author, date) — this does not affect verification of any other instance |

#### B9 — `WebSiteCmsPreviewEvent`

| Field | Value |
|-------|-------|
| Location | `applications/content/webapp/content/WEB-INF/controller.xml:1696–1700` |
| Declaration | `request-map uri="WebSiteCmsPreview"`, `security auth="true" https="true"`, `type="java" invoke="execute"`, `path="org.apache.ofbiz.content.content.WebSiteCmsPreviewEvent"` |
| Origin commit | `5d65e6e84bbc24f65e4caf5d4798ad42130afc2b` |
| Origin date | 2016-04-11 (Jacques Le Roux, OFBIZ-4502) |
| Introduced with | A stale pre-ASF package name (`org.ofbiz.content.content.WebSiteCmsPreviewEvent`) — consistent with copy-paste from a stale reference |
| Mechanical rename | `834691742b77`, Deepak Dixit, 2016-07-16 (OFBIZ-6274) — repository-wide `org.ofbiz.*` → `org.apache.ofbiz.*` rename touched this line without verifying the class existed |
| Implementation status | Never implemented under either package name; confirmed by pickaxe across all `.java`/`.groovy` history |
| `invoke="execute"` uniqueness | Confirmed as the only occurrence repository-wide |
| Controller wiring | Single location, structurally identical to B7's request-map (same `auth="true" https="true"` gate, same `type="java"` dispatch, same webapp and servlet mount) |
| Plugins verification | Absent from both repositories |
| Age | ~10 years (2016–2026) |
| Reachability | **Reachable phantom** — authenticated request to `/control/WebSiteCmsPreview` is structurally expected to reach `JavaEventHandler` via the same mechanism as B7/B8. Not independently confirmed by runtime invocation (structural analysis only). |

---

### Disconfirmed Candidates (3)

These three candidates were examined and excluded through structured classification.
Their rejection rationale is documented here because disconfirmed candidates
demonstrate that the framework was not applied to cherry-pick positive examples.

#### `storeForwardedEmail` — **Struck: explicitly disabled**

Service declaration exists but is wrapped in a well-formed XML comment added by
David E. Jones on 2009-02-15 (`f27b4eeaaa03`), during a component reorganization
that moved email services from `content` to `framework/common`. The same diff hunk
that changed the `location` package added the comment wrapper, documenting the
missing implementation inline and disabling it in the same commit.

A developer caught this Case-B-shaped issue by hand during unrelated work in 2009
and self-documented it. **Fails the invisibility criterion by definition** — the
declaration is openly disabled, not silently failing.

#### `getProjectTask` — **Struck: live implementation exists**

No declaration in trunk servicedef. Trunk callers reference it via
`CustRequestForms.xml:695,721`. The declaration exists in the plugins-work
servicedef (`projectmgr/servicedef/services.xml:207–208`, `engine="groovy"`),
with a live Groovy implementation at `ProjectServicesScript.groovy:592`
and 11 genuine call sites. Fully live and reachable. **Not a detectability failure.**

#### `checkOwnership` — **Struck: fails invisibility criterion**

Real Groovy service at `applications/content/servicedef/services.xml:887`.
When called via `DataResourcePermissionServices.xml`, the `<call-simple-method>`
tag (never updated to `<call-service>`) causes `SimpleMethod.getSimpleMethod()`
to return null, throwing a loud `MiniLangRuntimeException` naming the exact missing
method. Widely reachable via 12+ live `updateDataResource*` entity-auto services.
**Loud, immediately diagnosable exception — fails the invisibility criterion.**

---

### Additional Instance Found via Detector

#### `updateProductQuickAdminShipping` — **Verified, disposition pending**

A service declaration exists at `applications/product/servicedef/services.xml:60–64`
(`engine="java"`, `location="...ProductServices"`, `invoke="updateProductQuickAdminShipping"`).
Never implemented in `ProductServices`. An identically named method exists at
`ProductEvents.java:434` but with a controller-event signature
(`HttpServletRequest, HttpServletResponse`) rather than a service-engine signature
(`DispatchContext, Map`) — structurally incompatible. Both declarations share
origin commit `94fe6d1cdaa`, 2006-07-01. UI posts only to the controller URI;
no XML or programmatic caller invokes the service by name. Not exported.

The detector flagged this under a `FOUND_IN_WRONG_CLASS` hypothesis (implementation
relocated). Source verification disconfirmed that hypothesis: the two declarations
were born together in the same commit under different dispatch mechanisms, sharing
a name by coincidence, not by relocation.

**Reachability:** Dormant declaration (no wiring to the service specifically).

**Disposition:** Verified but not counted in the seven confirmed instances above.
Whether this is an eighth confirmed instance, a footnote demonstrating independent
detector capability, or held for a revision pass is an open question not resolved here.

---

### Detector Validation

The detector was run against the OFBiz search scope after the manual sweep.

- **Confirmed matches:** 6 (matching instances identified by hand)
- **False positives:** 0
- Additional detector-found instance: `updateProductQuickAdminShipping` (disposition pending, see above)

---

### Failure-Mode Taxonomy

The reachability profile (above) and the failure mode are independent axes.
Three distinct failure shapes are on record across both projects:

| Failure shape | Instances | Mechanism |
|---------------|-----------|-----------|
| Unhandled crash | OFBiz B7, B8; B9 (inferred from structural match, not runtime-tested) | `NoSuchMethodException` thrown directly by `JavaEventHandler`, uncaught |
| Caught, wrapped exception | OFBiz `updateProductQuickAdminShipping` | `NoSuchMethodException` caught and re-thrown as `GenericServiceException` |
| Dispatch-layer miss, coded response | Fineract `CLIENTCHARGE|INACTIVATE`, `EMAIL|UPDATE`, `WORKINGCAPITALLOAN|UPDATEDISCOUNT` | Handler-registry key miss → `UnsupportedCommandException` → HTTP 400 with specific error code |

---

## Part 2: Salience Failure (Case C equivalent)

### Pattern definition

The same three-condition definition used for Fineract Case C applies:

1. **Loop membership:** The lookup sits inside a loop
2. **Key invariance:** The lookup's key arguments are provably never reassigned
   within that loop
3. **Uncached status:** No explicit entity-engine caching mechanism is present
   at the lookup site

### Entity-engine caching determination

Before classifying any instance, the OFBiz entity-engine execution path was
traced to verify that uncached lookups actually issue SQL statements.

**Result:** When `useCache=false`, the call chain is:

```
GenericDelegator.findOne() / findList()
  → GenericHelperDAO.findByPrimaryKey() / findListIteratorByCondition()
    → GenericDAO.select() / selectListIteratorByCondition()
      → sqlP.executeQuery()    ← JDBC execution
```

`EntityQuery.use().from().where().queryOne()` routes to
`GenericDelegator.findList()`, then follows the same chain.

There is no intermediate caching layer between `GenericDAO` and JDBC when
`useCache=false`. **Each uncached loop iteration issues a SQL statement.**

This was confirmed across all three lookup API families used by the confirmed
instances: `findOne`, `findList`, and `EntityQuery`.

**Execution frequency qualifier:** Five of six confirmed instances are
conditionally executed within their loop. The correct claim for those is
**up to N SQL statements per invocation**. One instance (C5,
`GiftCertificateServices.java`) is unconditional: **N statements per invocation**
without qualification.

**Note on latency experiments:** No latency experiments were conducted for OFBiz.
The Fineract Case C latency null result established that source-level lookup
reduction does not reliably translate into end-to-end latency improvement under
all workloads (particularly under low entity diversity where JPA/ORM caching
absorbs repeated calls). No equivalent measurement was attempted for OFBiz.

---

### Confirmed Instances (6 total)

#### C1 — `InvoiceServices.java:606` (`createInvoiceForOrder`)

| Field | Value |
|-------|-------|
| File | `applications/accounting/src/.../InvoiceServices.java` |
| Pattern | `delegator.findList()` keyed on `orderItem.orderId` + `orderItem.orderItemSeqId` (method parameter, derived from outer scope) inside `itemAdjustments` loop |
| Module | accounting |
| Origin commit | `720a4b3877e` |
| Origin date | 2016-11-05 (Divesh Dutta, OFBIZ-7012) |
| Age | ~10 years (2016–2026) |
| Caching | None at lookup site |
| Gating | Fires only when adjustment type is `PROMOTION_ADJUSTMENT` with non-null `productPromoId` |
| SQL executions | **Up to N** |

#### C2 — `OrderManagerEvents.java:255–256` (`receiveOfflinePayment`)

| Field | Value |
|-------|-------|
| File | `applications/order/src/.../OrderManagerEvents.java` |
| Pattern | `OrderPaymentPreference.queryFirst()` keyed on `orderId` (method-level, not loop-derived) inside `paymentMethodTypes` loop |
| Module | order |
| Origin commit | `a3059c097d` |
| Origin date | 2020-06-16 (Priya Sharma, OFBIZ-7377) |
| Age | ~6 years (2020–2026) |
| Caching | None at lookup site |
| Gating | Fires only when `UtilValidate.isEmpty(paymentReference)` |
| SQL executions | **Up to N** |

#### C3 — `OrderServices.java:1749` (`calcTax` context)

| Field | Value |
|-------|-------|
| File | `applications/order/src/.../OrderServices.java` |
| Pattern | `EntityQuery.use().from("PostalAddress").where(facilityId).queryOne()` keyed on `originFacilityId` (assigned before loop at line 1688) inside `shipGroups` loop |
| Module | order |
| Origin commit | `d37ce8cb21` |
| Origin date | 2008-07-18 (Scott Gray) — predates OFBiz's 2010 move to Apache |
| Age | ~18 years (2008–2026) |
| Caching | None at lookup site |
| Gating | Three nested gates: `shippingAddress == null` AND `facilityId != null` AND `facilityContactMech != null` |
| SQL executions | **Up to N** |
| Note | The gating was previously under-specified as a single condition; source verification confirmed three sequential nested gates |

#### C4 — `ShipmentServices.groovy:~968` (`createShipmentForFacilityAndShipGroup`)

| Field | Value |
|-------|-------|
| File | `applications/product/groovyScripts/.../ShipmentServices.groovy` |
| Pattern | `from('OrderRole')` (resolves via `GroovyBaseScript` to `EntityQuery.use(delegator).from('OrderRole')`) keyed on `orderHeader.orderId` (method parameter) inside `orderItemShipGroupList` loop |
| Module | product |
| Origin commit | `c177bccda0` |
| Origin date | 2009-06-06 (David E. Jones applying Pranay Pandey's patch, OFBIZ-2570) |
| Migration | Originally XML minilang; ported to Groovy 2020 by Sebastian Berg (OFBIZ-11462) — pure port, no logic change |
| Age | **~17 years** (from 2009 origin, not ~5 years from Groovy port date) |
| Caching | None at lookup site (confirmed in both Groovy version and 2009 minilang origin) |
| Gating | Fires only when `partyIdFrom` unresolved from vendor/facility owner |
| SQL executions | **Up to N** |
| Note | Sibling facility lookup a few lines away in the same method was correctly hoisted and cached by the same author |

#### C5 — `GiftCertificateServices.java:894` (`giftCertificatePurchase`)

| Field | Value |
|-------|-------|
| File | `applications/accounting/src/.../GiftCertificateServices.java` |
| Pattern | `ProductStoreEmailSetting` lookup keyed on `productStoreId` (method parameter) + literal `"PRDS_GC_PURCHASE"` inside `qtyLoop` for-loop |
| Module | accounting |
| Origin | File introduced in 2006 SVN import; no specific author traced for this method |
| Age | ~20 years (2006–2026) — oldest confirmed Case C instance |
| Caching | None at lookup site |
| Gating | **None** — runs on every loop iteration unconditionally |
| SQL executions | **N (unconditional)** — the only unconditional instance among the six |

#### C6 — `CommunicationEventServices.java:546` (`sendEmailToContactList`)

| Field | Value |
|-------|-------|
| File | `applications/party/src/.../CommunicationEventServices.java` |
| Pattern | `WebSite` lookup via `EntityQuery.use().queryOne()` keyed on `contactList.verifyEmailWebSiteId` (method-level, not loop-derived) inside per-recipient send loop |
| Module | party |
| Origin commit | `a31572006b` |
| Origin date | 2010-12-24 (Hans Bakker) |
| Age | ~16 years (2010–2026) |
| Caching | None at lookup site |
| Gating | `contactListPartyStatus != null` (plus upstream email-validity filter) |
| SQL executions | **Up to N** |
| Note | Line number drifted from 397 (2010) to 546 (2026) — confirmed via pickaxe and current-tree grep. Modernized from `delegator.findOne()` to `EntityQuery` but logic unchanged. |

---

### Authorship distribution

Confirmed instances span 6 independent authors across 5 modules:

| Author | Instance | Year |
|--------|----------|------|
| Divesh Dutta | C1 | 2016 |
| Priya Sharma | C2 | 2020 |
| Scott Gray | C3 | 2008 |
| David E. Jones / Pranay Pandey | C4 | 2009 |
| Hans Bakker | C6 | 2010 |
| C5 | Not traced (2006 SVN import) | ~2006 |

This distribution is consistent with the paper's claim that the pattern is not
one developer's coding habit. Each instance has an independent commit history.

---

### Disconfirmed Candidates (145 total)

145 candidates were examined and rejected. The rejection counts and rationale
are documented here because they constitute explicit negative cases for the
classification criteria.

#### Java/Groovy disconfirmed (5)

| Candidate | Location | Rejection reason |
|-----------|----------|-----------------|
| `TaxAuthorityServices.java:557` | `PartyRelationship/GROUP_ROLLUP` | `.cache()` present at origin (2006 SVN import used `findByAndCache`). Fails condition 3. |
| `ShoppingCartItem.java:1021` | `TechDataCalendarExcDay` | Two arguments: `calendarId` (invariant) AND `exceptionDateStartTime` (reassigned every iteration via `dayCount++`). Fails condition 2. Per-day date lookup — correct code. |
| `ProductPromoWorker.java:477` | `OrderProductPromoCode` | Lookup sits **outside** the `productPromo` loop at the same indentation level, executing once before the loop starts. Fails condition 1. False positive from line proximity. |
| `GiftCertificateServices.java:1124` | `giftCertificateReload` | Brace count confirms loop is already closed before line 1124 (Opens=14, Closes=14). Fails condition 1. Line 894 (C5) confirmed as the genuine instance. |
| "Two hits in qtyLoop" framing | `GiftCertificateServices` lines 894 + 1124 | Both claimed inside `qtyLoop` in `giftCertificateProcessor`. False: each sits in a different method entirely. Framing struck. Line 894 is C5; line 1124 is outside any loop. |

#### FTL disconfirmed (140 total)

FTL (FreeMarker Template Language) files were searched for `delegator.findOne()`
and `EntityQuery.use()` patterns. 101 delegator hits and 44 EntityQuery chains
were examined across 23 FTL files.

| Rejection category | Count | Condition failed | Example |
|-------------------|-------|------------------|---------|
| Explicit caching present | 53 | Condition 3 | `true` cache flag; `.cache()` (no arg, confirmed sets `useCache=true`) |
| Lookup key variant (references loop variable) | 65 | Condition 2 | Key is loop variable field (e.g., `visitObj.visitId`, per-row lookup) |
| Lookup outside loop | 13 | Condition 1 | Inside conditional only, or after loop closes |
| EntityQuery chains reclassified after manual verification | 9 (subset of above) | Various | Key-variant (5) or outside-loop (7, with 3 overlap) after manual context check |

**Arithmetic note:** The 9 EntityQuery reclassifications are a subset of the
65 key-variant and 13 outside-loop counts, not additive. Total FTL disconfirmed
= 53 + 65 + 13 = 131, plus 9 that were initially miscategorized and corrected.
The 140 total reflects the corrected count; the 9 reclassified are distributed
within the three category totals above.

---

### Scanner limitations identified

Three distinct scanner limitations surfaced during Case C analysis:

1. **Single-hop reassignment blindness:** A variable textually different from
   the loop variable but assigned from it a few lines into the loop body is not
   recognized as loop-variant by name matching alone.

2. **Iterate-map dual binding:** Minilang's `iterate-map key="K" value="V"` introduces
   two loop-scoped variables; an initial scanner version tracked only `value`,
   producing a false positive.

3. **Condition/map object opacity:** When a finder's argument is a pre-built
   `EntityCondition`/`Map` object, a scanner cannot determine whether the loop
   variable is embedded inside it without resolving the construction site.

None of these are fixable by more regex; full data-flow analysis is required.
This is consistent with the paper's argument that tooling to detect this pattern
class does not currently exist.

---

## Summary

| Aspect | Fineract | OFBiz | Cross-project result |
|--------|----------|-------|---------------------|
| Detectability failure: confirmed | CLIENTCHARGE\|INACTIVATE (~11y) + Teller/Cashier (~12y, supporting) + EMAIL\|UPDATE + WORKINGCAPITALLOAN\|UPDATEDISCOUNT | 7 confirmed (3 reachable phantom, 2 dormant, 2 orphaned-by-substitution); ages ~7–20y | **Strong replication** across different dispatch architectures |
| Detectability failure: disconfirmed | — | 3 candidates (explicit disable; live implementation; loud exception) | Classification criteria exclude non-invisible cases |
| Salience failure: confirmed | 7 instances, 5 modules, 4–13y persistence | 6 instances, 4 modules, 6–20y persistence | **Structural replication** |
| Salience failure: disconfirmed | — | 145 candidates (5 Java/Groovy + 140 FTL) | Negative cases documented |
| SQL/execution verification | PostgreSQL statement logging (direct measurement) | Execution-path trace to JDBC (source analysis) | Both confirm lookup-to-SQL path |
| Latency experiments | Conducted; null result; exploratory | Not conducted | Null result from Fineract informed decision not to repeat |
| Accountability failure | Case A: ~11y, Warn-Disable-Delete | No convincing analogue found in scoped modules | **Negative result** — boundary condition of framework |
