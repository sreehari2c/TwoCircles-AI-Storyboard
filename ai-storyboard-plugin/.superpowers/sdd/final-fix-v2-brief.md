# Final whole-branch fix v2

## Context

The previous hardening pass is at commit 037b7ac. Fix all findings from the second whole-branch review together. Work in:
C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin

Read:
- docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md
- docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md
- skills/storyboard/SKILL.md
- workflow/*.md
- templates/project.json
- templates/project-readme.md
- docs/storyboard-acceptance-runbook.md

## Required fixes

### 1. Make all runtime references skill-relative and portable

- Keep only skills/storyboard/SKILL.md discoverable.
- Move workflow/*.md into skills/storyboard/workflow/*.md.
- Move guides/ into skills/storyboard/resources/guides/ and templates/ into skills/storyboard/resources/templates/.
- Update the public skill and every workflow document to reference resources and workflow documents relative to skills/storyboard/.
- Keep storyboard-projects/ at plugin root as the portable project store.
- Update README, design, plan, runbook, and package validation paths to describe this layout.
- Remove the old root workflow/guides/templates copies so there is one source of truth.

### 2. Update the approved design and implementation plan

Amend the design and plan in-place with an explicit architecture amendment:
- only skills/storyboard/SKILL.md is a registered skill;
- workflow/*.md and resources/* are non-discoverable files loaded by the public skill;
- the project store remains storyboard-projects/<project-slug>/ at plugin root;
- validation counts one public skill and the workflow/reference files instead of eight internal SKILL.md files.
Remove or revise any old file map, task, or validation text that still requires eight registered internal skills.

### 3. Add durable review authorization

Update resources/templates/project.json and the review/public/generation workflow documents with:
    "review": {
      "decision": "pending",
      "plan_version": null,
      "authorized_for_generation": false,
      "reviewed_at": null,
      "change_summary": ""
    }

Allowed review decisions are pending, approved, bypassed.
- Planning or plan edits reset review decision to pending and authorization false.
- Review approval or explicit bypass writes decision approved/bypassed, plan_version, reviewed_at, authorization true, and status review.
- Generation requires authorization true and approved/bypassed decision; otherwise route to review.
- State fallback for status review routes to generation only when authorization is true; otherwise it routes to review.
- Include review fields in the version record and acceptance evidence.

### 4. Make final manifest state-consistent

Update the commit/export workflow:
- Build a prospective committed manifest candidate with status committed, selected active_version, and the export record already appended.
- Validate candidate manifest, candidate plan, candidate CSV, and candidate image under iterations/v###/commit-candidate/.
- Require final/manifest.json to be byte-for-byte equivalent in content to the prospective committed project manifest.
- After validation passes, promote all candidate artifacts and write the same committed manifest content to project.json and final/manifest.json.
- Treat promotion plus manifest update as the final commit step.
- On failure leave project.json, final/, active_version, status, and exports unchanged.

### 5. Fix the two minor contract gaps

- Public routing: when status is review and the request is add/remove/reorder/edit-plan, route to the review workflow, which returns status to planned after the edit; do not route that case to planning.
- Add an explicit frame record contract with revision_history and the production/export fields needed to map all 45 CSV columns. Define exact metadata keys in resources/templates/project.json or project-readme, including scene_name, narrative_purpose, time_of_day, camera_height, aperture_intent, playback_intent, priority, capture_type, assigned_shooter, scheduled_time, reference_type, production_notes, status, created_at, and updated_at.
- Document the deterministic mapping from canonical frame fields to the 45 CSV columns in the commit/export workflow or project README.

## Validation

Run:
- JSON parse for all JSON files.
- Confirm only skills/storyboard/SKILL.md exists under skills.
- Confirm eight skills/storyboard/workflow/*.md files and resources/guides/*.md/resources/templates/* exist.
- Confirm no root workflow/, guides/, or templates/ duplicates remain.
- Validate public routing references the eight workflow files and skill-relative resources.
- Validate review decision values/fields, six lifecycle states, four iteration artifact names, fallback choices, candidate ordering, and 45-column mapping terms.
- Run package-wide checks and git diff --check 28d60e9..HEAD.
- Inspect git status; do not add .superpowers scratch files.

## Commit and report

Commit:
    git add .codex-plugin README.md skills/storyboard docs/superpowers/specs docs/superpowers/plans docs/storyboard-acceptance-runbook.md storyboard-projects
    git commit -m "fix: make storyboard plugin portable and stateful"

Write full report to:
C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin\.superpowers\sdd\final-fix-v2-report.md

Return only status, commit, one-line validation summary, concerns, and report path.

