---
name: unattended
description: The operator's rules for a session with no user present, such as a scheduled routine running maintenance or autopilot. Read by the operator at session start when its task says no user is present. Never used by workers.
---

# Unattended sessions

An unattended session does the work its task names, such as maintenance or autopilot, with no user present. These rules hold alongside that work. Run only on a PhaseCraft tracker; otherwise stop and say why.

## Where to work

- All unattended work goes on one branch: the newest `claude/phasecraft/*` branch whose PR is open, or that has no PR yet. A merged PR ends that branch. If there is none, cut `claude/phasecraft/{date}` from the default branch. If the newest such branch's PR was closed unmerged, act on a comment on why it was closed, then cut a new branch; with no such comment, ask on that PR and stop.
- A phase in progress on the branch is finished before any other work.
- Sync with the default branch when Parallel work in the operator role file calls for it. A conflict that needs a decision: ask.
- While this branch’s PR is ready for review and unmerged, do nothing, not even a sync; stop.
- While a question on this branch is open, start no new phase. Reviews and autofix intake go on.
- A push rejected because the remote moved means another run is working: stop.
- While a branch other than this one, not yet merged into the default branch, has changed the tracker or the plan, build nothing; ask, naming it. Reviews and `autofix` intake go on.

## When a human is needed

No one is present, so to ask is to leave a question for a human to answer later. Where an attended session would stop for the user, ask: comment on the issue or PR the question is about, and label it `needs-human`. Where there is none, open an issue labelled `needs-human`. Keep it short: the question, the options, your recommendation. Record the open decision in the tracker with the issue or PR number. Stop that line of work, and go on with work that does not depend on it. Never infer past it.

At the start, read every issue and PR the tracker names for an open decision. The user's input is a later comment with no `[phasecraft:…]` prefix, or a `[phasecraft:response]` or `[phasecraft:decision]` comment. Act on it, and settle a question as the operator role file sets out.

- Discussion stages never run unattended. Work that needs one: ask.
- If the application cannot start because the session lacks something it needs, such as a service or a secret, that is not a defect: ask.
- A failure no role can fix within its boundaries, such as an action the environment refuses: do not look for a way around it; ask.

Where an active intent exists, its boundaries and settings hold. Three exceptions to the decisions that always go to the user:

- In maintenance, you may insert a corrective-only phase without asking. Move no `low` Minor findings into it; that still needs the user.
- A deviation from a `manual` UAT of an `autofix` issue counts as seen by the user, who reported it by labelling it.
- A Major UAT deviation routed to the developer, where the spec is clear, may go to a fix-build without the user seeing it first; list it in the PR. Critical deviations, and any you are in doubt about, still wait on the user.

## Delivery

Push the branch after every unit of work, whatever the active intent’s delivery setting says. Open its PR as a draft at the first push, with its Claims block; the branch claims its numbers and holds the PRD lock like any other. Before ending a session that added work, run the full UAT the operator role file requires, where one applies; mark the PR ready once it is CLEAR and no question on the branch is open.

## Ending

Run session end without confirmation. If a PR or other event wakes the session after it ends, do nothing.
