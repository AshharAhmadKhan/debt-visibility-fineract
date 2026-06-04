# Case C Instance Index: Loop-Invariant Query Amplification

All four instances were discovered during systematic code reading
between February and May 2026. Each instance placed a repository
lookup inside a loop where the queried identifier does not change
across iterations.

## Instance 1: FINERACT-2561

File: JournalEntryWritePlatformServiceImpl.java
Method: saveAllDebitOrCreditEntries
Invariant argument: currencyCode (method parameter, constant across all iterations)
DB calls before: N x 1
DB calls after: 1
Fix: hoisted findOneWithNotFoundDetection() above the loop
PR: https://github.com/apache/fineract/pull/5712

## Instance 2: FINERACT-2579

File: ProvisioningEntriesWritePlatformServiceImpl.java
Method: generateLoanProvisioningEntry
Invariant arguments: LoanProduct, Office, ProvisioningCategory, two GLAccounts (5 per row)
DB calls before: N x 5
DB calls after: 4
Fix: replaced per-iteration queries with bulk findAllById and in-memory map lookups
PR: https://github.com/apache/fineract/pull/5739

## Instance 3: FINERACT-2582

Files: UpdateEmailOutboundTasklet.java, NotificationWriteServiceImpl.java
Methods: email campaign dispatch, notification mapper insertion
Invariant arguments: campaignId, notificationId (method-level constants)
DB calls before: N x 1 per method
DB calls after: 1 per method
Fix: hoisted findById calls above both loops
PR: https://github.com/apache/fineract/pull/5747

## Instance 4: FINERACT-2583

Files: HolidayWritePlatformServiceImpl.java, ProvisioningCriteriaAssembler.java, CollateralAssembler.java
Methods: holiday office selection, provisioning criteria assembly, collateral assembly
Invariant arguments: Office, product, and collateral type IDs
DB calls before: N x 1 per method
DB calls after: 1 per method
Fix: bulk findAllById with in-memory map lookups across all three methods
PR: https://github.com/apache/fineract/pull/5748
