# PR, JIRA, and Commit Provenance Index

This index maps every pull request, JIRA ticket, commit, mailing list thread,
and community record cited in the paper to the specific claim it supports.
The structure follows the chain: origin → issue → PR → fix/removal → paper claim.

The paper states that 14 pull requests were merged to Fineract between February
and May 2026. Six provide direct evidentiary artefacts for the three cases.
Two further primary sources lie outside that set: PR #5655 (closed in favour of
merged PR #5764) and PR #4671 (self-service disable step, authored by a project
maintainer in 2025, not the study author).

---

## Case A: Accountability Failure (Self-Service Security Debt)

### Origin and accumulation

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Self-service module introduction | Git history | ~2015 | Module existed ~11 years before deletion |
| FINERACT-969 (OWASP ZAP scanning) | JIRA issue | 2020 | Security risk documented without remediation |
| FINERACT-879 (CORS policy hardening) | JIRA issue | 2020 | Security risk documented without remediation |
| FINERACT-854 (SQL prepared statements) | JIRA issue | 2020 | Security risk documented without remediation |
| FINERACT-853 (SpotBugs integration) | JIRA issue | 2020 | Security risk documented without remediation |
| CVE-2023-25195 (SSRF) | CVE record | 2023 | Documented public vulnerability |
| CVE-2023-25196 (SQL injection, v1.4–1.8.2) | CVE record | 2023 | Documented public vulnerability |
| CVE-2024-23537 (privilege escalation) | CVE record | 2024 | Documented public vulnerability |
| "Securing Fineract" wiki advisory | Confluence page | By 2024 | Risk documented; module remained operational |

### Phase 2: Disable

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Deprecation announcement email | ASF dev mailing list | May 17, 2025 | "vulnerable and, critically, no contributors have stepped forward" |
| **PR #4671** / FINERACT-2283 | GitHub PR (maintainer: ádám sághy) | Merged May 17, 2025 | Feature flag `fineract.module.self-service.enabled` defaulting to `false` |
| https://github.com/apache/fineract/pull/4671 | | | |

### Phase 3: Delete

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| **PR #5498** / FINERACT-2480 | GitHub PR | Merged March 13, 2026 | Deleted 142 files, ~11,500 lines, 18 REST controllers, Liquibase migration 0219 |
| https://github.com/apache/fineract/pull/5498 | | | |

### Community records

| Artefact | URL | Date | Paper claim supported |
|----------|-----|------|-----------------------|
| "Securing Fineract" wiki advisory | https://cwiki.apache.org/confluence/display/FINERACT/Securing+Fineract | By 2024 | Official advisory stating self-service APIs should be disabled |
| Deprecation mailing list post | https://lists.apache.org/thread/vrspyskchosslx9kqxfw4n3tmmp9hrvh | May 17, 2025 | Formal deprecation announcement, module described as unmaintained |

---

## Case B: Detectability Failure (CLIENTCHARGE|INACTIVATE)

### Origin

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Commit `d0fd3e4a6c` (Vishwas Babu A. J.) | Git commit | August 17, 2015 | Origin of null-returning stub `inactivateCharge()` and surrounding scaffold (DB columns, permissions, routing, builder method) |
| Subsequent commit (September 2015) | Git commit | September 2015 | Comment updated from `// TODO Auto-generated method stub` to `// functionality not yet supported`; still returns `null` |
| `git log --all -S "clientCharge.inactivate" --oneline` | Git pickaxe (zero results) | 2015–2026 | No handler registration was ever committed in any commit in the repository's full history |

### Discovery

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| **PR #5655** / FINERACT-2545 | GitHub PR (closed, not merged) | Opened March 2026; closed April 12, 2026 | Discovery event — implementation attempt that surfaced the phantom; closed in favour of write-path removal |
| https://github.com/apache/fineract/pull/5655 | | | |
| DISCUSS thread | ASF dev mailing list | March 20, 2026 | Community discussion revealing endpoint had never functioned; PMC member requested rigorous validation |
| https://lists.apache.org/thread/lhh0hx0t2y7vzdrs98fg3qyfx1xo9d0y | | | |

