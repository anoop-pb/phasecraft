# PhaseCraft — Agentic Development Framework

**When a slash command has an `agent:` field, adopt that agent's identity directly in this session. Do not delegate to a sub-agent via the Agent/Task tool. Do not use `context: fork` or background agent spawning.**

**First exception — the orchestrator.** The orchestrator agent (commands: `/build-loop` and `/uat-loop`) is authorized to spawn bounded workers (coding, architect, qa, and — for `/uat-loop` only — product) via the Agent tool. Each adopts the orchestrator identity directly in the main session via its `agent:` field with **no** `context: fork`, and from there spawns the workers. No other agent or command may spawn subagents, and the orchestrator must never itself be forked or run as a subagent — a forked/subagent context cannot spawn further workers.  For `/uat-loop`, which appends the UAT stage to the same loop, it may additionally spawn the product agent to run `/verify-uat`. The orchestrator may spawn only those four agents, only for those workflows, and only in Autonomous Mode (see `AGENT-STANDARDS.md` → Autonomous Mode).

**Second exception — Operator Mode.** When the user explicitly asks the main session to act as their **operator assistant** (no slash command run, no agent identity adopted), the main session may spawn any PhaseCraft agent **except the orchestrator** via the Agent tool, each in its own context, with a task prompt that explicitly states the agent is running in Operator Mode (see `AGENT-STANDARDS.md` → Operator Mode). The operator works through specifics with the agents on the user's behalf within the user's stated intent, pausing to ask the user only when a decision would conflict with or modify that intent. That intent is recorded in a confirmed intent file under `/intents/` before any work begins — the operator's one write target, and a path no agent reads or writes (see `AGENT-STANDARDS.md` → Operator Mode → The Intent File). The orchestrator is never spawned and neither `/build-loop` nor `/uat-loop` is used in operator sessions — the operator performs the build → verify → UAT sequencing itself.

A lifecycle-staged multi-agent framework for building software with AI. Each stage is worked on by one or more focused agents with explicit file access boundaries. Agents do not drift into each other's concerns.

---

## Lifecycle Stages

```
INIT → SPEC → UI → ARCH → PLAN → BUILD (per phase) → REVIEW (per phase) → UAT (per phase)
```
INIT    - project initialization
SPEC    - product specifications
UI      - UI/UX specifications
ARCH    - technical architecture
PLAN    - phase planning
BUILD   - implementation and testing
REVIEW  - architecture & QA verification
UAT     - agent-run acceptance testing of the running application

Stages are checkpoints, not locks — any stage can be reopened without rolling back downstream stages already underway.

### Phases

The plan stage divides features into dependency-ordered phases. Each phase is a set of features that can be fully tested in isolation. Phases are bounded by AI constraints for building and testing - preventing too much scope in a single session to avoid token exhaustion, containing context to avoid drift over long sessions and sequencing by dependency so that foundational layers are fully tested before the next phase builds on them.
BUILD, REVIEW and UAT repeat for each phase. A phase is not complete until both architecture and QA reviews are CLEAR and UAT is CLEAR (or marked `N/A` for that phase). UAT runs last, after both automated reviews pass — it exercises the running application, which is only worth doing once the build is settled. 

---

## Terminology

- **Progress** — the per-row TRACKER.md column holding the current in-flight unit of work (crash-resilience scratch). Not a communication channel.
- **Agent message** — a directed prose note from one agent to another via `.agent-messages/`. Not human-facing.
- **Handoff summary** — the human-facing recap printed in chat on session exit.

---

## TRACKER.md Format

TRACKER.md is the single source of truth for lifecycle and phase **state**, read and updated by all agents. State tracking only — directed agent-to-agent communication does not live here. That flows through the agent messages layer (format defined below under Agent Messages Format; behavioral rules in `AGENT-STANDARDS.md` → Agent Messages).

### Lifecycle Status Table

```
# Project Tracker

PRD version at last update: —
PhaseCraft version at last update: —

## Lifecycle Status

| Stage   | Status      | Last updated | Progress |
|---------|-------------|--------------|----------|
| SPEC    | not started | —            |          |
| UI      | not started | —            |          |
| ARCH    | not started | —            |          |
| PLAN    | not started | —            |          |
```

### Phase Status Table

```
## Phase Status

| Phase | PRD version | Build | Arch Review | QA Review | UAT | Complete | Progress |
|-------|-------------|-------|-------------|-----------|-----|----------|----------|

(phases added by /plan)
```

### The Progress Column

`Progress` is per-session crash-resilience scratch — it records the **in-flight unit of work** for the current stage or phase row, and is overwritten as work proceeds (not appended). Its only purpose is to ensure that an abruptly ended session loses at most the current unit of work. It is not a log, not an audit trail, and not a channel for handing context to another agent — directed handoffs go through the agent messages layer.

**Legacy note:** trackers created before this version may show this column labeled `Handoff notes`. Treat it as `Progress`. The first agent to write the row normalizes the header to `Progress`. No other migration is required — existing content remains valid in-flight context.

### Version Stamp

`PhaseCraft version at last update` records the framework version the project last ran under (read from `.claude/FRAMEWORK-VERSION`). The first agent to write TRACKER.md in a session sets this field to the current framework version. It exists so framework drift is detectable across upgrades.

### Phase Status Column Values

