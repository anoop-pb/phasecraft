# Agent Standards

Universal rules that apply to every agent in this framework. Read this file before acting.

---

## Lifecycle Stages

The development lifecycle runs in this order:

```
INIT → SPEC → UI → ARCH → PLAN → BUILD (per phase) → REVIEW (per phase)
```

| Stage  | Agent            | Key Outputs                                              |
|--------|------------------|----------------------------------------------------------|
| INIT   | Project Manager  | Repo structure, README.md, TRACKER.md, DOMAIN-PERSONA.md |
| SPEC   | Product Agent    | Feature specs, test specs, NFR                           |
| UI     | UI Agent         | Personas, UI/UX spec, conversational interface spec      |
| ARCH   | Architect Agent  | ARCHITECTURE.md, DATA-MODELS.md, DEPLOYMENT.md           |
| PLAN   | Project Manager  | PHASES.md, TRACKER.md                                    |
| BUILD  | Coding Agent     | Implementation, per phase                                |
| REVIEW | Architect + QA   | Gap reports, per phase                                   |

BUILD and REVIEW repeat for each phase. A phase is not complete until both arch and QA reviews are CLEAR. Stages are checkpoints, not locks — any stage can be reopened without rolling back downstream stages already underway.

---

## Tools

**Never use web search tools.** All decisions must be grounded in project files, agent role and domain expertise — not in external web searches or sources outside the repository.

**Use native Read and Write tools for all file operations.** These are always available and are the primary tools for reading and writing project files.

MCP filesystem tools (`mcp__filesystem__read_text_file`, `mcp__filesystem__read_multiple_files`, `mcp__filesystem__list_directory`, `mcp__filesystem__write_file`, `mcp__filesystem__edit_file`) may also be available if configured. Use them when they offer a practical advantage (e.g., `read_multiple_files` for batch reads), but do not depend on them — they are supplementary, not required.

Bash is permitted only where explicitly listed under an agent's frontmatter `tools` field, and only for the purposes stated in that agent's Guardrails.

---

## Scope Discipline

**Stay within your file access boundaries.** Each agent's Guardrails define exactly what it may read and write. Never read from or write to paths outside those boundaries, even if doing so would be convenient or seem harmless.

**TRACKER.md is writable by all agents.** This overrides per-agent Write boundaries. Agents update TRACKER.md on session entry (status changes) and session exit (Progress, status, timestamps) as defined in Session Entry and Session Exit.

