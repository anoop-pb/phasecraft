---
description: "PhaseCraft: Run a UI Agent session — define personas, UI/UX spec, and conversational interface spec"
agent: ui
allowed-tools: Read, Write, mcp__filesystem__*
---
You are running a UI Agent session. Your task is to define personas, UI/UX spec, and conversational interface spec through conversation with the user.

On session start, before anything else:
1. Read `.claude/agents/UI-AGENT.md`
2. Read `DOMAIN-PERSONA.md`
3. Read `.claude/agents/AGENT-STANDARDS.md`
4. Read `TRACKER.md` — verify a PRD exists in `/prd/output/`. If not, print entry banner with BLOCKED status and stop: tell the user to run /spec first. Also check `/prd/sections/03-functional-requirements/` — confirm that the features the user intends to cover this session have stable paired spec and test files. If not, note which features are missing specs and flag to the user before proceeding.
5. If no UI spec files exist yet in `/ui/` (first `/ui` run): list the features found in `/prd/sections/03-functional-requirements/` and ask the user to confirm which features are in scope for the MVP UI. Verify all confirmed MVP features have stable paired spec and test files before proceeding. If any are missing, tell the user to run `/spec` to complete them first.
6. Note any handoff notes from the UI row in TRACKER.md.
7. Print the entry banner per AGENT-STANDARDS.md Session Entry.
8. Always enter the session and print the entry banner regardless of current UI status — never silently exit or defer to another agent.
9. Begin the session per your Workflow.
