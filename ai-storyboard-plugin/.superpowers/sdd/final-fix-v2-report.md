# Final whole-branch fix v2 report

## Status

PASS. All second final-review findings in `final-fix-v2-brief.md` were implemented together in the no-code, file-based plugin scope.

## Commit

- Commit: `cc7a83b66b6c2effbf08a60432a8fb80687d1c89`
- Message: `fix: make storyboard plugin portable and stateful`
- Branch: `feat/ai-storyboard-plugin`

## Implemented fixes

### 1. Portable, skill-relative package layout

- Kept `skills/storyboard/SKILL.md` as the only public registered skill.
- Standardized `.codex-plugin/plugin.json` on `./skills/`, whose only direct skill is `storyboard`, and added the required plugin interface metadata.
- Moved the eight private stage contracts to `skills/storyboard/workflow/`.
- Moved starter guides to `skills/storyboard/resources/guides/`.
- Moved templates to `skills/storyboard/resources/templates/`.
- Removed the private orchestration duplicate `workflow/storyboard.md`; orchestration remains in the public skill.
- Removed the old root `workflow/`, `guides/`, and `templates/` sources of truth.
- Preserved `storyboard-projects/<project-slug>/` at plugin root.
- Removed eight verified-empty legacy `skills/storyboard-*` directories that were visible to plugin discovery despite containing no tracked files.

### 2. Approved design and plan amendment

- Replaced the obsolete eight-internal-skill architecture with one registered skill plus eight non-discoverable workflow files.
- Updated the file map, task boundaries, validation counts, package paths, and project-store semantics.
- Updated README and acceptance-runbook paths to match the portable layout.

### 3. Durable review authorization and routing

- Added the manifest review object with `decision`, `plan_version`, `authorized_for_generation`, `reviewed_at`, and `change_summary`.
- Defined decisions `pending`, `approved`, and `bypassed`.
- Made planning and plan edits reset review to pending/unauthorized.
- Made approval and explicit bypass record the selected plan version, review timestamp, authorization, change summary, and status `review`.
- Required generation to re-read durable authorization and match the selected plan version; invalid authorization routes to review.
- Made no-verb `review` fallback route to generation only when authorization is valid.
- Routed add/remove/reorder/edit-plan requests from status `review` through the review workflow, which writes the edit, resets authorization, and returns status to `planned`.
- Added all five review fields to the canonical version record and acceptance evidence.

### 4. Canonical frame and 45-column export contract

- Added `revision_history` to the frame contract.
- Added exact narrative, camera, production, scheduling, reference, status, and timestamp metadata keys.
- Added `storyboard_id` to the manifest contract.
- Documented an ordered canonical source mapping for all 45 CSV columns.
- Defined deterministic list and empty-value serialization without column shifting.

### 5. State-consistent final manifest promotion

- Made source validation precede candidate construction.
- Made the candidate include plan, CSV, image/image-reference metadata, and a prospective committed manifest under `iterations/v###/commit-candidate/`.
- Made the prospective manifest contain status `committed`, selected integer `active_version`, and appended export records before candidate validation.
- Required candidate manifest, plan, 45-column CSV, and image to validate together.
- Made promotion plus writing the exact buffered candidate manifest bytes to both `project.json` and `final/manifest.json` the final commit step.
- Required byte-for-byte manifest equivalence and rollback preservation until validation completes.
- Specified that failures leave `project.json`, prior `final/`, `active_version`, `status`, and `exports` unchanged.

## Validation evidence

All validation was rerun after commit.

| Check | Result |
| --- | --- |
| Parse every JSON file with `ConvertFrom-Json` | PASS: 2 JSON files parsed |
| Only `skills/storyboard/SKILL.md` under `skills/` | PASS: exactly 1 |
| Private workflow count | PASS: exactly 8 Markdown files |
| Skill resources | PASS: 7 guide Markdown files and 4 template files |
| Root duplicate directories | PASS: no root `workflow/`, `guides/`, or `templates/` |
| Public routing references | PASS: all 8 workflow files and skill-relative resource roots |
| Review contract | PASS: 3 decision values and all 5 fields in project/version records |
| Lifecycle contract | PASS: 6 states |
| Iteration artifact contract | PASS: `prompt-packet.md`, `frames.json`, `result.json`, `changes.md` |
| Refinement fallback contract | PASS: whole contact sheet, frame-only artifact, metadata-only |
| Commit candidate ordering | PASS: prospective manifest before validation before promotion |
| CSV contract | PASS: 45-column header and 45 ordered mapping rows |
| Frame metadata gaps | PASS: all requested keys plus `revision_history` |
| Placeholder/package scan | PASS: no forbidden implementation placeholders or stale runtime paths |
| Skill validator | PASS: `Skill is valid!` |
| Plugin validator | PASS: plugin validation passed |
| `git diff --check` | PASS |
| `git diff --check 28d60e9..HEAD` | PASS |
| Commit message/hash verification | PASS |
| Git status before report write | PASS: clean |

## Staging and scratch-file check

The commit used the exact scoped add command from the brief. `.superpowers` files were not staged or committed. This report was written only after the implementation commit.

## Concerns

No open implementation or validation concerns. The manual ChatGPT image-generation acceptance run remains an external capability exercise documented in `docs/storyboard-acceptance-runbook.md`; it was not part of this file-only fix pass.
