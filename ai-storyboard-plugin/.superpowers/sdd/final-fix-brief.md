# Final whole-branch fix: contract hardening

## Context

The whole-branch review found five Important blockers and two Minor documentation issues. Fix them together as one coherent change in:
C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin

Read:
- Design: docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md
- Plan: docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md
- Existing implementation files under skills/, templates/, and docs/.

## Required fixes

### 1. Make stage instructions non-discoverable

Keep only skills/storyboard/SKILL.md as a registered Codex skill:
- Move the eight stage SKILL.md instruction files to workflow/ as ordinary Markdown reference documents, for example workflow/storyboard-intake.md through workflow/storyboard-commit-export.md.
- Remove the eight stage folders/files from skills/.
- Change .codex-plugin/plugin.json skills property to ./skills/storyboard/.
- Update the public storyboard skill to load the corresponding workflow/*.md reference when routing.
- Preserve the stage contracts and state rules; these are now internal workflow modules, not independently triggerable skills.
- Update README wording if needed to make the one public workflow explicit.

### 2. Define canonical iteration and frame records

Update templates/project.json and the relevant workflow documents with explicit shapes:
- project status is one of proposed, planned, review, generated, refined, committed;
- active_version is an integer;
- frames is an array of frame records with frame_id, sequence_number, title, description, prompt, negative_prompt, metadata, continuity_locks, references, and image paths;
- versions is an array of version records with version_id, status, plan_path, prompt_packet_path, frames_path, result_path, changes_path, created_at, and change_summary;
- exports is an array of export records with artifact type/path, version, validation result, and timestamp.

Initial generation must write:
- iterations/v001/prompt-packet.md
- iterations/v001/frames.json
- iterations/v001/result.json
- iterations/v001/changes.md
and update project.json.frames, project.json.versions, and project.json.active_version.

Refinement and versioning must consume those exact files and create the same four-file contract for every new iteration.

### 3. Define state transitions and fallback routing

Add to workflow/storyboard.md (the moved public SKILL.md) explicit state transitions:
- intake creates proposed;
- planning proposed -> planned;
- review planned -> review;
- approval or explicit review bypass review -> generated after generation;
- refinement generated -> refined;
- commit refined or generated -> committed;
- plan edits from review return to planned;
- named committed projects may reopen for inspection, branching, version restore, or export; a new branch starts with a new active iteration.

For requests without a stage verb, route from project.json.status:
- proposed -> planning;
- planned -> review;
- review -> review;
- generated/refined -> show current iteration and offer refinement or commit;
- committed -> show final state and offer inspect, branch, restore, or export.
Do not guess when the status is missing or invalid; stop with a manifest error.

### 4. Define frame-only refinement fallback

Update workflow/storyboard-refinement.md and the acceptance runbook:
- If the image capability supports frame-level editing, regenerate only requested frame assets and recompose only when the capability provides that operation.
- If it does not support frame-level editing and no compositor is available, do not silently regenerate the full contact sheet.
- Ask the user to choose one explicit fallback: (a) regenerate the whole contact sheet with unchanged frame prompts locked and a continuity/drift warning, (b) create a frame-only revision artifact while retaining the previous contact sheet, or (c) update prompts/metadata only with no new image.
- In all cases preserve unaffected frame records and prompts and record the chosen fallback in changes.md and result.json.

### 5. Make commit/export transactional

Update workflow/storyboard-commit-export.md:
- Validate the selected iteration and all required files first.
- Generate high-resolution output and candidate final-plan.md, shot-list.csv, and manifest.json under iterations/v###/commit-candidate/ rather than final/.
- Validate candidate JSON, CSV, Markdown frame identifiers, enum values, and image reference.
- Only after all validation passes promote candidate files to final/, append the export record, set project status to committed, update active_version, and report success.
- On any failure, keep prior final outputs, active_version, and status unchanged; report the exact failure and leave the candidate isolated.

### 6. Fix Minor documentation issues

- Intake must create the project-local guides/ and exports/ directories as well as references, plans, iterations, and final.
- In docs/storyboard-acceptance-runbook.md, action 1 is intake/import only. Action 2 creates the eight-frame plan and is the first step that should expect plans/plan-v001.md.

## Validation

Run:
- JSON parse for every JSON file.
- Check that only skills/storyboard/SKILL.md exists under skills/ and all eight workflow/*.md files exist.
- Check the public skill references every workflow file and all stage route names.
- Check project template includes the stated lifecycle enums and record keys.
- Check the exact four iteration artifact names appear in generation, refinement, versioning, and runbook docs.
- Check fallback choices and commit-candidate ordering appear in the relevant docs.
- Run git diff --check 28d60e9..HEAD.
- Run the existing package-wide checks from Task 7.
- Inspect git status; do not add .superpowers scratch files.

## Commit and report

Commit:
    git add .codex-plugin/plugin.json README.md skills workflow templates docs
    git commit -m "fix: harden storyboard workflow contracts"

Write detailed report to:
C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin\.superpowers\sdd\final-fix-report.md

Return only status, commit, one-line validation summary, concerns, and report path.

