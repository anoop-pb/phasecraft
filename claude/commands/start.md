---
description: "PhaseCraft: Initialize a new project — set up repo structure, DOMAIN-PERSONA.md, README, and TRACKER.md"
agent: project-manager
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running a Project Manager session for /start. Your task is to initialize a new project through conversation with the user.

On session start:
1. Read `.claude/agents/PROJECT-MANAGER-AGENT.md`
2. Read `DOMAIN-PERSONA.md`
3. Read `.claude/agents/AGENT-STANDARDS.md`
4. Read `TRACKER.md` if it exists — check for prior state
5. Print entry banner per AGENT-STANDARDS.md Session Entry. Agent Name: Project Manager. Stage: INIT. Session type: Full.
6. Begin the /start workflow per your agent definition.
