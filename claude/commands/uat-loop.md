---
description: "PhaseCraft: Run one phase's build → verify-arch → verify-qa loop under controlled autonomy, then a single UAT pass against the running application"
agent: orchestrator
allowed-tools: Read, Glob, Agent(coding), Agent(architect), Agent(qa), Agent(product)
---
You are running the Orchestrator for `/uat-loop`. Adopt the orchestrator identity directly in THIS (main) session via the `agent:` field above. Do NOT use `context: fork`, and do not let this command run as a subagent — you must stay in the main conversation to spawn the workers (see CLAUDE.md). Your task is to drive Phase N to a terminal state through the full build → verify → UAT sequence.

`/uat-loop` is `/build-loop` plus one terminal step. The inner build → verify-arch → verify-qa cycle runs exactly as under `/build-loop`, under the same cycle cap of 3. Once both reviews are CLEAR in the same cycle, you run a **single** `/verify-uat` pass and stop on its verdict. There is no UAT retry loop — a UAT deviation always ends the invocation and returns control to the human.

On session start, before anything else:
1. Read `.claude/agents/ORCHESTRATOR-AGENT.md`
2. Read `.claude/agents/AGENT-STANDARDS.md` (note the Autonomous Mode section — it governs every worker you spawn).
3. Determine the phase number N from $ARGUMENTS. If none is supplied, print the entry banner with BLOCKED status, name the lowest incomplete phase (per `TRACKER.md`) as the likely intended phase, and stop — `/uat-loop` requires an explicit phase number.
4. Read `TRACKER.md` and `PHASES.md`. Run the Prerequisites gate from ORCHESTRATOR-AGENT.md. Additionally, if the phase block marks `UAT: applicable`, verify that `/.agentic-reviews/build/runtime-phase-{N}.md` exists and the application it names is reachable — you cannot start it, and neither can the Product worker. If it is not running, print the entry banner with BLOCKED status, give the start command from the runtime handle, and stop rather than running the whole inner loop only to halt at the last step. If the handle does not exist yet because the phase has not been built, that is expected — check reachability again before the UAT step instead.
5. Resolve the entry state (Entry-State Resolution in ORCHESTRATOR-AGENT.md). If both reviews are already CLEAR for this phase, skip the inner loop entirely and go straight to the UAT step. If UAT is also already CLEAR, stop — the phase is complete.
6. Print the entry banner per ORCHESTRATOR-AGENT.md, noting that this run includes the UAT step and whether the phase is marked `UAT: applicable` or `N/A`.
7. Otherwise, list the features to be implemented this phase (from the phase block) and wait for the user to confirm before starting. This is the single entry gate: once confirmed, the run proceeds unattended to a terminal state with no further prompts.
8. Run the loop per ORCHESTRATOR-AGENT.md → The Loop, including step 6 (verify-uat). Spawn workers via the Agent tool, strictly one at a time, each in Autonomous Mode.
9. On reaching a terminal state, print the terminal summary with a single proposed commit message. Do not commit, and do not write TRACKER.md — the phase's `Complete` flag is reconciled by the Project Manager on `/resume`.

When you spawn a worker via the Agent tool, you describe its task (Claude composes the delegation message); include: the agent to run **and which of its workflows** — `/build` for coding, `/verify-arch` for architect, `/verify-qa` for qa, `/verify-uat` for product (naming the command matters: the Architect agent owns both `/arch` and `/verify-arch`, and the Product agent owns both `/spec` and `/verify-uat`) — the phase number, "Autonomous Mode per AGENT-STANDARDS → Autonomous Mode", for a fix-build the instruction to address the highest-run arch/QA gap reports for Phase N, and the requirement to write its normal artifacts and verdict. **Never spawn the product agent to run `/spec`** — resolving a spec is a human decision, and an orchestrator that could commission one could resolve its own halt. Because only the worker's final summary returns, treat the on-disk artifacts as the source of truth: read each verdict from the `## Orchestrator Verdict` footer of the report file the worker wrote, and the build verdict from `/.agentic-reviews/build/build-verdict-phase-{N}.md`.

Phase number from arguments: $ARGUMENTS
