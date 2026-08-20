# Thematic Coding Schema

Applied to mailing list threads and JIRA comments across all three cases.

## Categories

1. Security risk acknowledgment
   A contributor explicitly names a vulnerability, CVE, or security
   concern associated with the subsystem under discussion.

2. Contributor availability
   A statement about whether anyone is willing or able to take
   ownership of a subsystem or remediation task.

3. Architectural isolation proposals
   A suggestion to move a subsystem behind a boundary, into a
   separate module, or out of the core project entirely.

4. Removal rationale
   The stated justification for deleting code, an endpoint, or
   a feature rather than fixing or maintaining it.

5. Dependency surface evidence
   Any claim or investigation about whether downstream consumers
   reference the item under discussion.

**Note:** "Dependency surface evidence" appears in earlier paper versions as a
thematic-coding category. In the current integrated paper, dependency-surface
analysis is described separately as a structured architectural/dependency
analysis. The category is retained here to preserve the coding schema used
during the broader evidence analysis.

## Application notes

Each mailing list post and JIRA comment was coded independently.
A single post could receive multiple codes where applicable.
Codes were applied to the smallest unit of text that carried
the meaning, typically a sentence or paragraph.
