# Task 7 report: package-wide validation and smoke review

## Scope and worktree

Validated `C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin` on branch `feat/ai-storyboard-plugin`. Git metadata confirmed this is a linked worktree, not a submodule.

## Commands run and output summaries

| Command / activity | Output summary |
| --- | --- |
| `git rev-parse --git-dir`, `git rev-parse --git-common-dir`, `git branch --show-current`, `git rev-parse --show-superproject-working-tree` | Confirmed linked worktree on `feat/ai-storyboard-plugin`; the superproject query returned no path. |
| `git status --short; git log --oneline 28d60e9..HEAD; rg --files -g '!*node_modules*'` | Initial working tree was clean; nine implementation commits existed after base `28d60e9`; package structure contained the expected files. |
| Initial combined JSON/path/guide/skill/CSV/marker/whitespace/status validation script | JSON and paths passed; the initial custom marker interpretation flagged `scene-guide.md`. |
| `rg -n -i 'provisional|placeholder|pending|future' guides`, `Get-Content guides/scene-guide.md`, `git show 467e7e9 -- guides`, and requirement searches | Established that the scene guide was not required to contain a provisional marker. Task 2 requires exact provisional markers only in style files; story and shoot retain their own provisional wording. No scene-guide change was made. |
| Full package validation script (JSON parse, required paths, seven guides and applicable markers, nine skill frontmatter/contracts/routes, exact CSV header, marker search, base-range whitespace, status) | All checks passed through the marker scan; `git diff --check 28d60e9..HEAD` found seven committed trailing blank lines at EOF in the guide files. |
| `git status --short; git diff -- guides;` plus byte-tail inspection | Confirmed the scoped removal affected only one final blank line in each of the seven reported guide files. |
| `git add guides; git diff --check; git diff --cached --check; git diff --cached --stat; git status --short` | Working-tree and staged whitespace checks passed. Staged change: seven deletions across the seven guide files. Windows LF-to-CRLF warnings were emitted by Git but did not produce whitespace errors. |
| `git commit -m "chore: validate storyboard plugin foundation"; git rev-parse HEAD; git status --short` | Created fix commit `a81e8c9eac4737f8e087531cec5daa0c4fdcc05a`; status was clean. |
| Final full package validation script | Passed JSON parsing, ten required paths, seven guides and markers, public plus eight internal skill contracts/routes, exact 45-column CSV contract, forbidden-marker scan, `git diff --check 28d60e9..HEAD`, and clean status. |
| Ordered `Get-Content -Raw` smoke-review reads | Read README, public skill, all eight internal skills, three general guides, four style guides, four templates, and the acceptance runbook in the required order. |
| `git check-ignore -v .superpowers/sdd/task-7-report.md; git status --short` | Confirmed the report is ignored by `.gitignore` (`.superpowers/`) and therefore is not a package commit candidate. |

## Validation results

- Every JSON file parsed successfully.
- All ten required package paths exist.
- Exactly seven guide files exist. The four style guides contain `Status: Provisional starter placeholder`; story and shoot retain their required provisional starter wording.
- The public `storyboard` skill and all eight internal skills have valid frontmatter. Internal skills expose input, output, stop/ask, and invariant contracts; the public skill references every internal stage and required routing phrases.
- `templates/shot-list.csv` has exactly 45 columns in the specified order.
- The forbidden unfinished-work-marker search returned no matches outside the excluded specification and plan folders.
- `git diff --check 28d60e9..HEAD` is clean after the correction.
- Git status is clean. No `.superpowers` execution or review artifact was staged or committed.

## Fix commit

`a81e8c9eac4737f8e087531cec5daa0c4fdcc05a` — `chore: validate storyboard plugin foundation`

Changed only these package files, removing one trailing blank line at EOF from each:

- `guides/scene-guide.md`
- `guides/shoot-guide.md`
- `guides/story-guide.md`
- `guides/styles/cinematic-realism.md`
- `guides/styles/clean-pitch-frame.md`
- `guides/styles/documentary-sports.md`
- `guides/styles/graphic-storyboard-sketch.md`

## Manual smoke self-review

- New one-sentence creative intent routes through project discovery and intake, then planning, with review as the default gate before generation.
- `skip review`, `generate immediately`, and `use your defaults` explicitly bypass review while preserving the internal plan and assumptions.
- Generation requires an approved or explicitly review-skipped plan, records a numbered contact-sheet prompt packet and returned image reference in the iteration, and does not update the active version before a result exists.
- Refinement identifies affected and untouched stable frame IDs, records continuity risks and `changes.md`, and carries unaffected records forward without regenerating the whole sequence.
- Commit/export produces and validates `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json` from a locked iteration.
- The public flow, generation, and export skills prohibit success claims unless the corresponding image reference or validated output paths are present; failure paths preserve prior valid state.

## Concerns

- The review found no remaining package concerns.
- PowerShell displayed UTF-8 em dashes as mojibake in console output, but this was a terminal-display encoding issue only; it did not affect JSON, Markdown contracts, CSV validation, Git whitespace validation, or the committed bytes.
