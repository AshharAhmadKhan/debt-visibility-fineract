# Dependency Surface Checklist

This file documents the searches used to confirm that each debt instance had
zero live consumers — no SDK method, no generated client, no integration test,
no frontend reference, and no plugin reference that would create a dependency
on the non-functional code.

A confirmed zero result across all surfaces is what made safe removal possible
and is cited as evidence in the paper. Each section states:

- the search command
- the expected result (what constitutes a "pass")
- why the surface matters for the specific case

---

## Part 1: Case B — CLIENTCHARGE|INACTIVATE (Fineract)

The paper claims five surfaces were checked and all returned zero. These are the
exact searches.

### Surface 1: Generated SDK client

```bash
grep -r "inactivateClientCharge" fineract-client/
```

**Expected:** no output (zero matches)

**Why this matters:** The `fineract-client` module is a generated Java SDK wrapping
the Fineract REST API. If a generated method existed for `inactivateClientCharge`,
downstream applications would have had a typed entry point to the endpoint. Zero
results confirm no SDK consumer was ever created.

### Surface 2: OpenAPI specification (built artifact)

```bash
grep "inactivate" fineract.yaml | grep -i "client"
```

**Expected:** no output (zero matches)

**Why this matters:** The built `fineract.yaml` is the generated OpenAPI spec for
the API surface. The `@Operation` annotation on `ClientChargesApiResource`
documented only `paycharge` and `waive` — never `inactivate`. A zero result
confirms the endpoint was not part of the published API contract.

**Supplementary check — annotation source:**

```bash
grep -n "command=inactivate\|commandParam.*inactivate" \
  fineract-provider/src/main/java/org/apache/fineract/portfolio/client/api/ClientChargesApiResource.java
```

**Expected:** no output

### Surface 3: Frontend source tree

```bash
grep -r "command=inactivate" fineract-backoffice-ui/src/
```

**Expected:** no output

**Why this matters:** If any Angular component had wired a button or form to
`command=inactivate` for client charges, a UI consumer would have existed even
without an SDK method. Zero results confirm no UI surface was ever built.

### Surface 4: Integration test suite

```bash
grep -r "inactivate" fineract-integration-tests/src/ | grep -i "clientcharge\|client.charge"
```

**Expected:** no output

**Why this matters:** Integration tests provide the strongest form of execution
evidence. Zero results mean no automated test ever attempted to exercise this
endpoint, which is consistent with the zero historical execution records in
`m_portfolio_command_source`.

### Surface 5: Full-history handler search (git archaeology)

```bash
git log --all -S "clientCharge.inactivate" --oneline
```

**Expected:** zero lines of output

**Why this matters:** This confirms no handler was ever registered at any point
in the repository's full history, not just the current state. Distinct from the
four surfaces above, which check the current artifact state.

### Summary for CLIENTCHARGE|INACTIVATE

All five surfaces returned zero. Together with zero rows in
`m_portfolio_command_source` (no historical execution), this confirms the
endpoint had no active callers and no downstream dependencies anywhere in the
project.

---

## Part 2: Teller/Cashier Supporting Instance (Fineract)

Five null-returning stub methods backed five read endpoints. The paper claims
no integration test, no SDK method, no operationId in the spec, no Cucumber
feature file, and no remaining references after removal.

### Surface 1: SDK and generated client

```bash
# Check fineract-client for any of the five operationIds
grep -r "retrieveAllCashiers\|retrieveAllTransactionsForTeller\|retrieveOneTransactionForTeller\|retrieveAllJournalsForTeller\|retrieveAllCashierJournals" \
  fineract-client/ fineract-client-feign/
```

**Expected:** no output (no generated SDK method ever wrapped these endpoints)

**Note:** Also check `@AlternativeOperationId` aliases — the five operationIds
listed are the Swagger `operationId` values from the removed resource classes.

### Surface 2: Integration tests

```bash
# Repository-wide search for method names
grep -r "getCashierData\|findTellerTransaction\|fetchTellerTransactionsByTellerId\|getJournals\|fetchTellerJournals" \
  --include="*Test.java" .
```

**Expected:** zero results

The one integration test touching the teller/cashier module is
`CashierTransactionsHelper.java`, which calls only `retrieveCashierTransactions`
and `retrieveCashierTransactionsWithSummary` — both implemented methods,
unaffected by the removal.

