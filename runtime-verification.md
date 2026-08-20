# Runtime Verification

This file documents the runtime verification procedures used across two cases:

- **Case B** — confirming that CLIENTCHARGE|INACTIVATE never produced a
  successful response and had no live callers
- **Case C** — measuring SQL statement counts for the fixed implementation of
  FINERACT-2579 via PostgreSQL statement logging

These are distinct verification procedures targeting distinct claims.
Do not conflate the Case C SQL measurement with the Case B SQL check;
they use different mechanisms, target different claims, and were performed
at different times.

---

# Case B: CLIENTCHARGE|INACTIVATE Runtime Verification

These commands confirm that the CLIENTCHARGE|INACTIVATE endpoint
had never produced a successful response since its introduction in August 2015.

## Environment

```
docker pull apache/fineract:latest
```

Verified April 2026. The image used was the `latest` tag at that date.
If reproducing this after a subsequent release, pin the image digest or use a
specific version tag to ensure you are testing against the same codebase.

## HTTP invocation

```
POST /fineract-provider/api/v1/clients/1/charges/1?command=inactivate
```

**Expected response:**

```json
HTTP/1.1 400 Bad Request
{"errors":[{"userMessageGlobalisationCode":"error.msg.command.unsupported"}]}
```

This response indicates that the request reached the command-dispatch layer
and was rejected because no handler was registered for the
`CLIENTCHARGE|INACTIVATE` key. It does not indicate a network or
authentication failure.

## Git archaeology: handler registration

Confirm that no handler was ever registered in any commit in the repository's
full history:

```bash
git clone https://github.com/apache/fineract.git
cd fineract
git log --all -S "clientCharge.inactivate" --oneline
```

**Expected result:** zero output (no commit ever introduced this string).

Inspect the origin commit of the null stub:

```bash
git show d0fd3e4a6c
```

**Expected:** the commit introduces `inactivateCharge()` returning `null`
with the comment `// TODO Auto-generated method stub`, dated August 17, 2015,
authored by Vishwas Babu A. J.

## Database verification

Run against the tenant database after the test invocation:

```sql
SELECT *
FROM m_portfolio_command_source
WHERE action_name = 'INACTIVATE'
  AND entity_name = 'CLIENTCHARGE';
```

**Expected result:**
- Zero rows with `made_on_date` not null prior to the test run, confirming
  no historical invocation ever completed processing.
- The test-invocation row shows `status = 5` (rejected before processing).

---

# Case C: SQL Statement Measurement (FINERACT-2579)

## What this measurement establishes

The paper claims (Section 6.1 and 6.3) that the fixed implementation of
FINERACT-2579 issues exactly 4 SQL SELECT statements at N = 1, 10, 100,
and 1,000 — i.e., query count is constant regardless of batch size.

This was verified by **PostgreSQL statement logging** against a running
Fineract instance. It is the only directly measured quantitative claim in
Case C. All other lookup counts (e.g., N × 5 for the old implementation)
are source-level observations, not SQL-level measurements.

## Critical terminology distinction

| Term | Meaning |
|------|---------|
| Repository lookup | A source-level method invocation (`findById`, `findAllById`, etc.) |
| SQL statement | A statement actually issued to the database, as observed via PostgreSQL logging |

These are not the same. The JPA persistence context (Hibernate first-level
cache) can absorb repeated `findById` calls for the same entity within a
transaction without issuing additional SQL. The **N × 5 figure for the old
implementation is a source-level count only** — it was never confirmed at
the SQL level and was subsequently withdrawn as a paper claim. See Evidence
Ledger below.

## What was measured

