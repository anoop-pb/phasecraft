# PhaseCraft — Agentic Development Framework

**When a slash command has an agent: field, adopt that agent's identity directly in this session. Do not delegate to a sub-agent via the Agent tool. Do not use context: fork or background agent spawning.**

A lifecycle-staged multi-agent framework for building software with AI. Each stage is worked on by one or more focused agents with explicit file access boundaries. Agents do not drift into each other's concerns.

---

## Lifecycle Stages

```
INIT → SPEC → UI → ARCH → PLAN → BUILD (per phase) → REVIEW (per phase)
```
INIT    - project initialization
SPEC    - product specifications
UI      - UI/UX specifications
ARCH    - technical architecture
PLAN    - phase planning
BUILD   - implementation and testing
REVIEW  - architecture & QA verification

Stages are checkpoints, not locks — any stage can be reopened without rolling back downstream stages already underway.

### Phases

The plan stage divides features into dependency-ordered phases. Each phase is a set of features that can be fully tested in isolation. Phases are bounded by AI constraints for building and testing - preventing too much scope in a single session to avoid token exhaustion, containing context to avoid drift over long sessions and sequencing by dependency so that foundational layers are fully tested before the next phase builds on them.
BUILD and REVIEW repeat for each phase. A phase is not complete until both architecture and QA reviews are CLEAR. 

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

| Phase | PRD version | Build | Arch Review | QA Review | Complete | Progress |
|-------|-------------|-------|-------------|-----------|----------|----------|

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
| PRD version | `—` → `v{X.Y}` (set by the agent that last worked on this phase — Coding Agent on build, Architect Agent on arch review, QA Agent on QA review) |
| Build | `not started` → `in progress` → `complete` |
| Arch Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| QA Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| Complete | `—` → `✓ complete` (set by `/resume` reconciliation when both reviews show CLEAR on latest run) |

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

When a spec change affects multiple phases, these rules determine impact:

**Completed phases** (both reviews CLEAR): Never reopened. If a spec change makes a completed phase's output incorrect, the fix is forward-only — add a corrective item to the next pending phase.

**In-progress phases** (build started but not yet CLEAR): Modified only if the change directly affects features currently being built in that phase AND the fix cannot be deferred to a later phase without causing integration failures. Examples: (1) the change alters an API contract or data model that code in this phase depends on, or (2) the change removes a feature currently being implemented.

**Pending phases** (not started): Freely modified. Re-run `/plan` to reassign features or adjust scope.

---

## When No Slash Command Is Active

If a project is underway (TRACKER.md exists), read TRACKER.md, report current project status, and suggest the user run `/resume`.

If no project exists (no TRACKER.md), suggest the user run `/start`.