### Removal

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| 72-hour removal notice | ASF dev mailing list | April 9, 2026 | Community governance process: notice + 3 binding/explicit +1 responses including PMC Chair |
| https://lists.apache.org/thread/lx2d977z1n7s46oonk87ykvpv3hwgrx8 | | | |
| **PR #5764** / FINERACT-2587 | GitHub PR | Merged April 22, 2026 | Removed API routing branch, null stub, builder method in `CommandWrapperBuilder`, two constants from `ClientApiConstants`, Liquibase changeset 0231 (inactivation permission records) |
| https://github.com/apache/fineract/pull/5764 | | | |
| **PR #5805** / FINERACT-2595 | GitHub PR | Merged April 25, 2026 | Follow-up: corrected foreign-key violation in Liquibase changeset manifesting on existing tenant databases |
| https://github.com/apache/fineract/pull/5805 | | | |

### Supporting instance: Teller/Cashier stub endpoints

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Commit `5afed47767` | Git commit | November 20, 2014 | **Actual** origin of five null-returning stub methods in `TellerManagementReadPlatformServiceImpl`. Note: JIRA ticket FINERACT-2755 and PR description both cite "the original Cash Management commit in 2013" — this is a discrepancy. Direct commit inspection gives `5afed47767` dated November 20, 2014. The JIRA version is treated as the authoritative writeup, but this discrepancy was not resolved in the source material and is documented here rather than silently resolved. |
| JIRA FINERACT-2755 | JIRA ticket | August 11, 2026 | Formal documentation of removal rationale including "no objections received" referencing the dev-list thread |
| Dev-list DISCUSS thread | ASF dev mailing list | July 28, 2026 | Community notice for Teller/Cashier stub removal; no replies received on dev list |
| https://lists.apache.org/thread/m972c6dgr46lob01smkryqwq6dxn7lms | | | |
| **PR #6258** | GitHub PR | Merged August 2026 (commit `e73e98d`) | Removed `CashierApiResource.java` (entirely), `TellerJournalApiResource.java` (entirely), three broken methods from `TellerApiResource.java`, all five method declarations from service interface and implementation |
| https://github.com/apache/fineract/pull/6258 | | | |

### Detector-found instances (Fineract)

| Instance | Artefact | Paper claim supported |
|----------|----------|-----------------------|
| EMAIL\|UPDATE | Builder `CommandWrapperBuilder.java:3686`, endpoint `EmailApiResource.java:188`, origin commit `c338c1758e` (November 14, 2017) | Detector-discovered handler-registry gap; never implemented since module's origin |
| WORKINGCAPITALLOAN\|UPDATEDISCOUNT | Origin commit `135e49c064` (March 27, 2026); deletion `13bafbdc8d` (April 13, 2026); resurrection `7bb04bc1f5` (May 12, 2026) | Detector-discovered; full life-death-reanimation cycle documented with named commits, authors, dates |

---

## Case C: Salience Failure (Loop-Invariant Query Amplification)

Each row maps an instance to its origin commit, fix PR, and the specific paper claim it supports. See `case-c-instance-index.md` for full per-instance detail.

### FINERACT-2561 (Instance 1)

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Origin commit `2b8da96474` | Git commit | 2013-09-17 | Introduction of currency lookup inside loop in `saveAllDebitOrCreditEntries` |
| **PR #5712** / FINERACT-2561 | GitHub PR | Merged April 2, 2026 | Hoist above loop; corrected inconsistency where sibling method already hoisted correctly |
| https://github.com/apache/fineract/pull/5712 | | | |

### FINERACT-2579 (Instance 2)

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Origin commit `7fabfa5cb9` | Git commit | 2015-10-20 | Introduction of 5 per-iteration entity lookups in `generateLoanProvisioningEntry` |
| Fix commit `4bae9dccce` | Git commit | April 2026 | Underlying fix (distinct from merge commit) |
| Merge commit `8408a4c81d` | Git commit | April 7, 2026 | Merge commit when PR landed (parent: `3b6b67278c`) |
| **PR #5739** / FINERACT-2579 | GitHub PR | Merged April 7, 2026 | Replaced 5 per-iteration lookups with 4 bulk `findAllById()` calls |
| https://github.com/apache/fineract/pull/5739 | | | |
| PostgreSQL statement logs | Empirical measurement | April 2026 | 4 SQL SELECT statements observed at N = 1, 10, 100, 1,000 (your own measurement — not in the PR) |