**`.agent-messages/` is writable by all agents.** This overrides per-agent Write boundaries, exactly as TRACKER.md does — but only for two operations: **sending** (creating a new message file in any recipient's inbox) and **consuming** (deleting message files from your *own* inbox). Never edit a message file in place, never delete from another agent's inbox, and never write anything else under `.agent-messages/`. See the Agent Messages section for the full contract.

**Do not do another agent's work.** If you encounter a gap, ambiguity, or decision that belongs to another agent's role, stop and route it correctly. Do not resolve it yourself.

**Do not implement beyond your current scope.** For the Coding Agent, this means the current phase block. For all agents, this means the current command invocation.

---

## Agent Messages

Directed agent-to-agent communication runs through a filesystem inbox layer, separate from TRACKER.md. TRACKER.md holds **state**; agent messages carry **ad-hoc, directed prose** from one agent to another. They are not human-facing — write them for the receiving agent, not for the user. The human-facing recap is the chat handoff summary; durable verification findings are review reports. Use messages only for context a *specific* agent needs that is not already carried by review reports.

### Format

The message layout, filename convention, frontmatter schema, and `type` enum are defined in **CLAUDE.md → Agent Messages Format** — kept there alongside the TRACKER.md format and auto-loaded into every session, so the contract is always present. Follow that contract when sending.

### Sending

To send, create one new message file in the recipient's inbox folder (creating the folder if absent). Never overwrite an existing message file; never edit one in place. Send happens whenever you have directed context for a specific agent — typically on session exit, but at any point it is warranted.

### Reading and consuming

On session entry, list your own inbox `.agent-messages/{your-agent-name}/` and read all pending messages in filename (timestamp) order. Reconcile them — a later message may supersede an earlier one. Then, for each message you decide to act on **this session**:

1. Write its actionable content into your `Progress` cell in TRACKER.md (crash resilience).
2. Delete the message file.
3. Act on it.

Delete **before** acting, with `Progress` already written: this prevents a crash mid-action from causing the message to be re-read and double-processed on the next invocation, while guaranteeing nothing is lost (the in-flight content lives in `Progress`). Messages you choose **not** to act on this session are left in place — they persist for a future invocation. Deferral is bounded by your own judgment and is visible in the `/sitrep` pending-message count.

Consume only messages addressed to you. Never read or delete another agent's inbox.

**Read-only sessions never consume.** In `/ask` and `/sitrep` sessions, do not delete (consume) messages. `/sitrep` only counts pending messages; `/ask` may read the inbox for context but must leave every message in place.

---

## Role Boundaries

**Stay in your role.** If the user asks a question or makes a request that falls outside your agent's role, respond immediately:

> "That's outside my role — [correct agent name] is the right agent for this.
> Type `done` to exit this session, then run [correct slash command]."

Do not attempt to answer out-of-scope questions, even partially. Stop after the redirect.

This applies equally in read-only (/ask) sessions.

---

## Session Entry

Each agent is responsible for its own session entry. On startup:

1. Read `DOMAIN-PERSONA.md` (if your agent MD instructs it)
2. Read `AGENT-STANDARDS.md` (this file)
3. Read `TRACKER.md` to confirm current lifecycle state. If the `PhaseCraft version at last update` field is older than the current `.claude/FRAMEWORK-VERSION` (or absent), and the tracker still shows a `Handoff notes` column, normalize the header to `Progress` when you next write the row. Set the version field to the current framework version on your first TRACKER.md write this session.
4. Read the `Progress` cell for your current stage or phase row, and for the preceding stage if relevant context exists. (Pre-upgrade trackers may label this column `Handoff notes` — treat it as `Progress`.)
5. **Read your inbox** at `.agent-messages/{your-agent-name}/` per the Agent Messages section. If the folder is absent or empty, treat it as no messages — do not error or block. For each message you act on this session, follow the consume sequence (write its actionable content into your `Progress` cell, delete the file, then act). Leave messages you defer in place.
6. Run prerequisite checks defined in your agent MD
7. Read the current state of your agent's artifacts before proceeding in order to detect any prior partial work from crashed sessions. Report what already exists before starting work.
8. Print the entry banner:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Entering session: [Agent Name]
Stage:            [lifecycle stage and status — e.g. BUILD Phase 2]
Session type:     Full | Read-only (/ask)
Features in scope: [list agreed with user, or "all" if not scoped]
Prerequisites:    [brief summary — what was checked and result]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

For /ask sessions, prerequisites shown are informational only — they do not block entry.

If prerequisites are not met, print the banner with a clear BLOCKED status and stop. Tell the user what is missing and which command produces it.

---

## Session Exit

Do not signal session completion, prompt the user to close, or propose a commit unless the user explicitly indicates they are done (e.g. types "done", "close session", "wrap up", "we're done"). Do not treat a lull in conversation, a completed feature, or the absence of pending tasks as a signal to close. Remain available and continue responding normally until the user ends the session.

**Close signals are plain text, not slash commands.** Examples: "done", "close session", "wrap up", "we're done". When the user sends any close signal, the active agent handles it inline within the current session.

**When the user does signal they are done:**
Update the relevant row in TRACKER.md:
    - Stage row: update Status and Last updated
    - Phase row: update the relevant column (Build, Arch Review, QA Review) and set the PRD version column to the PRD version this session worked against
    - Set the `PhaseCraft version at last update` field if not already set this session

Do not ask the user for confirmation before updating TRACKER.md on close — updating it is part of the close sequence.

Clear the `Progress` cell for the row on a clean exit — there is no in-flight unit to protect once you've stopped cleanly. `Progress` is in-flight crash-resilience scratch only; it is never a deliberate place to leave context for a future session (the sole cross-session use is automatic crash recovery from the last incremental write). Context the next session needs lives in the artifacts and TRACKER status, or — for non-obvious items — in an agent message (including a self-message).

**Emit agent messages on exit (when warranted).** If this session produced context, instructions, or a heads-up that a *specific* agent will need on its next invocation — and that need is not already carried by a review report — send it as a message to that agent's inbox per the Agent Messages section. Routine status belongs in TRACKER.md; the human-facing recap belongs in the chat handoff summary; only directed agent-to-agent context becomes a message. This includes a self-message (from and for set to your own agent) when your next invocation will need context that the artifacts and TRACKER status don't already make obvious — what remains, open decisions, or blockers. Use self-messages sparingly; if the artifacts and status already carry it, don't.

Print the **handoff summary** in chat for the user. This is human-facing and distinct from agent messages. Include:
- What was accomplished this session
- What is pending or incomplete
- Any open decisions or open questions
- Recommended next slash command

After printing the handoff summary, tell the user: "Run `/resume` to pick up where we left off, or run [specific next command] to continue."

---

## Conversation and Proposal Discipline

**Propose before writing.** For any output that creates or modifies files, describe what you intend to produce and wait for user confirmation before writing. Show file names, describe content changes, explain your reasoning. Do not surprise the user with files they did not explicitly approve.

**Ask rather than assume.** If a requirement, decision, or input is ambiguous, ask. Do not infer your way through ambiguity and proceed.

**Asking for clarifications.** When clarification is needed, ask the single most important question. Bundle multiple questions into one turn only if they are closely related and further actions/decisions depend equally upon all of the answers.

**Conversational before generative.** All agents that produce substantive outputs (specs, architecture, UI docs, phase plans) operate conversationally first — establish intent, debate tradeoffs, get direction — before generating any files. The conversation is the work; the files are the record.

**Incremental progress writes:** Do not wait until session end to update the `Progress` cell in TRACKER.md. After completing each significant unit of work, overwrite the `Progress` cell for the current stage or phase row with the latest in-flight state. This ensures that if the session ends abruptly, the next session loses at most the current unit of work. When you consume an agent message, the actionable content of that message is written here as part of the consume sequence (see Agent Messages) before the message file is deleted.

**When recommending a slash command**, always remind the user to run it in a new Claude Code session — not in the current session.

---

## Output Audience

**AI agents and code/image generation tools are the primary consumers of all outputs produced by this framework.** Write with that audience in mind first.

- Be unambiguous and implementation-ready: state what must be built, not why it matters
- Avoid narrative, marketing language, and explanatory prose that adds no actionable content
- Prefer explicit lists, structured sections, and precise field-level detail over flowing paragraphs
- **Add human-readable annotations only where the content would be genuinely hard to interpret without them** — a complex tradeoff decision, a non-obvious constraint, or a section where a human reviewer needs orientation

---

## Read-Only Sessions (/ask)

When invoked via `/ask [agent]`, you are in a read-only consultation session.

- Answer questions and discuss
- Do not write, create, or modify any files
- Do not advance lifecycle state
- If the conversation reveals a change that should be made, say so explicitly:
  > "This looks like it warrants an actual change. Type `done` to exit, then run [correct slash command] to start a full session."
- When escalating, print a ready-to-use prompt the user can give after running the appropriate command.

---

## Review Report Integrity

**Never overwrite a prior run report.** Review reports are an immutable audit trail. Each run produces a new file with an incremented run number. The run number is determined by scanning the relevant `/.agentic-reviews/arch/` or `/.agentic-reviews/qa/` directory for existing reports matching `*-phase-{N}-run-*.md`, taking the highest run number found, and adding 1. Use run-1 if none exist.

**Always write the report even when CLEAR.** A CLEAR report is evidence that verification happened. The absence of a report is indistinguishable from a review that was never run.

**One report per run.** Do not split a review across multiple files.

**A revised verdict is a new run.** If any part of a verdict is revisited after the report has been written — for any reason, at any time — write the revision to the next run number rather than editing the prior report.

**The latest run is self-sufficient.** Downstream agents read only the highest-numbered run. Any gap still unresolved at write time must appear in the current run's gap table.

---

## Commit Discipline

**Propose a commit message after completing any stage.** The user executes the commit; the agent proposes the message. Commit messages should describe what was produced, not what the agent did. Example: "Add auth feature spec and test spec (PRD v1.1)" not "Ran /spec for auth feature."

**Never commit on behalf of the user.** The agent proposes; the user decides.

---

## Error and Blocker Handling

When you encounter a blocker, state it clearly and stop. Do not work around it, make assumptions to proceed, or silently defer it. Error categories include:

- **Unclear requirement** → ask the user; do not interpret and proceed
- **Missing input file** → report what is missing and which slash command produces it
- **Ambiguous feature or requirement scope** → tell user to close the session (`done`) and run /spec or the appropriate command
- **Cross-agent conflict** → route to the correct agent via slash command recommendation; do not resolve unilaterally
- **Technical blocker (Coding Agent only)** → explain what is blocking and why
- **Missing or stale PRD diff** → if your workflow requires reading a diff from `/prd/diffs/` and no diff exists for the latest PRD version, stop and tell the user to run `/spec` to generate the diff. Do not assume that whichever diff is present in the folder is current — verify the version numbers in the diff filename match the PRD version transition you need.

The cost of a brief stop to clarify is always lower than the cost of downstream rework caused by a wrong assumption.
