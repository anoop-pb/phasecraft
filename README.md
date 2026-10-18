# PhaseCraft 2.0

PhaseCraft is a framework for building high-quality, maintainable software with Claude Code.

You talk to one main session, the **operator**. It records what you want. Then it hands each piece of work to a fresh worker session with one role: product manager, UI designer, architect, planner, developer, security reviewer, QA engineer, researcher, or codebase analyst. Work moves in **phases** - a set of features built and verified together. Each phase is built, reviewed, and tested before the next phase builds on it.

Each worker produces artifacts as per its role, and these artifacts are stored in the repo - feature specs, test specs, UI/UX specs, architecture, plans, code, tests, review reports, research findings, and so on. Appropriate artifacts form the bulk of the context for future sessions, keeping things maintainable over time.

The operator asks before it assumes. It records what you decide. It brings back only what needs your attention.

## Core Principles

- Generated artifacts (specs, architecture, etc.) and decisions live in the repo, providing context for long term maintainability across sessions.
- Scoped agent roles mean each output has clear ownership and session context stays focused on one kind of work.
- Each phase of work is bounded based on dependencies and sized to fit reasonably within a session's context.
- Controlled autonomy and intent-driven development, with guardrails and review loops.
- Independent verification: no role checks its own work. The operator runs the build checks itself. Separate roles review architecture, security and test coverage. UAT tests the running product as a user would.

## Coming from PhaseCraft 0.x

