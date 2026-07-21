# Task 6 Report: Manual Acceptance Runbook

## Status

Implemented the manual, no-code acceptance runbook for the storyboard flow.

## Files Changed

- `docs/storyboard-acceptance-runbook.md` (committed): setup requirements, nine acceptance actions, expected evidence, and failure checks.
- `.superpowers/sdd/task-6-report.md` (this report): validation and self-review record.

## Commit

- `eb78641db9f8647775b402c9630d4c3dad3e7a6b` — `docs: add storyboard acceptance runbook`

## Validation Output

```text
Required phrase validation: PASS (6/6)
git diff --check: PASS
```

The required phrase check verified `streetcar`, `frame-001`, `final-plan.md`, `shot-list.csv`, `manifest.json`, and `without the original chat`. The runbook was also reviewed against the brief for all setup constraints, nine numbered actions, evidence requirements, and the three required failure checks.

## Self-Review

- The document remains a concise manual runbook and does not introduce a hosted API, custom script, or runtime requirement.
- Setup explicitly starts from a clean project folder, uses the mural/streetcar/high-school-field soccer example, and requires ChatGPT image generation access.
- All nine acceptance actions are present in the required order.
- Each action has concrete expected evidence, including stable frame IDs, plan and contact-sheet references, unchanged unaffected prompts, frame-scoped reference metadata, iteration change summaries, final Markdown/CSV/manifest artifacts, and disk-based reopen evidence.
- Failure checks explicitly preserve the active version after generation failure, pause lock conflicts for approval, and protect prior final outputs after CSV/manifest validation failure.
- Existing artifact names and project paths match the repository contracts: `storyboard-projects/<project-slug>/`, `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json`.

## Concerns

None. The task commit contains only the requested runbook; this report is intentionally separate from that commit.

## Documentation Finding Fix

Updated the expected-evidence checks to require the concrete approved paths `plans/plan-v001.md`, `iterations/v###/prompt-packet.md`, iteration result-reference/image metadata in the same iteration folder, `iterations/v###/changes.md`, and `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json`.

Fix validation:

```text
Command: phrase validation from task-6-brief.md
Required phrase validation: PASS (6/6)

Command: git diff --check
git diff --check: PASS
```
