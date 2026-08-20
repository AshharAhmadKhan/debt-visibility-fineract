# Claim-to-Artifact Index

This is the reviewer entry point. It maps every major empirical claim in the
paper to the specific artifact that supports it and the step needed to verify it
independently.

Structure per row:
- **Paper claim** — the exact proposition as stated in the paper
- **Artifact** — the file in this package, or an external primary source
- **Verification** — what a reviewer does to check the claim

If a claim is based on your own empirical measurement (not an external source),
that is stated explicitly so the reviewer knows what they are verifying.

---

## Case A: Accountability Failure (Self-Service Security Debt)

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Self-service module introduced ~2015, existed ~11 years before deletion | `pr-jira-mapping.md` → Case A origin | `git log --follow` on any self-service file in apache/fineract; confirm introduction date vs. PR #5498 merge date (March 13, 2026) |
| JIRA security issues FINERACT-969, 879, 854, 853 documented from 2020 | `pr-jira-mapping.md` → Case A accumulation | Open each JIRA ticket; confirm creation dates and security-component classification |
| CVE-2023-25195 (SSRF), CVE-2023-25196 (SQL injection), CVE-2024-23537 (privilege escalation) | `pr-jira-mapping.md` → Case A accumulation | NVD/MITRE CVE records for each number |
| "Securing Fineract" wiki advisory stated APIs should be disabled and apps should not use them | `pr-jira-mapping.md` → community records | https://cwiki.apache.org/confluence/display/FINERACT/Securing+Fineract |
| Formal deprecation announcement posted May 17, 2025, stated "no contributors have stepped forward" | `pr-jira-mapping.md` → community records | https://lists.apache.org/thread/vrspyskchosslx9kqxfw4n3tmmp9hrvh |
| PR #4671 introduced feature flag defaulting to `false`, making module opt-in | `pr-jira-mapping.md` → Phase 2 | https://github.com/apache/fineract/pull/4671 — inspect diff for flag name and default value |
| PR #5498 deleted 142 files, ~11,500 lines, 18 REST controllers, Liquibase migration 0219 | `pr-jira-mapping.md` → Phase 3 | https://github.com/apache/fineract/pull/5498 — inspect diff stats |

---

## Case B: Detectability Failure (CLIENTCHARGE\|INACTIVATE)

### Historical reconstruction

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| `inactivateCharge()` introduced in commit `d0fd3e4a6c` by Vishwas Babu A. J., August 17, 2015, returning `null` | `pr-jira-mapping.md` → Case B origin | `git show d0fd3e4a6c` in apache/fineract — confirm author, date, null return, TODO comment |
| Surrounding scaffold (DB columns, permissions, routing, builder method) established in same commit | `pr-jira-mapping.md` → Case B origin | Same `git show d0fd3e4a6c` — inspect diff for `m_client_charge` columns, `m_permission` records, `ClientChargesApiResource`, `CommandWrapperBuilder` |
| September 2015 commit updated comment to `// functionality not yet supported`, still returning `null` | `pr-jira-mapping.md` → Case B origin | `git log --follow -p fineract-provider/src/main/java/.../ClientChargeWritePlatformServiceImpl.java` — find September 2015 commit |
| No handler registration was ever committed: `git log --all -S "clientCharge.inactivate" --oneline` returns zero results | `runtime-verification.md` → git archaeology | Run the command in a clone of apache/fineract; expected output: empty |
| Endpoint returned HTTP 400 `error.msg.command.unsupported` on every invocation | `runtime-verification.md` → Case B HTTP invocation | Run the HTTP invocation against `apache/fineract:latest` (April 2026); see exact command in runtime-verification.md |
| `m_portfolio_command_source` showed zero historical rows before test run; test row shows `status=5`, `made_on_date=NULL` | `runtime-verification.md` → Case B database verification | Run the SQL query in runtime-verification.md against a live Fineract tenant database |

### Dependency surface (zero consumers)

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| No SDK method wraps `inactivateClientCharge` | `dependency-surface-checklist.md` → SDK search | Search queries in checklist; run against fineract-client module |
| Swagger `@Operation` on endpoint documents only `paycharge` and `waive`, not `inactivate` | `dependency-surface-checklist.md` → OpenAPI check | Inspect `ClientChargesApiResource.java` annotation or built `fineract.yaml` |
| No front-end file references `command=inactivate` for client charges | `dependency-surface-checklist.md` → frontend search | Search query in checklist; run against `fineract-backoffice-ui` or equivalent |
| No integration test exercises the endpoint | `dependency-surface-checklist.md` → test search | Search query in checklist |