| Column | Valid states |
|--------|-------------|
| PRD version | `—` → `v{X.Y}` or `v{X.Y.Z}` (set by the agent that last worked on this phase — Coding Agent on build, Architect Agent on arch review, QA Agent on QA review). A patch the phase has picked up **is** recorded as `v{X.Y.Z}` for provenance (so verified state never silently predates a clarification). The scoping anchor remains the base minor: for all drift, rework, and equivalence purposes `v{X.Y.Z}` is treated as `v{X.Y}`. |
| Build | `not started` → `in progress` → `complete` |
| Arch Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| QA Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| UAT | `—` (not run) → `run-{R} CLEAR` or `run-{R} DEVIATIONS FOUND` → `N/A` (phase has no user-facing surface, or the project is headless; set by `/plan` from the phase block's `UAT:` line) |
| Complete | `—` → `✓ complete` (set by `/resume` reconciliation when both reviews show CLEAR on latest run **and** UAT shows CLEAR or `N/A`) |

---

## Agent Messages Format

The agent messages layer is the channel for directed agent-to-agent communication, separate from TRACKER.md state. The *behavioral rules* (when to read, the consume sequence, the write carve-out, sending on exit) live in `AGENT-STANDARDS.md` → Agent Messages. This section defines only the static contract: where messages live, how they are named, and what they contain.

### Layout

```
.agent-messages/                 ← transient; agent-to-agent communication
  product/                       ← one inbox folder per recipient agent
  ui/
  architect/
  coding/
  qa/
  project-manager/
```

Recipient folder names match agent names: `product`, `ui`, `architect`, `coding`, `qa`, `project-manager`. One file per message. The structure is created lazily — the sender creates the recipient's folder if it does not exist (`mkdir -p` semantics). A fresh clone has no `.agent-messages/` until the first message is sent; an absent inbox simply means no messages.

### Filename

```
{timestamp}__{from-agent}__{discriminator}.md
```

- `timestamp` — UTC ISO-8601 with filesystem-safe separators, e.g. `2026-05-21T08-57-03Z` (use `-` in place of `:`). The leading timestamp gives a deterministic chronological read order within an inbox.
- `from-agent` — the sending agent's name, for uniqueness and at-a-glance triage.
- `discriminator` — the operator value if known, else a short random nonce (e.g. 4 hex chars). Guarantees filename uniqueness across operators working independently, so concurrent pushes never collide.

If a same-instant, same-sender collision would occur, append a numeric suffix: `{timestamp}__{from-agent}-2.md`.

### Message contents

YAML frontmatter carries the deterministic fields; the body is prose written for the receiving agent.

```
---
from: coding
for: qa
stage: REVIEW          # lifecycle stage the message concerns
phase: 2               # phase number, or — if not phase-scoped
operator: —            # human owner/operator if known, else —
type: handoff          # see enum below
created: 2026-05-21T08:57:03Z
---

Prose message to the receiving agent.
```

`type` is a closed enum:

| Type | Meaning |
|------|---------|
| `handoff` | Context the next agent needs to continue the work |
| `gap-routed` | Pointing the recipient at something to address that is *not* already captured in a review report (formal arch/QA gaps still travel via reports, not messages) |
| `escalation` | Needs a decision or attention from the recipient |
| `ratification` | The sender already acted on or decided something (at the user's explicit request) that falls in the recipient's area of authority, and asks the recipient to validate or flag it on its next invocation. Use only when acting now was absolutely necessary — not as a routine way to cross role boundaries. |
| `blocker` | Something the recipient owns is preventing progress |
| `info` | FYI, no action required |

---

## Phase Rework Policy After Spec Changes

When a spec change affects phases, these rules determine impact:

**Patch bumps (clarifications) do not trigger rework — with one carve-out for completed phases.** A patch version bump (`v{X.Y.Z}`) is a clarification of existing language — no new or changed behavior or scope (as per the Product Agent's patch-eligibility test). It triggers no reassessment of any **in-progress or pending** phase: an in-progress phase picks the clarification up on its next `/build` (which records `v{X.Y.Z}` in its PRD version column), and a pending phase reads the clarified sections when it is first built. **Exception:** if a clarified section maps to a **completed** phase, the Coding Agent's current-phase scope cannot pick it up, so `/plan` detects this on its next run it scans the `PATCH-v*` changelogs, maps clarified sections to phases via PHASES.md, and adds a corrective item to the next pending or in-progress phase — the same forward-only rule completed phases follow for any spec change. This is the only downstream action a patch produces.

**Completed phases** (arch and QA reviews CLEAR, and UAT CLEAR or `N/A`): Never reopened. If a spec change makes a completed phase's output incorrect, the fix is forward-only — add a corrective item to the next pending phase.

**In-progress phases** (build started but not yet through every applicable gate): Modified only if the change directly affects features currently being built in that phase AND the fix cannot be deferred to a later phase without causing integration failures. Examples: (1) the change alters an API contract or data model that code in this phase depends on, or (2) the change removes a feature currently being implemented.

**Pending phases** (not started): Freely modified. Re-run `/plan` to reassign features or adjust scope.

---

## When No Slash Command Is Active

If a project is underway (TRACKER.md exists), read TRACKER.md, report current project status, and suggest the user run `/resume`.

If no project exists (no TRACKER.md), suggest the user run `/start`.
