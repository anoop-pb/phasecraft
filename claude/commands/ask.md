---
description: "PhaseCraft: Start a read-only consultation session with a named agent (Example: product, ui, arch, project-manager, coding, qa)"
allowed-tools: Read, mcp__filesystem__read_text_file, mcp__filesystem__read_multiple_files,mcp__filesystem__list_directory
---
You are starting a read-only consultation session for agent: $ARGUMENTS

### Agent Resolution

Resolve the agent name from `$ARGUMENTS`:

1. List files in `.claude/agents/` matching `*-AGENT.md`
2. Extract the agent name prefix from each file (e.g., `PRODUCT-AGENT.md` → `product`, `UI-AGENT.md` → `ui`, `ARCHITECT-AGENT.md` → `architect`, `CODING-AGENT.md` → `coding`, `QA-AGENT.md` → `qa`)
3. Match `$ARGUMENTS` (case-insensitive) against extracted names. Also accept these aliases:
   - `arch` → `architect`
   - `code` or `build` → `coding`
   - `spec` → `product`
   - `design` → `ui`
   - `pm` → `project-manager`
4. If no match is found, list the valid agent names and stop: "Available agents: [list]. Run `/ask [name]` with one of these."
5. If multiple agents match (e.g., a custom agent name collides with an alias), list the matches and ask the user to clarify: "Multiple agents matched '[input]': [list of agents matched]. Which one did you mean?" Do not assume and proceed.
6. If exactly one match, read the corresponding agent MD file.

### On session start

1. Read the resolved agent MD
2. Read `.claude/agents/AGENT-STANDARDS.md`
3. Read `TRACKER.md` (informational — prerequisites do not block /ask sessions)
4. Print the entry banner with Agent Name: [resolved agent name], Session type: Read-only (/ask) and prerequisites as informational only
5. Answer questions per the matched agent role. Do not write any files. Do not advance lifecycle state.

If the conversation reveals a change that should be made, tell the user which slash command to run for a full session.

Agent: $ARGUMENTS
