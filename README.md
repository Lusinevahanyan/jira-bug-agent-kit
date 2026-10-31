# Jira Bug Agent Kit for Claude Code (IntelliJ)

Copy everything in this folder into the ROOT of your project (next to `pom.xml`
or `build.gradle`). The Claude Code plugin in IntelliJ runs the Claude Code
CLI, so it reads these same files.

```
your-project/
├── CLAUDE.md                         # 1. Memory: project rules, always loaded
├── .mcp.json                         # 2. Tools: Atlassian MCP server (Jira)
├── .claude/
│   ├── settings.json                 # 3. Guardrails: what needs your approval
│   ├── skills/
│   │   ├── jira-bug-report/          # 4. Knowledge: how a good bug looks
│   │   │   ├── SKILL.md
│   │   │   └── templates/bug-template.md
│   │   └── report-bugs/SKILL.md      # 5. Workflow: /report-bugs command
│   └── agents/
│       ├── bug-hunter.md             # 6a. Worker: finds bugs (read-only)
│       └── jira-bug-reporter.md      # 6b. Worker: files approved bugs
└── bug-reports/{pending,filed}/      # 7. Audit trail
```

---

## Lesson 0 - What each piece is (learn this first)

| Piece | Analogy | Answers the question |
|---|---|---|
| CLAUDE.md | Team wiki page pinned on the wall | "What are the rules here?" |
| MCP server | A hand that can touch another system | "What can it DO outside the code?" |
| Skill | A checklist / standard / template | "HOW should it do this well?" |
| Subagent | A specialist colleague with own desk & tools | "WHO does this job, with which tools?" |
| Permissions / hooks | The manager's signature | "What needs approval?" |
| Slash command (skill with `disable-model-invocation`) | The button you press | "How do I start the process?" |

**Senior-lead rule:** an agent is *not* one big prompt. It's small pieces, each with one job.

---

## Lesson 1 - Connect Jira (MCP)  ~10 min

1. `.mcp.json` is already in the kit. Alternatively run in the IntelliJ terminal:
   ```bash
   claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2 --scope project
   ```
   Do **not** use the old `/v1/sse` URL from older tutorials - it's deprecated.
2. Start Claude Code, type `/mcp`, select `atlassian`, authenticate in the browser.
3. Test (read-only): *"List the Jira projects I can see."*
4. **Verify tool names:** in `/mcp` look at the atlassian tool list. If any name
   differs from the ones in `settings.json` and `jira-bug-reporter.md`, fix them there.
5. If OAuth is blocked by your company: your Jira admin must allow the
   Atlassian Rovo MCP server, or enable API-token auth. Ask them; don't work around it.

✅ Done when: Claude can list your projects and search one issue by JQL.

## Lesson 2 - Write the rules (CLAUDE.md)  ~5 min
Fill in `<YOUR-SITE>`, `<PROJECT_KEY>`, components. Ask Claude:
*"Show the issue types and required fields for project <KEY>."* If `Bug` needs
extra required fields (e.g. Affects Version), add them to CLAUDE.md.

## Lesson 3 - The knowledge (skill `jira-bug-report`)  ~15 min
Open `SKILL.md` and the template. **Change them to match your team's real
Jira style** - copy the format of the best bug ticket your team ever wrote.
This is the part that makes output look professional, not the AI model.

## Lesson 4 - The workers (subagents)
Run `/agents` in Claude Code - you should see `bug-hunter` and `jira-bug-reporter`.
Note the design:
- `bug-hunter` has **no** Jira tools and **no** Edit/Write. It cannot cause damage.
- `jira-bug-reporter` has Jira tools but **no** code analysis job. It cannot invent bugs.
This is *least privilege* + *separation of duties*.

## Lesson 5 - Guardrails (settings.json)
Reads from Jira are auto-allowed. `createJiraIssue` is in `ask`, so Claude Code
asks you before every ticket - a second safety net after the human gate in the workflow.
Confluence writes are denied entirely (not this agent's job).

## Lesson 6 - Report YOUR two bugs (first real run)
In the same IntelliJ Claude chat where the 2 bugs were found:
```
/report-bugs already found: the two bugs you identified above
```
Check the generated reports in `bug-reports/pending/`, approve, watch the tickets appear.
Review the tickets in Jira critically. Whatever you had to fix by hand ->
put it into the skill. That's how the agent improves.

## Lesson 7 - Hunting mode
```
/report-bugs src/main/java/com/yourco/backup/restore
```
Start with a SMALL scope (one package). Measure: of N findings, how many were real?
If < 70% are real, tighten the bug-hunter prompt before scaling up.

---

## What "ideal" looks like later (don't build yet)
1. **Evaluation set** - 10 known bugs + 10 clean files; rerun after every prompt change.
2. **Hook** - a PreToolUse hook on `createJiraIssue` that blocks a description
   containing secrets (regex) - deterministic, unlike a prompt.
3. **Link to tests** - the hunter proposes a failing JUnit/Playwright test as proof;
   then label becomes `reproduced` honestly.
4. **Plugin** - package `.claude/` as a plugin to share with the QA team.
5. **CI** - run hunter on PR diffs in CI, but file nothing automatically:
   post findings as a PR comment, humans decide.

## Common mistakes
- One agent that finds AND files automatically -> Jira flooded with false positives, team stops trusting it.
- Scanning the whole repo -> context overflow, shallow findings.
- No duplicate check -> 5 tickets for the same bug.
- Calling static analysis "reproduced".
