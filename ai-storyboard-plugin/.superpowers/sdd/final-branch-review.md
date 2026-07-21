diff --git a/.codex-plugin/plugin.json b/.codex-plugin/plugin.json
new file mode 100644
index 0000000..f119696
--- /dev/null
+++ b/.codex-plugin/plugin.json
@@ -0,0 +1,17 @@
+{
+  "name": "two-circles-ai-storyboard",
+  "version": "0.1.0",
+  "description": "A guided AI storyboard workflow for turning creative briefs into production-ready visual plans and shot lists.",
+  "author": {
+    "name": "Two Circles"
+  },
+  "license": "Proprietary",
+  "keywords": [
+    "storyboard",
+    "creative planning",
+    "cinematography",
+    "shot list",
+    "image generation"
+  ],
+  "skills": "./skills/"
+}
diff --git a/.gitignore b/.gitignore
index e458ed5..276e69c 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1 +1,2 @@
 .worktrees/
+.superpowers/
diff --git a/README.md b/README.md
index a352d70..17ec51c 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,15 @@
-# TwoCircles-AI-Storyboard
-Ai Storyboard generation project for hackathon
+# Two Circles AI Storyboard
+
+A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
+
+## Start
+
+Use the top-level storyboard workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It will discover the project state, ask only high-impact questions, create a plan, offer review, and hand approved prompts to ChatGPT image generation.
+
+## Project storage
+
+Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.
+
+## Scope
+
+This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.
diff --git a/docs/storyboard-acceptance-runbook.md b/docs/storyboard-acceptance-runbook.md
new file mode 100644
index 0000000..e0a55aa
--- /dev/null
+++ b/docs/storyboard-acceptance-runbook.md
@@ -0,0 +1,75 @@
+# Manual Acceptance Runbook
+
+Use this runbook to verify the complete storyboard flow from a clean project folder. This is a manual acceptance test: it requires access to ChatGPT image generation, but does not require a hosted API, custom script, or runtime.
+
+## Setup
+
+1. Start from a clean project folder with no prior project outputs or chat-dependent state.
+2. Have the supplied soccer production example ready: a sequence spanning a mural, a streetcar, and a high-school field.
+3. Confirm that ChatGPT image generation is available for the contact-sheet request.
+
+Record the project slug and use the durable files under `storyboard-projects/<project-slug>/` as the source of truth throughout the run.
+
+## Acceptance actions
+
+### 1. Import or describe the sequence
+
+Import or describe the soccer production sequence in order: mural, streetcar, and high-school field. Confirm the project is created in `storyboard-projects/<project-slug>/` and that the sequence is represented in the project plan.
+
+Expected evidence: `plans/plan-v001.md` containing all three locations, with stable frame IDs such as `frame-001` and sequence numbers that express the current order.
+
+### 2. Generate an eight-frame plan
+
+Create an eight-frame plan covering the sequence. Use practical variations in lens, camera movement, frame rate, and shot size; keep the production constraints and continuity constants consistent across frames.
+
+Expected evidence: `plans/plan-v001.md` with exactly eight identifiable frame records, each with a stable frame ID and explicit lens, movement, frame-rate, and shot-size values or intentional defaults.
+
+### 3. Review the plan and verify assumptions
+
+Review every frame before generation. Check that user-approved values are distinguishable from inferred assumptions, and label each assumption rather than presenting it as confirmed input. Resolve or accept assumptions before proceeding.
+
+Expected evidence: the reviewed `plans/plan-v001.md`, including labeled assumptions and a visible review/approval outcome.
+
+### 4. Generate the contact sheet
+
+Request a numbered, low-resolution contact sheet through ChatGPT image generation using the approved eight-frame plan. Confirm that the returned image is recorded with the active iteration only after the image result is available.
+
+Expected evidence: a numbered contact-sheet reference plus the exact generation packet at `iterations/v###/prompt-packet.md`; the same `iterations/v###/` folder must contain the iteration result-reference/image metadata pointing to the returned image.
+
+### 5. Change only the streetcar frame
+
+Request a scoped change to the streetcar frame only—for example, adjust its movement while preserving the approved project constants. Compare the new iteration with the prior one.
+
+Expected evidence: the streetcar frame is the only changed frame; unaffected frame records and unchanged prompts remain unchanged; the new immutable `iterations/v###/` folder contains the result-reference/image metadata and scope record.
+
+### 6. Replace one frame with an uploaded reference
+
+Upload a reference and replace one selected frame with it. Confirm that the reference is attached to the selected frame, not applied as a project-wide reference or copied into unrelated frame prompts.
+
+Expected evidence: frame-scoped reference metadata naming the target frame ID in the same `iterations/v###/` folder as the iteration result-reference/image metadata, plus unchanged reference metadata and prompts for unaffected frames.
+
+### 7. Inspect iteration history
+
+List the iterations and inspect the change summary for the scoped edit and reference replacement. Confirm that earlier approved iterations remain available and have not been overwritten.
+
+Expected evidence: iteration history, selected source and destination versions, affected and untouched frame IDs, and the refinement summary at `iterations/v###/changes.md`.
+
+### 8. Commit and validate exports
+
+Commit the selected iteration. Validate that the final Markdown plan, CSV shot list, manifest, and image references are complete and internally consistent.
+
+Expected evidence: the committed outputs `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json`, plus the final image-reference metadata. Confirm that every exported frame maps to a stable frame ID and that the manifest references the selected iteration and generated or supplied images.
+
+### 9. Reopen from disk
+
+Close or leave the original conversation, then reopen the project from `storyboard-projects/<project-slug>/` without the original chat thread. Inspect the project, active iteration, plan, and final artifacts.
+
+Expected evidence: successful reopen without the original chat, with the same project slug, stable frame IDs, iteration history, and final Markdown/CSV/manifest available from disk.
+
+## Failure checks
+
+- Force or observe a failed generation and verify that the active version remains unchanged and the prior approved plan and outputs are still present.
+- Create a lock conflict and verify that the workflow pauses for approval instead of silently overwriting or changing the locked iteration.
+- Introduce a CSV or manifest validation failure and verify that the prior final outputs remain untouched; no partial replacement appears in `final/`.
+
+Record pass/fail results, the observed artifact paths, and any deviations from the expected evidence before accepting the task.
diff --git a/guides/scene-guide.md b/guides/scene-guide.md
new file mode 100644
index 0000000..c4dcb93
--- /dev/null
+++ b/guides/scene-guide.md
@@ -0,0 +1,29 @@
+# Scene guide
+
+Treat the scene as a continuity system. Infer and lock each value from the brief, references, or surrounding frames; mark an inference as `inferred` until approved.
+
+## Scene fields
+
+- Location: identify the specific setting and the visual features that must persist.
+- Time of day: infer from story context, schedule, or light quality, then keep the sun position and ambient level consistent.
+- Lighting direction: lock the key direction and its relationship to the subject and environment.
+- Weather: record conditions such as clear, overcast, rain, or wind and keep their visible effects consistent.
+- Crowd level: specify empty, sparse, moderate, or dense, including background activity.
+- Background depth: choose shallow, medium, or deep space according to story needs and preserve major layers.
+- Wardrobe: lock colors, silhouette, layers, footwear, and visible wear or wetness.
+- Props: record story-critical objects, their appearance, and who holds or moves them.
+- Signage: transcribe only supplied or confirmed text; otherwise describe placement and visual treatment without inventing claims.
+- Atmosphere: capture the intended sensory quality—quiet, tense, celebratory, humid, dusty, intimate, or similar.
+
+## Per-frame continuity checklist
+
+Apply this checklist to every frame:
+
+- [ ] Location and persistent landmarks match.
+- [ ] Time of day and lighting direction match.
+- [ ] Weather and atmosphere match.
+- [ ] Crowd level and background depth match.
+- [ ] Wardrobe and subject appearance match.
+- [ ] Props and signage remain consistent and legible.
+- [ ] Action state, screen direction, and subject position follow the preceding beat.
+- [ ] Any inferred value is marked `inferred` until approved.
diff --git a/guides/shoot-guide.md b/guides/shoot-guide.md
new file mode 100644
index 0000000..798ce0b
--- /dev/null
+++ b/guides/shoot-guide.md
@@ -0,0 +1,21 @@
+# Shoot guide
+
+Use these provisional capture defaults unless the brief calls for a different approach:
+
+- 16:9 delivery.
+- 23.976 fps.
+- 18/25/35/50/85/100mm lens family.
+- Tripod for static shots.
+- Handheld for action or intimacy.
+- Gimbal or dolly for controlled movement.
+- 120 fps only when slow motion serves the story.
+
+## Plain-language recommendations
+
+- Lens: choose a wide lens to establish place and relationship, a normal lens for natural perspective, and a longer lens to isolate a subject or compress distance. Describe the intended feeling before specifying a focal length.
+- Support: use a tripod when the frame should feel stable and repeatable; handheld when proximity, urgency, or human presence matters; use a gimbal or dolly when movement should feel deliberate and controlled.
+- Frame rate: stay at the delivery rate for normal motion. Use 120 fps only when slow motion adds meaning, emphasis, or clarity.
+- Movement: begin with a motivated move—follow action, reveal information, connect subjects, or change emotional distance. Keep static frames static when movement adds no information.
+- Capture type: identify whether a frame is an establishing view, action coverage, detail, reaction, transition, or resolution beat so coverage serves the story.
+
+Prefer creative descriptions of mood, energy, perspective, and intent over unnecessary exposure or shutter technicalities.
diff --git a/guides/story-guide.md b/guides/story-guide.md
new file mode 100644
index 0000000..d089eb2
--- /dev/null
+++ b/guides/story-guide.md
@@ -0,0 +1,18 @@
+# Story guide
+
+This is a provisional starter default, not official Two Circles guidance. Use it as a practical starting point and adapt it to the brief.
+
+## Default structure
+
+General short concepts default to six frames; infer another count when narrative beats, duration, or event coverage require it. Use this sequence grammar as a flexible starting sequence:
+
+1. Establishing
+2. Introduce subject/action
+3. Detail
+4. Reaction/emotion
+5. Progression/escalation
+6. Resolution/call to action
+
+Ask only questions that materially change story, subject, location, action, duration, or delivery. Vary shot size and camera language while avoiding redundant frames. Duration is optional for non-linear event coverage and recorded when supplied or inferred.
+
+Mark inferred values as `inferred` until approved. Keep each frame focused on one meaningful beat, and make changes in action, emotion, or visual information legible from frame to frame.
diff --git a/guides/styles/cinematic-realism.md b/guides/styles/cinematic-realism.md
new file mode 100644
index 0000000..e21b0cc
--- /dev/null
+++ b/guides/styles/cinematic-realism.md
@@ -0,0 +1,49 @@
+# Cinematic realism
+
+Status: Provisional starter placeholder
+
+Use photorealistic, premium commercial realism with controlled contrast and intentional depth of field. Describe executable dynamic camera positions that a crew could stage and repeat.
+
+## Rendering method
+
+Photorealistic live-action treatment with premium commercial realism and physically plausible detail.
+
+## Contrast
+
+Controlled contrast with readable shadows and purposeful highlight separation.
+
+## Saturation
+
+Moderate, selective saturation that supports the subject without looking artificial.
+
+## Lighting
+
+Motivated, shaped light with clear direction and refined falloff.
+
+## Texture
+
+Natural skin, material, and environmental texture with restrained polish.
+
+## Typical lenses
+
+18mm for context, 35mm for spatial storytelling, 50mm for natural perspective, and 85/100mm for selective intimacy.
+
+## Camera movement
+
+Executable dolly, gimbal, crane, or controlled handheld moves motivated by the beat.
+
+## Framing tendencies
+
+Balanced compositions, deliberate negative space, and intentional depth-of-field changes.
+
+## Subject treatment
+
+Confident, specific, and human; preserve believable gesture, expression, and physical interaction.
+
+## Continuity locks
+
+Lock screen direction, light direction, wardrobe, props, time of day, and lens language across adjacent frames.
+
+## Prohibited traits
+
+Avoid plastic skin, impossible camera positions, generic stock-photo posing, excessive bloom, and inconsistent anatomy or physics.
diff --git a/guides/styles/clean-pitch-frame.md b/guides/styles/clean-pitch-frame.md
new file mode 100644
index 0000000..ebb0561
--- /dev/null
+++ b/guides/styles/clean-pitch-frame.md
@@ -0,0 +1,49 @@
+# Clean pitch frame
+
+Status: Provisional starter placeholder
+
+Favor polished presentation composition, controlled backgrounds, strong visual hierarchy, and client-review readability.
+
+## Rendering method
+
+Clean, presentation-ready visual treatment with clear hierarchy and selective detail.
+
+## Contrast
+
+Controlled contrast that keeps the primary message readable at a glance.
+
+## Saturation
+
+Confident but disciplined color, with accents reserved for key subjects or actions.
+
+## Lighting
+
+Evenly controlled, flattering light with enough shape to separate subject and background.
+
+## Texture
+
+Crisp, refined materials and surfaces without distracting micro-detail.
+
+## Typical lenses
+
+35mm or 50mm for clear spatial storytelling; 85mm when isolation improves pitch clarity.
+
+## Camera movement
+
+Simple, intentional moves that can be explained in a pitch and understood in one viewing.
+
+## Framing tendencies
+
+Strong visual hierarchy, controlled backgrounds, clean negative space, and readable subject placement.
+
+## Subject treatment
+
+Specific, approachable, and easy to assess; prioritize the story beat over visual noise.
+
+## Continuity locks
+
+Lock palette, background treatment, subject scale, key props, typography-safe space, and action direction.
+
+## Prohibited traits
+
+Avoid cluttered backgrounds, competing focal points, illegible details, arbitrary camera flourishes, and unsupported claims.
diff --git a/guides/styles/documentary-sports.md b/guides/styles/documentary-sports.md
new file mode 100644
index 0000000..f5c192a
--- /dev/null
+++ b/guides/styles/documentary-sports.md
@@ -0,0 +1,49 @@
+# Documentary sports
+
+Status: Provisional starter placeholder
+
+Prioritize natural light, observational framing, handheld energy, authentic expressions, and less polished composition.
+
+## Rendering method
+
+Naturalistic documentary rendering that preserves the immediacy of real sports coverage.
+
+## Contrast
+
+Moderate contrast with detail retained in changing outdoor light.
+
+## Saturation
+
+Natural, environment-led color with authentic team and venue tones.
+
+## Lighting
+
+Available or naturally motivated light; accept honest variation across coverage.
+
+## Texture
+
+Tactile surfaces, sweat, grass, weather, equipment wear, and lived-in environments.
+
+## Typical lenses
+
+35mm for proximity, 50mm for natural observation, and 85/100mm for action or reaction from a respectful distance.
+
+## Camera movement
+
+Handheld energy, responsive pans, and motivated follow movement.
+
+## Framing tendencies
+
+Observational, slightly imperfect compositions that leave room for action to unfold.
+
+## Subject treatment
+
+Authentic expressions, unscripted gestures, effort, concentration, and shared emotion.
+
+## Continuity locks
+
+Lock play direction, uniforms, weather, field orientation, equipment, score context, and action state.
+
+## Prohibited traits
+
+Avoid staged smiles, sterile studio polish, impossible athletic poses, invented game details, and exaggerated spectacle.
diff --git a/guides/styles/graphic-storyboard-sketch.md b/guides/styles/graphic-storyboard-sketch.md
new file mode 100644
index 0000000..79b0dae
--- /dev/null
+++ b/guides/styles/graphic-storyboard-sketch.md
@@ -0,0 +1,49 @@
+# Graphic storyboard sketch
+
+Status: Provisional starter placeholder
+
+Use a restrained monochrome or limited-color palette, hand-drawn appearance, clear blocking, and strong silhouettes.
+
+## Rendering method
+
+Hand-drawn storyboard panels with confident linework and readable visual shorthand.
+
+## Contrast
+
+Strong value separation so blocking and action read immediately.
+
+## Saturation
+
+Monochrome by default or a tightly limited accent palette.
+
+## Lighting
+
+Simple directional indication with clear shadow shapes rather than rendered realism.
+
+## Texture
+
+Paper, pencil, ink, or restrained marker texture that supports legibility.
+
+## Typical lenses
+
+Describe wide, normal, or telephoto perspective in panel notes; use focal lengths only when they clarify staging.
+
+## Camera movement
+
+Show movement with arrows, panel progression, and concise action notes.
+
+## Framing tendencies
+
+Clear blocking, strong silhouettes, readable eyelines, and purposeful panel borders.
+
+## Subject treatment
+
+Expressive but economical poses with distinctive silhouettes and easy-to-follow action.
+
+## Continuity locks
+
+Lock screen direction, silhouette, prop placement, staging landmarks, and panel-to-panel action progression.
+
+## Prohibited traits
+
+Avoid muddy rendering, decorative detail that obscures blocking, ambiguous silhouettes, and uncontrolled color variety.
diff --git a/skills/storyboard-commit-export/SKILL.md b/skills/storyboard-commit-export/SKILL.md
new file mode 100644
index 0000000..6027728
--- /dev/null
+++ b/skills/storyboard-commit-export/SKILL.md
@@ -0,0 +1,43 @@
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
+4. Validate required manifest fields; stable frame IDs; prompt presence; allowed status, reference, and capture values; required CSV header order and 45-column count; output paths; and Markdown frame identifiers. The allowed values are:
+   - `status`: `proposed`, `approved`, `assigned`, `captured`, `completed`, `omitted`
+   - `reference_type`: `internal`, `external`, `uploaded`, `AI-generated`, `none`
+   - `capture_type`: `must-capture`, `inspiration`, `optional`, `alternate`
+   Commit validation must reject any value outside these lists.
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
diff --git a/skills/storyboard/SKILL.md b/skills/storyboard/SKILL.md
new file mode 100644
index 0000000..c7e5918
--- /dev/null
+++ b/skills/storyboard/SKILL.md
@@ -0,0 +1,82 @@
+---
+name: storyboard
+description: Turn a creative brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
+---
+
+# Smart Storyboard
+
+Coordinate one storyboard project from creative intent through a reviewable plan, image-generation handoff, targeted revisions, and production exports. Keep the work file-based, scope changes precisely, and preserve approved history.
+
+## Project discovery
+
+Follow this order exactly:
+
+1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`; an explicitly named committed project may be reopened.
+2. Scan `storyboard-projects/*/project.json` for project manifests. A project is active when the current workspace is inside that project's folder, or when exactly one project manifest exists under `storyboard-projects/`.
+3. If multiple manifests exist and none is in the current project folder, ask the user to choose; do not guess based on file ordering or timestamps.
+4. If no project exists and the user supplied meaningful story input, create one from `templates/` with `storyboard-intake`.
+5. If no project exists and no meaningful story input was supplied, ask for one sentence of creative intent.
+
+Treat `project.json` as the source of truth. `frame_id` is stable (for example, `frame-001`); `sequence_number` may change when frames are reordered.
+
+## Routing
+
+Load only the relevant internal skill and guides for the current request. Route requests as follows:
+
+| Request | Load |
+| --- | --- |
+| New brief, script, concept, or reference | `skills/storyboard-intake/SKILL.md` (`storyboard-intake`) |
+| Missing plan, make a plan, add/remove/reorder frames | `skills/storyboard-planning/SKILL.md` (`storyboard-planning`) |
+| Review or assumptions request | `skills/storyboard-review/SKILL.md` (`storyboard-review`) |
+| Approved plan plus generate/create | `skills/storyboard-generation/SKILL.md` (`storyboard-generation`) |
+| Change frame, make wider, keep everything else | `skills/storyboard-refinement/SKILL.md` (`storyboard-refinement`) |
+| Show versions, restore, branch | `skills/storyboard-versioning/SKILL.md` (`storyboard-versioning`) |
+| Finalize, commit, export, make a shot list | `skills/storyboard-commit-export/SKILL.md` (`storyboard-commit-export`) |
+| Reference attachment or reference association | `skills/storyboard-references/SKILL.md` (`storyboard-references`) |
+
+When a request spans stages, use the smallest necessary sequence, preserving the normal lifecycle: intake -> plan -> review -> generate -> refine/version -> commit/export.
+
+## Defaults and context
+
+- Review is enabled by default. `skip review`, `generate immediately`, and `use your defaults` bypass review while retaining the internal plan; record the explicit bypass.
+- General concepts default to six frames; the soccer benchmark uses eight unless an approved frame count overrides it.
+- Starter guides are provisional; project-local guides override them.
+- Apply context precedence in this order: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, system inference.
+- Keep supplied, approved, and inferred values distinct. Do not change an approved lock without explicit user choice.
+- For generation, hand approved prompts to ChatGPT image generation and retain the exact prompt packet with the returned image reference.
+
+## Stage handling
+
+1. Discover the project, then load its `project.json` and only the guides needed for the routed stage.
+2. Associate references through `storyboard-references` before a plan, generation, refinement, or export relies on them.
+3. Use `storyboard-intake` to normalize new meaningful story input before planning.
+4. Use `storyboard-planning` to produce or alter the file-based plan. Frame additions, removals, and reordering update the plan while preserving each `frame_id`.
+5. Use `storyboard-review` as the default decision gate. An explicitly skipped review still preserves the internal plan and its assumptions.
+6. Use `storyboard-generation` only for an approved or explicitly review-skipped plan. Show the exact frame prompts or a readable prompt summary before requesting image generation.
+7. Use `storyboard-refinement` for scoped frame or continuity changes. Show a written change summary and the untouched frame IDs. Never silently regenerate an entire storyboard after a frame-specific request.
+8. Use `storyboard-versioning` to inspect, restore, or branch immutable iteration history.
+9. Use `storyboard-commit-export` to validate and produce final plan, shot list, and manifest artifacts.
+
+## User-visible response format
+
+Keep responses concise and structured. Include:
+
+- **Current stage:** the routed workflow stage and project.
+- **Changed files:** created or modified paths, or `none` before a write.
+- **Inferred assumptions:** labeled defaults or recommendations.
+- **Warnings/conflicts:** unresolved rights, locks, ambiguities, or validation issues.
+- **Frame IDs affected:** affected stable IDs and, for refinements, untouched IDs.
+- **Next action:** the required decision or the next safe stage.
+
+For image generation, include exact frame prompts or a readable prompt summary. For refinement, include a written change summary and untouched frame IDs.
+
+## Stop conditions
+
+Stop for explicit user choice when:
+
+- a lock would change;
+- a reference rights issue is unresolved;
+- a high-impact ambiguity cannot be resolved with a recommendation; or
+- commit validation fails.
+
+Never claim image generation, export, or commit success without the corresponding file or reference being present. Preserve prior approved records and outputs when a later operation fails.
diff --git a/storyboard-projects/.gitkeep b/storyboard-projects/.gitkeep
new file mode 100644
index 0000000..e69de29
diff --git a/templates/plan.md b/templates/plan.md
new file mode 100644
index 0000000..1cd982d
--- /dev/null
+++ b/templates/plan.md
@@ -0,0 +1,29 @@
+# [Project Title] Storyboard Plan
+## Project Summary
+## Creative Constants
+## Production Assumptions
+## Story Progression
+## Frame Plan
+
+Duplicate the Frame section for every stable frame ID. Final plans use machine-readable Frame ID fields.
+
+### Frame 001 — [Frame Title]
+- Frame ID: frame-001
+- Purpose:
+- Description:
+- Subject:
+- Action:
+- Shot size:
+- Camera angle:
+- Camera position:
+- Lens:
+- Depth of field:
+- Movement:
+- Location:
+- Lighting:
+- Duration:
+- Transition:
+- Continuity locks:
+- Reference source:
+- Prompt:
+- Negative constraints:
diff --git a/templates/project-readme.md b/templates/project-readme.md
new file mode 100644
index 0000000..aadccac
--- /dev/null
+++ b/templates/project-readme.md
@@ -0,0 +1,20 @@
+# Storyboard Project
+
+`project.json` is the source of truth for the storyboard project.
+
+## Project folders
+
+- `guides/` contains project guidance, conventions, and creative direction.
+- `references/` contains source references used to inform the storyboard.
+- `plans/` contains planning documents and frame-level shot plans.
+- `iterations/` contains working generations and revision history.
+- `final/` contains approved final storyboard assets.
+- `exports/` contains deliverables prepared for downstream use.
+
+## Frame identity
+
+`frame_id` is stable and identifies the same storyboard frame across revisions. `sequence_number` describes its current order and may change as the story is reorganized.
+
+## Lifecycle
+
+The project lifecycle is: intake -> plan -> review -> generate -> refine/version -> commit/export.
diff --git a/templates/project.json b/templates/project.json
new file mode 100644
index 0000000..a6a354c
--- /dev/null
+++ b/templates/project.json
@@ -0,0 +1,19 @@
+{
+  "project_id": "project-slug",
+  "title": "Untitled Storyboard",
+  "schema_version": "0.1.0",
+  "status": "proposed",
+  "active_version": 0,
+  "story": {},
+  "shoot": {},
+  "scene": {},
+  "style": {},
+  "creative_constants": {
+    "locks": [],
+    "prohibited_elements": []
+  },
+  "references": [],
+  "frames": [],
+  "versions": [],
+  "exports": []
+}
diff --git a/templates/shot-list.csv b/templates/shot-list.csv
new file mode 100644
index 0000000..419b858
--- /dev/null
+++ b/templates/shot-list.csv
@@ -0,0 +1 @@
+project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at
