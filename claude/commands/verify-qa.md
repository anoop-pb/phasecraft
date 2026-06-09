---
description: "PhaseCraft: Run a QA Agent review for the specified phase — audits test coverage against feature specs and test specs"
agent: qa
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running a QA Agent review session. Your task is to audit test coverage for the specified phase against feature specs and test specs, and produce a gap report.

On session start, before anything else:
1. Read `.claude/agents/QA-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — confirm build is complete for this phase. Verify `/.agentic-reviews/qa/test-results-phase-{N}.txt` exists. If not, print entry banner with BLOCKED status and stop: tell the user the Coding Agent must run tests and save results before QA review can proceed.
4. Determine run number: scan `/.agentic-reviews/qa/` for files matching `qa-review-phase-{N}-run-*.md`, take the highest run number found, add 1. Use run-1 if none exist.
5. Read the PRD version column and Progress notes for this phase from TRACKER.md.
6. Read your agent inbox at `.agent-messages/qa/` — directed messages from other agents — and consume per AGENT-STANDARDS Session Entry (record actionable content in your Progress cell → delete → act); leave deferred messages in place. An absent or empty inbox means nothing to consume.
7. Print the entry banner per AGENT-STANDARDS.md Session Entry.
8. Complete the review per your agent definition.
9. Write report to `/.agentic-reviews/qa/qa-review-phase-{N}-run-{R}.md`.
10. Update TRACKER.md QA review column and PRD version column for this phase.
11. If the review routes work to another agent beyond what the report itself carries, send a message to that agent's inbox per AGENT-STANDARDS (formal gaps stay in the report; messages are for ad-hoc context only).
12. Propose a commit message.

Phase number: $ARGUMENTS
