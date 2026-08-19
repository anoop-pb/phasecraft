---
description: "PhaseCraft: Run a Product Agent UAT session — exercise the running application against feature and UI specs, for one phase, as a full-product sweep, or as a write-up of the user's own findings"
agent: product
allowed-tools: Read, Write, Edit, Glob, mcp__Claude_Browser__navigate, mcp__Claude_Browser__read_page, mcp__Claude_Browser__get_page_text, mcp__Claude_Browser__find, mcp__Claude_Browser__computer, mcp__Claude_Browser__form_input, mcp__Claude_Browser__read_console_messages, mcp__Claude_Browser__read_network_requests, mcp__Claude_Browser__resize_window, mcp__Claude_Browser__tabs_context
---
You are running a Product Agent /verify-uat session. Your task is to exercise the running application for the specified phase against its feature specs and UI specs, and produce a deviation report. This is agent-run UAT — human acceptance is separate and happens after your report is read.

On session start, before anything else:
1. Read `.claude/agents/PRODUCT-AGENT.md`
2. Read `DOMAIN-PERSONA.md`
3. Read `.claude/agents/AGENT-STANDARDS.md`
4. Read `TRACKER.md` and `PHASES.md` — determine the phase number and mode from $ARGUMENTS, per PRODUCT-AGENT.md → /verify-uat → Modes, which governs every mode difference below. If neither a phase number nor `full` is supplied, name the lowest phase whose arch and QA reviews are CLEAR but whose UAT column is `—`, and confirm with the user before proceeding. State the mode in the entry banner.
5. Verify prerequisites, applying the Mode carve-outs in PRODUCT-AGENT.md: PHASES.md marks this phase `UAT: applicable` (if `N/A`, note it in the entry banner and stop — there is nothing to exercise); TRACKER.md shows both Arch Review and QA Review CLEAR on their latest run for this phase; `/.agentic-reviews/build/runtime-phase-{N}.md` exists and the application it names responds. If any prerequisite is not met, print the entry banner with BLOCKED status and stop, naming what is missing and which command produces it. If the application is simply not running, say so and give the start command from the runtime handle — you cannot start it yourself. The entry banner is mandatory in all cases, even when blocked. Always use the exact formatted block from AGENT-STANDARDS.md, not prose.
6. Determine run number: scan `/.agentic-reviews/uat/` for files matching this run's report name pattern, take the highest run number found, add 1. Use run-1 if none exist.
7. Read the PRD version column and Progress notes for this phase from TRACKER.md.
8. Read your agent inbox at `.agent-messages/product/` and consume per AGENT-STANDARDS Session Entry (record actionable content in your Progress cell → clear the message file → act); leave deferred messages in place. An absent or empty inbox means nothing to consume.
9. Print the entry banner per AGENT-STANDARDS.md Session Entry.
10. Complete the review per your /verify-uat workflow for the mode in force. Work from the specs outward — read a requirement, then find out whether the running product satisfies it. Do not browse the application and write up what you notice.
11. Write the report to `/.agentic-reviews/uat/`, named per the mode in force, including the mandatory `## Orchestrator Verdict` footer.
12. Update the TRACKER.md UAT column and PRD version column for this phase. Where any deviation routes to the Coding Agent, apply PRODUCT-AGENT.md → Routing a build fix.
13. If deviations route to another agent beyond what the report itself carries, send a message to that agent's inbox per AGENT-STANDARDS (formal deviations stay in the report; messages are for ad-hoc context only).
14. Propose a commit message.

**You never write to `/prd/` in this session, and never read `/app/` or `/arch/`.** If the specs turn out to be wrong, ambiguous, or contradicted by what was built, that is a deviation routed to the Product Agent — resolved in a separate `/spec` session with the user present, not here.

Arguments (phase number, `full`, and/or `manual`): $ARGUMENTS
