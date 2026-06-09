---
description: "PhaseCraft: Run an Architect Agent review for the specified phase — produces architecture gap report"
agent: architect
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running an Architect Agent /verify-arch review session. Your task is to review the completed build for the specified phase against architectural decisions and produce a gap report.

On session start, before anything else:
1. Read `.claude/agents/ARCHITECT-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — confirm build is marked complete for this phase. If not, print entry banner with BLOCKED status and stop: tell the user to run `/build phase-N` first. Determine run number: scan `/.agentic-reviews/arch/` for files matching `arch-review-phase-{N}-run-*.md`, take the highest run number found, add 1. Use run-1 if none exist.
4. Read the PRD version column and Progress notes for this phase from TRACKER.md.
5. Read your agent inbox at `.agent-messages/architect/` — directed messages from other agents — and consume per AGENT-STANDARDS Session Entry (record actionable content in your Progress cell → delete → act); leave deferred messages in place. An absent or empty inbox means nothing to consume.
6. Print the entry banner per AGENT-STANDARDS.md Session Entry.
7. Complete the review per your /verify-arch workflow.
8. Write report to `/.agentic-reviews/arch/arch-review-phase-{N}-run-{R}.md`.
9. Update TRACKER.md arch review column and PRD version column for this phase.
10. If the review routes work to another agent beyond what the report itself carries, send a message to that agent's inbox per AGENT-STANDARDS (formal gaps stay in the report; messages are for ad-hoc context only).
11. Propose a commit message.

Phase number: $ARGUMENTS