### Surface 3: Cucumber feature files

```bash
grep -r "cashier\|teller.*transaction\|teller.*journal" \
  --include="*.feature" .
```

**Expected:** no reference to the five removed endpoints

### Surface 4: Post-removal reference check

After PR #6258 was merged:

```bash
# Confirm no remaining references to removed types
grep -r "TellerJournalData\|TellerTransactionData\|getJournalData\|getTransactionData\|findTransactionData" .
```

**Expected:** zero results outside the removed code (confirming clean removal)

---

## Part 3: OFBiz Detectability Failure — Plugins Verification

For each of the seven confirmed OFBiz Case B instances, the plugins repositories
(ofbiz-plugins-work and ofbiz-plugins-bare) were checked to confirm the phantom
service declaration does not appear there either.

**Repositories:**

```bash
git clone https://github.com/apache/ofbiz-plugins ofbiz-plugins-work
# ofbiz-plugins-bare: the same repository cloned without working-tree checkout,
# used for git grep HEAD verification across full tree
```

### Search command (per instance)

```bash
# Run against both repositories
git grep HEAD "<invoke-name>"
```

Substitute `<invoke-name>` with the service name for each instance:

| Instance | `invoke-name` | Expected result |
|----------|---------------|-----------------|
| B1 | `creditOrderPaymentPreference` | No output (absent from both repos) |
| B2 | `testPermissionExistingTesting` | No output (absent from both repos) |
| B5 | `deleteProductionRunRoutingTask` | No output (absent from both repos) |
| B6 | `deleteProductionRunComponent` | No output (absent from both repos) |
| B7 | `updateLayoutImageOnly` | No output (absent from both repos) |
| B8 | `addSeparator` | **Present in ofbiz-plugins-bare** (ecommerce plugin route — expected, documented in ofbiz-validation.md) |
| B9 | `WebSiteCmsPreviewEvent` | No output (absent from both repos) |

**Note on B8:** `addSeparator` is the only one of the seven present in the
plugins repositories. The ecommerce plugin contains a `controller.xml` entry
with `uri="addseperator"` (note the typo) targeting the same unimplemented
method. This is documented in `ofbiz-validation.md` — it does not disconfirm
B8 but it means B8 has a dual-route profile (trunk + ecommerce plugin) with
differing auth settings.

**Note on cross-repo commit hashes:** Trunk and plugins-repo commit hashes are
repository-local and not directly comparable. The same commit (same content,
date, author, message) will have different hashes in each repository. This
affects only commit-hash cross-referencing, not the search results themselves.

### Caller search (per instance)

In addition to plugins verification, confirm no callers exist anywhere in trunk:

```bash
# In apache/ofbiz-framework
git grep HEAD "<invoke-name>"
```

**Expected for B1, B2, B5, B6, B7, B9:** no output beyond the declaration
itself in its servicedef XML file.

**Expected for B8:** the declaration in trunk `controller.xml` (expected) plus
the ecommerce plugin entry (expected, documented).

---

## Part 4: OFBiz Salience Failure — Plugins Verification

The OFBiz Case C analysis also checked the plugins repositories for
loop-invariant uncached patterns.

```bash
# In ofbiz-plugins-work
grep -r "delegator\.findOne\|delegator\.findList\|EntityQuery\.use" \
  --include="*.ftl" --include="*.java" --include="*.groovy" .
```

**Expected:** No Case C instances found in either plugins repository.

The confirmed six instances are all in apache/ofbiz-framework trunk.
Both ofbiz-plugins-work and ofbiz-plugins-bare were confirmed empty of
loop-invariant uncached patterns.

---

## What constitutes a positive result

In each search above, a positive result (finding a match where none is expected)
would indicate either:

1. A consumer existed and was missed in the original analysis — which would
   weaken the claim that removal was safe
2. The search pattern is too broad and is matching unrelated code — which
   requires manual inspection to distinguish

If reproducing these searches returns unexpected results, manually inspect each
match to determine whether it is:

- A reference to the actual non-functional endpoint (a true positive — significant)
- An unrelated use of the same string (a false positive — inspect and discard)
- A post-removal reference in a changelog, comment, or historical document
  (expected — not a live consumer)
