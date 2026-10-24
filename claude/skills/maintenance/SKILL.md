---
name: maintenance
description: The operator's rules for maintenance runs, full architecture reviews, and taking up `autofix` issues. Read by the operator when the user or a schedule asks for any of these. Never used by workers.
---

# Maintenance

## Maintenance runs

When the user or a schedule asks for maintenance, keep the codebase healthy without new scope. Work down this list:

1. A phase other than a corrective-only one has completed since the latest full architecture review, or none has run yet: run one.
2. Open `autofix` issues not yet in progress: take them up.
3. Carry-forward fixes are open and need no decision: insert one corrective-only phase for them and build it.

## Full architecture review

Run one when the user asks, or in maintenance. The architect reviews the whole code root; name in the task any areas the latest full review did not reach, which it covers first. Before it runs, remove accepted findings whose code has changed since they were accepted.

Completed phases are frozen, so Critical and Major findings routed to the developer, and `rising` Minor findings, become carry-forward fixes; the next phase's architecture review confirms them closed. Other Minor findings go to the Minor findings list, with `found in full review run R` in place of the phase. Findings the tracker already holds stay as they are. A finding routed to another role goes to that role, or to the user where the intent does not settle it. Record the run and verdict on the tracker's Latest full arch review line.

## Autofix issues

Issues labelled `autofix` are problem reports against the current specs. Take them up when the user asks, or in maintenance, through a `manual` UAT. The label is the authority to take one up.

Label an issue `in-progress` once taken up, and record that in the tracker. Where the UAT finds the product behaves as the specs say, the report is not a defect: remove `in-progress`, label it `needs-human`, and ask whether to close it or change the specs.