### Community governance and removal

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| 72-hour mailing list notice posted April 9, 2026; three +1 responses including PMC Chair | `pr-jira-mapping.md` → Case B removal | https://lists.apache.org/thread/lx2d977z1n7s46oonk87ykvpv3hwgrx8 — count replies, check PMC Chair identity |
| PR #5764 removed write path (routing, stub, builder method, constants, Liquibase changeset 0231) | `pr-jira-mapping.md` → Case B removal | https://github.com/apache/fineract/pull/5764 — inspect diff |
| PR #5805 corrected FK violation in Liquibase changeset on existing tenant databases | `pr-jira-mapping.md` → Case B removal | https://github.com/apache/fineract/pull/5805 — inspect diff |

### Teller/Cashier supporting instance

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Five null-returning stub methods originated in commit `5afed47767`, November 20, 2014 | `pr-jira-mapping.md` → Teller/Cashier; `ofbiz-validation.md` N/A (Fineract case) | `git show 5afed47767` — confirm date and method stubs. **Note:** JIRA FINERACT-2755 and PR description cite "2013" — discrepancy is documented and unresolved; direct commit inspection gives November 20, 2014 |
| All five endpoints returned HTTP 204 with empty body including for non-existent resource IDs | `runtime-verification.md` → (Teller/Cashier section) | Direct HTTP invocation against develop-branch Docker build with minimal teller setup |
| PR #6258 removed the five stub methods and deleted two API resource classes | `pr-jira-mapping.md` → Teller/Cashier | https://github.com/apache/fineract/pull/6258 — inspect diff |

---

## Case C: Salience Failure (Loop-Invariant Query Amplification)

### Seven instances — commit archaeology

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Seven instances across five independent modules; persistence 4–13 years | `case-c-instance-index.md` → summary table | Run `git show <origin-commit>` for each of the seven commits listed; confirm date and method content |
| Instances span independent contributors — not one developer's coding habit | `case-c-instance-index.md` → authorship note | Compare committer fields across the seven origin commits |
| Instances 2 and 6 share origin commit `7fabfa5cb9` (2015-10-20) | `case-c-instance-index.md` → instances 2 and 6 | `git show 7fabfa5cb9` — confirm both files/methods appear in the diff |

