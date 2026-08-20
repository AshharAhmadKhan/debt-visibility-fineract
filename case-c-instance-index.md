# Case C Instance Index: Loop-Invariant Query Amplification

This index covers all seven confirmed instances of loop-invariant query
amplification identified in Apache Fineract between February and May 2026.
The paper presents these across four JIRA issues (FINERACT-2561, 2579, 2582,
2583), each covering one or more instances.

Persistence durations are calculated from the origin commit date to the
fix merge date (April 2026). Origin commits were confirmed by inspecting the
actual diff, not only by pickaxe search, since a pickaxe hit can
false-positive when a call is hoisted without changing occurrence count.

**Terminology note:** "Repository lookups" refers to source-level method
invocations (`findById`, `findOneWithNotFoundDetection`, etc.).
"SQL statements" refers only to what was directly observed via PostgreSQL
statement logging. These are not the same thing; see runtime-verification.md
for the distinction and the measurement methodology.

---

## Instance 1 — FINERACT-2561

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2561 |
| File | `JournalEntryWritePlatformServiceJpaRepositoryImpl.java` |
| Method | `saveAllDebitOrCreditEntries` |
| Module | Journal Entry |
| Invariant argument | `currencyCode` (method parameter, constant across all loop iterations) |
| Pattern | `findOneWithNotFoundDetection(currencyCode)` called once per loop iteration |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (hoisted above loop) |
| Origin commit | `2b8da96474` |
| Origin date | 2013-09-17 |
| Persistence | ~12 years 7 months |
| Fix | Hoist above loop |
| Fix PR | https://github.com/apache/fineract/pull/5712 |
| Fix merged | April 2, 2026 |
| Note | The sibling method handling opening balance entries in the same class already hoisted this call correctly before the loop — the pattern coexisted with a correct implementation in the same file. |

---

## Instance 2 — FINERACT-2579

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2579 |
| File | `ProvisioningEntriesWritePlatformServiceJpaRepositoryImpl.java` |
| Method | `generateLoanProvisioningEntry` |
| Module | Provisioning |
| Invariant arguments | 5 entity IDs per row: LoanProduct, Office, ProvisioningCategory, two GLAccounts |
| Pattern | Five separate `findById()` calls per loop iteration, all on identifiers that do not change within the loop |
| Repository lookups before | N × 5 |
| Repository lookups after | 4 (bulk `findAllById()` calls with in-memory map lookups) |
| Origin commit | `7fabfa5cb9` |
| Origin date | 2015-10-20 |
| Persistence | ~10 years 6 months |
| Fix commit | `4bae9dccce` (the underlying fix commit within the PR) |
| Merge commit | `8408a4c81d` (the merge commit when the PR landed; distinct from the fix commit) |
| Fix PR | https://github.com/apache/fineract/pull/5739 |
| Fix merged | April 7, 2026 |
| SQL measurement | PostgreSQL statement logging confirmed 4 SELECT statements at N = 1, 10, 100, 1,000. See runtime-verification.md for methodology. |
| Note | The N × 5 source-level lookup count was **not** confirmed as N × 5 SQL statements. The old implementation's SQL count was never directly measured. The withdrawn 1,250× database-traffic claim was based on source-level extrapolation that does not hold at SQL level due to JPA first-level cache absorption. |

---

## Instance 3 — FINERACT-2582a

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2582 |
| File | `UpdateEmailOutboundWithCampaignMessageTasklet.java` |
| Method | Email campaign dispatch (loop body) |
| Module | Email/Campaign |
| Invariant argument | `campaignId` (method-level constant across all loop iterations) |
| Pattern | `findById(campaignId)` called once per iteration inside the campaign dispatch loop |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (hoisted above loop) |
| Origin commit | `ff9181cc47` |
| Origin date | 2022-05-30 |
| Persistence | ~3 years 10 months |
| Fix | Hoist above loop |
| Fix PR | https://github.com/apache/fineract/pull/5747 |
| Fix merged | April 8, 2026 |

---

