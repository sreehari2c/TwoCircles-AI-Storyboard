# Final Whole-Branch Fix Report

## Status

PASS - all final-review blockers and documentation issues were implemented in the specified worktree and committed.

## Commit

- Foundation hardening commit: `965f1a1` (`fix: harden storyboard workflow contracts`)
- State-route follow-up commit: `037b7ac` (`fix: align storyboard state routes`)
- Branch: `feat/ai-storyboard-plugin`
- Worktree: `C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin`

## Scope preserved

The package remains instruction-and-file based. No hosted API, custom runtime, database, script, compositor, or image-processing implementation was added.

## Implemented fixes

### 1. One registered public skill

- Scoped `.codex-plugin/plugin.json` to `./skills/storyboard/`.
- Kept only `skills/storyboard/SKILL.md` as a `SKILL.md`.
- Removed all eight stage `SKILL.md` files from `skills/`.
- Added eight ordinary stage references under `workflow/`.
- Added `workflow/storyboard.md` as the explicit ordinary orchestration/state reference named by the brief.
- Updated the public skill to reference every workflow file and all eight stage route names.
- Updated README wording to state that the public `storyboard` workflow is the only registered skill and workflow modules are not independently triggerable.

### 2. Canonical project, frame, version, and export contracts

- Added lifecycle values: `proposed`, `planned`, `review`, `generated`, `refined`, and `committed`.
- Kept `active_version` as an integer.
- Defined the frame contract with stable identity, ordering, content, prompt, metadata, lock, reference, and image-path fields.
- Defined the version contract with all required artifact paths, status, timestamp, and change summary.
- Defined the export contract with artifact type/path, version, validation result, and timestamp.
- Required generation to write:
  - `iterations/v001/prompt-packet.md`
  - `iterations/v001/frames.json`
  - `iterations/v001/result.json`
  - `iterations/v001/changes.md`
- Required refinement, restoration, and branching to consume and create the same four-file contract.
- Required project frame, version, and active-version updates only after iteration validation.

### 3. Lifecycle transitions and state routing

- Defined intake, planning, review, generation, refinement, commit, review-edit, reopen, restore, and branch transitions.
- Added status-only routing for requests without a stage verb.
- Added a hard manifest gate for missing/invalid `status` or non-integer `active_version`.
- Defined committed-project reopen choices and required new branches to begin with a new active iteration.

### 4. Frame-only refinement fallback

- Required frame-only regeneration when the image capability supports it.
- Limited contact-sheet recomposition to capabilities that provide that operation.
- Added three explicit user choices when frame editing and compositing are unavailable:
  1. whole-contact-sheet regeneration with locked unchanged prompts and drift warning;
  2. frame-only revision artifact with prior contact sheet retained;
  3. prompt/metadata-only update with no new image.
- Required unaffected frame records/prompts to remain unchanged and the chosen path to be recorded in both `changes.md` and `result.json`.
- Added the same fallback verification to the acceptance runbook.

### 5. Transactional commit/export

- Required selected-iteration and source-file validation before output generation.
- Required high-resolution output/reference and candidate final artifacts under `iterations/v###/commit-candidate/`.
- Required complete candidate JSON, CSV, Markdown frame ID, enum, path, and image-reference validation.
- Required candidate promotion before export records, `active_version`, and `committed` status are updated.
- Required rollback/preservation of prior finals and manifest state on source, generation, validation, promotion, or manifest-update failure.
- Required failed candidates to remain isolated for inspection.

### 6. Documentation corrections

- Intake now creates project-local `guides/` and `exports/` together with `references/`, `plans/`, `iterations/`, and `final/`.
- Acceptance action 1 is intake/import only.
- Acceptance action 2 is the first action that creates and expects `plans/plan-v001.md`.

### 7. State-route consistency follow-up

- Fixed review re-entry: status-only routing may send `review` back to the review workflow, which now accepts both `planned` and `review`, performs `planned -> review` only on first entry, and preserves `review` during re-entry.
- Fixed committed export: committed projects already offered export/re-export, and the commit/export workflow now accepts `committed` for explicit re-export while retaining the same transactional candidate ordering.
- Defined `committed -> committed` after successful re-export and preserved committed state on failure.
- Added runbook evidence for transactional export from a reopened committed project.

## Validation results

All checks passed with exit code 0:

- Parsed every JSON file with PowerShell `ConvertFrom-Json`.
- Confirmed all existing Task 7 required package paths.
- Confirmed no forbidden planning placeholders.
- Confirmed exactly one `SKILL.md`, at `skills/storyboard/SKILL.md`.
- Confirmed manifest skill registration is `./skills/storyboard/`.
- Confirmed all eight stage workflow references exist.
- Confirmed the public skill references every workflow Markdown file and all stage route names.
- Confirmed lifecycle enums, integer `active_version`, and frame/version/export record keys.
- Confirmed the four exact iteration artifacts appear in generation, refinement, versioning, and runbook documents.
- Confirmed state transitions, status fallback routing, and invalid-manifest stop behavior.
- Confirmed the route/input matrix, including review re-entry and committed re-export, with focused checks that failed before the follow-up and passed afterward.
- Confirmed all three contact-sheet fallback choices in refinement and runbook documents.
- Confirmed source-validation -> candidate-build -> candidate-validation -> promotion -> manifest-state ordering.
- Confirmed intake directory creation and action 1/action 2 runbook separation.
- Confirmed the 45-column CSV contract and frame identity column positions.
- Completed the Task 7 manual smoke read in the required order across README, public workflow, ordinary workflow modules, four styles, templates, and acceptance runbook.
- Ran `git diff --check 28d60e9..HEAD`.
- Ran working-tree `git diff --check`.
- Confirmed clean Git status with no `.superpowers` scratch files staged or tracked.
- Confirmed the latest commit subject and full commit ID `037b7ac1288bf77f4825309da934ae1c86479186`.

Git emitted LF-to-CRLF advisory messages during staging/pre-commit diff checks; these were warnings only, and both whitespace validation commands exited 0.

## Concerns

None.
