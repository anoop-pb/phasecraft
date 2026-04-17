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

## TRACKER.md Format

TRACKER.md is the single source of truth for lifecycle and phase status, read and updated by all agents.

### Lifecycle Status Table

```
# Project Tracker

PRD version at last update: —

## Lifecycle Status

| Stage   | Status      | Last updated | Handoff notes |
|---------|-------------|--------------|---------------|
| SPEC    | not started | —            |               |
| UI      | not started | —            |               |
| ARCH    | not started | —            |               |
| PLAN    | not started | —            |               |
```

### Phase Status Table

```
## Phase Status

| Phase | PRD version | Build | Arch Review | QA Review | Complete | Handoff notes |
|-------|-------------|-------|-------------|-----------|----------|---------------|

(phases added by /plan)
```

### Phase Status Column Values

| Column | Valid states |
|--------|-------------|
| PRD version | `—` → `v{X.Y}` (set by the agent that last worked on this phase — Coding Agent on build, Architect Agent on arch review, QA Agent on QA review) |
| Build | `not started` → `in progress` → `complete` |
| Arch Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| QA Review | `—` (not run) → `run-{R} CLEAR` or `run-{R} GAPS FOUND` |
| Complete | `—` → `✓ complete` (set by `/resume` reconciliation when both reviews show CLEAR on latest run) |

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
