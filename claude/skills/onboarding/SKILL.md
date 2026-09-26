---
name: onboarding
description: The operator's onboarding workflows, greenfield and brownfield. Read by the operator only while TRACKER.md does not exist. Never used by workers.
---

# Onboarding

Read this file when session start finds no `TRACKER.md`.

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
- The codebase analyst's outputs. Start the application first where it can be started, so the analyst can drive it. Adopted docs are historical claims, not verified truth; contradictions with observed behaviour are resolved with the user.
- Domain context and the Interface answer, as in greenfield.
- `CODING-STANDARDS.md` from the analyst's proposal, reviewed, edited, or replaced by the user.
- `REPO-MAP.md` from the analyst's proposal. Only the root rows may differ from the template; the layout under each root is fixed. Its scratch folder is added to `.gitignore`.
- The user's answer on adopting any written verification evidence found in the repo.
- `TRACKER.md` with the baseline recorded.

Prior process state. The repo may carry another process's records: a tracker, intent files, verified-work records, open fixes, open decisions, spec versions. Inventory them with the spec-like sources. Work in flight under that process is finished under it before onboarding. With the user's confirmation, open fixes become carry-forward fixes and open decisions become tracker entries. The prior process's last spec version and verified work go in the tracker's Baseline line.

A task given for this session proceeds once onboarding is confirmed.
