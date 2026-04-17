---
description: "PhaseCraft: Resume from current project state — read TRACKER.md and recommend the next slash command"
agent: project-manager
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running a Project Manager session for /resume. Your task is to read current project state and recommend the next slash command.

On session start:
1. Read `.claude/agents/PROJECT-MANAGER-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — if it does not exist, stop and tell the user to run `/start` first
4. Read `DOMAIN-PERSONA.md` — check for placeholder/TBD text
5. Run phase completion reconciliation per your agent definition
6. Report status and recommend next command per your agent definition
