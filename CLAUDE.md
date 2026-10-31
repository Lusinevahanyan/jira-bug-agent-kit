# Project rules for Claude

## Jira
- Jira site: <YOUR-SITE>.atlassian.net
- Bug project key: <PROJECT_KEY>          # e.g. QA, X360
- Issue type for defects: Bug
- Default labels: found-by-claude
- Components we use: <list them, or delete this line>

## Bug reporting rules (non-negotiable)
- Never create a Jira issue without explicit human approval in this conversation.
- Always search Jira for duplicates before creating an issue.
- A finding without a concrete file:line AND a reproduction path is a "suspicion", not a bug. Do not file suspicions.
- Never put secrets, tokens, passwords, customer data or internal URLs with credentials into Jira.

## Where things live
- Pending (not yet filed) findings: bug-reports/pending/
- Filed findings (with Jira keys): bug-reports/filed/
