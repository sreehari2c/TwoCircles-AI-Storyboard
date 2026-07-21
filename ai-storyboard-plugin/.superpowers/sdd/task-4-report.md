# Task 4 Report: Internal Stage Skills

## Status

Completed the eight internal, file-based storyboard stage skills required by Task 4.

## Files Changed

- `skills/storyboard-intake/SKILL.md`
- `skills/storyboard-planning/SKILL.md`
- `skills/storyboard-review/SKILL.md`
- `skills/storyboard-generation/SKILL.md`
- `skills/storyboard-refinement/SKILL.md`
- `skills/storyboard-versioning/SKILL.md`
- `skills/storyboard-references/SKILL.md`
- `skills/storyboard-commit-export/SKILL.md`

Each file has lowercase YAML metadata and explicit Inputs, Files, Outputs, Stop and Ask, and Invariants sections. Together they define intake, planning, review, contact-sheet generation, scoped refinement, immutable versioning, rights-aware reference handling, and validated final export.

## Commit

- `5f35715519e54339efe83c81665a6de49cec7f2f` — `feat: add storyboard stage skills`

The commit intentionally contains only the eight requested skill files, matching the task's specified `git add` scope. This report is therefore uncommitted task documentation.

## Validation Output

Prescribed validation:

```text
PASS: prescribed validation found 8 internal skills with name/input/output contracts.
```

Additional static contract audit:

```text
PASS: shared-contract and stage-specific audit passed for all eight files.
PASS: no forbidden placeholders in internal skills.
```

`git diff --check` completed with no whitespace errors before commit. The committed tree contains eight new skill files and no remaining staged or unstaged task-file changes. Git emitted informational LF-to-CRLF conversion warnings after the commit because of the repository's Windows line-ending configuration; no content or whitespace validation failed.

## Self-Review

- Confirmed every file begins with a unique lowercase `name` and a one-sentence description.
- Confirmed all skills declare what they read, what they may change, what they hand to the next stage, stop conditions, and invariants.
- Confirmed the shared rules are represented where relevant: `project.json` authority, stable `frame_id`, changeable `sequence_number`, context precedence, provisional defaults, six/eight-frame defaults, capture defaults, numbered contact-sheet artifact, no-code/file-based scope, and approved-iteration preservation.
- Confirmed generation records its exact packet and result before updating the active version; refinement scopes changes and preserves unaffected material; version restoration branches rather than overwrites; references are frame-scoped and rights-aware; export validates outputs before replacing final files.
- Confirmed internal instructions keep the top-level storyboard flow as the primary user interface rather than directing users to invoke stage skills individually.

## Concerns

- No functional concerns found.
- The repository's line-ending configuration reports LF-to-CRLF conversion warnings when Git touches the new Markdown files. The files pass `git diff --check`; no line-ending change was made because this is an existing repository policy rather than a Task 4 defect.

## Reviewer Fix

Fix command:

```powershell
git add skills/storyboard-commit-export/SKILL.md
git commit -m "fix: define storyboard export enum validation"
```

Result: focused contract/grep validation passed for all allow-lists and the rejection rule; `git diff --check` passed; commit `180e6aa1bb19d27358a97f88c7b36efff275c593` created with only the skill file staged.
