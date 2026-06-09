---
description: "PhaseCraft: Run a Coding Agent session — implement features for the specified phase"
agent: coding
allowed-tools: Read, Write, Bash, mcp__filesystem__*
---
You are running a Coding Agent build session. Your task is to implement the features listed in the specified phase block.

On session start, before anything else:
1. Read `.claude/agents/CODING-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — determine the phase number from $ARGUMENTS. If no phase number was provided, check the lowest phase with build status not started or in progress, and confirm with the user on what to do next. Wait for a response before proceeding.
4. After reading TRACKER.md, if the requested phase is marked complete, note this in the entry banner and ask the user what they need — do not treat it as a blocker. Do not update TRACKER.md and do not prompt the user to confirm features for implementation.
5. Verify prerequisites: PHASES.md has a block for this phase, ARCHITECTURE.md and DATA-MODELS.md exist in /arch/, latest PRD and corresponding PRD summary exist in /prd/output/. If any prerequisite is not met, print entry banner with BLOCKED status and stop. The entry banner is mandatory in all cases — even when blocked. Always use the exact formatted block from AGENT-STANDARDS.md, not prose.
6. Note any Progress notes from the phase row in TRACKER.md.
7. Read your agent inbox at `.agent-messages/coding/` — directed messages from other agents — and consume per AGENT-STANDARDS Session Entry (record actionable content in your Progress cell → delete → act); leave deferred messages in place. An absent or empty inbox means nothing to consume. Send messages to other agents' inboxes as warranted during the session and on exit.
8. Print the entry banner per AGENT-STANDARDS.md Session Entry.
9. After the entry banner, list any features to be implemented this phase and wait for the user to confirm before writing any code.
10. Update TRACKER.md: set Build column for this phase to `in progress` and update Last updated field - skip if phase is already marked complete.
11. Begin per your Workflow.

Phase number from arguments: $ARGUMENTS