- **Implementation:** fixed implementation at merge commit `8408a4c81d`
  (PR #5739, merged April 7, 2026)
- **Method:** `generateLoanProvisioningEntry` in
  `ProvisioningEntriesWritePlatformServiceJpaRepositoryImpl`
- **N values tested:** 1, 10, 100, 1,000 provisioning entries
- **Instrumentation:** PostgreSQL `log_min_duration_statement` and
  `log_statement = 'all'` in `postgresql.conf`, with statement counts
  extracted from the PostgreSQL server log
- **Scaling check:** The aggregate reserve amount was tracked at each N and
  scaled exactly linearly (1×, 10×, 100×, 1,000× baseline), confirming that
  no rows were dropped or undercounted by the bulk-fetch logic

## Observed result

| N | SQL SELECT statements observed |
|---|-------------------------------|
| 1 | 4 |
| 10 | 4 |
| 100 | 4 |
| 1,000 | 4 |

The 4 statements correspond to the four `findAllById()` calls in the fixed
implementation: LoanProduct, Office, ProvisioningCategory, and the two
GLAccount types (bulk-fetched as one call each).

## How to reproduce

### Prerequisites

- Apache Fineract running against PostgreSQL (not H2)
- A tenant database seeded with provisioning-ready loan data
- Access to `postgresql.conf` to enable statement logging

### Step 1 — Enable PostgreSQL statement logging

In `postgresql.conf`:

```
log_statement = 'all'
log_min_duration_statement = 0
```

Reload PostgreSQL configuration:

```sql
SELECT pg_reload_conf();
```

### Step 2 — Checkout the fixed commit

```bash
cd fineract
git checkout 8408a4c81d
```

Build and start Fineract against your PostgreSQL instance.

### Step 3 — Invoke the provisioning entries endpoint

```
POST /fineract-provider/api/v1/runaccruals
```

or the relevant provisioning generation endpoint for your tenant, with
N loans in a provisionable state.

Repeat for N = 1, 10, 100, and 1,000.

### Step 4 — Count SELECT statements in the PostgreSQL log

```bash
grep "SELECT" /var/log/postgresql/postgresql-*.log \
  | grep -i "provisioning\|loanproduct\|glaccounts" \
  | wc -l
```

Isolate the statements issued during the provisioning call by timestamp.
A more precise approach is to use `pg_stat_statements` or wrap the call
in a transaction and count statements in the log between BEGIN and COMMIT.

**Expected:** exactly 4 SELECT statements per provisioning run regardless
of N.

### Step 5 — Verify the old implementation for comparison (optional)

```bash
git checkout 3b6b67278c   # parent commit of the fix merge
```

Repeat the measurement. The old implementation's SQL count was **not**
directly measured in the original study. Reproducing it would provide
independent evidence for or against the source-level N × 5 extrapolation.

## Evidence ledger

| Claim | Evidence type | Status |
|-------|---------------|--------|
| Old implementation: up to N × 5 repository lookups | Source code inspection | Supported |
| New implementation: 4 bulk repository calls | Source code inspection (PR #5739 diff) | Supported |
| New implementation: 4 SQL SELECT statements at N = 1, 10, 100, 1,000 | PostgreSQL statement logs | **Directly measured** |
| Old implementation: N × 5 SQL statements | Not measured | Unsupported / withdrawn |
| 1,250× reduction in database traffic | Source-level extrapolation | Withdrawn |
| Latency improves proportionally with lookup reduction | Timing experiments | Not supported (exploratory null result) |
| JPA cache explains null latency result | Source analysis + JPA behavior | Plausible mechanism (not independently confirmed) |

## Latency experiments (exploratory — not paper evidence)

A controlled timing comparison was conducted under three workloads using
two live stacks with identical schema, seed data, and auth.

- Fixed stack: commit `8408a4c81d`
- Prefix (unfixed) stack: parent commit `3b6b67278c`

**Single-product workload (N = 1 to 1,000):** Latency ratio rose to
approximately 1.1× by N = 10 and remained flat through N = 1,000.
No statistical test was computed; reported descriptively.

**Multi-product workloads (N = 100 only, 30 repetitions per stack):**

| Product diversity | Latency ratio | p-value |
|-------------------|---------------|---------|
| 5 distinct products | 0.99× | 0.83 |
| 20 distinct products | 0.99× | 0.77 |

**No detectable latency improvement was observed.** The plausible
explanation is JPA first-level cache absorption: when entities are
repeated across iterations (low diversity), the old implementation's
individual `findById` calls are cache-satisfied without hitting the
database, making the SQL-level improvement invisible in end-to-end latency.

**These experiments are exploratory and are not treated as evidence for
the paper's debt-visibility framework.** They are included here for
completeness and reproducibility.

**Note on confidence intervals:** An earlier draft included 95% CIs, but
they were computed assuming n = 20 repetitions. The actual experiment used
n = 30. Since raw timing data is no longer available, the CIs cannot be
honestly recomputed and are not included here.
