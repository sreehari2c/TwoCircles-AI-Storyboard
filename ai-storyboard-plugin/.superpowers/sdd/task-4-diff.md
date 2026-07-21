diff --git a/skills/storyboard-commit-export/SKILL.md b/skills/storyboard-commit-export/SKILL.md
new file mode 100644
index 0000000..21ae8b0
--- /dev/null
+++ b/skills/storyboard-commit-export/SKILL.md
@@ -0,0 +1,39 @@
+---
+name: storyboard-commit-export
+description: Use when a selected storyboard iteration is ready to be locked and exported as production files.
+---
+
+# Storyboard Commit and Export
+
+Lock a validated iteration and publish final file artifacts without disturbing earlier finals when validation fails.
+
+## Inputs
+
+Read `project.json`, the selected iteration's frame records, prompt packet, result reference, change summary, references, `templates/plan.md`, and `templates/shot-list.csv`.
+
+## Procedure
+
+1. Lock the selected iteration and request high-resolution output at commit. Warn that deterministic upscaling may be unavailable and regeneration may differ.
+2. Build `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json` from the locked iteration.
+3. Record export metadata, generation settings, source references, timestamps, and change summary.
+4. Validate required manifest fields; stable frame IDs; prompt presence; allowed status, reference, and capture values; required CSV header order and 45-column count; output paths; and Markdown frame identifiers.
+5. Report success only after all validation passes. If it fails, leave prior final outputs untouched.
+
+## Files
+
+May create or update `final/final-plan.md`, `final/shot-list.csv`, `final/manifest.json`, and export metadata in `project.json` only after a complete validation pass. May create temporary candidate outputs outside `final/` for validation.
+
+## Outputs
+
+Produce a locked iteration record and validated final plan, shot list, manifest, high-resolution output reference or availability warning, and export metadata for delivery.
+
+## Stop and Ask
+
+Stop and ask the user when no iteration is selected, high-resolution regeneration would alter an approved result without confirmation, required manifest data is missing, a status/reference/capture value is not allowed, or validation fails.
+
+## Invariants
+
+- Approved iterations are never overwritten.
+- `frame_id` is stable and appears in Markdown and CSV outputs.
+- Prior final outputs remain untouched if validation fails.
+- Never report export success without present, validated output paths.
diff --git a/skills/storyboard-generation/SKILL.md b/skills/storyboard-generation/SKILL.md
new file mode 100644
index 0000000..981df22
--- /dev/null
+++ b/skills/storyboard-generation/SKILL.md
@@ -0,0 +1,38 @@
+---
+name: storyboard-generation
+description: Use when an approved storyboard plan is ready for its contact-sheet image request.
+---
+
+# Storyboard Generation
+
+Generate only from an approved or explicitly review-skipped plan, preserving prior valid state on failure.
+
+## Inputs
+
+Read `project.json`, the approved `plans/plan-v###.md`, selected style guide, approved creative constants, negative constraints, aspect ratio, references, and per-frame prompts.
+
+## Procedure
+
+1. Build a prompt packet with the approved plan, constants, negative constraints, selected style, aspect ratio, references, and numbered frame prompts.
+2. Request a coherent, numbered, low-resolution contact sheet through ChatGPT image generation.
+3. After an image result is available, create `iterations/v###/`, record the exact prompt packet and returned image reference there, and update `project.json` with the new active version and result metadata.
+4. If generation fails, record the failure only where it cannot alter the prior plan or active version.
+
+## Files
+
+May create `iterations/v###/prompt-packet.md` and result-reference metadata under the new iteration. May update `project.json` only after the image result is available.
+
+## Outputs
+
+Produce a numbered contact-sheet reference, exact prompt packet, iteration metadata, and an active-version update for refinement, versioning, or export.
+
+## Stop and Ask
+
+Stop and ask the user when the plan has not been approved or explicitly review-skipped, references have unresolved rights, required prompts are absent, or the requested result would violate an approved constant.
+
+## Invariants
+
+- The default image artifact is a numbered contact sheet.
+- Preserve the prior plan and active version if generation fails.
+- Approved iterations are never overwritten.
+- Do not claim a result exists until its returned image reference is recorded.
diff --git a/skills/storyboard-intake/SKILL.md b/skills/storyboard-intake/SKILL.md
new file mode 100644
index 0000000..df118a2
--- /dev/null
+++ b/skills/storyboard-intake/SKILL.md
@@ -0,0 +1,39 @@
+---
+name: storyboard-intake
+description: Use when a storyboard request needs a normalized project record before planning.
+---
+
+# Storyboard Intake
+
+Normalize supplied information into a portable project without repeating questions that the brief already answers.
+
+## Inputs
+
+Read the supplied story, shoot, scene, style, reference, duration, frame-count, aspect-ratio, and production constraints. Read `templates/project.json`, `templates/project-readme.md`, and an existing `storyboard-projects/<project-slug>/project.json` when the request names a project.
+
+## Procedure
+
+1. Extract supplied facts first. Ask only for a missing fact that materially changes the storyboard; do not ask for information already supplied.
+2. Mark every derived value as `inferred` and keep it distinct from supplied or approved values.
+3. Create a unique, filesystem-safe project slug from the title or creative intent.
+4. Create `storyboard-projects/<project-slug>/` from the two templates and create `plans/`, `iterations/`, `references/`, and `final/` beneath it. Do not overwrite an existing project.
+5. Record the normalized intake fields in `project.json` and stop before planning.
+
+## Files
+
+May create `storyboard-projects/<project-slug>/project.json`, `README.md`, `plans/`, `iterations/`, `references/`, and `final/`. May update only the new project's `project.json` during intake.
+
+## Outputs
+
+Produce a normalized intake summary containing the project slug, supplied facts, `inferred` values, unresolved material gaps, and the project path. This summary and `project.json` are the input to planning.
+
+## Stop and Ask
+
+Stop and ask the user when no meaningful story intent exists, a requested slug collides with a different project, or a missing fact materially changes the story, delivery, rights, or production constraints.
+
+## Invariants
+
+- `project.json` is the source of truth.
+- Preserve supplied facts verbatim and never present an inference as approved.
+- Starter defaults are provisional and replaceable.
+- Do not create a plan or image during intake.
diff --git a/skills/storyboard-planning/SKILL.md b/skills/storyboard-planning/SKILL.md
new file mode 100644
index 0000000..51795c3
--- /dev/null
+++ b/skills/storyboard-planning/SKILL.md
@@ -0,0 +1,40 @@
+---
+name: storyboard-planning
+description: Use when a normalized storyboard project needs a reviewable production plan.
+---
+
+# Storyboard Planning
+
+Turn normalized intake into a file-based, reviewable plan without generating imagery.
+
+## Inputs
+
+Read the project's `project.json`, applicable project-local guides, plugin `guides/`, and `templates/plan.md`. Apply precedence: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, then system inference.
+
+## Procedure
+
+1. Load the applicable story, shoot, scene, and selected-style guides; project-local guidance overrides starter guides.
+2. Use six frames for a general concept and eight for the soccer benchmark unless an approved frame count says otherwise.
+3. Assign stable IDs such as `frame-001`; use `sequence_number` for ordering only.
+4. Produce project summary, creative constants, production assumptions, story progression, frame plan, prompts, negative constraints, and continuity requirements.
+5. Separate approved values from `inferred` assumptions. Apply default shoot values: 16:9, 23.976 fps, 18/25/35/50/85/100mm, tripod static, handheld action/intimacy, gimbal/dolly controlled movement, and 120 fps only when story-justified.
+6. Write `plans/plan-v001.md` from the template and update `project.json` with frames, constants, assumptions, and proposed version metadata.
+
+## Files
+
+May create `storyboard-projects/<project-slug>/plans/plan-v001.md`. May update that project's `project.json`. Never create images, image references, or iteration assets.
+
+## Outputs
+
+Produce a reviewable plan and manifest records with stable frame IDs, prompts, negative constraints, continuity locks, approved values, and labeled assumptions for review.
+
+## Stop and Ask
+
+Stop and ask the user when a material ambiguity cannot be resolved by a labeled recommendation, an approved lock would change, or guide conflicts affect story intent, capture, rights, or delivery.
+
+## Invariants
+
+- `project.json` remains the source of truth.
+- `frame_id` never changes after assignment; `sequence_number` may change.
+- Preserve explicit and approved context over defaults or inference.
+- Planning is no-code and image-free.
diff --git a/skills/storyboard-references/SKILL.md b/skills/storyboard-references/SKILL.md
new file mode 100644
index 0000000..f5a24aa
--- /dev/null
+++ b/skills/storyboard-references/SKILL.md
@@ -0,0 +1,35 @@
+---
+name: storyboard-references
+description: Use when storyboard reference material needs scoped metadata and rights-aware handling.
+---
+
+# Storyboard References
+
+Record reference provenance, permitted influence, and frame scope before a plan or image request relies on it.
+
+## Inputs
+
+Read the supplied reference, `project.json`, applicable frame IDs, and any existing `references/` metadata. Support `internal`, `external`, `uploaded`, `AI-generated`, and `none` reference types.
+
+## Procedure
+
+For every reference, store a reference ID, type, source/path/URL, rights note, applicable frames, what to borrow, and what not to copy. Use `none` as an explicit record when no reference is supplied. Treat uploaded material as unscoped until the user identifies applicable frames. Record unresolved rights uncertainty without inferring permission.
+
+## Files
+
+May create or update `references/<reference-id>.md` or equivalent project-local reference metadata. May update the `references` collection in `project.json` with the same fields and frame scope.
+
+## Outputs
+
+Produce rights-aware, frame-scoped reference records for planning, generation, refinement, and export, including any unresolved-rights warning.
+
+## Stop and Ask
+
+Stop and ask the user when rights, source, permitted use, applicable frames, or requested borrowing are unresolved. Stop before implying that unresolved material is approved.
+
+## Invariants
+
+- Do not assume an uploaded reference applies to the whole project.
+- Preserve source provenance and rights uncertainty.
+- Do not copy protected visual identity, text, or other prohibited elements beyond confirmed permission.
+- `project.json` is the source of truth for reference linkage.
diff --git a/skills/storyboard-refinement/SKILL.md b/skills/storyboard-refinement/SKILL.md
new file mode 100644
index 0000000..f54f754
--- /dev/null
+++ b/skills/storyboard-refinement/SKILL.md
@@ -0,0 +1,39 @@
+---
+name: storyboard-refinement
+description: Use when an existing storyboard needs targeted changes without losing continuity or history.
+---
+
+# Storyboard Refinement
+
+Create a new iteration for a scoped change while preserving unaffected storyboard records and assets.
+
+## Inputs
+
+Read `project.json`, the active iteration, its prompt packet and frame records, the approved plan, references, and the latest explicit user instruction.
+
+## Procedure
+
+1. Identify affected `frame_id` values and classify the request as a project-constant change or frame-only change.
+2. List continuity risks to location, time, lighting, wardrobe, props, subject state, screen direction, and sequence progression.
+3. Create the next `iterations/v###/` snapshot and write `changes.md` with the request, scope, risks, changed fields, and untouched frame IDs.
+4. Support one-frame and multi-frame changes, reference replacement, same-prompt regeneration, and sequence-ending replacement requests.
+5. Carry forward unaffected frame records, prompts, metadata, and assets unchanged; regenerate only the affected scope after required approval.
+
+## Files
+
+May create the next `iterations/v###/` folder and its `changes.md`, frame records, prompt packet, and result metadata. May update `project.json` only after the new iteration has a recorded result or approved non-image change.
+
+## Outputs
+
+Produce a scoped change summary, continuity-risk list, changed and untouched frame IDs, and a new iteration suitable for review, generation, versioning, or export.
+
+## Stop and Ask
+
+Stop and ask the user when a frame-level request changes a project constant or approved lock, a continuity risk has no acceptable recommendation, replacement-reference rights are unresolved, or the affected scope is unclear.
+
+## Invariants
+
+- Never silently regenerate the whole sequence for a frame-specific request.
+- Preserve unaffected frame records, prompts, metadata, and assets.
+- `frame_id` remains stable; only `sequence_number` may change.
+- Approved iterations are never overwritten.
diff --git a/skills/storyboard-review/SKILL.md b/skills/storyboard-review/SKILL.md
new file mode 100644
index 0000000..5ac9c2e
--- /dev/null
+++ b/skills/storyboard-review/SKILL.md
@@ -0,0 +1,35 @@
+---
+name: storyboard-review
+description: Use when a storyboard plan needs an explicit user decision before creation.
+---
+
+# Storyboard Review
+
+Make review the default gate between a proposed plan and image creation.
+
+## Inputs
+
+Read `project.json`, the active `plans/plan-v###.md`, project-local overrides, and the active iteration summary when present.
+
+## Procedure
+
+Show the project summary, creative constants, ordered frame sequence with stable IDs, approved values, `inferred` assumptions, warnings, and conflicts. Support: approve; edit the overall plan; edit, add, remove, or reorder one frame; request alternatives; skip review; and proceed to creation. Record approved instructions and selected alternatives in `project.json` and the active plan when a decision changes them.
+
+## Files
+
+May update `project.json` and the active plan Markdown to record approved or edited plan data. May create no images, prompt packets, or iteration assets.
+
+## Outputs
+
+Produce an explicit review decision, changed frame IDs, remaining warnings, and an approved or revised plan for generation. `skip review` is an explicit decision, not an implicit default.
+
+## Stop and Ask
+
+Stop and ask the user when review is neither approved nor explicitly skipped, a requested change conflicts with an approved lock, or alternatives need a user selection.
+
+## Invariants
+
+- Review is default.
+- `frame_id` is stable even when frames are reordered.
+- Keep approved values distinct from assumptions.
+- Never generate images in this skill.
diff --git a/skills/storyboard-versioning/SKILL.md b/skills/storyboard-versioning/SKILL.md
new file mode 100644
index 0000000..c0070a3
--- /dev/null
+++ b/skills/storyboard-versioning/SKILL.md
@@ -0,0 +1,35 @@
+---
+name: storyboard-versioning
+description: Use when a storyboard iteration must be inspected, restored, or branched without deleting history.
+---
+
+# Storyboard Versioning
+
+Treat iteration folders as immutable history and restore through new snapshots rather than overwrites.
+
+## Inputs
+
+Read `project.json`, `iterations/v###/` folders, each iteration's `changes.md`, frame records, prompt packet, and result metadata.
+
+## Procedure
+
+List iteration folders with change summaries and status. Restore a whole version by creating a new branch snapshot from the selected iteration. Restore one frame by copying its prior frame record into a new iteration and recording the source version. Branch from any existing version by creating a new iteration lineage without deleting history.
+
+## Files
+
+May create a new `iterations/v###/` branch snapshot, `changes.md`, and copied frame records. May update `project.json` to register lineage and select a newly created active version.
+
+## Outputs
+
+Produce an iteration inventory, restore or branch summary, source and destination versions, affected frame IDs, and a new immutable-history-compatible iteration.
+
+## Stop and Ask
+
+Stop and ask the user when the requested source version or frame ID does not exist, the target iteration is ambiguous, or restoring a version would replace an approved active choice without explicit confirmation.
+
+## Invariants
+
+- Never overwrite an approved iteration.
+- Never delete iteration history.
+- Whole-version restores and single-frame restores always create a new iteration.
+- `frame_id` is stable across copies and branches.