## Instance 4 — FINERACT-2582b

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2582 |
| File | `NotificationWritePlatformServiceImpl.java` |
| Method | Notification mapper insertion (loop body) |
| Module | Notification |
| Invariant argument | `notificationId` (method-level constant across all loop iterations) |
| Pattern | `findById(notificationId)` called once per iteration inside the notification insertion loop |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (hoisted above loop) |
| Origin commit | `c60c6601c0` |
| Origin date | 2017-01-03 |
| Persistence | ~9 years 3 months |
| Fix | Hoist above loop |
| Fix PR | https://github.com/apache/fineract/pull/5747 |
| Fix merged | April 8, 2026 |
| Note | Instances 3 and 4 share the same fix PR (PR #5747) because both methods were corrected in the same change. |

---

## Instance 5 — FINERACT-2583a

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2583 |
| File | `HolidayWritePlatformServiceJpaRepositoryImpl.java` |
| Method | Holiday office selection loop |
| Module | Holiday |
| Invariant argument | Office ID |
| Pattern | `findOneWithNotFoundDetection()` on office ID called per iteration |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (bulk `findAllById()`) |
| Origin commit | `59f1c0ce8a` |
| Origin date | 2013-06-12 |
| Persistence | ~12 years 10 months |
| Fix | Bulk `findAllById()` with in-memory map lookup |
| Fix PR | https://github.com/apache/fineract/pull/5748 |
| Fix merged | April 8, 2026 |

---

## Instance 6 — FINERACT-2583b

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2583 |
| File | `ProvisioningCriteriaAssembler.java` |
| Method | Provisioning criteria assembly loop |
| Module | Provisioning |
| Invariant argument | Product ID |
| Pattern | `findOneWithNotFoundDetection()` on product ID called per iteration |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (bulk `findAllById()`) |
| Origin commit | `7fabfa5cb9` |
| Origin date | 2015-10-20 |
| Persistence | ~10 years 6 months |
| Fix | Bulk `findAllById()` with in-memory map lookup |
| Fix PR | https://github.com/apache/fineract/pull/5748 |
| Fix merged | April 8, 2026 |
| Note | Shares origin commit `7fabfa5cb9` with Instance 2 (FINERACT-2579). Both were introduced in the same commit on 2015-10-20. |

---

## Instance 7 — FINERACT-2583c

| Field | Value |
|-------|-------|
| JIRA issue | FINERACT-2583 |
| File | `CollateralAssembler.java` |
| Method | Collateral assembly loop |
| Module | Collateral |
| Invariant argument | Collateral type ID |
| Pattern | `findOneWithNotFoundDetection()` on collateral type ID called per iteration |
| Repository lookups before | N × 1 |
| Repository lookups after | 1 (bulk `findAllById()`) |
| Origin commit | `69d228a502` |
| Origin date | 2013-03-27 |
| Persistence | ~13 years 0 months |
| Fix | Bulk `findAllById()` with in-memory map lookup |
| Fix PR | https://github.com/apache/fineract/pull/5748 |
| Fix merged | April 8, 2026 |
| Note | Instances 5, 6, and 7 share the same fix PR (PR #5748). |

---

## Summary Table

| Instance | JIRA | Module | File (short) | Origin Commit | Origin Date | Persistence | Fix PR |
|----------|------|--------|--------------|---------------|-------------|-------------|--------|
| 1 | 2561 | Journal Entry | `JournalEntryWritePlatformServiceJpaRepositoryImpl` | `2b8da96474` | 2013-09-17 | ~12y 7mo | #5712 |
| 2 | 2579 | Provisioning | `ProvisioningEntriesWritePlatformServiceJpaRepositoryImpl` | `7fabfa5cb9` | 2015-10-20 | ~10y 6mo | #5739 |
| 3 | 2582a | Email/Campaign | `UpdateEmailOutboundWithCampaignMessageTasklet` | `ff9181cc47` | 2022-05-30 | ~3y 10mo | #5747 |
| 4 | 2582b | Notification | `NotificationWritePlatformServiceImpl` | `c60c6601c0` | 2017-01-03 | ~9y 3mo | #5747 |
| 5 | 2583a | Holiday | `HolidayWritePlatformServiceJpaRepositoryImpl` | `59f1c0ce8a` | 2013-06-12 | ~12y 10mo | #5748 |
| 6 | 2583b | Provisioning | `ProvisioningCriteriaAssembler` | `7fabfa5cb9` | 2015-10-20 | ~10y 6mo | #5748 |
| 7 | 2583c | Collateral | `CollateralAssembler` | `69d228a502` | 2013-03-27 | ~13y 0mo | #5748 |

**Instance count vs. issue count:** There are 7 instances across 4 JIRA issues.
FINERACT-2582 covers 2 instances (3 and 4); FINERACT-2583 covers 3 instances
(5, 6, and 7). This distinction matters: the paper reports 7 instances in
Table 3 and 4 JIRA issues in Table 4. Both figures are correct for what they
are counting.

**Shared origin commit:** Instances 2 and 6 (`7fabfa5cb9`, 2015-10-20) were
introduced in the same commit. The pattern was introduced in multiple locations
simultaneously, not through independent repetition.

**Author attribution:** The seven instances span five independent contributors
and organizationally distinct code areas (Journal Entry, Provisioning,
Email/Campaign, Notification, Holiday/Collateral), consistent with the paper's
claim that the pattern is not one developer's coding habit.

---

## Verification

To verify any origin commit against the Apache Fineract repository:

```bash
git clone https://github.com/apache/fineract.git
cd fineract

# Inspect the origin commit diff (example: instance 1)
git show 2b8da96474

# Confirm the commit date
git log --format="%H %ai %s" | grep 2b8da96474

# Confirm the method existed at that commit
git show 2b8da96474:fineract-provider/src/main/java/org/apache/fineract/accounting/journalentry/service/JournalEntryWritePlatformServiceJpaRepositoryImpl.java | grep -A 20 "saveAllDebitOrCreditEntries"
```

Substitute the relevant commit hash and file path for the other instances.
