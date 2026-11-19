---
name: sync
description: The operator's rules for what a merge of the default branch into its own branch brings - conflicts, PRD versions to join, change notes from another branch, and merged code. Read by the operator before every such merge. Never used by workers.
---

# Sync

Parallel work in the operator role file says when to merge the default branch into yours, and sets out the PRD lock and the Claims block. These rules say how to handle what the merge brings.

- Resolve conflicts outside code by file owner: you for the files you write, the prd-version script for the PRD summary, the owning role for the rest. In the tracker, keep this branch's Active intent, Review preference, and Phases per session lines. Where both sides added a research file of the same name, rename this branch's. A conflict inside a spec section file is a spec change: run it as a specs task, under the PRD lock. Its version joins the two, so cut no integration version for that merge. Commit and tag it once the merge is complete.
- Where your branch has cut PRD versions and the default branch now carries a higher one, run the prd-version skill's integrate command, holding the PRD lock while you do. It cuts a new version that joins them; pick its number as the skill sets out. It needs no prd-version check. Commit it, tag it `prd-v{version}`, and add it to your claims. Earlier change notes, tags, and reports stay as they are.
- Change notes the merge brought in: the branch that wrote them has already designed and planned for them. Re-run a role only to check those notes against work of your own branch that the other branch never saw: the product manager where this branch has cut PRD versions, for cross-feature consistency, where a conflict it finds is a specs task; the UI designer or the architect where this branch has changed that role's files; the planner where this branch has changed the plan or has phases not started. Run them in that order, before the build goes on, naming the change notes in each task as landed from another branch. This is not a discussion stage.
- Code, before a phase's build: the merge is part of that build. Where it has no conflict in code, run the verification commands first. In the build task, name the conflicts in code for the developer to resolve, and any verification failure the merge caused. The phase's gates then cover the merged code.
- Code, with no phase left to build: a conflict in code, or a verification failure once the merge is in, is a corrective-only phase of its own, and needs no confirmation. Have the planner add it. The developer makes the change as its build, and it passes every gate like any other phase.
