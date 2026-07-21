# Storyboard Project

`project.json` is the source of truth for the storyboard project.

## Project folders

- `guides/` contains project guidance, conventions, and creative direction.
- `references/` contains source references used to inform the storyboard.
- `plans/` contains planning documents and frame-level shot plans.
- `iterations/` contains working generations and revision history.
- `final/` contains approved final storyboard assets.
- `exports/` contains deliverables prepared for downstream use.

Every iteration contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Commit candidates are validated under `iterations/v###/commit-candidate/` before promotion to `final/`.

## Frame identity

`frame_id` is stable and identifies the same storyboard frame across revisions. `sequence_number` describes its current order and may change as the story is reorganized.

Every frame follows the canonical record in the skill-relative `resources/templates/project.json`. It includes `revision_history`, production metadata, reference metadata, prompts, continuity notes, timestamps, and image paths sufficient to map all 45 shot-list columns. Metadata key names are exact and must not be aliased; the commit/export workflow is the authoritative ordered mapping.

## Durable review authorization

Planning and every plan edit reset `review.decision` to `pending`, clear `plan_version` and `reviewed_at`, and set `authorized_for_generation` to `false`. Approval or explicit bypass records `approved` or `bypassed`, the reviewed plan version, review timestamp, change summary, and authorization while status remains `review`. Generation is invalid without that durable state, and every version record snapshots it.

## Lifecycle

The manifest status lifecycle is: `proposed -> planned -> review -> generated -> refined -> committed`. Explicit review bypass is recorded in status `review`; initial commit may use `generated` or `refined`, and re-export may use `committed` without changing status.

Commit first builds a prospective committed manifest containing the selected `active_version` and appended export records. Candidate plan, CSV, image, and manifest validate together. The exact candidate manifest bytes are then written to both `project.json` and `final/manifest.json` as part of the same final promotion step; a failed transaction leaves prior state unchanged.
