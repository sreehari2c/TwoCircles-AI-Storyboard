diff --git a/.codex-plugin/plugin.json b/.codex-plugin/plugin.json
new file mode 100644
index 0000000..f3f879e
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
+  "skills": "./skills/storyboard/"
+}
diff --git a/.gitignore b/.gitignore
index e458ed5..276e69c 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1 +1,2 @@
 .worktrees/
+.superpowers/
diff --git a/README.md b/README.md
index a352d70..097cdfc 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,17 @@
-# TwoCircles-AI-Storyboard
-Ai Storyboard generation project for hackathon
+# Two Circles AI Storyboard
+
+A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
+
+## Start
+
+Use the single registered `storyboard` workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It discovers project state, asks only high-impact questions, creates a plan, offers review, and hands approved prompts to ChatGPT image generation.
+
+The stage instructions in `workflow/` are ordinary internal references loaded by the public workflow. They are not separately registered or independently triggerable skills.
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
index 0000000..7d137ea
--- /dev/null
+++ b/docs/storyboard-acceptance-runbook.md
@@ -0,0 +1,82 @@
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
+Record the project slug and use durable files under `storyboard-projects/<project-slug>/` as the source of truth throughout the run.
+
+## Acceptance actions
+
+### 1. Import or describe the sequence
+
+Import or describe the soccer production sequence in order: mural, streetcar, and high-school field. This action is intake/import only; do not create the plan yet.
+
+Expected evidence: `project.json` has status `proposed`, active version `0`, normalized story input covering all three locations, and empty `frames` and `versions` arrays. Project-local `guides/`, `references/`, `plans/`, `iterations/`, `final/`, and `exports/` directories exist, with no planning artifact created yet.
+
+### 2. Generate an eight-frame plan
+
+Create an eight-frame plan covering the sequence. Use practical variations in lens, camera movement, frame rate, and shot size; keep production constraints and continuity constants consistent across frames.
+
+Expected evidence: this is the first action that creates `plans/plan-v001.md`. It contains exactly eight identifiable frame records, each with a stable frame ID such as `frame-001` and explicit lens, movement, frame-rate, and shot-size values or intentional defaults; project status is `planned`.
+
+### 3. Review the plan and verify assumptions
+
+Review every frame before generation. Check that approved values are distinguishable from inferred assumptions. Resolve or accept assumptions before proceeding.
+
+Expected evidence: the reviewed `plans/plan-v001.md`, labeled assumptions, and a visible review or bypass decision. Plan edits return status to `planned`; a completed review uses status `review`.
+
+### 4. Generate the contact sheet
+
+Request a numbered, low-resolution contact sheet through ChatGPT image generation using the approved eight-frame plan.
+
+Expected evidence: initial generation creates all four files: `iterations/v001/prompt-packet.md`, `iterations/v001/frames.json`, `iterations/v001/result.json`, and `iterations/v001/changes.md`. `result.json` contains the numbered contact-sheet reference, while `project.json.frames`, `project.json.versions`, integer `active_version`, and status `generated` agree with v001.
+
+### 5. Change only the streetcar frame
+
+Request a scoped change to the streetcar frame only, such as adjusting its movement while preserving approved project constants. Compare the new iteration with the prior one.
+
+If frame-level editing is unavailable and there is no compositor, verify the workflow asks for one explicit fallback:
+
+- (a) regenerate the whole contact sheet with unchanged frame prompts locked and a continuity/drift warning;
+- (b) create a frame-only revision artifact while retaining the previous contact sheet; or
+- (c) update prompts/metadata only with no new image.
+
+Expected evidence: the streetcar frame is the only changed frame; unaffected frame records and prompts remain unchanged. The new immutable `iterations/v###/` folder contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; both `result.json` and `changes.md` record the capability path or fallback and retained image references.
+
+### 6. Replace one frame with an uploaded reference
+
+Upload a reference and replace one selected frame with it. Confirm the reference is attached to the selected frame, not applied project-wide or copied into unrelated frame prompts.
+
+Expected evidence: frame-scoped reference metadata names the target frame ID. The same new `iterations/v###/` folder contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`, with unchanged reference metadata and prompts for unaffected frames.
+
+### 7. Inspect iteration history
+
+List iterations and inspect change summaries for the scoped edit and reference replacement. Confirm earlier approved iterations remain available.
+
+Expected evidence: iteration history, selected source and destination versions, affected and untouched frame IDs, and complete `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md` files in every generated, refined, restored, or branched `iterations/v###/` folder.
+
+### 8. Commit and validate exports
+
+Commit the selected iteration. Validate the final Markdown plan, CSV shot list, manifest, and image references.
+
+Expected evidence: source validation occurs first; candidate `final-plan.md`, `shot-list.csv`, `manifest.json`, and image-reference metadata are built and validated under `iterations/v###/commit-candidate/`; only then are validated files promoted to `final/`. Confirm every exported frame maps to a stable frame ID, export records are appended, and status becomes `committed` only after promotion.
+
+### 9. Reopen from disk
+
+Close or leave the original conversation, then reopen the project from `storyboard-projects/<project-slug>/` without the original chat thread. Inspect the project, active iteration, plan, and final artifacts.
+
+Expected evidence: successful reopen without the original chat, with the same project slug, stable frame IDs, iteration history, and final Markdown/CSV/manifest available from disk. A named committed project offers inspect, branch, restore, or export. Choose export once and confirm the transactional candidate flow runs from status `committed`, appends validated export records, and leaves status `committed` after promotion.
+
+## Failure checks
+
+- Force or observe a failed generation and verify that the active version, status, prior plan, and outputs remain unchanged.
+- Create a lock conflict and verify the workflow pauses for approval.
+- Introduce a CSV or manifest validation failure and verify that prior final outputs remain untouched and no partial replacement appears in `final/`.
+- Verify a failed commit leaves the candidate isolated under `iterations/v###/commit-candidate/` and leaves prior final outputs, status, active version, and export records unchanged.
+
+Record pass/fail results, observed artifact paths, and deviations before accepting the task.
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
diff --git a/skills/storyboard/SKILL.md b/skills/storyboard/SKILL.md
new file mode 100644
index 0000000..017a395
--- /dev/null
+++ b/skills/storyboard/SKILL.md
@@ -0,0 +1,80 @@
+---
+name: storyboard
+description: Turn a creative brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
+---
+
+# Smart Storyboard
+
+Coordinate one storyboard project from creative intent through a reviewable plan, image-generation handoff, targeted revisions, and production exports. This is the plugin's only registered skill.
+
+On every invocation, load `workflow/storyboard.md` as the orchestration contract. Then load only the routed ordinary reference below; files in `workflow/` are not skills and must not be offered as independently triggerable commands.
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
+Treat `project.json` as the source of truth. Before routing an existing project, require `status` to be one of `proposed`, `planned`, `review`, `generated`, `refined`, or `committed`, and require `active_version` to be an integer. Stop with a manifest error when either value is missing or invalid.
+
+## Routing
+
+Load only the relevant internal workflow reference and guides for the current request. Route requests as follows:
+
+| Stage route name | Request | Load |
+| --- | --- | --- |
+| `storyboard-intake` | New brief, script, concept, or project import | `workflow/storyboard-intake.md` |
+| `storyboard-planning` | Missing plan, make a plan, add/remove/reorder frames | `workflow/storyboard-planning.md` |
+| `storyboard-review` | Review or assumptions request | `workflow/storyboard-review.md` |
+| `storyboard-generation` | Approved plan plus generate/create | `workflow/storyboard-generation.md` |
+| `storyboard-refinement` | Change frame, make wider, keep everything else | `workflow/storyboard-refinement.md` |
+| `storyboard-versioning` | Show versions, restore, branch | `workflow/storyboard-versioning.md` |
+| `storyboard-commit-export` | Finalize, commit, export, make a shot list | `workflow/storyboard-commit-export.md` |
+| `storyboard-references` | Reference attachment or reference association | `workflow/storyboard-references.md` |
+
+Explicit stage verbs control routing only when the requested transition is valid. For requests without a stage verb, route from `project.json.status`: `proposed` to planning; `planned` or `review` to review; `generated` or `refined` to a current-iteration summary offering refinement or commit; and `committed` to a final-state summary offering inspect, branch, restore, or export. Never infer a route from missing or invalid state.
+
+## Defaults and context
+
+- Review is enabled by default. `skip review`, `generate immediately`, and `use your defaults` bypass review while retaining the internal plan; record the explicit bypass.
+- General concepts default to six frames; the soccer benchmark uses eight unless an approved frame count overrides it.
+- Starter guides are provisional; project-local guides override them.
+- Apply context precedence in this order: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, system inference.
+- Keep supplied, approved, and inferred values distinct. Do not change an approved lock without explicit user choice.
+- For generation, hand approved prompts to ChatGPT image generation and retain `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md` for every iteration with the returned image reference.
+
+## Stage handling
+
+1. Discover and validate the project using `workflow/storyboard.md`.
+2. Load exactly one routed stage reference unless a valid multi-stage request requires the smallest necessary sequence.
+3. Apply lifecycle transitions only after the routed stage's durable artifacts exist and pass its validation.
+4. Associate references through `storyboard-references` before another stage relies on them.
+5. Preserve stable `frame_id` values, unaffected records, immutable iterations, and prior valid finals.
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
index 0000000..ff6bdfc
--- /dev/null
+++ b/templates/project-readme.md
@@ -0,0 +1,22 @@
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
+Every iteration contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Commit candidates are validated under `iterations/v###/commit-candidate/` before promotion to `final/`.
+
+## Frame identity
+
+`frame_id` is stable and identifies the same storyboard frame across revisions. `sequence_number` describes its current order and may change as the story is reorganized.
+
+## Lifecycle
+
+The manifest status lifecycle is: `proposed -> planned -> review -> generated -> refined -> committed`. Explicit review bypass may generate from `planned`; initial commit may use `generated` or `refined`, and re-export may use `committed` without changing status.
diff --git a/templates/project.json b/templates/project.json
new file mode 100644
index 0000000..ebe3552
--- /dev/null
+++ b/templates/project.json
@@ -0,0 +1,63 @@
+{
+  "project_id": "project-slug",
+  "title": "Untitled Storyboard",
+  "schema_version": "0.1.0",
+  "status": "proposed",
+  "status_values": [
+    "proposed",
+    "planned",
+    "review",
+    "generated",
+    "refined",
+    "committed"
+  ],
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
+  "record_contracts": {
+    "frame": {
+      "frame_id": "frame-001",
+      "sequence_number": 1,
+      "title": "",
+      "description": "",
+      "prompt": "",
+      "negative_prompt": "",
+      "metadata": {},
+      "continuity_locks": [],
+      "references": [],
+      "image_paths": {
+        "contact_sheet": "",
+        "frame": "",
+        "thumbnail": ""
+      }
+    },
+    "version": {
+      "version_id": "v001",
+      "status": "generated",
+      "plan_path": "plans/plan-v001.md",
+      "prompt_packet_path": "iterations/v001/prompt-packet.md",
+      "frames_path": "iterations/v001/frames.json",
+      "result_path": "iterations/v001/result.json",
+      "changes_path": "iterations/v001/changes.md",
+      "created_at": "",
+      "change_summary": ""
+    },
+    "export": {
+      "artifact_type": "final-plan",
+      "artifact_path": "final/final-plan.md",
+      "version": 1,
+      "validation_result": "pending",
+      "timestamp": ""
+    }
+  },
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
diff --git a/workflow/storyboard-commit-export.md b/workflow/storyboard-commit-export.md
new file mode 100644
index 0000000..efd155e
--- /dev/null
+++ b/workflow/storyboard-commit-export.md
@@ -0,0 +1,22 @@
+# Storyboard Commit and Export Workflow
+
+Commit or re-export a selected generated, refined, or committed iteration as one validated transaction. Candidate work must never alter prior final outputs or project state before promotion succeeds.
+
+## Inputs
+
+Require status `generated`, `refined`, or `committed`. Use `generated` or `refined` for an initial commit and `committed` only for an explicit export or re-export of a selected existing iteration. Read `project.json`; the selected iteration's `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; references; `templates/plan.md`; and `templates/shot-list.csv`.
+
+## Transaction order
+
+1. **Validate source first.** Before generating output, validate the selected iteration number, all four required iteration files, version record paths, complete canonical frame records, stable frame IDs, prompts, references, and an existing image reference. Do not lock or mutate project state yet.
+2. **Build an isolated candidate.** Request the high-resolution output, warning when regeneration may drift. Place the returned high-resolution file, or its candidate image-reference metadata when the capability returns only a reference, together with candidate `final-plan.md`, `shot-list.csv`, and `manifest.json` under `iterations/v###/commit-candidate/`, never directly under `final/`.
+3. **Validate the complete candidate.** Parse candidate JSON; verify candidate CSV header order, 45-column rows, and stable frame IDs; verify Markdown frame identifiers; validate project lifecycle status values and shot-list enum values; verify every output path and image reference exists. Allowed shot-list values are:
+   - `status`: `proposed`, `approved`, `assigned`, `captured`, `completed`, `omitted`
+   - `reference_type`: `internal`, `external`, `uploaded`, `AI-generated`, `none`
+   - `capture_type`: `must-capture`, `inspiration`, `optional`, `alternate`
+4. **Promote as one set.** Only after every candidate validation passes, promote the candidate files to `final/` as a directory-level set. Preserve the prior `final/` as rollback data until promotion and manifest update both complete.
+5. **Commit state last.** After successful promotion, append export records containing `artifact_type`, `artifact_path`, `version`, `validation_result`, and `timestamp`; set `active_version` to the selected integer version; set status to `committed` for an initial commit or keep status `committed` for a re-export; and report success.
+
+## Failure behavior
+
+On source, generation, candidate validation, promotion, or manifest-update failure, report the exact failure, leave `commit-candidate/` isolated for inspection, restore any promotion rollback, and keep prior `final/`, `project.json.exports`, `active_version`, and status unchanged. Never report commit or export success without the promoted validated files and committed manifest state.
diff --git a/workflow/storyboard-generation.md b/workflow/storyboard-generation.md
new file mode 100644
index 0000000..73115ab
--- /dev/null
+++ b/workflow/storyboard-generation.md
@@ -0,0 +1,30 @@
+# Storyboard Generation Workflow
+
+Create an immutable first or subsequent generated iteration from an approved or explicitly review-bypassed plan.
+
+## Inputs
+
+Read `project.json`, the selected `plans/plan-v###.md`, all canonical frame records, style guidance, constants, negative constraints, aspect ratio, and references. Require `review` with approval, or `planned` with a recorded explicit review bypass.
+
+## Four-file iteration contract
+
+Initial generation writes exactly these required artifacts under `iterations/v001/`:
+
+- `iterations/v001/prompt-packet.md`
+- `iterations/v001/frames.json`
+- `iterations/v001/result.json`
+- `iterations/v001/changes.md`
+
+Every later generation uses the same names under its next immutable `iterations/v###/` folder.
+
+## Procedure
+
+1. Build `prompt-packet.md` from the approved plan, constants, negative constraints, selected style, aspect ratio, references, and numbered frame prompts.
+2. Request a coherent numbered low-resolution contact sheet through ChatGPT image generation.
+3. After a returned image reference exists, write `frames.json` as the complete array of canonical frame records, including image paths; write `result.json` with the contact-sheet reference, settings, validation result, and timestamp; and write `changes.md` with the generation scope and summary.
+4. Validate all four files and their frame IDs before mutating project state.
+5. Copy the complete frame array to `project.json.frames`; append a version record containing `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, `created_at`, and `change_summary`; set integer `active_version`; then set status to `generated`.
+
+## Failure behavior
+
+If image generation or artifact validation fails, do not update `project.json.frames`, `project.json.versions`, `active_version`, or status. Do not claim generation success until the image reference and all four files exist and validate.
diff --git a/workflow/storyboard-intake.md b/workflow/storyboard-intake.md
new file mode 100644
index 0000000..168f835
--- /dev/null
+++ b/workflow/storyboard-intake.md
@@ -0,0 +1,25 @@
+# Storyboard Intake Workflow
+
+Normalize supplied information into a portable project without repeating questions that the brief already answers.
+
+## Inputs
+
+Read the supplied story, shoot, scene, style, references, duration, frame count, aspect ratio, and production constraints. Read `templates/project.json` and `templates/project-readme.md`.
+
+## Procedure
+
+1. Extract supplied facts first and ask only about missing facts that materially alter story, delivery, rights, or production constraints.
+2. Mark derived values as `inferred` and keep them distinct from supplied or approved values.
+3. Create a unique filesystem-safe project slug.
+4. Copy the templates to `storyboard-projects/<project-slug>/project.json` and `README.md` without overwriting an existing project.
+5. Create project-local `guides/`, `references/`, `plans/`, `iterations/`, `final/`, and `exports/` directories.
+6. Initialize `frames`, `versions`, and `exports` as empty arrays, `active_version` as integer `0`, and `status` as `proposed`.
+7. Record normalized intake fields and stop before planning.
+
+## Files and outputs
+
+May create only the new project manifest, README, and directories during intake. Produce a normalized summary with slug, supplied facts, inferred values, unresolved material gaps, and project path.
+
+## Stop conditions and invariants
+
+Stop when meaningful story intent is absent, a slug collides, or a material gap requires user choice. Preserve supplied facts verbatim. Do not create `plans/plan-v001.md`, iteration artifacts, or images during intake.
diff --git a/workflow/storyboard-planning.md b/workflow/storyboard-planning.md
new file mode 100644
index 0000000..f4c2f82
--- /dev/null
+++ b/workflow/storyboard-planning.md
@@ -0,0 +1,23 @@
+# Storyboard Planning Workflow
+
+Turn a `proposed` project into a reviewable, image-free production plan.
+
+## Inputs
+
+Read `project.json`, project-local and plugin guides, `templates/plan.md`, reference metadata, and the latest explicit user instruction. Apply the shared context precedence from `workflow/storyboard.md`.
+
+## Procedure
+
+1. Require status `proposed` for initial planning or `planned` for further plan edits. Plan edits returned from review also enter this workflow as `planned`.
+2. Load applicable story, shoot, scene, and style guides; project-local guidance overrides starter guidance.
+3. Use six frames for a general concept and eight for the soccer benchmark unless an approved frame count overrides it.
+4. Assign stable `frame_id` values such as `frame-001` and use `sequence_number` only for order.
+5. Populate every frame record with `frame_id`, `sequence_number`, `title`, `description`, `prompt`, `negative_prompt`, `metadata`, `continuity_locks`, `references`, and `image_paths`.
+6. Write the next `plans/plan-v###.md` with project summary, creative constants, assumptions, progression, frame plan, prompts, and negative constraints.
+7. Update `project.json.frames`, keep approved values distinct from inferred values, and set `status` to `planned` only after the plan validates.
+
+Planning does not create an iteration. The plan path is captured in the version record when generation creates that iteration.
+
+## Outputs
+
+Produce a validated plan and canonical manifest frame records for review. Stop for material ambiguity, lock conflicts, or guide conflicts that require user choice. Never create images or iteration files.
diff --git a/workflow/storyboard-references.md b/workflow/storyboard-references.md
new file mode 100644
index 0000000..3167caa
--- /dev/null
+++ b/workflow/storyboard-references.md
@@ -0,0 +1,15 @@
+# Storyboard References Workflow
+
+Record reference provenance, permitted influence, and frame scope before another stage relies on the material.
+
+## Inputs and procedure
+
+Read supplied material, `project.json`, applicable frame IDs, and project-local reference metadata. Support `internal`, `external`, `uploaded`, `AI-generated`, and `none` types. Store reference ID, type, source/path/URL, rights note, applicable frames, what to borrow, and what not to copy.
+
+Treat uploaded material as unscoped until the user identifies applicable frames. Use `none` when no reference is supplied. Record rights uncertainty without inferring permission.
+
+## Files and outputs
+
+May create `references/<reference-id>.md` and update the matching `project.json.references` record. When an iteration uses a reference, copy its stable reference ID into affected `frames.json` records; do not duplicate or broaden scope.
+
+Stop when rights, source, permitted use, frame scope, or requested borrowing is unresolved. Do not assume an uploaded reference applies to the whole project.
diff --git a/workflow/storyboard-refinement.md b/workflow/storyboard-refinement.md
new file mode 100644
index 0000000..1f5aa5b
--- /dev/null
+++ b/workflow/storyboard-refinement.md
@@ -0,0 +1,28 @@
+# Storyboard Refinement Workflow
+
+Create a new immutable iteration for scoped changes while preserving unaffected frame records, prompts, metadata, references, and image paths.
+
+## Inputs and required outputs
+
+Read `project.json` and the active iteration's exact `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Require status `generated` or `refined`. Every new `iterations/v###/` refinement must write the same four files:
+
+- `prompt-packet.md`
+- `frames.json`
+- `result.json`
+- `changes.md`
+
+## Procedure
+
+1. Identify affected frame IDs, distinguish frame-only changes from project-constant changes, and list continuity risks.
+2. Copy the active iteration into the next version as working data. Change only requested frame fields and keep every unaffected frame record and prompt byte-for-byte equivalent where the file format permits.
+3. If image capability supports frame-level editing, regenerate only requested frame assets. Recompose the contact sheet only when the capability also provides that operation.
+4. If frame-level editing is unsupported and no compositor is available, do not silently regenerate the full contact sheet. Ask the user to choose exactly one fallback:
+   - **(a) Whole contact sheet:** regenerate the whole contact sheet with unchanged frame prompts locked, with an explicit continuity/drift warning.
+   - **(b) Frame-only artifact:** create a frame-only revision artifact while retaining the previous contact sheet.
+   - **(c) Metadata only:** update prompts/metadata only with no new image.
+5. Record the selected capability path or fallback in both `changes.md` and `result.json`, including affected and untouched frame IDs, continuity warnings, prior image references retained, and every new image path.
+6. Validate `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; then update `project.json.frames`, append the canonical version record, increment integer `active_version`, and set `generated -> refined` or retain `refined` for later refinements.
+
+## Stop conditions and invariants
+
+Stop for lock conflicts, unresolved reference rights, unclear scope, or a missing explicit fallback choice. Never overwrite an iteration or silently regenerate the full contact sheet. On any failure, leave the prior active iteration and project state unchanged.
diff --git a/workflow/storyboard-review.md b/workflow/storyboard-review.md
new file mode 100644
index 0000000..cd0bc39
--- /dev/null
+++ b/workflow/storyboard-review.md
@@ -0,0 +1,19 @@
+# Storyboard Review Workflow
+
+Make review the default gate between a planned storyboard and image generation.
+
+## Inputs
+
+Read `project.json`, the active `plans/plan-v###.md`, project-local overrides, reference warnings, and the latest explicit user instruction. Require status `planned` or `review`.
+
+## Procedure
+
+1. Present the summary, constants, ordered stable frame IDs, approved values, inferred assumptions, warnings, and conflicts.
+2. When entering from `planned`, set `planned -> review` after the complete review presentation exists. When re-entering from `review`, keep status `review` and show the current unresolved decisions without replaying the transition.
+3. Support approval; overall edits; edit/add/remove/reorder frame; alternatives; `skip review`; and proceed to creation.
+4. Any plan edit from `review` writes the revised plan and frame records, sets `review -> planned`, and returns to planning/review unless the user explicitly bypasses review.
+5. Record approval or explicit bypass without marking the project generated. Generation sets `generated` only after its four-file iteration contract and image result exist.
+
+## Outputs
+
+Produce an explicit review decision, changed frame IDs, remaining warnings, and a generation authorization or revised plan. Stop for unresolved decisions, lock conflicts, or alternatives requiring a choice. Never generate images or iteration artifacts.
diff --git a/workflow/storyboard-versioning.md b/workflow/storyboard-versioning.md
new file mode 100644
index 0000000..11c3b06
--- /dev/null
+++ b/workflow/storyboard-versioning.md
@@ -0,0 +1,20 @@
+# Storyboard Versioning Workflow
+
+Inspect, restore, or branch immutable iteration history without deleting or overwriting prior snapshots.
+
+## Inputs
+
+Read `project.json` and each iteration's exact `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Refuse restore or branch operations when any selected source artifact is missing or invalid.
+
+## Procedure
+
+- Listing or comparing versions reads the four-file contracts and does not mutate state.
+- A whole-version restore creates a new `iterations/v###/` snapshot from the selected source.
+- A one-frame restore copies the prior canonical frame record into a new iteration while preserving all unaffected records.
+- A branch from any version, including a named committed project, creates a new active iteration and lineage without deleting history.
+
+Every restore or branch writes the same four required files in the destination: `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. `changes.md` and `result.json` record source version, operation, affected frames, retained image references, and validation outcome.
+
+After all four files validate, copy destination frames to `project.json.frames`, append a canonical version record with `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, `created_at`, and `change_summary`, and update integer `active_version`. A new branch sets status to `generated`; a restore uses `generated` or `refined` according to the restored result and recorded change.
+
+Stop for missing versions, missing frame IDs, ambiguous targets, or unconfirmed replacement of the active choice. Never overwrite or delete history.
diff --git a/workflow/storyboard.md b/workflow/storyboard.md
new file mode 100644
index 0000000..0dfb91b
--- /dev/null
+++ b/workflow/storyboard.md
@@ -0,0 +1,64 @@
+# Storyboard Orchestration Reference
+
+This ordinary workflow reference defines project discovery, routing, and lifecycle transitions for the registered `storyboard` skill. It is not an independently triggerable skill.
+
+## Project discovery and manifest gate
+
+1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`.
+2. Otherwise, use the project containing the current workspace, or the sole manifest under `storyboard-projects/`.
+3. If multiple projects remain possible, ask the user to choose.
+4. If no project exists and meaningful story input was supplied, route to `storyboard-intake`.
+5. If no project exists and no meaningful story input was supplied, ask for one sentence of creative intent.
+
+For an existing project, require `status` to be exactly one of `proposed`, `planned`, `review`, `generated`, `refined`, or `committed`. Require `active_version` to be an integer. Stop with a manifest error that names the missing or invalid field; do not guess a stage or repair state silently.
+
+## Stage routes
+
+Load only the applicable ordinary reference:
+
+- `storyboard-intake`: `workflow/storyboard-intake.md`
+- `storyboard-planning`: `workflow/storyboard-planning.md`
+- `storyboard-review`: `workflow/storyboard-review.md`
+- `storyboard-generation`: `workflow/storyboard-generation.md`
+- `storyboard-refinement`: `workflow/storyboard-refinement.md`
+- `storyboard-versioning`: `workflow/storyboard-versioning.md`
+- `storyboard-references`: `workflow/storyboard-references.md`
+- `storyboard-commit-export`: `workflow/storyboard-commit-export.md`
+
+Explicit stage verbs select a route only when its input state is valid. A reference attachment may route through `storyboard-references` before the requested lifecycle stage without changing status by itself.
+
+## State transitions
+
+- Intake creates a project with `status: proposed`.
+- Planning changes `proposed -> planned` after `plans/plan-v###.md` and the manifest frame plan are complete.
+- Review changes `planned -> review` when the review presentation is ready.
+- Plan edits made from `review` return the project to `planned` and require review again unless the user explicitly bypasses review.
+- Approval or an explicit review bypass authorizes generation; generation changes `review -> generated` or `planned -> generated` only after all generation artifacts and the image result exist.
+- Refinement changes `generated -> refined`; further valid refinement changes may keep `refined -> refined` while creating a new iteration.
+- Commit changes `generated -> committed` or `refined -> committed` only after transactional candidate validation and promotion succeed.
+- An explicit export or re-export from `committed` uses the same candidate transaction and keeps `committed -> committed` after promotion.
+- A named committed project may reopen for inspection, branching, version restore, or export. Reopening alone does not change `committed`.
+- A new branch starts with a new active iteration, appends a version record, and sets the branch state to `generated` until it is refined or committed.
+
+Never update status before the files required by the destination state exist and validate.
+
+## Routing without a stage verb
+
+When the request does not contain a stage verb, route only from `project.json.status`:
+
+| Status | Route or response |
+| --- | --- |
+| `proposed` | Route to `storyboard-planning`. |
+| `planned` | Route to `storyboard-review`. |
+| `review` | Remain in `storyboard-review` and show unresolved decisions. |
+| `generated` | Show the current iteration and offer refinement or commit. |
+| `refined` | Show the current iteration and offer further refinement or commit. |
+| `committed` | Show final state and offer inspect, branch, restore, or export. |
+
+## Shared invariants
+
+- `project.json` is the source of truth; `active_version` identifies the active `iterations/v###/` folder.
+- `frame_id` is stable; only `sequence_number` changes when ordering changes.
+- Every generated, refined, restored, or branched iteration contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`.
+- Apply precedence: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, then inference.
+- Stop for lock conflicts, unresolved rights, invalid manifests, or failed validation.