### SQL measurement (your own empirical measurement)

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Fixed implementation (PR #5739) replaced N×5 per-iteration lookups with 4 bulk `findAllById()` calls | `pr-jira-mapping.md` → FINERACT-2579; `case-c-instance-index.md` → Instance 2 | https://github.com/apache/fineract/pull/5739 — inspect diff for removal of `findById()` loop and introduction of `findAllById()` |
| PostgreSQL statement logging confirmed 4 SQL SELECT statements at N = 1, 10, 100, 1,000 (your measurement) | `runtime-verification.md` → Case C SQL measurement | Follow the reproduction steps in runtime-verification.md: checkout commit `8408a4c81d`, enable PostgreSQL `log_statement = 'all'`, invoke provisioning endpoint at each N, count SELECT statements in log |
| N×5 source-level lookup count for old implementation was **not** confirmed at SQL level — claim withdrawn | `runtime-verification.md` → evidence ledger | Evidence ledger explicitly marks "Unsupported / withdrawn" — no artifact supports the withdrawn claim |
| Aggregate reserve amount scaled linearly at each N (no rows dropped by bulk-fetch) | `runtime-verification.md` → Case C SQL measurement | Described as a scaling check; no separate artifact — reviewer can reproduce by comparing reserve totals at each N |

### Individual instance fixes

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| PR #5712 corrected currency lookup in `saveAllDebitOrCreditEntries` | `pr-jira-mapping.md` → FINERACT-2561 | https://github.com/apache/fineract/pull/5712 |
| PR #5739 replaced five per-iteration lookups with four bulk calls in `generateLoanProvisioningEntry` | `pr-jira-mapping.md` → FINERACT-2579 | https://github.com/apache/fineract/pull/5739 |
| PR #5747 hoisted constant lookups in email campaign and notification services | `pr-jira-mapping.md` → FINERACT-2582 | https://github.com/apache/fineract/pull/5747 |
| PR #5748 applied bulk-fetch pattern across holiday, provisioning criteria, and collateral assembly | `pr-jira-mapping.md` → FINERACT-2583 | https://github.com/apache/fineract/pull/5748 |

---

## OFBiz Cross-Project Validation

### Detectability failure replication

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| 7 confirmed detectability failures in OFBiz; 3 reachable phantom, 2 dormant, 2 orphaned-by-substitution | `ofbiz-validation.md` → Part 1 confirmed instances | For each instance: `git show <origin-commit>` in apache/ofbiz-framework; `git log --all -S "<invoke-name>"` to confirm no implementation; inspect controller wiring |
| 3 candidates disconfirmed with rejection rationale | `ofbiz-validation.md` → disconfirmed candidates | See rationale per candidate; `storeForwardedEmail` commented-out since 2009, `getProjectTask` has live implementation, `checkOwnership` throws loud exception |
| Detector validated against OFBiz scope: 6 confirmed matches, 0 false positives | `ofbiz-validation.md` → detector validation | Detector results reported; independent replication requires running the same detector against apache/ofbiz-framework |
| Ages span approximately 7 to 20 years | `ofbiz-validation.md` → per-instance tables | Origin commit dates in each instance table; current date minus origin date |

### Salience failure replication

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| 6 confirmed instances of loop-invariant query amplification in OFBiz; ages 6–20 years | `ofbiz-validation.md` → Part 2 confirmed instances | For each instance: `git show <origin-commit>`; confirm loop membership, key invariance, absence of caching |
| OFBiz entity engine confirmed to issue SQL per uncached loop iteration — execution path traced to JDBC | `ofbiz-validation.md` → entity-engine caching determination | Source inspection of `GenericDelegator` → `GenericHelperDAO` → `GenericDAO.select()` → `sqlP.executeQuery()` chain, as documented with code excerpts |
| 5 of 6 instances are conditional: up to N SQL statements; 1 instance (C5) is unconditional: N SQL statements | `ofbiz-validation.md` → confirmed instances (gating column) | Inspect gating conditions in each instance's source location |
| 145 candidates disconfirmed: 5 Java/Groovy + 140 FTL (53 cached, 65 key-variant, 13 outside-loop) | `ofbiz-validation.md` → disconfirmed candidates | Rejection rationale and example locations documented per category |
| No latency experiments conducted for OFBiz | `ofbiz-validation.md` → explicit statement | Stated explicitly — no measurement artifact exists or is claimed |

### Negative result

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Accountability failure did not replicate in OFBiz within scoped modules | `ofbiz-validation.md` → summary table | Negative result stated with scope boundary; accounting, product, order, and party modules examined |

---

## Methodology

| Paper claim | Artifact | Verification |
|-------------|----------|--------------|
| Prospective OFBiz protocol established August 15, 2026 before search began | `ofbiz-validation.md` → prospective protocol section | Protocol described; dated before systematic search — ordering is asserted, not separately archived |
| 14 PRs merged to Fineract February–May 2026; 6 are direct case artefacts | `pr-jira-mapping.md` → complete PR list | GitHub profile for AshharAhmadKhan; filter by apache/fineract, date range |
| Thematic analysis of mailing list threads and JIRA comments | `thematic-coding-schema.md` | Coding categories and definitions |

---

## How to use this index

1. Identify the paper claim you want to verify.
2. Find it in the table above and note the artifact column.
3. If the artifact is a file in this package, open it for detail.
4. If the artifact is an external primary source (GitHub PR, mailing list, CVE), the URL is in `pr-jira-mapping.md`.
5. Follow the verification step.

Claims marked as **your own measurement** (e.g., the PostgreSQL statement counts)
cannot be verified by clicking an external URL — they require reproducing the
experiment. The reproduction procedure is in `runtime-verification.md`.

Claims marked as **withdrawn** (e.g., the 1,250× database-traffic reduction)
have no supporting artifact because they were explicitly retracted. Do not
expect to find evidence for them.