### FINERACT-2582 (Instances 3 and 4)

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Origin commit `ff9181cc47` | Git commit | 2022-05-30 | Instance 3: `UpdateEmailOutboundWithCampaignMessageTasklet` |
| Origin commit `c60c6601c0` | Git commit | 2017-01-03 | Instance 4: `NotificationWritePlatformServiceImpl` |
| **PR #5747** / FINERACT-2582 | GitHub PR | Merged April 8, 2026 | Hoisted both calls above their respective loops |
| https://github.com/apache/fineract/pull/5747 | | | |

### FINERACT-2583 (Instances 5, 6, and 7)

| Artefact | Type | Date | Paper claim supported |
|----------|------|------|-----------------------|
| Origin commit `59f1c0ce8a` | Git commit | 2013-06-12 | Instance 5: `HolidayWritePlatformServiceJpaRepositoryImpl` |
| Origin commit `7fabfa5cb9` | Git commit | 2015-10-20 | Instance 6: `ProvisioningCriteriaAssembler` (shares origin with FINERACT-2579) |
| Origin commit `69d228a502` | Git commit | 2013-03-27 | Instance 7: `CollateralAssembler` |
| **PR #5748** / FINERACT-2583 | GitHub PR | Merged April 8, 2026 | Applied bulk-fetch pattern across all three methods |
| https://github.com/apache/fineract/pull/5748 | | | |

---

## Complete PR List (14 merged, February–May 2026)

The paper states 14 PRs were merged between February and May 2026.
Six are direct case artefacts; the remaining eight addressed unrelated areas.

| PR | JIRA | Role | Merged |
|----|------|------|--------|
| #5498 | FINERACT-2480 | Case A Delete | March 13, 2026 |
| #5655 | FINERACT-2545 | Case B Discovery (closed, not merged) | Closed April 12, 2026 |
| #5712 | FINERACT-2561 | Case C fix (Instance 1) | April 2, 2026 |
| #5739 | FINERACT-2579 | Case C fix (Instance 2) | April 7, 2026 |
| #5747 | FINERACT-2582 | Case C fix (Instances 3+4) | April 8, 2026 |
| #5748 | FINERACT-2583 | Case C fix (Instances 5+6+7) | April 8, 2026 |
| #5764 | FINERACT-2587 | Case B Delete | April 22, 2026 |
| #5805 | FINERACT-2595 | Case B Follow-up (FK fix) | April 25, 2026 |
| #6258 | FINERACT-2755 | Teller/Cashier supporting instance | August 2026 |
| #4671 | FINERACT-2283 | Case A Disable (maintainer PR, 2025) | May 17, 2025 |
| Remaining 8 | — | Unrelated Fineract contributions | Feb–May 2026 |

Note: PR #5655 is closed (not merged) — it is the discovery artefact for Case B,
superseded by PR #5764. It is listed in the paper as a primary source alongside
the merged PRs.

---

## Mailing List Threads

| Thread | URL | Date | Paper claim supported |
|--------|-----|------|-----------------------|
| Self-service deprecation announcement | https://lists.apache.org/thread/vrspyskchosslx9kqxfw4n3tmmp9hrvh | May 17, 2025 | Case A formal deprecation — module described as vulnerable and unmaintained |
| CLIENTCHARGE\|INACTIVATE DISCUSS | https://lists.apache.org/thread/lhh0hx0t2y7vzdrs98fg3qyfx1xo9d0y | March 20, 2026 | Case B community discussion; PMC member validation request |
| CLIENTCHARGE\|INACTIVATE removal notice | https://lists.apache.org/thread/lx2d977z1n7s46oonk87ykvpv3hwgrx8 | April 9, 2026 | Case B 72-hour governance notice; 3 +1 responses including PMC Chair |
| Teller/Cashier DISCUSS | https://lists.apache.org/thread/m972c6dgr46lob01smkryqwq6dxn7lms | July 28, 2026 | Supporting instance; no replies received on dev list |
