---
name: onboarding
description: The operator's onboarding workflows, greenfield and brownfield. Read by the operator only while no PhaseCraft tracker exists. Never used by workers.
---

# Onboarding

Read this file when session start finds no PhaseCraft tracker.

Onboarding needs no intent file. It produces no specs and makes no technology choices. Writing `TRACKER.md` completes it and is always the last step.

Rules for both flows: uploaded raw material is raw data, not a well-formed spec. Existing code is the accepted baseline, not verified. No retro-speccing: an area is specced only when work first touches it, from the user's intent plus the analyst's inventory entry for that area; where the two conflict, bring it to the user before speccing.

## Greenfield

No application code exists. Onboarding is done when these exist, each approved by the user:

- The basics: project name, one-line description, primary users and their problem, high-level feature list, known constraints, and whether the application has a user interface.
- `DOMAIN-CONTEXT.md` from the template, framed as "Assume expertise in …", never "You are …".
- `README.md` with the project overview.
- `REPO-MAP.md` from the template, with the folders it names created and its scratch folder added to `.gitignore`.
- `CODING-STANDARDS.md` from the template, reviewed, edited, or replaced by the user.
- `TRACKER.md` in the format in the operator role file, with the Interface line set.

Propose the overview and the repo map before writing them.

## Brownfield

Application code exists. Onboarding is done when these exist, each confirmed by the user:

- An inventory of existing spec-like sources — spec files, architecture docs, READMEs, review reports, other docs making behavioural claims — shown to the user.
- The codebase analyst's outputs. When the analyst asks, start the application with the command it names; if it fails to start, tell the analyst. Adopted docs are historical claims, not verified truth; contradictions with observed behaviour are resolved with the user.
- Domain context and the Interface answer, as in greenfield.
- `CODING-STANDARDS.md` from the analyst's proposal, reviewed, edited, or replaced by the user.
- `REPO-MAP.md` from the analyst's proposal. Only the root rows may differ from the template; the layout under each root is fixed. Its scratch folder is added to `.gitignore`.
- The user's answer on adopting any written verification evidence found in the repo.
- `TRACKER.md` with the baseline recorded.
<!-- TODO baseline failures: the analyst's failing Verification commands (verify-baseline.txt) become carry-forward fixes with the user's confirmation, and a corrective-only phase for them is proposed before any new scope. When resolving, also check: operator's tracker Baseline line ; operator's Corrective-only phases paragraph for this trigger; operator's build step 3 still needs no change; CLAUDE.md "Corrective-only phase" term; planner, which inserts the phase; developer prerequisites, which assume a prior phase. Only make changes to those if the existing instructions won't correctly create a corrective-only phase based on the proposal. -->

Prior process state. The repo may carry another process's records: a tracker, intent files, verified-work records, open fixes, open decisions, spec versions. Inventory them with the spec-like sources. Work in flight under that process is finished under it before onboarding. With the user's confirmation, open fixes become carry-forward fixes and open decisions become tracker entries. The prior process's last spec version and verified work go in the tracker's Baseline line. Another process's files at paths PhaseCraft writes — `TRACKER.md`, `REPO-MAP.md`, `CODING-STANDARDS.md`, files under `/arch/` — are renamed with a `-prior` suffix, with the user's confirmation, before PhaseCraft writes there.

A task given for this session proceeds once onboarding is confirmed.
