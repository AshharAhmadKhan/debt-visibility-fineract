# PR and JIRA Mapping

## Case A: Visible but Unowned (Self-Service Security Debt)

- PR #4671 / FINERACT-2283: Disable phase (self-service disabled by default)
  https://github.com/apache/fineract/pull/4671

- PR #5498 / FINERACT-2480: Delete phase (full module removal)
  https://github.com/apache/fineract/pull/5498

## Case B: Invisible at Execution Surface (Phantom Command Path)

- PR #5655 / FINERACT-2545: Discovery (implementation attempt that surfaced the phantom)
  https://github.com/apache/fineract/pull/5655

- PR #5764 / FINERACT-2587: Delete phase (write-path removal)
  https://github.com/apache/fineract/pull/5764

- PR #5805 / FINERACT-2595: Follow-up fix (FK violation on existing tenants)
  https://github.com/apache/fineract/pull/5805

## Case C: Economically Tolerated (Loop-Invariant Query Amplification)

- PR #5712 / FINERACT-2561: Currency lookup hoisted in saveAllDebitOrCreditEntries
  https://github.com/apache/fineract/pull/5712

- PR #5739 / FINERACT-2579: Bulk fetch in generateLoanProvisioningEntry
  https://github.com/apache/fineract/pull/5739

- PR #5747 / FINERACT-2582: Constant lookups hoisted in email and notification services
  https://github.com/apache/fineract/pull/5747

- PR #5748 / FINERACT-2583: Bulk fetch in holiday, provisioning criteria, and collateral assembly
  https://github.com/apache/fineract/pull/5748
