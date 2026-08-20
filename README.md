# Replication Package: Stewardship Gaps and Debt Visibility

Replication package for "Stewardship Gaps and Debt Visibility:
A Multi-Case Study of Latent Technical Debt in Apache Fineract"

Ashhar Ahmad Khan, Jamia Hamdard, 2026

All pull requests, JIRA issues, commits, and mailing list threads referenced
here are publicly archived under Apache Software Foundation governance and can
be independently verified.

Apache Fineract repository: https://github.com/apache/fineract
Apache OFBiz repository: https://github.com/apache/ofbiz-framework

## How to use this package

Start with **claim-to-artifact-index.md**. It maps every major empirical claim
in the paper to the specific artifact that supports it and the step needed to
verify it independently.

From there, each file below corresponds to a distinct part of the methodology.

## Contents

- **claim-to-artifact-index.md** — reviewer entry point; maps paper claims to
  artifacts and verification steps across all three cases and the OFBiz validation

- **pr-jira-mapping.md** — provenance index mapping every PR, JIRA issue, commit,
  and mailing list thread to its case and the specific paper claim it supports

- **runtime-verification.md** — exact commands used to verify Case B at runtime
  (HTTP invocation, git archaeology, database inspection) and the Case C
  PostgreSQL statement measurement protocol (N = 1, 10, 100, 1,000)

- **dependency-surface-checklist.md** — search queries confirming zero consumers
  for Case B (SDK, OpenAPI spec, frontend, integration tests) and OFBiz plugins
  verification for all confirmed detectability-failure instances

- **case-c-instance-index.md** — all seven loop-invariant query amplification
  instances in Fineract: file paths, methods, origin commits, persistence
  durations, and fix PRs

- **ofbiz-validation.md** — cross-project validation in Apache OFBiz: seven
  confirmed detectability failures, three disconfirmed candidates, six confirmed
  salience failures, 145 disconfirmed candidates, entity-engine execution-path
  analysis, and the negative accountability-failure result

- **thematic-coding-schema.md** — category definitions used during thematic
  analysis of mailing list threads and JIRA comments across all three cases
