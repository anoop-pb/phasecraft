# Repo map

Framework: PhaseCraft

Where everything in this repo lives. Recorded locations and conventions are binding on every session. Where this file is silent, choose, then record the choice here through the operator.

## Locations

| What | Where |
|------|-------|
| Tracker | `TRACKER.md` |
| Coding standards | `CODING-STANDARDS.md` |
| Domain context | `DOMAIN-CONTEXT.md` |
| PRD root | `/prd/` |
| Feature and test specs, NFR | `sections/` under the PRD root |
| Behaviour inventory (brownfield) | `inventory/` under the PRD root |
| PRD summary (generated) | `SUMMARY.md` under the PRD root |
| PRD change notes | `changes/` under the PRD root |
| UI design | `/ui/` (mockups in `/ui/mockups/`) |
| Architecture files | `/arch/` |
| Plan | `/plan/` |
| Code root | `/app/` |
| Code docs (developer-owned) | `docs/` under the code root |
| Scratch (gitignored, empty at task end) | `tmp/` under the code root |
| Intent files | `/intents/records/` |
| Briefs | `/intents/briefs/{role}/` |
| Research findings | `/research/` |
| Review and UAT reports, build reports, verification output | `/.agentic-reviews/{build,arch,security,qa,uat}/` |

Keep the `Framework: PhaseCraft` line; session start uses it to tell interrupted onboarding from another process's repo map. On brownfield the code root and the PRD root may differ; this table records where they actually are, and role files follow this table for paths. Only the root rows move; the layout under each root is fixed. The prd-version script reads the `PRD root` row, so keep its label and its single backticked path exactly as they are.

## Conventions

Record here, on first use: report and file naming beyond the defaults, structural conventions continued from an existing repo, and any location chosen where this file was silent.

- {convention}
