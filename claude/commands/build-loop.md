---
description: "PhaseCraft: Run one phase's build → verify-arch → verify-qa loop under controlled autonomy until both reviews are CLEAR or a human decision is required"
agent: orchestrator
allowed-tools: Read, Glob, Agent(coding), Agent(architect), Agent(qa)
---
You are running the Orchestrator for `/build-loop`. Adopt the orchestrator identity directly in THIS (main) session via the `agent:` field above. Do NOT use `context: fork`, and do not let this command run as a subagent — you must stay in the main conversation to spawn the build/verify workers (see CLAUDE.md). Your task is to drive Phase N's build → verify loop to a terminal state (PHASE COMPLETE or a HALT).

On session start, before anything else:
1. Read `.claude/agents/ORCHESTRATOR-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md` (note the Autonomous Mode section — it governs every worker you spawn).
3. Determine the phase number N from $ARGUMENTS. If none is supplied, print the entry banner with BLOCKED status, name the lowest incomplete phase (per `TRACKER.md`) as the likely intended phase, and stop — `/build-loop` requires an explicit phase number.
4. Read `TRACKER.md` and `PHASES.md`. Run the Prerequisites gate from ORCHESTRATOR-AGENT.md. If any prerequisite is not met, print the entry banner with BLOCKED status, name what is missing and which command produces it, and stop. The entry banner is mandatory in all cases, including when blocked — use the exact formatted block from ORCHESTRATOR-AGENT.md, not prose.
5. Resolve the entry state (Entry-State Resolution in ORCHESTRATOR-AGENT.md) from the phase row's `Build`/`Arch Review`/`QA Review` columns in TRACKER — they carry the CLEAR/GAPS status you need. Identify (do not read) the latest arch and QA report run numbers and paths so a fix-build can be pointed at them; you read a report's contents only in-loop, for the `## Orchestrator Verdict` footer of one a worker has just written. You read no `Progress` cell from TRACKER and no agent inbox of your own — you are read-only and persist no state, and each worker you spawn performs its own Session Entry (reading its `Progress` and consuming its inbox) per AGENT-STANDARDS.
6. Print the entry banner per ORCHESTRATOR-AGENT.md. (It intentionally overrides the standard AGENT-STANDARDS Session-Entry banner — the orchestrator is a loop controller, so the banner shows loop state, not the usual agent/stage fields.)
7. If the requested phase is already complete (both latest reviews CLEAR), note this in the entry banner and stop without spawning anything — ask the user what they need rather than re-running the loop.
8. Otherwise, list the features to be implemented this phase (from the phase block) and wait for the user to confirm before starting the loop. This is the single entry gate: once confirmed, the loop runs unattended to a terminal state with no further prompts.
9. Run the loop per ORCHESTRATOR-AGENT.md → The Loop. Spawn the coding, architect, and qa agents via the Agent tool, strictly one at a time, each in Autonomous Mode.
10. On reaching a terminal state, print the terminal summary with a single proposed commit message. Do not commit, and do not write TRACKER.md — the phase's `Complete` flag is reconciled by the Project Manager on `/resume` (see ORCHESTRATOR-AGENT.md → Successful Completion / Halt Handling).

When you spawn a worker via the Agent tool, you describe its task (Claude composes the delegation message); include: the agent to run **and which of its workflows** — `/build` for coding, `/verify-arch` for architect, `/verify-qa` for qa (naming the command matters: the Architect agent owns both `/arch` and `/verify-arch`, so it must be told to run `/verify-arch`), the phase number, "Autonomous Mode per AGENT-STANDARDS → Autonomous Mode", for a fix-build the instruction to address the highest-run arch/QA gap reports for Phase N, and the requirement to write its normal artifacts and verdict. Because only the worker's final summary returns, treat the on-disk artifacts as the source of truth: read each verify verdict from the `## Orchestrator Verdict` footer of the report file it wrote, and read the build verdict from `/.agentic-reviews/build/build-verdict-phase-{N}.md`. The first gap that requires a spec or architecture decision halts the loop and returns control to the user.

Phase number from arguments: $ARGUMENTS
