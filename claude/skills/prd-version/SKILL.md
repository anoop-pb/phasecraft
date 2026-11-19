---
name: prd-version
description: Deterministic PRD summary (version, feature index, dependency map, NFR lines), a check of the product manager's change note - complete against the section files that actually changed, every row typed, the version step matching the row types, no number used twice - and the change note of an integration version. Run by the operator after any spec change and at a sync; never hand-write or hand-edit the summary or an integration note.
---

# prd-version

Run from the repo root with the project's Python 3 and git. The script finds the PRD root in the `PRD root` row of `REPO-MAP.md`, or uses `/prd/` when the row is absent. Below, `{prd}` is that root. Section files live in `{prd}/sections/`. Each version is identified by the git tag `prd-v{version}`, which the operator sets on the commit that records it.

Check the change note, before committing anything from the specs task:

```
python3 .claude/skills/prd-version/prd_version.py check --from 1.1 --to 1.2
```

Checks `{prd}/changes/CHANGES-v1.2.md` four ways, and exits 1 naming each problem:

- Its first column lists exactly the section files changed since the last commit, including new untracked files. While a merge is in progress and not yet committed, a file counts only where it differs from both this branch’s last commit and the last commit of the branch being merged in. This leaves out files the specs task did not touch.
- Every row's Type is `new`, `changed`, or `clarified`.
- The step from 1.1 to 1.2 is the bump the highest row type requires: `new` → major, `changed` → minor, `clarified` only → patch. A step may skip numbers another branch holds: 1.1 to 1.3 is still a minor bump.
- No feature number and no NFR ID is used twice.

`1.1` and `1.1.0` name the same version; either form finds the change note. For the first version, omit `--from`; it then skips the comparison and the step.

Regenerate the summary:

```
python3 .claude/skills/prd-version/prd_version.py summary --version 1.2
```

Writes `{prd}/SUMMARY.md`: the version, a feature index (number, name, one-line description, depends on, depended on by, NFR flag), the cross-feature interaction map, and one line per NFR item.

Cut an integration version, at a sync where the default branch brought in a higher version than the ones cut on this branch:

```
python3 .claude/skills/prd-version/prd_version.py integrate --version 1.5 --previous 1.4 --integrates 1.3 1.3.1
```

`--previous` is the highest version the merge brought in. `--integrates` lists the versions cut on this branch that are not yet on the default branch. `--version` is the next free version above both, of the bump the integrated versions' highest row type requires. Writes `{prd}/changes/CHANGES-v1.5.md` with an `Integrates:` line and an empty change table, then the summary. It changes no other file: the integrated versions keep their change notes and tags.

The summary is extracted mechanically: a feature's one-line description is the first prose line of its Description section; dependencies are the feature numbers in its Dependencies section; the NFR flag is `yes` unless NFR implications is empty or "None". A summary that extracts wrongly means a section file breaks these rules; send it back to the product manager.
