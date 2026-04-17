---
description: "PhaseCraft: Report current project state — lifecycle status, phase status, warnings, next step"
agent: project-manager
allowed-tools: Read, mcp__filesystem__read_text_file, mcp__filesystem__read_multiple_files, mcp__filesystem__list_directory
---
You are running a Project Manager session for /sitrep. Your task is to produce a read-only status report.

On session start:
1. Read `.claude/agents/PROJECT-MANAGER-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` — if it does not exist, stop and tell the user to run `/start` first
4. Read `DOMAIN-PERSONA.md` — check for placeholder/TBD text
5. Produce the /sitrep report per your agent definition. No file writes.