PhaseCraft 2.0 is a major rewrite of PhaseCraft 0.7. There is no migration path. A project already under way on 0.7 finishes on 0.7; use 2.0 for new work. For what triggered the rewrite and how 2.0 is better, see [Why 2.0](#why-phasecraft-20).

## Getting started

You need Claude Code, git, and Python 3. Pushing branches or opening PRs needs your code host's CLI, such as Github CLI (`gh`). UAT of a web interface uses Claude’s built-in browser, which is available when you run Claude Code in the Claude desktop app.

If your repo has been running another SDLC agentic process, finish or drop its work in flight first. Rename an existing CLAUDE.md or .claude/ folder with a -prior suffix.

1. Copy `CLAUDE.md` from PhaseCraft to your repo root
2. Copy PhaseCraft's `claude/` folder to your repo root and rename the folder to `.claude/`.
3. Start a Claude Code session in the repo. (`.claude/settings.json` sets `"agent": "operator"`, so every session starts as the operator.)
4. Send a first message: say "onboard this project", or say what you want built. The operator checks the repo first. On a new repo it offers greenfield onboarding; on a repo with code, brownfield onboarding runs before anything else (see below). You confirm what it sets up.
5. Once onboarding is done, tell the operator what you want built, if you have not already. It shapes the intent with you and writes it to an intent file. Work starts once you confirm it.

Later sessions also start with your first message. The operator reads the tracker. It then carries on with the task you give, or tells you where things stand and recommends the next step.

**Permission mode.** Workers inherit the session's permission mode. If you run the operator under accept-edits or bypass-permissions, every worker it spawns edits files unattended under that mode. Choose the mode with that in mind.

## Onboarding

Onboarding sets up the files every later session relies on. It runs once per repo. It ends when the operator writes `TRACKER.md` at the repo root. As later work progresses, the operator writes to the Tracker to record where things stand: phase status, verification state, in-flight work, open decisions. The Repo map (`REPO-MAP.md`) at the repo root records where everything lives.

**Greenfield** — no application code yet. The operator asks about the project, its users, its domain, its constraints, and whether it has a user interface. With your approval, it then writes the domain context, a project README, the repo map, the coding standards, and the tracker.

**Brownfield** — application code exists. Onboarding is required before any work, even if you give a task. The operator lists the repo's existing docs. A codebase analyst reads the code, runs the tests and the application, and derives what is missing: architecture files, a behaviour inventory, coding standards, and a repo map. You confirm each one.
- Existing code is the accepted baseline. It is not treated as verified.
- Existing code is not specced up front. An area gets specs when work first touches it.
- If the repo carries another process's records, open fixes and open decisions carry over with your confirmation.

A task you gave at the start proceeds once onboarding is confirmed.

## How work flows

```
specs → UI design → architecture → plan → per phase: build → architecture review → security review → QA review → UAT
```

- **Discussion stages.** You discuss specs, UI design, and architecture with the operator. It records the outcome in a brief. A worker writes the formal artifact from that brief. UI design is skipped for a project without a user interface.
- **Specs are versioned.** Each spec change gets a change note and a git tag, `prd-v{version}`. The type of each change decides what runs next. New and changed features go back through design and architecture. Clarifications go straight to the planner.
- **Phases and gates.** The planner divides features into phases. Each phase passes up to five gates, in order: build, architecture review, security review, QA review, and UAT. Security review and UAT run where the plan says they apply. UAT drives a browser, so it covers web interfaces only. The build gate passes on verification output the operator produces itself, not on the developer's word. Any code change sends the phase back through its reviews.
- **Findings.** Critical and Major findings are fixed in one batch, then reviewed again. Minor findings never block. A phase whose same gate still finds problems after two fixes comes to you.
- **Completed phases are frozen.** A defect found later is fixed as forward work in a later phase.
- **Wider UAT.** A user flow that spans phases is tested end to end once its last phase is complete. A full UAT runs when the intent's work is complete, and before any PR.

**Sessions stay short on purpose.** A session's context fills up with discussion and worker handoffs. As it fills, quality drops. When Claude Code compacts it, detail is lost. So each discussion stage gets its own session, and a build session covers at most 2 phases by default. You can change that number in the intent. At a limit, the operator stops at a clean boundary and asks you to start a new session. The tracker and the other files carry everything across, so nothing is lost.

## Your intent and what reaches you

Work runs on a confirmed intent file the operator records under `/intents/records/`. It holds:

- What you want, and why.
- Boundaries: what the operator must not do, touch, or decide alone.
- Review preference: pause for your review at the end of each discussion stage and each phase, or run through.
- Number of phases per session.
- Delivery options: commit only, push a branch, or push and open a PR.

Your answers to the operator's questions are appended to the file. A change of intent replaces it with a new file. The intent file is the contract the operator answers to. It is also how a new session knows what you authorized.

Irrespective of the review preference, the following always come to you:

- Any decision the operator cannot settle based on the intent.
- UAT deviations rated Major or Critical, and any other that the operator is unsure about.
- Changes to Non-Negotiables — architecture decisions marked as costly to reverse — and security decisions that change cost, the risk you accept, or what the app's end users must do.
- A phase whose same gate still finds problems after two rounds of fixes.
- A proposed corrective-only phase, for fixes that have piled up.
- Moving a task to a cheaper model.

You can also report problems you find yourself. The operator passes them to a UAT run, which checks them against the specs.

The operator never pushes to the default branch and never force-pushes. It merges a PR only when you tell it to merge that PR in the current session, and only after the PR's checks pass.

## The roles

The operator invokes the required worker role as a sub-agent, for each task needed. You never need to invoke them directly.

| Role | Does |
|------|------|
| operator | The main session. Intent, briefs, sequencing, tracker, gates, verification runs, spec versions, delivery. Never does role work. |
| product-manager | Feature specs, test specs, NFRs — and UAT of the running application. Never reads code or architecture. |
| ui-designer | Personas, UI/UX spec, conversational interface spec, mockups. |
| architect | The architecture files, including security decisions. Reviews each built phase against them, and for maintainability. |
| planner | Divides features into dependency-ordered phases, sized to fit a session. |
| developer | Implements a phase with tests. The only role that writes to the code root. |
| security-reviewer | Reviews each security-relevant phase for weaknesses. Fixes nothing. |
| qa-engineer | Audits test coverage against the specs. Writes no tests. |
| researcher | Answers one question from the web. Gets the question only (no project context); findings go to the operator alone. |
| codebase-analyst | Brownfield onboarding only. Derives artifacts from an existing codebase. |

Each role has its own tools and read/write boundaries. Specs stay free of implementation, reviews stay independent, and acceptance testing stays black-box.

## Repo layout

This is the default layout. On brownfield, the operator may set a different code root or specs root, and records them in `REPO-MAP.md`. The layout under each root stays the same.

```
CLAUDE.md              shared rules every session loads
TRACKER.md             where things stand (operator-written)
REPO-MAP.md            where everything lives; binding conventions
CODING-STANDARDS.md    code and commit standards, verification commands
DOMAIN-CONTEXT.md      domain expertise the roles assume
/prd/                  specs, summary, change notes
/ui/                   design artifacts and mockups
/arch/                 architecture files
/plan/                 phase plan
/app/                  the product (code root)
/intents/              intent files and briefs
/research/             research findings (operator-only)
/.agentic-reviews/     review and UAT reports, build reports, verification output
/.claude/              agents, templates, skills, settings
```

## Models and cost

Every role runs on Opus. The architect, planner, security reviewer, and QA engineer run at high effort, set in their role files. The others use the session default. Effort cannot be changed per task. With your confirmation, the operator may move a single mechanical, fully specified task to Sonnet. It never does this for architecture, reviews, or UAT, and it records any such move in the tracker. Expect Opus-level token use. Phase sizing keeps each worker well inside one session.

## Optional Configuration (use with caution)

**Permission prompts.** Worker boundaries do not pre-approve shell commands; Claude Code still asks. To avoid repeated prompts, you could add your test, build, and verification commands, and the prd-version script, to `permissions.allow` in `.claude/settings.json` (shared) or `.claude/settings.local.json` (personal). For example:

```json
{
  "permissions": {
    "allow": [
      "Bash(npx vitest run:*)",
      "Bash(.venv/bin/python3 -m pytest:*)",
      "Bash(python3 .claude/skills/prd-version/prd_version.py:*)"
    ]
  }
}
```

**Plain session, without the operator.** Remove the `agent` line from `.claude/settings.json` for that session, and restore it afterwards. Personal overrides go in `.claude/settings.local.json`, which is not committed and takes precedence over the shared file.

## Working in a team

Everything that matters is in files in git: the tracker, intent files, briefs, and reports. Commit and push per the intent's delivery setting. Another engineer can then pick up the project by starting a session; the operator orients from the repo map and the tracker. Coding standards live in `CODING-STANDARDS.md`. If you have not filled in a stack-specific section, the developer adds one at the first build.

## Updating the framework

Copy the new `CLAUDE.md` over the old one. From the new `claude` folder, copy agents/, templates/, skills/ and PHASECRAFT-VERSION into your `.claude` folder, replacing the old ones. Keep your local settings files.

Sessions pick up the changes on their next start. Existing artifacts and reports are unaffected.

## Why PhaseCraft 2.0

PhaseCraft 0.x told models how to work: step lists, entry banners, orchestration modes, and fourteen slash commands. Current models get the defaults right without that. Extra mechanics also cost context and crowd out the model's judgement. 2.0 replaces mechanics with intent:

- Outcomes are stated. The steps are the model's.
- Formats are fixed only where two sessions must interoperate. For instance: the tracker, reports, the plan, and the runtime handle.
- Firm rules remain only where models tend to cut corners: weakening tests to pass, quiet scope changes, and touching live cloud resources.

The three main benefits over 0.7:

1. **One conversation.** 0.7 had fourteen slash commands and three modes. In 2.0 you talk to the operator, and the review preference in your intent sets where it stops for you.
2. **Stronger verification.** 2.0 adds a security review gate and a maintainability check in the architecture review. The build gate rests on output the operator re-runs itself — tests, lint, type checks, and a dependency audit — not on test results the developer saved.
3. **Existing codebases.** Brownfield onboarding derives the missing artifacts from existing code (best-effort basis). 0.7 supported new projects only.

2.0 is also more than 50% smaller than 0.7. Each session loads only the shared rules and its own role file, about 2,000 to 5,000 words.

Dropped from 0.7, and what replaced each:

- **Slash commands and the three modes** → the operator, with the review preference.
- **The Orchestrator agent and the build and UAT loops** → the operator, which sequences every gate itself.
- **The Project Manager agent** → the operator runs onboarding and keeps the tracker; the planner writes the plan.
- **Entry banners** → the operator's status report and each worker's handoff.
- **Messages between agents** → one-line notes in a worker's handoff, which the operator records and passes on. The operator now sees everything that moves between roles.
- **Reopening a completed phase for UAT defects** → completed phases stay frozen; fixes go forward in a later phase.
- **Compiled PRD snapshots** → a git tag per spec version, `prd-v{version}`.
