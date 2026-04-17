---
description: "PhaseCraft: Run an Architect Agent session — produce ARCHITECTURE.md, DATA-MODELS.md, and DEPLOYMENT.md"
agent: architect
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running an Architect Agent /arch session. Your task is to produce architecture, data models, and deployment specs from the PRD and UI inputs.

On session start, before anything else:
1. Read `.claude/agents/ARCHITECT-AGENT.md`
2. Read `DOMAIN-PERSONA.md`
3. Read `.claude/agents/AGENT-STANDARDS.md`
4. Read `TRACKER.md` — verify a PRD exists in `/prd/output/`. If not, print entry banner with BLOCKED status and stop: tell the user to run /spec first. If ARCH has already run (ARCHITECTURE.md exists in `/arch/`), note this is a re-run and ask the user which features are new or changed before proceeding.
5. Note any handoff notes from the ARCH row in TRACKER.md.
6. Print the entry banner per AGENT-STANDARDS.md Session Entry.
7. Always enter the session and print the entry banner regardless of current ARCH status — never silently exit or defer to another agent.
8. Begin the session per your /arch workflow.
