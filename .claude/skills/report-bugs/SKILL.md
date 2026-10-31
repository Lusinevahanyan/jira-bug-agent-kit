---
name: report-bugs
description: End-to-end workflow - find bugs in a scope (or take bugs already found), get human approval, then file them in Jira. Run as /report-bugs <scope or bug description>.
disable-model-invocation: true
argument-hint: "<files | package | branch | 'already found: ...'>"
---

# /report-bugs workflow

Input: $ARGUMENTS

## Step 1 - Get findings
- If the input starts with "already found" or describes concrete bugs from this
  conversation: convert them into findings using the jira-bug-report template.
  Do NOT re-hunt.
- Otherwise: delegate to the `bug-hunter` subagent with the given scope.

## Step 2 - Save
Write all findings to `bug-reports/pending/<YYYY-MM-DD>-<short-scope>.md`.

## Step 3 - Human gate (mandatory)
Show the user a compact table: id | summary | severity | confidence.
Ask which ids to file. Accept "all", a list of ids, or "none".
Wait for the answer. Never continue on silence or ambiguity.

## Step 4 - File
Delegate to the `jira-bug-reporter` subagent with ONLY the approved findings
(full text) and the explicit line: "Approved by user: <ids>".

## Step 5 - Close the loop
Show the reporter's result table (created / duplicate / failed + keys).
Move the pending file to `bug-reports/filed/` with the Jira keys added.
