---
name: prd-version
description: Deterministic PRD summary (version, feature index, dependency map, NFR lines) and a check of the product manager's change note - complete against the section files that actually changed, every row typed, and the version step matching the row types. Run by the operator after any spec change; never hand-write or hand-edit the summary.
---

# prd-version

Run from the repo root with the project's Python 3 and git. The script finds the PRD root in the `PRD root` row of `REPO-MAP.md`, or uses `/prd/` when the row is absent. Below, `{prd}` is that root. Section files live in `{prd}/sections/`. Each version is identified by the git tag `prd-v{version}`, which the operator sets on the commit that records it.

Check the change note, before committing a new version:

```
python3 .claude/skills/prd-version/prd_version.py check --from 1.1 --to 1.2
```

Checks `{prd}/changes/CHANGES-v1.2.md` three ways, and exits 1 naming each problem:

- Its first column lists exactly the section files changed since tag `prd-v1.1`, including new untracked files.
- Every row's Type is `new`, `changed`, or `clarified`.
- The step from 1.1 to 1.2 is the bump the highest row type requires: `new` → major, `changed` → minor, `clarified` only → patch.

`1.1` and `1.1.0` name the same version; either form finds the tag and the change note. For the first version, omit `--from`; it then checks only that the change note exists and its rows are typed.

Regenerate the summary:

```
python3 .claude/skills/prd-version/prd_version.py summary --version 1.2
```

Writes `{prd}/SUMMARY.md`: the version, a feature index (number, name, one-line description, depends on, depended on by, NFR flag), the cross-feature interaction map, and one line per NFR item.

The summary is extracted mechanically: a feature's one-line description is the first prose line of its Description section; dependencies are the feature numbers in its Dependencies section; the NFR flag is `yes` unless NFR implications is empty or "None". Write section files so this extraction lands correctly.
