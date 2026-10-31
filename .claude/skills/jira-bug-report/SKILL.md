---
name: jira-bug-report
description: Team standard for writing a Jira bug report - summary rules, required sections, severity/priority mapping, labels and redaction rules. Use whenever a bug is described, analysed or filed in Jira.
---

# How we write bug reports

This skill is KNOWLEDGE, not a workflow. Agents and commands use it so every
bug in Jira looks the same, whoever (or whatever) wrote it.

## Summary (title) rules
- Format: `[Component] <what is wrong> when <condition>`
- Max ~90 characters, no trailing period, no "Bug:" prefix.
- Describe the symptom, not the fix.
  - Good: `[Backup] Restore job reports success when 0 files were restored`
  - Bad:  `Fix null check in RestoreService`

## Description
Use the template in `templates/bug-template.md`. Every section is required;
write `Unknown` rather than inventing content.

## Severity -> Jira priority
| Severity | Meaning | Priority |
|---|---|---|
| S1 Critical | data loss, security hole, crash in main flow, no workaround | Highest |
| S2 Major | core feature wrong, workaround exists | High |
| S3 Minor | edge case, non-core feature, cosmetic with functional impact | Medium |
| S4 Trivial | cosmetic, typo | Low |

If unsure between two levels, choose the lower one and say why in the report.

## Labels
Always: `found-by-claude`. Add `static-analysis` when found by reading code,
`reproduced` only if someone actually executed the reproduction steps.

## Redaction (mandatory before filing)
Remove or mask: passwords, API tokens, connection strings, customer names,
emails, tenant IDs, internal hostnames with credentials.

## Honesty rules
- Static analysis is not reproduction. Never write "Reproduced" unless it was run.
- State the confidence level in the report.
