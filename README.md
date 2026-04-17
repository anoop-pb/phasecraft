# PhaseCraft — An Opinionated Agentic Development Framework

PhaseCraft is a multi-agent SDLC framework for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It gives you a team of specialized AI agents — Product, UI, Architect, Coding, QA and Project Manager, each with strict role boundaries and file access controls. You run slash commands; agents handle spec, design, architecture, implementation, and review through a structured lifecycle.

Work is organized into phases — coherent, dependency-ordered sets of features, each fully built, tested, and reviewed before the next phase begins. Think of a **phase as a sprint scoped for AI**: bounded by token limits and context drift, not time.

> **Beta** — this framework is under active development and testing on Claude Code via Claude Desktop. Other environments have not been tested. Expect rough edges, submit feedback/bugs and check back for updates.

A *session* in this README means one Claude Code conversation — from the moment you type a slash command to when the agent prints its handoff summary.  
**Important:** Each slash command should run in its own session to avoid context bleed between agents.

---

### How this README is structured

> **Start here:** [Lifecycle & Workflow](#lifecycle--workflow) → [Quick Start & First Project](#quick-start--first-project)  
> **Using PhaseCraft:** [Command Reference](#command-reference)  
> **Understanding PhaseCraft:** [Framework Internals](#framework-internals)  
> **Reference:** [Practical Notes](#practical-notes) (models, multi-engineer, troubleshooting)  

---

## Lifecycle & Workflow

```
INIT → SPEC → UI → ARCH → PLAN → BUILD (per phase) → REVIEW (per phase)
```

| Stage | What gets done | Agent |
|-------|-------------|-------|
| **INIT** | Project setup, domain for agents, repo structure | Project Manager |
| **SPEC** | Feature specs, test specs, non-functional requirements | Product Agent |
| **UI** | Personas, UI/UX spec, conversational interface spec | UI Agent |
| **ARCH** | Architecture, data models, deployment spec | Architect Agent |
| **PLAN** | Divide features into dependency-ordered phases | Project Manager |
| **BUILD** | Implement and test one phase at a time | Coding Agent |
| **REVIEW** | Architecture review + QA review per phase | Architect + QA Agents |

  
While Phases are sequential, Stages are checkpoints, not locks. Any stage can be reopened without rolling back downstream work.

### Phases and review gates

The `/plan` command divides your features into dependency-ordered phases. Each phase is small enough for one Claude Code session (avoids token exhaustion), self-contained enough to test in isolation, and sequenced so foundational layers are built and verified before features that depend on them.

BUILD and REVIEW repeat for each phase. After each build, two independent reviews must pass before the next phase begins:

- **`/verify-arch`** — checks code against architecture decisions, data models, and non-negotiables
- **`/verify-qa`** — audits test coverage against feature specs and test specs

Both must report **CLEAR**. If gaps are found, the reports tell you exactly what to fix and which agent handles it. Review reports are immutable — each run creates a new versioned file as an audit trail.

### Dependencies between stages

Stages generally run in order, and the framework will block you if a prerequisite is missing. The main nuances worth knowing upfront:

- Not all phases need to be fully spec'd upfront — the current phase (or two) needs complete feature spec, UI and architecture. Additional features can be evolved incrementally by re-running `/spec`, `/ui`, `/arch` and `/plan`.
- SPEC must cover foundational features before UI can begin — typically the core user flows and anything with non-obvious UI infrastructure needs
- ARCH needs enough UI stability for foundational decisions, but does not require complete UI coverage
- After ARCH is in place, UI for later phases may proceed incrementally (without re-running `/arch`) as long as no new UI infrastructure requirements are introduced 


---

## Quick Start & First Project

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- A git-initialized repository for your project

### Setup

```bash
# 1. Create project repo
mkdir my-project && cd my-project && git init

# 2. Download and Copy PhaseCraft framework files into your project root
cp /path/to/phasecraft/CLAUDE.md .
cp /path/to/phasecraft/DOMAIN-PERSONA.md .
cp -r /path/to/phasecraft/claude/ .claude/

# 3. Open Claude Code and run
/start
```

### Walkthrough

Each entry below should be run in a separate Claude Code session. Commit after each. To close a session with an agent, type 'wrap up', 'done' or 'close session' - this will trigger the agent to wrap up its work, and print a hand-off summary including any pending decisions or open questions (also recorded in TRACKER.md). The agent will also propose a commit message for you to commit.  

---

**`/start`** — The Project Manager asks what you're building, proposes a repo structure, and creates initial files after you confirm. This includes `DOMAIN-PERSONA.md`, which defines the domain expertise agents assume throughout the lifecycle — fill it out carefully.

---

**`/spec`** — Conversational. Each feature produces a paired feature spec and test spec file. NFR is captured in its own file. Run as many times as needed. When complete, a versioned compiled PRD and PRD Summary are generated in `/prd/output/`.

---

**`/ui`** — Produces personas, UI/UX spec, and conversational interface spec in `/ui/`. Skip if your app is headless (CLI, API-only, background service). After spec files are written, the UI Agent identifies which features warrant mockups and provides image-generation prompts. Use any image generation tool to create mockups, save them to `/ui/mockups/`, then proceed to `/arch`.

---

**`/arch`** — Reads PRD and UI specs, asks clarifying questions, proposes architecture. Pay close attention to the **Non-Negotiables** section it produces — these are decisions that cannot be changed without significant downstream rework.

---

**`/plan`** — Proposes phases with explicit scope and stop conditions. Treat the generated PHASES.md as a contract, at least for the current and upcoming phase.

---

**`/build {phase-number}`** — Implements exactly what is listed in the numbered phase block of PHASES.md . Writes tests alongside each feature, runs the full test suite, and iterates until all tests pass. Iterates up to 3 times on failures before stopping to ask for direction. Reports scope alignment on completion.

---

**`/verify-arch {phase-number}`** then **`/verify-qa {phase-number}`** — Architecture review followed by QA review for the specified phase number. Each produces a versioned report in `/.agentic-reviews/`.

If **GAPS FOUND**:

| Gap type | Action (with example phase number) |
|----------|------------------------------------|
| Architecture gap | `/build 1` to fix → re-run `/verify-arch 1` |
| QA gap (test implementation) | `/build 1` to fix → re-run `/verify-qa 1` |
| QA gap (spec ambiguity) | `/spec` to clarify → `/build 1` → re-run `/verify-qa 1` |

When both reviews are **CLEAR** → move to `/build 2` and repeat.

---

### Understanding the Situation and Next Steps

With work potentially spanning days and weeks, you do not need to recall or separately track the situation.

**After the first cycle**, use `/resume` at the start of each session — it reads TRACKER.md, tells you where things stand, and also recommends the next command (run in a separate session).

Running `/sitrep` in a new sesssion at any time will provide a brief report on the current state of the project.  

If you need to ask a question to a specific agent without wanting to run a full work session, use `/ask [agent]`.   

### Mid-lifecycle spec changes or new requirements

For requirement changes or new requirements after phases have been built, or even when a specific phase has been partially built (eg. pending verification), you may invoke the following for spec modifications:

1. `/spec` — update feature spec, generate new PRD version, generate diff
2. `/ui` — re-run if the change has UI/UX implications
3. `/arch` — re-run to check if the change has architectural implications
4. `/plan` — re-run always after spec changes or additions
5. `/build {phase-N}` — continue running current in-progress phase or next phase if no phase is in-progress
6. `/verify-arch {phase-N}` then `/verify-qa {phase-N}` — re-verify
  
**Important:** Completed Phases are never reopened. If a spec change makes a completed phase's output incorrect, the fix is forward-only — Project Manager's `/plan` session will add a corrective item to the next pending phase or the currently in-progress phase. In-progress phases are modified only if the change directly affects code being built or can't be deferred.  

PRD versions get a minor bump (+0.1) for modifications to existing features and a major bump (+1.0) for new features or sections added.

### Tips

- **One command per session.** This keeps context clean and avoids token exhaustion.
- **Let agents ask questions.** The framework is conversational — agents ask before assuming.
- **Don't skip stages.** The framework will block you if prerequisites aren't met.
- **Use `/resume` liberally.** If you're ever unsure what to do next, run it.

---

## Command Reference

| Command | Agent | Key Outputs |
|---------|-------|--------|
| `/start` | Project Manager | Repo structure, README, DOMAIN-PERSONA, TRACKER.md |
| `/resume` | Project Manager | Current state report + recommended next command |
| `/sitrep` | Project Manager | Current state report (read-only) |
| `/plan` | Project Manager | PHASES.md, TRACKER.md |
| `/spec` | Product Agent | Feature specs, test specs, NFR |
| `/ui` | UI Agent | Personas, UI/UX spec, conversational interface |
| `/arch` | Architect Agent | ARCHITECTURE.md, DATA-MODELS.md, DEPLOYMENT.md |
| `/build {phase-number}` | Coding Agent | Code + tests for phase N |
| `/verify-arch {phase-number}` | Architect Agent | Architecture gap report |
| `/verify-qa {phase-number}` | QA Agent | Test coverage gap report |
| `/ask [agent]` | Named agent (read-only) | Answers only; no file writes |

Each agent handles its own session entry, prerequisite checks, and TRACKER.md updates on close.


---

## Framework Internals

### Why PhaseCraft

Claude Code is powerful out of the box, but on non-trivial projects it tends to drift — mixing concerns, losing context over long sessions, skipping test coverage, and making architectural decisions on the fly. PhaseCraft solves this by imposing staged lifecycle gates, strict agent boundaries, phased builds and review checkpoints, including human-in-the-loop for commits.

### Agent overview

| Agent | Role | Reads | Writes |
|-------|------|-------|--------|
| **Project Manager** | Lifecycle, init, planning, status | Everywhere | TRACKER.md, PHASES.md, DOMAIN-PERSONA.md, README.md |
| **Product Agent** | Feature specs, test specs, NFR | `/prd/`, domain persona | `/prd/` only |
| **UI Agent** | Personas, UI/UX spec, conversational interface | `/ui/`, `/prd/`, domain persona | `/ui/` only |
| **Architect Agent** | Architecture, data models, deployment; arch review | `/prd/`, `/ui/`, `/arch/`, `/app/` (review only) | `/arch/`, review reports to `/.agentic-reviews/arch` |
| **Coding Agent** | Implementation, tests | Everywhere | `/app/`, test results |
| **QA Agent** | Test coverage audit | `/prd/`, `/app/tests/`, test results | Review reports to `/.agentic-reviews/qa` |  

In addition, all agents write status to TRACKER.md .  
NOTE: `/` here represents project root.


Agents are explicitly instructed not to read or write outside their boundaries. Also, if an agent encounters something outside its role, it stops and tells you which command to run instead.

### Folder structure (from project root)

```
CLAUDE.md                    ← framework entry point; auto-loaded into every session
DOMAIN-PERSONA.md            ← domain expertise persona; created during /start
TRACKER.md                   ← lifecycle and phase status; updated by all agents
PHASES.md                    ← phase scope and boundaries; generated by /plan

.claude/
  FRAMEWORK-VERSION
  agents/
    AGENT-STANDARDS.md       ← universal rules; read by all agents
    PROJECT-MANAGER-AGENT.md ← /start, /plan, /resume, /sitrep
    PRODUCT-AGENT.md         ← /spec
    UI-AGENT.md              ← /ui
    ARCHITECT-AGENT.md       ← /arch, /verify-arch
    CODING-AGENT.md          ← /build
    QA-AGENT.md              ← /verify-qa
  commands/                  ← slash command definitions

prd/                         ← generated by /spec
  sections/
    01-overview.md
    02-objectives.md
    03-functional-requirements/
      01-feature-name.md
      01-feature-name-tests.md
    04-non-functional-requirements.md
    05-future-features.md
  output/                    ← versioned compiled PRDs and summaries
  diffs/                     ← PRD version diffs

ui/                          ← generated by /ui
  01-personas.md
  02-ui-ux-spec.md
  03-conversational-interface.md
  mockups/

arch/                        ← generated by /arch
  ARCHITECTURE.md
  DATA-MODELS.md
  DEPLOYMENT.md

app/                         ← all application code; generated by /build
  tests/

.agentic-reviews/            ← generated by /verify-arch and /verify-qa
  arch/
  qa/
```

### Framework file roles

**CLAUDE.md** is the universal framework foundation — auto-loaded into every session. It contains lifecycle stage ordering, TRACKER.md format definitions, and Phase Rework Policy. It contains no agent persona or command workflows.

**Agent files** (`.claude/agents/`) define each agent's role, guardrails (file access boundaries), workflow logic, output format, and exit criteria.

**Agent Standards** (`AGENT-STANDARDS.md`) owns behavioral rules that apply universally: session entry/exit, scope discipline, proposal discipline, commit conventions, and review report integrity.

**Command files** (`.claude/commands/`) bootstrap each session: dispatch to the correct agent, enforce tool permissions, and run prerequisite checks. They contain procedural steps that overlap with agent files — this duplication is intentional for context resilience.

**DOMAIN-PERSONA.md** defines the domain expertise that various agents assume. A legal case management app gets agents that understand the legal profession; an e-commerce app gets agents that understand retail/commerce. This is filled conversationally during `/start` and significantly improves spec and design quality. Without it, agents operate as generalists. The domain persona file layers domain knowledge (using "assume expertise in ...", not "you are a ...") onto relevant agent's existing roles.  

### How sessions work

Agent sessions run directly — the user types a slash command and talks directly to that agent for the full session. The agent handles its own entry banner, prerequisite checks, conversational workflow, TRACKER.md updates, and handoff summary on close.

**Design decision:** Commands use the `agent:` frontmatter field (without `context: fork`) because PhaseCraft agents are conversational — the user interacts directly with the agent for the full session. `context: fork` would run the agent as a delegated task, preventing direct conversation. The `agent:` field loads the agent MD as the system prompt while the command file serves as the task prompt, and CLAUDE.md is auto-loaded as universal context. AGENT-STANDARDS.md is read by all agents.

### Extending the framework

To add a custom agent (e.g., a Security Agent or Deploy Agent), create a new `*-AGENT.md` in `.claude/agents/` following the structure of existing agent files, and a corresponding command file in `.claude/commands/` with the `agent:` frontmatter pointing to it.

---

## Practical Notes

### Recommended models

PhaseCraft does not enforce a model per command — choose based on your account, budget, and preference.

| Command | Recommended | Reason |
|---------|-------------|--------|
| `/start`, `/resume`, `/sitrep`, `/plan` | Sonnet | Coordination and file work; reasoning depth not critical |
| `/spec`, `/ui` | Sonnet | Conversational spec work; quality driven by domain persona and inputs |
| `/arch` | Opus | Highest reasoning depth; Non-Negotiables have large downstream impact |
| `/build` | Sonnet | Code generation; quality driven by spec and arch inputs |
| `/verify-arch`, `/verify-qa` | Opus | Gap detection benefits from careful reasoning |
| `/ask` | Sonnet | Conversational Q&A |

When in doubt, use Sonnet for speed and cost, Opus when the output has significant downstream consequences.

### Multi-engineer usage

Phases are strictly sequential — one phase complete before the next begins. Different engineers can own different stages or phases; the handoff mechanism is always files (TRACKER.md, PHASES.md, review reports). Human role coordination as well as orchestration among humans and AI is outside the scope of this framework.

### Updating the framework

Copy new agent MDs into `.claude/agents/`, command MDs into `.claude/commands/`, `CLAUDE.md` into the project root, and `FRAMEWORK-VERSION` into `.claude/`. All agents pick up changes automatically on their next invocation. Restarting Claude Desktop is recommended. Existing review reports are unaffected.

### Troubleshooting

**"BLOCKED" on session entry** — A prerequisite isn't met. The message says exactly what's missing and which command produces it.

**Agent refuses to do something** — Agents have strict role boundaries. If you ask the Architect Agent about test coverage, it will redirect you to the QA Agent. Follow the redirect.

**Session crashed or ended unexpectedly** — Agents write progress to TRACKER.md incrementally. Run `/resume` to pick up where you left off. At most you lose the current unit of work.

**Phase review found gaps** — This is the system working as intended. The review report routes each gap to the right agent. Run `/build N` for coding gaps, `/spec` for spec issues, then re-verify.

**Check status without changing anything** — `/sitrep` for a full read-only report, or `/ask [agent]` to consult any agent without making changes.

### Known issues

Occasionally, slash commands with some models may trigger delegation to the agent in the background, spawned via the Agent tool (similar to context: fork behaviour). This happens despite the `agent:` field defined correctly in the command YAML frontmatter. To prevent this, an additional instruction has been added to the top of CLAUDE.md . This has been specifically observed with Opus.

Nevertheless if and when you face this problem, you will not see the session entry banner (which has the correct agent name sandwiched between ____ horizontal lines). In such a case, try starting a new session and running the command again, or use a different model.