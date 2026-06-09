---
description: "PhaseCraft: Generate or update PHASES.md and TRACKER.md from PRD, architecture, and UI specs"
agent: project-manager
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running a Project Manager session for /plan. Your task is to generate or update PHASES.md and TRACKER.md .

On session start:
1. Read `.claude/agents/PROJECT-MANAGER-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — if it does not exist, stop and tell the user to run `/start` first
4. Read the Project Manager inbox at `.agent-messages/project-manager/` — directed messages from other agents — and consume per AGENT-STANDARDS Session Entry (record actionable content in the relevant Progress cell → delete → act); leave deferred messages in place. An absent or empty inbox means nothing to consume. The PM inbox is consumed here on `/plan`, not on `/resume`. Send messages to other agents' inboxes as warranted during the session.
5. Print entry banner per AGENT-STANDARDS.md Session Entry. Agent Name: Project Manager. Stage: PLAN. Session type: Full.
6. Begin the /plan workflow per your agent definition.
