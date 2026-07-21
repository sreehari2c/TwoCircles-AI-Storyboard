diff --git a/.codex-plugin/plugin.json b/.codex-plugin/plugin.json
new file mode 100644
index 0000000..a043e37
--- /dev/null
+++ b/.codex-plugin/plugin.json
@@ -0,0 +1,33 @@
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
+  "skills": "./skills/",
+  "interface": {
+    "displayName": "Two Circles AI Storyboard",
+    "shortDescription": "Plan, review, refine, and export production storyboards.",
+    "longDescription": "A portable, file-based storyboard workflow from creative brief through image-generation handoff and production shot list.",
+    "developerName": "Two Circles",
+    "category": "Productivity",
+    "capabilities": [
+      "Interactive",
+      "Write"
+    ],
+    "defaultPrompt": [
+      "Turn this creative brief into a storyboard plan.",
+      "Review and refine my current storyboard.",
+      "Export my approved storyboard and shot list."
+    ]
+  }
+}
diff --git a/.gitignore b/.gitignore
index e458ed5..276e69c 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1 +1,2 @@
 .worktrees/
+.superpowers/
diff --git a/README.md b/README.md
index a352d70..5eb17f7 100644
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
+Only `skills/storyboard/SKILL.md` is registered. Its eight stage instructions live in `skills/storyboard/workflow/`, and its starter guides and templates live in `skills/storyboard/resources/`. These are ordinary skill-relative resources, not separately discoverable skills.
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
index 0000000..8d97d4f
--- /dev/null
+++ b/docs/storyboard-acceptance-runbook.md
@@ -0,0 +1,86 @@
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
+Confirm the installed package exposes only `skills/storyboard/SKILL.md`; its eight stage references are under `skills/storyboard/workflow/`, and starter files are under `skills/storyboard/resources/`.
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
+Expected evidence: this is the first action that creates `plans/plan-v001.md`. It contains exactly eight identifiable frame records, each with a stable frame ID such as `frame-001` and explicit lens, movement, frame-rate, and shot-size values or intentional defaults; project status is `planned`, and review is `pending` with authorization false.
+
+### 3. Review the plan and verify assumptions
+
+Review every frame before generation. Check that approved values are distinguishable from inferred assumptions. Resolve or accept assumptions before proceeding.
+
+Expected evidence: the reviewed `plans/plan-v001.md`, labeled assumptions, and a durable `project.json.review` object. Approval records `decision: approved`; explicit bypass records `decision: bypassed`. Both record `plan_version: 1`, a non-null `reviewed_at`, `authorized_for_generation: true`, and a `change_summary` while status is `review`. Plan edits reset the five fields to pending/unauthorized values and return status to `planned`.
+
+### 4. Generate the contact sheet
+
+Request a numbered, low-resolution contact sheet through ChatGPT image generation using the approved eight-frame plan.
+
+Expected evidence: initial generation creates all four files: `iterations/v001/prompt-packet.md`, `iterations/v001/frames.json`, `iterations/v001/result.json`, and `iterations/v001/changes.md`. `result.json` contains the numbered contact-sheet reference, while `project.json.frames`, `project.json.versions`, integer `active_version`, and status `generated` agree with v001. The v001 version record contains the same five review fields that authorized generation.
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
+Expected evidence: source validation occurs first. Candidate `final-plan.md`, `shot-list.csv`, image/image-reference metadata, and a prospective `manifest.json` with status `committed`, selected `active_version`, and appended export records are built under `iterations/v###/commit-candidate/`. Validate the candidate plan, all 45 CSV columns, image, and manifest together. Only then promote the set and write the same candidate manifest bytes to both `project.json` and `final/manifest.json`; verify those files are byte-for-byte identical.
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
+- Clear or corrupt durable review authorization and verify a generate request routes back to review without creating an iteration.
+- While status is `review`, request an add/remove/reorder/edit-plan change and verify the review workflow handles it, resets authorization, and returns status to `planned`.
+
+Record pass/fail results, observed artifact paths, and deviations before accepting the task.
diff --git a/docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md b/docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md
index 0f39d8c..0046c19 100644
--- a/docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md
+++ b/docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md
@@ -1,548 +1,173 @@
 # AI Storyboard Codex Plugin Foundation Implementation Plan
 
-> For agentic workers: REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox syntax for tracking.
+> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.
 
-**Goal:** Build a portable, no-code Codex plugin with one smart storyboard flow, focused internal skills, starter guide defaults, file-based project templates, and a manual acceptance runbook for ChatGPT image-generation handoff.
+**Goal:** Deliver a portable, no-code storyboard plugin with one registered skill, eight private stage workflows, durable review authorization, immutable iterations, and state-consistent final exports.
 
-**Architecture:** The repository becomes an installable plugin package. The plugin manifest declares the package and points to the skills directory. The public storyboard skill routes user intent and project state to focused internal skills. Markdown skills, JSON/CSV/Markdown templates, and project-local files provide the complete runtime behavior without a server, database, or custom code.
+**Architecture:** `.codex-plugin/plugin.json` declares the standard `./skills/` registry root, which contains only the public `storyboard` skill. That `SKILL.md` loads stage Markdown and starter resources relative to its own directory; nested supporting files are not discoverable skills. Durable projects remain at plugin-root `storyboard-projects/<project-slug>/`, and all lifecycle decisions live in `project.json` rather than chat history.
 
-**Tech Stack:** Codex plugin manifest JSON, Markdown SKILL.md instructions, JSON project manifests, Markdown production plans, CSV shot lists, PowerShell read-only validation commands, and ChatGPT image-generation handoff.
+**Tech Stack:** Codex plugin manifest JSON, one Markdown skill, non-discoverable Markdown workflow/reference files, JSON project manifests, Markdown plans, CSV shot lists, and ChatGPT image-generation handoff. No custom code or runtime.
 
 ## Global Constraints
 
-- The plugin is pure instructions plus Markdown/JSON/CSV files; it has no hosted API, custom runtime, database, or image-processing implementation.
-- The user interacts through one top-level storyboard flow; internal skills are selected automatically from intent and project state.
-- Projects are stored under storyboard-projects/<project-slug>/ inside the plugin repository.
-- Review is enabled by default; skip review, generate immediately, and use your defaults bypass review while retaining the internal plan.
-- Story information is mandatory; shoot and scene information may be inferred and must be labeled as inferred.
-- Stable frame IDs use the format frame-001; sequence numbers may change when frames are reordered.
-- Context precedence is latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, then system inference.
-- Starter guides are provisional, replaceable, and must not be presented as official Two Circles guidance.
-- General concepts default to six frames; the soccer benchmark uses eight frames.
-- Starter shoot defaults are 16:9 delivery, 23.976 fps, 18/25/35/50/85/100mm lenses, tripod for static shots, handheld for action/intimacy, gimbal or dolly for controlled movement, and 120 fps only when slow motion serves the story.
-- The default low-resolution image artifact is a numbered contact sheet; separate frame images are recorded when generated or supplied.
-- Approved iterations are never overwritten. Failed generations and invalid exports leave the last valid state intact.
-- Commit output includes a final storyboard image reference, Markdown production plan, expanded-schema CSV shot list, final manifest snapshot, and generation metadata.
+- Only `skills/storyboard/SKILL.md` is registered or discoverable.
+- Exactly eight stage files live under `skills/storyboard/workflow/`.
+- Starter guides and templates live under `skills/storyboard/resources/`; root `workflow/`, `guides/`, and `templates/` directories do not exist.
+- Projects remain under plugin-root `storyboard-projects/<project-slug>/`.
+- The six status values are `proposed`, `planned`, `review`, `generated`, `refined`, and `committed`.
+- Review decisions are `pending`, `approved`, and `bypassed`; generation requires durable authorization for the selected plan.
+- Every generated/refined/restored/branched iteration has `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`.
+- Every version record snapshots the five review fields.
+- Commit/export validates candidate plan, 45-column CSV, image, and prospective committed manifest before state changes.
+- A failed generation or export leaves prior project and final state unchanged.
 
 ---
 
 ## File map and responsibilities
 
 | Path | Responsibility |
-|---|---|
-| .codex-plugin/plugin.json | Plugin identity, version, metadata, and skills directory declaration. |
-| README.md | Installation/use overview and the top-level command contract. |
-| skills/storyboard/SKILL.md | User-facing routing, project discovery, lifecycle, and handoff rules. |
-| skills/storyboard-*/SKILL.md | One focused internal capability per stage. |
-| guides/*.md | Provisional story, shoot, scene, and style defaults. |
-| templates/project.json | Minimal canonical manifest shape and field examples. |
-| templates/project-readme.md | Per-project operating instructions and lifecycle summary. |
-| templates/plan.md | Machine-readable Markdown plan structure. |
-| templates/shot-list.csv | Exact CSV header contract from the product specification. |
-| storyboard-projects/.gitkeep | Keeps the portable project root present in git. |
-| docs/storyboard-acceptance-runbook.md | Manual eight-frame soccer acceptance procedure and checks. |
-
-This is one plan because all tasks produce one installable plugin package and each task can be validated by inspecting its own files plus the package contract. No hosted subsystem is being implemented.
-
-## Task 1: Scaffold the plugin package
+| --- | --- |
+| `.codex-plugin/plugin.json` | Plugin identity and standard `./skills/` root containing one public skill. |
+| `README.md` | Public package layout, portable store, and no-code scope. |
+| `skills/storyboard/SKILL.md` | The sole public skill: discovery, routing, shared state fallback, response contract. |
+| `skills/storyboard/workflow/*.md` | Eight private stage contracts loaded by the public skill. |
+| `skills/storyboard/resources/guides/` | Seven provisional starter guide files. |
+| `skills/storyboard/resources/templates/project.json` | Canonical manifest, durable review, frame/version/export contracts. |
+| `skills/storyboard/resources/templates/project-readme.md` | Project lifecycle and record semantics. |
+| `skills/storyboard/resources/templates/plan.md` | Machine-readable storyboard plan shape. |
+| `skills/storyboard/resources/templates/shot-list.csv` | Exact 45-column CSV header. |
+| `storyboard-projects/.gitkeep` | Portable project-store root. |
+| `docs/storyboard-acceptance-runbook.md` | Manual lifecycle and failure evidence. |
+
+## Task 1: Consolidate the registered skill package
 
 **Files:**
-- Create: .codex-plugin/plugin.json
-- Modify: README.md
-- Create: storyboard-projects/.gitkeep
 
-**Interfaces:**
-- Consumes: The approved design document at docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md.
-- Produces: A valid plugin manifest whose skills property points to ./skills/, plus a repository README that names storyboard as the public workflow.
-
-- [ ] Step 1: Write the manifest
-
-Create .codex-plugin/plugin.json with this content:
-
-    {
-      "name": "two-circles-ai-storyboard",
-      "version": "0.1.0",
-      "description": "A guided AI storyboard workflow for turning creative briefs into production-ready visual plans and shot lists.",
-      "author": {
-        "name": "Two Circles"
-      },
-      "license": "Proprietary",
-      "keywords": [
-        "storyboard",
-        "creative planning",
-        "cinematography",
-        "shot list",
-        "image generation"
-      ],
-      "skills": "./skills/"
-    }
-
-- [ ] Step 2: Update the README
-
-Replace the current README with a concise package overview containing:
-
-    # Two Circles AI Storyboard
-
-    A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
-
-    ## Start
-
-    Use the top-level storyboard workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It will discover the project state, ask only high-impact questions, create a plan, offer review, and hand approved prompts to ChatGPT image generation.
-
-    ## Project storage
-
-    Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.
-
-    ## Scope
-
-    This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.
-
-- [ ] Step 3: Keep the project root in git
-
-Create an empty storyboard-projects/.gitkeep file. Do not create a sample project yet; project creation is handled by the top-level flow from templates/.
-
-- [ ] Step 4: Validate the package scaffold
-
-Run from the repository root:
-
-    Get-Content .codex-plugin/plugin.json -Raw | ConvertFrom-Json | Out-Null
-    if (-not (Test-Path .codex-plugin/plugin.json)) { throw 'Missing plugin manifest' }
-    if (-not (Test-Path storyboard-projects/.gitkeep)) { throw 'Missing project root marker' }
-
-Expected: the command completes without an exception and produces no validation error.
-
-- [ ] Step 5: Commit
-
-    git add .codex-plugin/plugin.json README.md storyboard-projects/.gitkeep
-    git commit -m "feat: scaffold storyboard plugin package"
-
-## Task 2: Add provisional guides with usable defaults
-
-**Files:**
-- Create: guides/story-guide.md
-- Create: guides/shoot-guide.md
-- Create: guides/scene-guide.md
-- Create: guides/styles/cinematic-realism.md
-- Create: guides/styles/documentary-sports.md
-- Create: guides/styles/graphic-storyboard-sketch.md
-- Create: guides/styles/clean-pitch-frame.md
+- Modify: `.codex-plugin/plugin.json`
+- Modify: `README.md`
+- Modify: `skills/storyboard/SKILL.md`
+- Move: `workflow/*.md` to `skills/storyboard/workflow/*.md`
+- Move: `guides/` to `skills/storyboard/resources/guides/`
+- Move: `templates/` to `skills/storyboard/resources/templates/`
 
 **Interfaces:**
-- Consumes: The guide rules in the design document and Global Constraints.
-- Produces: Replaceable guide files that internal skills can load directly and project-local guides/ files can override.
-
-- [ ] Step 1: Write the story guide
-
-Create guides/story-guide.md with these required sections and decisions:
-
-- Purpose: convert incomplete creative intent into a visual sequence.
-- Provisional status: clearly state that the guide is a starter default, not official Two Circles guidance.
-- Default frame count: six frames for a general short concept; infer another count when duration, narrative beats, or event coverage require it.
-- Sequence grammar: establish, introduce subject/action, detail, reaction/emotion, progression/escalation, resolution/call to action.
-- Question policy: ask only questions that materially change story, subject, location, action, duration, or delivery.
-- Coverage rules: vary shot size and camera language while avoiding redundant frames.
-- Duration rules: keep duration optional for non-linear event coverage and record it when supplied or inferred.
-- Inference labels: every inferred story value is marked inferred until approved.
-
-- [ ] Step 2: Write the shoot guide
-
-Create guides/shoot-guide.md with the exact defaults from Global Constraints, plus plain-language recommendations for choosing lens, support, frame rate, movement, and capture type. Include a rule that creative descriptions should be preferred over unnecessary exposure or shutter technicalities.
-
-- [ ] Step 3: Write the scene guide
-
-Create guides/scene-guide.md with rules for inferring and locking location, time of day, lighting direction, weather, crowd level, background depth, wardrobe, props, signage, and atmosphere. Include a continuity checklist applied to every frame.
-
-- [ ] Step 4: Write four style placeholders
-
-Each style file must contain Status: Provisional starter placeholder, Rendering method, Contrast, Saturation, Lighting, Texture, Typical lenses, Camera movement, Framing tendencies, Subject treatment, Continuity locks, and Prohibited traits.
-
-Use these starting distinctions:
 
-- cinematic-realism.md: photorealistic, premium commercial realism, controlled contrast, intentional depth of field, executable dynamic camera positions.
-- documentary-sports.md: natural light, observational framing, handheld energy, authentic expressions, less polished composition.
-- graphic-storyboard-sketch.md: restrained monochrome or limited color, hand-drawn appearance, clear blocking, strong silhouettes.
-- clean-pitch-frame.md: polished presentation composition, controlled backgrounds, strong visual hierarchy, client-review readability.
+- Consumes: plugin-root project paths and the existing stage contracts.
+- Produces: one registered skill with skill-relative private resources.
 
-Do not include official logos, proprietary claims, or unverified Two Circles brand rules.
+- [ ] Move every stage, guide, and template to the canonical skill-relative destination and remove root duplicates.
+- [ ] Keep exactly eight stage workflow files; orchestration belongs in the public `SKILL.md`.
+- [ ] Resolve `workflow/...` and `resources/...` relative to `skills/storyboard/` in every runtime instruction.
+- [ ] Keep `storyboard-projects/<project-slug>/` paths relative to the plugin root.
+- [ ] Verify every path named by the public skill exists.
 
-- [ ] Step 5: Validate guide completeness
-
-Run:
-
-    $guideFiles = Get-ChildItem guides -Recurse -Filter *.md
-    if ($guideFiles.Count -ne 7) { throw "Expected 7 guide files, found $($guideFiles.Count)" }
-    Select-String -Path $guideFiles.FullName -Pattern 'Provisional|Status' | Out-Null
-
-Expected: seven Markdown guide files are found and every style file contains a provisional-status marker.
-
-- [ ] Step 6: Commit
-
-    git add guides
-    git commit -m "feat: add provisional storyboard guides"
-
-## Task 3: Add project and export templates
+## Task 2: Make review authorization durable
 
 **Files:**
-- Create: templates/project.json
-- Create: templates/project-readme.md
-- Create: templates/plan.md
-- Create: templates/shot-list.csv
-
-**Interfaces:**
-- Consumes: The canonical project model, expanded CSV schema, and lifecycle from the design document.
-- Produces: Templates used by intake, planning, review, and commit/export skills. The frame_id and sequence_number fields are the shared interface between JSON, Markdown, and CSV.
-
-- [ ] Step 1: Write the manifest template
-
-Create templates/project.json as valid JSON with these top-level keys and representative empty values:
-
-    {
-      "project_id": "project-slug",
-      "title": "Untitled Storyboard",
-      "schema_version": "0.1.0",
-      "status": "proposed",
-      "active_version": 0,
-      "story": {},
-      "shoot": {},
-      "scene": {},
-      "style": {},
-      "creative_constants": {
-        "locks": [],
-        "prohibited_elements": []
-      },
-      "references": [],
-      "frames": [],
-      "versions": [],
-      "exports": []
-    }
-
-- [ ] Step 2: Write the project README template
-
-Create templates/project-readme.md explaining that project.json is the source of truth, listing the folder meanings, stating that frame_id is stable, and documenting the lifecycle intake -> plan -> review -> generate -> refine/version -> commit/export.
-
-- [ ] Step 3: Write the plan template
-
-Create templates/plan.md with these exact headings:
-
-    # [Project Title] Storyboard Plan
-
-    ## Project Summary
-    ## Creative Constants
-    ## Production Assumptions
-    ## Story Progression
-    ## Frame Plan
 
-    ### Frame 001 — [Frame Title]
-    - Frame ID: frame-001
-    - Purpose:
-    - Description:
-    - Subject:
-    - Action:
-    - Shot size:
-    - Camera angle:
-    - Camera position:
-    - Lens:
-    - Depth of field:
-    - Movement:
-    - Location:
-    - Lighting:
-    - Duration:
-    - Transition:
-    - Continuity locks:
-    - Reference source:
-    - Prompt:
-    - Negative constraints:
-
-The planning skill will duplicate the Frame section for every stable frame ID and use Frame 03 — Title style headings in final plans while retaining the machine-readable Frame ID field.
-
-- [ ] Step 4: Write the exact CSV header
-
-Create templates/shot-list.csv with this single header row, preserving column order:
-
-    project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at
-
-- [ ] Step 5: Validate template contracts
-
-Run:
-
-    Get-Content templates/project.json -Raw | ConvertFrom-Json | Out-Null
-    $header = (Get-Content templates/shot-list.csv -First 1).Split(',')
-    if ($header.Count -ne 45) { throw "Expected 45 CSV columns, found $($header.Count)" }
-    if ($header[3] -ne 'frame_id' -or $header[4] -ne 'sequence_number') { throw 'Frame identity columns are not in the required positions' }
-
-Expected: JSON parses successfully and the CSV has 45 columns with frame_id before sequence_number.
-
-- [ ] Step 6: Commit
-
-    git add templates
-    git commit -m "feat: add storyboard project and export templates"
-
-## Task 4: Write the internal stage skills
-
-**Files:**
-- Create: skills/storyboard-intake/SKILL.md
-- Create: skills/storyboard-planning/SKILL.md
-- Create: skills/storyboard-review/SKILL.md
-- Create: skills/storyboard-generation/SKILL.md
-- Create: skills/storyboard-refinement/SKILL.md
-- Create: skills/storyboard-versioning/SKILL.md
-- Create: skills/storyboard-references/SKILL.md
-- Create: skills/storyboard-commit-export/SKILL.md
+- Modify: `skills/storyboard/resources/templates/project.json`
+- Modify: `skills/storyboard/workflow/storyboard-planning.md`
+- Modify: `skills/storyboard/workflow/storyboard-review.md`
+- Modify: `skills/storyboard/workflow/storyboard-generation.md`
+- Modify: `skills/storyboard/workflow/storyboard-refinement.md`
+- Modify: `skills/storyboard/workflow/storyboard-versioning.md`
+- Modify: `skills/storyboard/SKILL.md`
 
 **Interfaces:**
-- Consumes: The project manifest and templates from Task 3, and starter/project-local guides from Task 2.
-- Produces: Focused instruction boundaries that the top-level skill can load and execute without exposing separate user-facing commands.
-
-- [ ] Step 1: Define shared skill conventions
-
-Every file must begin with YAML frontmatter containing a unique lowercase name and a one-sentence description. Every file must state:
-
-- Inputs it reads.
-- Files it may create or update.
-- Outputs it produces for the next stage.
-- Conditions under which it must stop and ask the user.
-- Invariants it must preserve.
-
-- [ ] Step 2: Write intake skill behavior
 
-skills/storyboard-intake/SKILL.md must extract supplied story, shoot, scene, style, reference, duration, frame-count, aspect-ratio, and production constraints; avoid asking for supplied facts; mark inferred values; create a project slug; copy the project and README templates; and stop with a normalized intake summary before planning.
+- Consumes: selected `plans/plan-v###.md` and current manifest status.
+- Produces: a five-field review object and review snapshot on every version record.
 
-- [ ] Step 3: Write planning skill behavior
+- [ ] Add `decision`, `plan_version`, `authorized_for_generation`, `reviewed_at`, and `change_summary` to the manifest review object and version contract.
+- [ ] Reset review to pending/unauthorized on initial planning and every plan edit.
+- [ ] Record approval or explicit bypass with selected plan version and timestamp while status remains `review`.
+- [ ] Route review-state add/remove/reorder/edit-plan through the review workflow; after editing, return status to `planned`.
+- [ ] Require approved/bypassed durable authorization for the selected plan before generation; otherwise route to review.
+- [ ] Route a no-verb `review` state to generation only when authorization is valid.
 
-skills/storyboard-planning/SKILL.md must load the applicable guides, create stable frame IDs, produce the canonical plan, populate creative constants, separate assumptions from approved values, write plans/plan-v001.md, and update project.json without creating images.
-
-- [ ] Step 4: Write review skill behavior
-
-skills/storyboard-review/SKILL.md must show project summary, creative constants, frame sequence, assumptions, warnings, and conflicts. It must support approve, edit overall plan, edit/add/remove/reorder one frame, request alternatives, skip review, and proceed to creation. It must not generate images.
-
-- [ ] Step 5: Write generation skill behavior
-
-skills/storyboard-generation/SKILL.md must build a prompt packet containing the approved plan, constants, negative constraints, style, aspect ratio, references, and frame prompts; request the numbered low-resolution contact sheet through ChatGPT image generation; record the exact packet and returned image reference under iterations/v001/; and update project.json only after the image result is available.
-
-- [ ] Step 6: Write refinement skill behavior
-
-skills/storyboard-refinement/SKILL.md must identify affected frame IDs, classify whether a project constant is affected, list continuity risks, preserve unaffected records, create the next iteration, and write changes.md. It must support one-frame, multi-frame, reference replacement, same-prompt regeneration, and sequence-ending replacement requests.
-
-- [ ] Step 7: Write versioning skill behavior
-
-skills/storyboard-versioning/SKILL.md must list iteration folders, show change summaries, restore a whole version by creating a new branch snapshot, restore one frame by copying its prior frame record into a new iteration, and branch from an existing version without deleting history.
-
-- [ ] Step 8: Write references skill behavior
-
-skills/storyboard-references/SKILL.md must support internal, external, uploaded, AI-generated, and none reference types; store source/path/URL, rights note, applicable frames, what to borrow, and what not to copy; and avoid assuming an uploaded reference applies to the whole project.
-
-- [ ] Step 9: Write commit/export skill behavior
-
-skills/storyboard-commit-export/SKILL.md must lock the selected iteration, request high-resolution output while warning about regeneration differences when deterministic upscaling is unavailable, write final/final-plan.md, final/shot-list.csv, and final/manifest.json, record export metadata, and validate required fields before reporting success.
-
-- [ ] Step 10: Validate internal skill boundaries
-
-Run:
-
-    $skillFiles = Get-ChildItem skills -Recurse -Filter SKILL.md | Where-Object { $_.FullName -notlike '*skills\storyboard\SKILL.md' }
-    if ($skillFiles.Count -ne 8) { throw "Expected 8 internal skill files, found $($skillFiles.Count)" }
-    foreach ($file in $skillFiles) {
-      $content = Get-Content $file.FullName -Raw
-      if ($content -notmatch '(?m)^name:') { throw "Missing name frontmatter in $($file.FullName)" }
-      if ($content -notmatch 'Inputs|Reads') { throw "Missing input contract in $($file.FullName)" }
-      if ($content -notmatch 'Outputs|Produces') { throw "Missing output contract in $($file.FullName)" }
-    }
-
-Expected: eight internal skill files are found, and every file exposes name, input, and output contracts.
-
-- [ ] Step 11: Commit
-
-    git add skills/storyboard-intake skills/storyboard-planning skills/storyboard-review skills/storyboard-generation skills/storyboard-refinement skills/storyboard-versioning skills/storyboard-references skills/storyboard-commit-export
-    git commit -m "feat: add storyboard stage skills"
-
-## Task 5: Write the top-level smart storyboard flow
+## Task 3: Complete the canonical frame/export contract
 
 **Files:**
-- Create: skills/storyboard/SKILL.md
-
-**Interfaces:**
-- Consumes: The eight internal skills from Task 4, templates from Task 3, guides from Task 2, and project folders under storyboard-projects/.
-- Produces: The public storyboard command contract and routing behavior used by all later manual tests.
-
-- [ ] Step 1: Define the public skill metadata
-
-Start the file with this frontmatter:
-
-    ---
-    name: storyboard
-    description: Turn a creative brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
-    ---
-
-- [ ] Step 2: Define project discovery
-
-Document the exact discovery order:
-
-1. If the user names a project, resolve storyboard-projects/<project-slug>/project.json.
-2. If the current workspace has one active storyboard project, use it.
-3. If there are multiple projects and none is named, ask the user to choose.
-4. If there is no project and the user supplied meaningful story input, create one from templates.
-5. If there is no project and no meaningful story input, ask for one sentence of creative intent.
-
-- [ ] Step 3: Define routing and defaults
-
-Include the routing table from the design document, the lifecycle, the context precedence, default review behavior, six-frame general default, eight-frame soccer benchmark exception, and the rule to load only the relevant internal skill and guide files.
-
-- [ ] Step 4: Define user-visible responses
-
-Require concise, structured responses that show current stage, changed files, assumptions, warnings, frame IDs, and next action. For image generation, show the exact frame prompts or a readable prompt summary. For refinement, show the change summary and untouched frame IDs.
-
-- [ ] Step 5: Define stop conditions
-
-The flow must stop for explicit user choice when a lock would change, a reference rights issue is unresolved, a high-impact ambiguity cannot be resolved with a recommendation, or commit validation fails. It must never claim image generation, export, or commit success without the corresponding file/reference being present.
-
-- [ ] Step 6: Validate top-level routing references
 
-Run:
-
-    $content = Get-Content skills/storyboard/SKILL.md -Raw
-    foreach ($name in @('storyboard-intake','storyboard-planning','storyboard-review','storyboard-generation','storyboard-refinement','storyboard-versioning','storyboard-references','storyboard-commit-export')) {
-      if ($content -notmatch [regex]::Escape($name)) { throw "Top-level flow does not reference $name" }
-    }
-    foreach ($phrase in @('skip review','frame-001','storyboard-projects','project.json','ChatGPT image generation')) {
-      if ($content -notmatch [regex]::Escape($phrase)) { throw "Top-level flow is missing required contract phrase: $phrase" }
-    }
-
-Expected: all eight internal skills and all five contract phrases are referenced.
-
-- [ ] Step 7: Commit
-
-    git add skills/storyboard/SKILL.md
-    git commit -m "feat: add smart storyboard orchestrator"
-
-## Task 6: Add the manual acceptance runbook
-
-**Files:**
-- Create: docs/storyboard-acceptance-runbook.md
+- Modify: `skills/storyboard/resources/templates/project.json`
+- Modify: `skills/storyboard/resources/templates/project-readme.md`
+- Modify: `skills/storyboard/workflow/storyboard-commit-export.md`
 
 **Interfaces:**
-- Consumes: The public flow from Task 5 and the acceptance scenario from the design document.
-- Produces: A repeatable, no-code smoke test for new users and future plugin revisions.
-
-- [ ] Step 1: Document setup
-
-State that the test starts from a clean project folder, uses the supplied soccer production example, and requires access to the ChatGPT image-generation capability. The runbook must not require a hosted API or custom script.
-
-- [ ] Step 2: Document the nine acceptance actions
-
-Include the exact sequence from the design: import/describe sequence, create eight-frame plan, verify inferred labels, generate contact sheet, change only streetcar frame, replace one frame with uploaded reference, inspect history, commit and validate outputs, reopen from disk.
-
-- [ ] Step 3: Add evidence checks
-
-For each action, list the expected file or visible result: stable frame IDs, plan file, contact-sheet reference, unchanged prompts for unaffected frames, frame-scoped reference metadata, iteration change summary, final Markdown/CSV/manifest, and successful reopen without chat history.
-
-- [ ] Step 4: Add failure checks
-
-Document that a failed generation leaves the active version unchanged, a lock conflict pauses for approval, and a CSV/manifest validation failure leaves prior final outputs untouched.
-
-- [ ] Step 5: Validate the runbook references
-
-Run:
 
-    $content = Get-Content docs/storyboard-acceptance-runbook.md -Raw
-    foreach ($phrase in @('streetcar','frame-001','final-plan.md','shot-list.csv','manifest.json','without the original chat')) {
-      if ($content -notmatch [regex]::Escape($phrase)) { throw "Acceptance runbook is missing: $phrase" }
-    }
+- Consumes: canonical selected-iteration frame records.
+- Produces: one deterministic RFC 4180 row per frame using the exact 45-column header.
 
-Expected: the runbook contains all benchmark and evidence terms.
+- [ ] Add `revision_history`, image paths, and exact narrative/production metadata keys to the frame record.
+- [ ] Include all contract gaps explicitly: `scene_name`, `narrative_purpose`, `time_of_day`, `camera_height`, `aperture_intent`, `playback_intent`, `priority`, `capture_type`, `assigned_shooter`, `scheduled_time`, `reference_type`, `production_notes`, `status`, `created_at`, and `updated_at`.
+- [ ] Document an ordered source mapping for every CSV column from `project_id` through `updated_at`.
+- [ ] Define list serialization and empty optional values without changing column count.
 
-- [ ] Step 6: Commit
-
-    git add docs/storyboard-acceptance-runbook.md
-    git commit -m "docs: add storyboard acceptance runbook"
-
-## Task 7: Run package-wide validation and perform the smoke review
+## Task 4: Make final manifest promotion state-consistent
 
 **Files:**
-- Modify: Any file that fails the checks below.
-
-**Interfaces:**
-- Consumes: The complete plugin package from Tasks 1–6.
-- Produces: A clean, self-contained package ready for plugin installation or export.
-
-- [ ] Step 1: Validate every JSON file
 
-Run:
+- Modify: `skills/storyboard/workflow/storyboard-commit-export.md`
+- Modify: `skills/storyboard/resources/templates/project-readme.md`
 
-    Get-ChildItem -Recurse -Filter *.json | ForEach-Object {
-      Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null
-    }
-
-Expected: no JSON parse errors.
-
-- [ ] Step 2: Validate the required package paths
-
-Run:
+**Interfaces:**
 
-    $required = @(
-      '.codex-plugin/plugin.json',
-      'skills/storyboard/SKILL.md',
-      'guides/story-guide.md',
-      'guides/shoot-guide.md',
-      'guides/scene-guide.md',
-      'templates/project.json',
-      'templates/plan.md',
-      'templates/shot-list.csv',
-      'docs/storyboard-acceptance-runbook.md',
-      'storyboard-projects/.gitkeep'
-    )
-    foreach ($path in $required) {
-      if (-not (Test-Path $path)) { throw "Missing required package path: $path" }
-    }
+- Consumes: a valid generated/refined iteration, or selected committed iteration for re-export.
+- Produces: candidate artifacts and two byte-identical committed manifests.
 
-Expected: no missing-path errors.
+- [ ] Validate the selected source before requesting or building candidate output.
+- [ ] Build candidate plan, CSV, image/image-reference, and prospective committed `manifest.json` under `iterations/v###/commit-candidate/`.
+- [ ] Put status `committed`, selected active version, and appended export records into the candidate manifest before validation.
+- [ ] Validate the candidate manifest, plan, 45-column CSV, and image together.
+- [ ] Promote the set and write the exact candidate manifest bytes to both `project.json` and `final/manifest.json` as the final step.
+- [ ] Retain rollback data until byte equivalence passes; restore prior state on any failure.
 
-- [ ] Step 3: Search for forbidden planning placeholders
+## Task 5: Amend design and acceptance evidence
 
-Run:
+**Files:**
 
-    rg -n "TODO|TBD|implement later|fill in details|add appropriate error handling" . --glob '!docs/superpowers/specs/**' --glob '!docs/superpowers/plans/**'
+- Modify: `docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md`
+- Modify: `docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md`
+- Modify: `docs/storyboard-acceptance-runbook.md`
 
-Expected: no matches. Provisional guide markers are allowed because they are intentional product content, not unfinished implementation steps.
+**Interfaces:**
 
-- [ ] Step 4: Check repository diff and line endings
+- Consumes: the final file and state contracts.
+- Produces: architecture and manual-test documentation that match runtime behavior.
 
-Run:
+- [ ] State explicitly that there is one public skill and eight private workflow files.
+- [ ] Replace the obsolete eight-internal-skills file map and validation model.
+- [ ] Add review reset, approval/bypass, fallback routing, and version-record evidence.
+- [ ] Add prospective-manifest, 45-column, byte-equivalence, and rollback evidence.
+- [ ] Preserve the soccer benchmark and no-chat reopen check.
 
-    git diff --check
-    git status --short
+## Task 6: Run package-wide validation
 
-Expected: no whitespace errors. Only intended plugin files are modified.
+**Files:**
 
-- [ ] Step 5: Perform the manual smoke review
+- Modify only files that fail a check.
 
-Read README.md, skills/storyboard/SKILL.md, all internal skills, the four guide styles, the templates, and the acceptance runbook in that order. Confirm that every referenced path exists and that the top-level flow can describe the next action for a new project, a reviewable plan, a generated iteration, a frame refinement, and a committed project.
+**Interfaces:**
 
-- [ ] Step 6: Commit the validated package
+- Consumes: the complete package.
+- Produces: fresh evidence for portability, state contracts, and clean version-control content.
 
-    git add .
-    git commit -m "chore: validate storyboard plugin foundation"
+- [ ] Parse every JSON file with `ConvertFrom-Json`.
+- [ ] Confirm exactly one `SKILL.md` under `skills/`, at `skills/storyboard/SKILL.md`.
+- [ ] Confirm exactly eight `skills/storyboard/workflow/*.md` files, seven guide Markdown files, and four template files.
+- [ ] Confirm root `workflow/`, `guides/`, and `templates/` are absent.
+- [ ] Confirm public routing names all eight workflow files and all runtime paths resolve.
+- [ ] Confirm three review decisions, five review fields, six lifecycle states, four iteration filenames, review fallback choices, prospective candidate ordering, byte-identical manifests, and all 45 mapping terms.
+- [ ] Run the package-wide placeholder/path checks and the skill/plugin validators.
+- [ ] Run `git diff --check 28d60e9..HEAD` and inspect `git status --short`.
+- [ ] Stage only `.codex-plugin`, `README.md`, `skills/storyboard`, the design, this plan, runbook, and `storyboard-projects`; never stage `.superpowers` scratch files.
 
 ## Spec coverage self-review
 
-- Product goal and no-code plugin scope: Tasks 1, 4, and 5.
-- One smart top-level flow with associated skills: Tasks 4 and 5.
-- Provisional story, shoot, scene, and style defaults: Task 2.
-- Canonical JSON project model and stable frame IDs: Task 3.
-- Project-local guide overrides and portable repository storage: Tasks 1, 2, 3, and 5.
-- Review-by-default and explicit bypass: Task 5.
-- ChatGPT image-generation handoff and contact sheet: Tasks 4 and 5.
-- Frame-level refinement, continuity locks, and change summaries: Task 4.
-- Versioning, restoration, and branching behavior: Task 4.
-- Markdown plan and expanded CSV shot-list export: Tasks 3 and 4.
-- Failure handling, validation, and no-overwrite rules: Tasks 4, 5, 6, and 7.
-- Soccer benchmark acceptance scenario: Task 6.
-- Deferred hosted API, SharePoint, collaboration, UI, and office exports: Global Constraints and Task 6 documentation.
-
-## Execution handoff
-
-Plan complete and saved to docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md. Two execution options:
-
-1. Subagent-Driven (recommended) — dispatch a fresh subagent per task and review between tasks.
-2. Inline Execution — execute the tasks in this session with checkpoints.
-
-Choose one approach before implementation begins.
+- One registered skill and private skill-relative files: Tasks 1 and 5.
+- Portable project store and no-code scope: Task 1 and Global Constraints.
+- Durable review and state routing: Task 2.
+- Canonical frame records and 45-column mapping: Task 3.
+- State-consistent manifest promotion and rollback: Task 4.
+- Soccer acceptance and disk-only reopen: Task 5.
+- Package-level structural and content checks: Task 6.
diff --git a/docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md b/docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md
index ba3ef2b..7b67111 100644
--- a/docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md
+++ b/docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md
@@ -1,206 +1,183 @@
 # AI Storyboard Codex Plugin Foundation
 
-**Status:** Design approved for written-spec review  
-**Date:** 2026-07-21  
+**Status:** Design approved, including the second final-review architecture amendment
+**Date:** 2026-07-21
 **Scope:** Hackathon foundation for a portable, no-code Codex plugin paired with ChatGPT image generation
 
 ## 1. Product goal
 
-Create a portable Codex plugin that turns a brief, script, concept, or shot description into a consistent, production-readable storyboard workflow. The plugin should make the user experience feel like one smart `storyboard` flow while internally routing work through focused skills and a file-based project model.
+Create a portable Codex plugin that turns a brief, script, concept, or shot description into a consistent, production-readable storyboard workflow. The user experiences one smart `storyboard` flow. That public skill routes to private workflow references and persists all operational state in a file-based project model.
 
-The first milestone validates the end-to-end planning workflow rather than building a hosted application. A user should be able to start from one meaningful creative-intent statement, review a generated plan, hand it to ChatGPT image generation, refine individual frames, and commit durable Markdown, CSV, JSON, and image references inside the plugin repository.
+The foundation validates intake, planning, durable review, image-generation handoff, scoped refinement, versioning, and transactional export. It contains no hosted service, custom runtime, database, or image-processing implementation.
 
 ## 2. Design principles
 
-- Story information is the only mandatory creative input. Shoot and scene details may be inferred when reasonable.
-- Fast first value comes before a long questionnaire. Questions must materially affect the storyboard.
-- Defaults are recommendations, not hidden decisions. Inferred values are labeled clearly.
-- Consistency is prioritized over novelty. Project-wide constants and frame-level locks are explicit.
-- A frame refinement must preserve unaffected frame records, prompts, metadata, and assets as closely as the image-generation capability permits.
-- The manifest, Markdown plan, CSV shot list, and iteration snapshots are the durable source of truth; the chat thread is not.
-- The plugin contains no hosted service, custom runtime, database, or image-processing implementation.
+- Story information is the only mandatory creative input; shoot and scene details may be inferred when reasonable.
+- Questions must materially affect the storyboard. Defaults and inferred values are labeled.
+- Project-wide constants and frame locks preserve consistency.
+- A refinement preserves unaffected frame records, prompts, metadata, references, and assets.
+- `project.json`, plans, iteration snapshots, final manifests, and CSV exports are durable truth; chat history is not.
+- Failed generation or export never mutates the last valid project or final state.
 
-## 3. Plugin architecture
+## 3. Architecture amendment: one registered skill
 
-### User-facing entry point
+Only `skills/storyboard/SKILL.md` is a registered skill. It owns project discovery, routing, shared invariants, and user-visible response shape.
 
-The plugin exposes one top-level `storyboard` flow. It determines the user’s intent and current project stage from the request and the active project manifest. Users do not need to know the names of internal skills.
+Eight stage instruction sets live under `skills/storyboard/workflow/`:
 
-### Internal associated skills
+1. `storyboard-intake.md`
+2. `storyboard-planning.md`
+3. `storyboard-review.md`
+4. `storyboard-generation.md`
+5. `storyboard-refinement.md`
+6. `storyboard-versioning.md`
+7. `storyboard-references.md`
+8. `storyboard-commit-export.md`
 
-The top-level flow delegates to focused instruction sets:
+Starter guides and templates live under `skills/storyboard/resources/`. Workflow and resource files are loaded relative to `skills/storyboard/`; they are non-discoverable files, not internal skills or independent commands.
 
-1. **Intake** — normalize briefs, scripts, concepts, schedules, shot lists, and references; extract known facts; identify only high-impact gaps.
-2. **Planning** — create the project summary, creative constants, production assumptions, story progression, frame plan, prompts, and negative constraints.
-3. **Review** — present assumptions, warnings, frame sequence, and editable plan decisions; support approval, edits, reordering, adding/removing frames, alternatives, and review bypass.
-4. **Generation handoff** — build a complete prompt packet for ChatGPT image generation and record the returned image reference and settings.
-5. **Refinement** — identify affected frame IDs, preserve unaffected records, update only requested changes, warn about continuity impact, and create a new iteration.
-6. **Versioning** — list, compare, restore, and branch storyboard iterations without destroying prior snapshots.
-7. **Reference management** — store uploaded, external, internal, and AI-generated reference metadata and associate references with the project or specific frames.
-8. **Commit and export** — lock the selected iteration, validate output completeness, write the final plan and CSV, and record final artifact metadata.
+The portable project store remains at plugin-root `storyboard-projects/<project-slug>/`. Project-local `guides/` override the skill's starter guides, below explicit user instructions and approved project/frame decisions.
 
-### Project-local overrides
+The starter guides remain provisional and must not be represented as official Two Circles guidance. Defaults include six frames for a general short concept, eight for the soccer benchmark, 16:9 delivery, 23.976 fps, the 18/25/35/50/85/100mm lens family, and camera movement/support chosen to serve the story.
 
-Each project may override the starter guides by placing replacements in its own `guides/` directory. Project-local instructions take precedence over plugin defaults but remain below explicit user instructions and approved frame changes.
+## 4. Routing and lifecycle
 
-### Starter guide placeholders
+The public skill discovers and validates the project first, then routes by valid intent and durable state:
 
-The plugin will include non-official, replaceable starter guide files with usable baseline values:
+| Request or state | Workflow |
+| --- | --- |
+| New brief, script, concept, or import | Intake |
+| Missing plan or make-plan request from `proposed`/`planned` | Planning |
+| Review, assumptions, approval, or bypass | Review |
+| Add/remove/reorder/edit-plan while status is `review` | Review; write edit, reset authorization, return to `planned` |
+| Generate/create with durable authorization | Generation |
+| Scoped frame change | Refinement |
+| List, compare, restore, or branch versions | Versioning |
+| Attach or associate a reference | References |
+| Finalize, commit, export, or shot list | Commit/export |
 
-- **Story:** six frames for a general short concept, covering establishing, action, detail, reaction, progression, and resolution. Frame count may be changed when the narrative or duration warrants it.
-- **Shoot:** 16:9 delivery, 23.976 fps, a general 18/25/35/50/85/100mm lens family, tripod for static shots, handheld for action or intimacy, and gimbal/dolly for controlled movement. 120 fps is recommended only when slow motion serves the story.
-- **Scene:** preserve one coherent location, time of day, lighting direction, wardrobe, and prop set across frames unless the approved plan intentionally changes them.
-- **Styles:** placeholder definitions for cinematic realism, documentary sports, graphic storyboard sketch, and clean pitch-frame. These are provisional and must not be presented as official Two Circles brand guidance.
+The six lifecycle states are:
 
-## 4. Smart flow and state routing
+`proposed -> planned -> review -> generated -> refined -> committed`
 
-The flow first resolves the project context, then classifies intent, then loads only the relevant internal skill and guides.
+Without an explicit stage verb, route `proposed` to planning, `planned` to review, and `review` to generation only when durable authorization is true and the decision is approved or bypassed. Otherwise `review` routes back to review. Generated/refined projects offer refinement or commit. Committed projects offer inspect, branch, restore, or export.
 
-| User request or state | Routed action |
-|---|---|
-| New brief, script, concept, or reference | Intake |
-| Missing plan, “make a plan,” add/remove/reorder frames | Planning |
-| “Review,” “what assumptions did you make?” | Review |
-| Approved plan plus “generate/create” | Generation handoff |
-| “Change frame 3,” “make this wider,” “keep everything else” | Refinement |
-| “Show versions,” “restore,” “branch” | Versioning |
-| “Finalize,” “commit,” “export,” “make a shot list” | Commit and export |
+## 5. Durable review authorization
 
-The normal lifecycle is:
+Every manifest contains:
 
-`intake → plan → review → generate → refine/version → commit/export`
+```json
+"review": {
+  "decision": "pending",
+  "plan_version": null,
+  "authorized_for_generation": false,
+  "reviewed_at": null,
+  "change_summary": ""
+}
+```
+
+Allowed decisions are `pending`, `approved`, and `bypassed`.
 
-Review is enabled by default. The user may explicitly bypass it with requests such as “skip review,” “generate immediately,” or “use your defaults.” A bypass still stores the internal plan and its assumptions.
+- Initial planning and every plan edit reset review to pending, clear plan version/timestamp, and set authorization false.
+- Approval writes `approved`; an explicit `skip review`, `generate immediately`, or `use your defaults` request writes `bypassed`.
+- Approval and bypass both record the reviewed plan version, ISO 8601 UTC timestamp, authorization true, and a concise change summary while status remains `review`.
+- Generation requires status `review`, authorization true, an approved/bypassed decision, and a matching plan version. Otherwise it routes to review.
+- Every generated, refined, restored, or branched version record snapshots all five review fields.
 
-If a project is reopened, the manifest determines the next valid action. A committed project may be inspected, branched, restored, or exported again without relying on prior conversation history.
+## 6. Canonical project and frame model
 
-## 5. Canonical project model
+`project.json` is the source of truth. It includes project/storyboard identity, six-state lifecycle, integer active version, durable review, story/shoot/scene/style context, creative constants, references, canonical frames, immutable versions, and export records.
 
-Each project has one source-of-truth `project.json` manifest. The manifest includes:
+Stable frame IDs (`frame-001`) identify frames across revisions. `sequence_number` alone changes for reordering. Every canonical frame includes:
 
-- `project_id`, `title`, `schema_version`, `status`, and `active_version`.
-- `story`, `shoot`, `scene`, and `style` context.
-- `creative_constants` and explicit continuity locks.
-- `references` with type, source, rights note, applicable frames, and intended borrowing constraints.
-- `frames` with stable IDs, sequence numbers, descriptions, prompts, negative prompts, metadata, continuity locks, references, image paths, and revision history.
-- `versions` with immutable iteration snapshots, plans, prompts, image references, settings, timestamps, and change summaries.
-- `exports` with artifact type, path, version, validation result, and timestamp.
+- identity, title, description, prompts, continuity locks, reference IDs, image paths, and `revision_history`;
+- exact metadata keys for narrative and production values;
+- exact export fields needed for the 45-column shot list, including `scene_name`, `narrative_purpose`, `time_of_day`, `camera_height`, `aperture_intent`, `playback_intent`, `priority`, `capture_type`, `assigned_shooter`, `scheduled_time`, `reference_type`, `production_notes`, `status`, `created_at`, and `updated_at`.
 
-Frame IDs are stable (`frame-001`, `frame-002`); `sequence_number` may change when frames are reordered. User-approved values and inferred values are distinguished in the data and displayed in the workflow.
+The template defines every metadata key and the commit/export workflow defines the deterministic mapping for all 45 CSV columns.
 
 Context precedence is:
 
 1. Latest explicit user instruction.
 2. Approved frame-specific instruction.
 3. Approved project plan.
 4. Project-local guide.
-5. Plugin starter guide.
+5. Skill starter guide.
 6. System inference.
 
-Locked values cannot be changed silently. A request that conflicts with a lock must identify the conflict and request explicit approval before changing it.
-
-## 6. Image-generation handoff
-
-The plugin does not implement or host an image provider. It prepares a generation packet for the ChatGPT image-generation capability.
+Locked values cannot change silently.
 
-The packet contains:
+## 7. Image-generation and iteration contract
 
-- Approved project summary and story progression.
-- Project-wide creative constants and negative constraints.
-- Selected provisional or project-local style guide.
-- Aspect ratio and generation intent.
-- Frame-by-frame descriptions, prompts, metadata, continuity requirements, and reference associations.
-- A request for a numbered, coherent low-resolution storyboard contact sheet.
+The plugin prepares a prompt packet for ChatGPT image generation; it does not implement an image provider. Every generated, refined, restored, or branched `iterations/v###/` folder contains exactly these required artifacts:
 
-The default low-resolution output is a numbered contact sheet. Per-frame images are recorded when generated or supplied, but a custom compositor is not part of this no-code foundation. The exact prompt packet and returned image reference are stored in the iteration manifest.
+- `prompt-packet.md`
+- `frames.json`
+- `result.json`
+- `changes.md`
 
-For refinement, the flow first lists affected frames and possible continuity impacts. It regenerates only requested frames when the image-generation capability supports that behavior. Otherwise, it preserves unaffected frame assets and records the limitation and any suspected drift. It never silently regenerates the whole sequence after a frame-specific request.
+Initial generation requests a coherent numbered low-resolution contact sheet. Refinement identifies affected and untouched frame IDs. When frame-only generation is unavailable and no compositor exists, the user chooses whole-sheet regeneration with drift warning, a frame-only artifact retaining the previous sheet, or metadata-only revision. No workflow silently regenerates the whole sequence.
 
-## 7. File layout
-
-The plugin repository will contain the plugin definition, skills, starter guides, templates, and portable projects:
+## 8. Portable file layout
 
 ```text
 <plugin-repo>/
-├── .codex-plugin/
-│   └── plugin.json
-├── skills/
-│   ├── storyboard/
-│   ├── storyboard-intake/
-│   ├── storyboard-planning/
-│   ├── storyboard-review/
-│   ├── storyboard-generation/
-│   ├── storyboard-refinement/
-│   ├── storyboard-versioning/
-│   ├── storyboard-references/
-│   └── storyboard-commit-export/
-├── guides/
-│   ├── story-guide.md
-│   ├── shoot-guide.md
-│   ├── scene-guide.md
-│   └── styles/
-├── templates/
-│   ├── project.json
-│   ├── project-readme.md
-│   ├── plan.md
-│   └── shot-list.csv
-└── storyboard-projects/
-    └── <project-slug>/
-        ├── project.json
-        ├── README.md
-        ├── guides/
-        ├── references/
-        │   ├── uploaded/
-        │   ├── external/
-        │   └── internal/
-        ├── plans/
-        ├── iterations/
-        ├── final/
-        └── exports/
+|-- .codex-plugin/
+|   `-- plugin.json
+|-- skills/
+|   `-- storyboard/
+|       |-- SKILL.md
+|       |-- workflow/
+|       |   |-- storyboard-intake.md
+|       |   |-- storyboard-planning.md
+|       |   |-- storyboard-review.md
+|       |   |-- storyboard-generation.md
+|       |   |-- storyboard-refinement.md
+|       |   |-- storyboard-versioning.md
+|       |   |-- storyboard-references.md
+|       |   `-- storyboard-commit-export.md
+|       `-- resources/
+|           |-- guides/
+|           `-- templates/
+`-- storyboard-projects/
+    `-- <project-slug>/
+        |-- project.json
+        |-- README.md
+        |-- guides/
+        |-- references/
+        |-- plans/
+        |-- iterations/
+        |-- final/
+        `-- exports/
 ```
 
-Project folders are saved inside the plugin repository so the complete project can be exported, reopened, and shared as a portable file set.
-
-## 8. Commit and export behavior
-
-Commit locks the selected iteration and writes:
-
-- A final storyboard image reference, with high-resolution output requested at commit where the image-generation capability supports it.
-- `final-plan.md` containing the approved project summary and machine-readable frame headings.
-- `shot-list.csv` using the expanded schema from the product specification.
-- A final manifest snapshot.
-- Generation settings, source references, timestamps, and a concise change summary.
+Package validation counts one public `SKILL.md`, eight workflow Markdown files, and the guide/template resources. Root `workflow/`, `guides/`, and `templates/` copies are invalid.
 
-The high-resolution request must preserve the approved composition. If exact deterministic upscaling is unavailable and regeneration is required, the artifact metadata and user-facing summary must warn that small visual differences may occur.
+## 9. State-consistent commit and export
 
-The plugin must validate stable frame IDs, required manifest fields, CSV column names, prompt presence, status values, output paths, and Markdown frame identifiers before marking commit complete.
+Commit/export is one transaction:
 
-## 9. Failure handling and safety
+1. Validate the selected source iteration and its durable review evidence.
+2. Build candidate final plan, 45-column CSV, and high-resolution image/image-reference metadata under `iterations/v###/commit-candidate/`.
+3. Build a prospective committed manifest with status `committed`, selected integer `active_version`, and all export records already appended.
+4. Validate candidate manifest, plan, CSV, and image together, including enum values, frame IDs, all paths, and all 45 deterministic mappings.
+5. Promote the artifact set and write the same buffered candidate manifest bytes to both project-root `project.json` and `final/manifest.json` as the final commit step.
 
-- Missing meaningful story input: ask for the smallest useful creative-intent statement.
-- Ambiguous high-impact decisions: provide recommended defaults and a small set of alternatives.
-- Physically or logistically implausible shot: explain the issue and suggest an executable alternative.
-- Generation failure: preserve the prior plan and active iteration; do not overwrite approved outputs.
-- Continuity risk: identify which constants or frames may drift and ask for confirmation when a locked value is affected.
-- Invalid export: report the exact validation failure and leave the last valid export untouched.
-- Reference rights uncertainty: record the uncertainty and do not imply usage approval.
+The two committed manifests must be byte-for-byte identical. Promotion keeps rollback data until both manifest writes and equivalence validation succeed. On any failure, prior `project.json`, `final/`, active version, status, and exports remain unchanged; the isolated candidate may remain for inspection.
 
-## 10. Acceptance runbook
+## 10. Failure handling and safety
 
-The first manual test uses the supplied soccer production example:
+- Ask for the smallest useful creative-intent statement when story input is absent.
+- Explain implausible production choices and recommend executable alternatives.
+- Stop for lock conflicts, unresolved rights, invalid manifests, and high-impact ambiguity.
+- Preserve prior plan/state after generation failure.
+- Preserve prior final/state after candidate or promotion failure.
+- Never imply that reference rights are approved when uncertain.
 
-1. Import or describe the mural, streetcar, and high-school-field sequence.
-2. Generate an eight-frame plan using the practical lens, movement, frame-rate, and shot-size variations.
-3. Review the plan and verify assumptions are labeled.
-4. Generate a numbered low-resolution contact sheet through ChatGPT image generation.
-5. Change only the streetcar frame and verify unaffected frame records and prompts remain unchanged.
-6. Replace one frame with an uploaded reference and verify the reference is frame-scoped.
-7. Inspect iteration history and the change summary.
-8. Commit the result and validate the Markdown plan, CSV shot list, manifest, and image references.
-9. Reopen the project from `storyboard-projects/<project-slug>/` without the original chat thread.
+## 11. Acceptance runbook
 
-The foundation is successful when the user can complete this flow from one top-level storyboard command while retaining readable, portable project files at every stage.
+The soccer benchmark imports the mural, streetcar, and high-school-field sequence; builds an eight-frame plan; verifies durable review evidence; generates a contact sheet; changes only the streetcar frame; applies one frame-scoped uploaded reference; inspects immutable history; commits and validates plan, all 45 CSV columns, identical manifests, and image references; then reopens from `storyboard-projects/<project-slug>/` without chat history.
 
-## 11. Out of scope for this foundation
+## 12. Out of scope
 
-Hosted APIs, SharePoint integration, multi-user collaboration, live production tracking, client approval portals, mobile-specific UI, PDF/PowerPoint/Word export, custom image compositing, and a standalone web application are deferred. The manifest and skill boundaries leave room for these later without changing the core project model.
+Hosted APIs, SharePoint integration, multi-user collaboration, live production tracking, client portals, mobile UI, Office/PDF export, custom image compositing, and a standalone web application are deferred.
diff --git a/skills/storyboard/SKILL.md b/skills/storyboard/SKILL.md
new file mode 100644
index 0000000..d715bee
--- /dev/null
+++ b/skills/storyboard/SKILL.md
@@ -0,0 +1,84 @@
+---
+name: storyboard
+description: Use when turning a creative brief, script, concept, reference, or shot description into a storyboard plan, image-generation handoff, refinement, version, or production shot list.
+---
+
+# Smart Storyboard
+
+Coordinate one storyboard project from creative intent through a reviewable plan, image-generation handoff, targeted revisions, and production exports. This is the plugin's only registered skill.
+
+Resolve every `workflow/...` and `resources/...` path below relative to this `skills/storyboard/` directory. Load only the routed ordinary workflow reference; these files and all resources are non-discoverable and must not be offered as independently triggerable skills or commands.
+
+## Project discovery
+
+Follow this order exactly:
+
+1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`; an explicitly named committed project may be reopened.
+2. Scan `storyboard-projects/*/project.json` for project manifests. A project is active when the current workspace is inside that project's folder, or when exactly one project manifest exists under `storyboard-projects/`.
+3. If multiple manifests exist and none is in the current project folder, ask the user to choose; do not guess based on file ordering or timestamps.
+4. If no project exists and the user supplied meaningful story input, create one from `resources/templates/` with `storyboard-intake`.
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
+| `storyboard-planning` | Missing plan or make a plan from `proposed`/`planned` | `workflow/storyboard-planning.md` |
+| `storyboard-review` | Review, assumptions, or add/remove/reorder/edit-plan while status is `review` | `workflow/storyboard-review.md` |
+| `storyboard-generation` | Generate/create with durable review authorization | `workflow/storyboard-generation.md` |
+| `storyboard-refinement` | Change frame, make wider, keep everything else | `workflow/storyboard-refinement.md` |
+| `storyboard-versioning` | Show versions, restore, branch | `workflow/storyboard-versioning.md` |
+| `storyboard-commit-export` | Finalize, commit, export, make a shot list | `workflow/storyboard-commit-export.md` |
+| `storyboard-references` | Reference attachment or reference association | `workflow/storyboard-references.md` |
+
+Explicit stage verbs control routing only when the requested transition is valid. When status is `review`, route add/remove/reorder/edit-plan requests to `storyboard-review`; that workflow writes the edit and returns status to `planned`. Never route that case directly to planning.
+
+For requests without a stage verb, route from `project.json.status`: `proposed` to planning; `planned` to review; `review` to generation only when `review.authorized_for_generation` is `true` and `review.decision` is `approved` or `bypassed`, otherwise back to review; `generated` or `refined` to a current-iteration summary offering refinement or commit; and `committed` to a final-state summary offering inspect, branch, restore, or export. Never infer a route from missing or invalid state.
+
+## Defaults and context
+
+- Review is enabled by default. `skip review`, `generate immediately`, and `use your defaults` are explicit bypass decisions, not permission inferred from chat history. Record the durable review object before generation.
+- General concepts default to six frames; the soccer benchmark uses eight unless an approved frame count overrides it.
+- Starter guides under `resources/guides/` are provisional; project-local guides override them.
+- Apply context precedence in this order: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, system inference.
+- Keep supplied, approved, and inferred values distinct. Do not change an approved lock without explicit user choice.
+- For generation, require durable authorization, hand approved prompts to ChatGPT image generation, and retain `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md` for every iteration with the returned image reference.
+- Use the canonical frame record and 45-column export mapping defined by `resources/templates/project.json`, `resources/templates/project-readme.md`, and `workflow/storyboard-commit-export.md`.
+
+## Stage handling
+
+1. Discover and validate the project using this public skill's project discovery and routing contracts.
+2. Load exactly one routed stage reference unless a valid multi-stage request requires the smallest necessary sequence.
+3. Apply lifecycle transitions only after the routed stage's durable artifacts exist and pass its validation.
+4. Associate references through `storyboard-references` before another stage relies on them.
+5. Preserve stable `frame_id` values, unaffected records, immutable iterations, and prior valid finals.
+6. Treat `review` as durable state: planning or plan edits reset it to pending and unauthorized; approval or explicit bypass records `plan_version`, `reviewed_at`, and authorization before generation.
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
diff --git a/skills/storyboard/resources/guides/scene-guide.md b/skills/storyboard/resources/guides/scene-guide.md
new file mode 100644
index 0000000..c4dcb93
--- /dev/null
+++ b/skills/storyboard/resources/guides/scene-guide.md
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
diff --git a/skills/storyboard/resources/guides/shoot-guide.md b/skills/storyboard/resources/guides/shoot-guide.md
new file mode 100644
index 0000000..798ce0b
--- /dev/null
+++ b/skills/storyboard/resources/guides/shoot-guide.md
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
diff --git a/skills/storyboard/resources/guides/story-guide.md b/skills/storyboard/resources/guides/story-guide.md
new file mode 100644
index 0000000..d089eb2
--- /dev/null
+++ b/skills/storyboard/resources/guides/story-guide.md
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
diff --git a/skills/storyboard/resources/guides/styles/cinematic-realism.md b/skills/storyboard/resources/guides/styles/cinematic-realism.md
new file mode 100644
index 0000000..e21b0cc
--- /dev/null
+++ b/skills/storyboard/resources/guides/styles/cinematic-realism.md
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
diff --git a/skills/storyboard/resources/guides/styles/clean-pitch-frame.md b/skills/storyboard/resources/guides/styles/clean-pitch-frame.md
new file mode 100644
index 0000000..ebb0561
--- /dev/null
+++ b/skills/storyboard/resources/guides/styles/clean-pitch-frame.md
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
diff --git a/skills/storyboard/resources/guides/styles/documentary-sports.md b/skills/storyboard/resources/guides/styles/documentary-sports.md
new file mode 100644
index 0000000..f5c192a
--- /dev/null
+++ b/skills/storyboard/resources/guides/styles/documentary-sports.md
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
diff --git a/skills/storyboard/resources/guides/styles/graphic-storyboard-sketch.md b/skills/storyboard/resources/guides/styles/graphic-storyboard-sketch.md
new file mode 100644
index 0000000..79b0dae
--- /dev/null
+++ b/skills/storyboard/resources/guides/styles/graphic-storyboard-sketch.md
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
diff --git a/skills/storyboard/resources/templates/plan.md b/skills/storyboard/resources/templates/plan.md
new file mode 100644
index 0000000..1cd982d
--- /dev/null
+++ b/skills/storyboard/resources/templates/plan.md
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
diff --git a/skills/storyboard/resources/templates/project-readme.md b/skills/storyboard/resources/templates/project-readme.md
new file mode 100644
index 0000000..25c9fd3
--- /dev/null
+++ b/skills/storyboard/resources/templates/project-readme.md
@@ -0,0 +1,30 @@
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
+Every frame follows the canonical record in the skill-relative `resources/templates/project.json`. It includes `revision_history`, production metadata, reference metadata, prompts, continuity notes, timestamps, and image paths sufficient to map all 45 shot-list columns. Metadata key names are exact and must not be aliased; the commit/export workflow is the authoritative ordered mapping.
+
+## Durable review authorization
+
+Planning and every plan edit reset `review.decision` to `pending`, clear `plan_version` and `reviewed_at`, and set `authorized_for_generation` to `false`. Approval or explicit bypass records `approved` or `bypassed`, the reviewed plan version, review timestamp, change summary, and authorization while status remains `review`. Generation is invalid without that durable state, and every version record snapshots it.
+
+## Lifecycle
+
+The manifest status lifecycle is: `proposed -> planned -> review -> generated -> refined -> committed`. Explicit review bypass is recorded in status `review`; initial commit may use `generated` or `refined`, and re-export may use `committed` without changing status.
+
+Commit first builds a prospective committed manifest containing the selected `active_version` and appended export records. Candidate plan, CSV, image, and manifest validate together. The exact candidate manifest bytes are then written to both `project.json` and `final/manifest.json` as part of the same final promotion step; a failed transaction leaves prior state unchanged.
diff --git a/skills/storyboard/resources/templates/project.json b/skills/storyboard/resources/templates/project.json
new file mode 100644
index 0000000..850c39a
--- /dev/null
+++ b/skills/storyboard/resources/templates/project.json
@@ -0,0 +1,120 @@
+{
+  "project_id": "project-slug",
+  "storyboard_id": "storyboard-project-slug",
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
+  "review_decision_values": [
+    "pending",
+    "approved",
+    "bypassed"
+  ],
+  "review": {
+    "decision": "pending",
+    "plan_version": null,
+    "authorized_for_generation": false,
+    "reviewed_at": null,
+    "change_summary": ""
+  },
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
+      "metadata": {
+        "scene_name": "",
+        "narrative_purpose": "",
+        "subject": "",
+        "action": "",
+        "location": "",
+        "time_of_day": "",
+        "shot_size": "",
+        "camera_angle": "",
+        "camera_height": "",
+        "camera_position": "",
+        "focal_length_mm": null,
+        "lens_type": "",
+        "aperture_intent": "",
+        "depth_of_field": "",
+        "camera_movement": "",
+        "camera_support": "",
+        "frame_rate_fps": null,
+        "playback_intent": "",
+        "duration_seconds": null,
+        "transition_to_next": "",
+        "lighting": "",
+        "weather": "",
+        "wardrobe": [],
+        "props": [],
+        "priority": "",
+        "capture_type": "must-capture",
+        "assigned_shooter": "",
+        "scheduled_time": null,
+        "reference_type": "none",
+        "reference_path": "",
+        "continuity_notes": "",
+        "production_notes": "",
+        "status": "proposed",
+        "created_at": "",
+        "updated_at": ""
+      },
+      "continuity_locks": [],
+      "references": [],
+      "image_paths": {
+        "contact_sheet": "",
+        "frame": "",
+        "thumbnail": ""
+      },
+      "revision_history": []
+    },
+    "version": {
+      "version_id": "v001",
+      "status": "generated",
+      "plan_path": "plans/plan-v001.md",
+      "prompt_packet_path": "iterations/v001/prompt-packet.md",
+      "frames_path": "iterations/v001/frames.json",
+      "result_path": "iterations/v001/result.json",
+      "changes_path": "iterations/v001/changes.md",
+      "review": {
+        "decision": "approved",
+        "plan_version": 1,
+        "authorized_for_generation": true,
+        "reviewed_at": "",
+        "change_summary": ""
+      },
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
diff --git a/skills/storyboard/resources/templates/shot-list.csv b/skills/storyboard/resources/templates/shot-list.csv
new file mode 100644
index 0000000..419b858
--- /dev/null
+++ b/skills/storyboard/resources/templates/shot-list.csv
@@ -0,0 +1 @@
+project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at
diff --git a/skills/storyboard/workflow/storyboard-commit-export.md b/skills/storyboard/workflow/storyboard-commit-export.md
new file mode 100644
index 0000000..93fcc92
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-commit-export.md
@@ -0,0 +1,79 @@
+# Storyboard Commit and Export Workflow
+
+Commit or re-export a selected generated, refined, or committed iteration as one validated transaction. Candidate work must never alter prior final outputs or project state before the complete promotion succeeds.
+
+## Inputs
+
+Require status `generated`, `refined`, or `committed`. Use `generated` or `refined` for an initial commit and `committed` only for an explicit export or re-export of a selected existing iteration. Read `project.json`; the selected iteration's `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; references; and skill-relative `resources/templates/plan.md`, `resources/templates/project.json`, and `resources/templates/shot-list.csv`.
+
+## Transaction order
+
+1. **Validate source first.** Validate the selected iteration number, all four required iteration files, version record paths, complete canonical frame records, stable frame IDs, prompts, references, durable review authorization, and an existing image reference. Do not lock or mutate project state.
+2. **Build an isolated artifact candidate.** Request the high-resolution output, warning when regeneration may drift. Build candidate `final-plan.md`, `shot-list.csv`, and the high-resolution image or image-reference metadata under `iterations/v###/commit-candidate/`, never directly under `final/`.
+3. **Build the prospective committed manifest.** Starting from the unchanged project manifest, set `status` to `committed`, set integer `active_version` to the selected version, and append every prospective export record with final artifact path, selected version, `validation_result: passed`, and one transaction timestamp. Serialize this complete prospective state as candidate `manifest.json` in `commit-candidate/`.
+4. **Validate the complete prospective state.** Parse candidate `manifest.json`; validate candidate `final-plan.md`, candidate `shot-list.csv`, and the candidate image/image-reference metadata; verify the CSV header and every 45-column row against the deterministic mapping below; verify Markdown frame IDs; validate the six project lifecycle states, three review decisions, shot-list enum values, final paths, selected active version, appended export records, and image references. Require the candidate manifest bytes intended for `project.json` and `final/manifest.json` to be identical.
+5. **Promote and commit as one final step.** Preserve the prior `final/` and project manifest as rollback data. Promote the candidate plan, CSV, image/image-reference metadata, and manifest as one directory-level set, then write the exact same buffered candidate manifest bytes to both `project.json` and `final/manifest.json`. Verify the two manifest files are byte-for-byte identical before discarding rollback data and reporting success.
+
+Allowed shot-list values are:
+
+- `status`: `proposed`, `approved`, `assigned`, `captured`, `completed`, `omitted`
+- `reference_type`: `internal`, `external`, `uploaded`, `AI-generated`, `none`
+- `capture_type`: `must-capture`, `inspiration`, `optional`, `alternate`
+
+## Deterministic 45-column mapping
+
+Emit columns in the exact order below. `frame` means the canonical record from the selected iteration's `frames.json`; `manifest` means the prospective committed candidate.
+
+| # | CSV column | Canonical source |
+| ---: | --- | --- |
+| 1 | `project_id` | `manifest.project_id` |
+| 2 | `storyboard_id` | `manifest.storyboard_id` |
+| 3 | `storyboard_version` | `manifest.active_version` |
+| 4 | `frame_id` | `frame.frame_id` |
+| 5 | `sequence_number` | `frame.sequence_number` |
+| 6 | `shot_name` | `frame.title` |
+| 7 | `scene_name` | `frame.metadata.scene_name` |
+| 8 | `description` | `frame.description` |
+| 9 | `narrative_purpose` | `frame.metadata.narrative_purpose` |
+| 10 | `subject` | `frame.metadata.subject` |
+| 11 | `action` | `frame.metadata.action` |
+| 12 | `location` | `frame.metadata.location` |
+| 13 | `time_of_day` | `frame.metadata.time_of_day` |
+| 14 | `shot_size` | `frame.metadata.shot_size` |
+| 15 | `camera_angle` | `frame.metadata.camera_angle` |
+| 16 | `camera_height` | `frame.metadata.camera_height` |
+| 17 | `camera_position` | `frame.metadata.camera_position` |
+| 18 | `focal_length_mm` | `frame.metadata.focal_length_mm` |
+| 19 | `lens_type` | `frame.metadata.lens_type` |
+| 20 | `aperture_intent` | `frame.metadata.aperture_intent` |
+| 21 | `depth_of_field` | `frame.metadata.depth_of_field` |
+| 22 | `camera_movement` | `frame.metadata.camera_movement` |
+| 23 | `camera_support` | `frame.metadata.camera_support` |
+| 24 | `frame_rate_fps` | `frame.metadata.frame_rate_fps` |
+| 25 | `playback_intent` | `frame.metadata.playback_intent` |
+| 26 | `duration_seconds` | `frame.metadata.duration_seconds` |
+| 27 | `transition_to_next` | `frame.metadata.transition_to_next` |
+| 28 | `lighting` | `frame.metadata.lighting` |
+| 29 | `weather` | `frame.metadata.weather` |
+| 30 | `wardrobe` | `frame.metadata.wardrobe` |
+| 31 | `props` | `frame.metadata.props` |
+| 32 | `priority` | `frame.metadata.priority` |
+| 33 | `capture_type` | `frame.metadata.capture_type` |
+| 34 | `assigned_shooter` | `frame.metadata.assigned_shooter` |
+| 35 | `scheduled_time` | `frame.metadata.scheduled_time` |
+| 36 | `reference_type` | `frame.metadata.reference_type` |
+| 37 | `reference_path` | `frame.metadata.reference_path` |
+| 38 | `image_path` | `frame.image_paths.frame` |
+| 39 | `prompt` | `frame.prompt` |
+| 40 | `negative_prompt` | `frame.negative_prompt` |
+| 41 | `continuity_notes` | `frame.metadata.continuity_notes` |
+| 42 | `production_notes` | `frame.metadata.production_notes` |
+| 43 | `status` | `frame.metadata.status` |
+| 44 | `created_at` | `frame.metadata.created_at` |
+| 45 | `updated_at` | `frame.metadata.updated_at` |
+
+Serialize list-valued metadata such as wardrobe or props as semicolon-delimited text inside one RFC 4180-quoted field. Use an empty field for an absent optional value; never shift columns or invent a second source.
+
+## Failure behavior
+
+On source, generation, prospective-manifest, candidate validation, promotion, manifest-write, or byte-equivalence failure, report the exact failure, leave `commit-candidate/` isolated for inspection, restore any promotion rollback, and leave `project.json`, prior `final/`, `active_version`, `status`, and `exports` unchanged. Never report commit or export success without the promoted validated files and identical committed manifest state.
diff --git a/skills/storyboard/workflow/storyboard-generation.md b/skills/storyboard/workflow/storyboard-generation.md
new file mode 100644
index 0000000..8a8770d
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-generation.md
@@ -0,0 +1,30 @@
+# Storyboard Generation Workflow
+
+Create an immutable first or subsequent generated iteration from an approved or explicitly review-bypassed plan.
+
+## Inputs
+
+Read `project.json`, the selected `plans/plan-v###.md`, all canonical frame records, skill-relative style guidance, constants, negative constraints, aspect ratio, and references. Require status `review`, `review.authorized_for_generation: true`, `review.decision` equal to `approved` or `bypassed`, and `review.plan_version` equal to the selected plan version. Otherwise route to `storyboard-review` without creating generation artifacts.
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
+5. Copy the complete frame array to `project.json.frames`; append a version record containing `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, a complete snapshot of the five review fields, `created_at`, and `change_summary`; set integer `active_version`; then set status to `generated`.
+
+## Failure behavior
+
+If image generation or artifact validation fails, do not update `project.json.frames`, `project.json.versions`, `active_version`, or status. Do not claim generation success until the image reference and all four files exist and validate.
diff --git a/skills/storyboard/workflow/storyboard-intake.md b/skills/storyboard/workflow/storyboard-intake.md
new file mode 100644
index 0000000..80ef62a
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-intake.md
@@ -0,0 +1,25 @@
+# Storyboard Intake Workflow
+
+Normalize supplied information into a portable project without repeating questions that the brief already answers.
+
+## Inputs
+
+Read the supplied story, shoot, scene, style, references, duration, frame count, aspect ratio, and production constraints. Read skill-relative `resources/templates/project.json` and `resources/templates/project-readme.md`.
+
+## Procedure
+
+1. Extract supplied facts first and ask only about missing facts that materially alter story, delivery, rights, or production constraints.
+2. Mark derived values as `inferred` and keep them distinct from supplied or approved values.
+3. Create a unique filesystem-safe project slug.
+4. Copy the templates to `storyboard-projects/<project-slug>/project.json` and `README.md` without overwriting an existing project.
+5. Create project-local `guides/`, `references/`, `plans/`, `iterations/`, `final/`, and `exports/` directories.
+6. Initialize `frames`, `versions`, and `exports` as empty arrays, `active_version` as integer `0`, `status` as `proposed`, and `review` as pending with null `plan_version`/`reviewed_at` and `authorized_for_generation: false`.
+7. Record normalized intake fields and stop before planning.
+
+## Files and outputs
+
+May create only the new project manifest, README, and directories during intake. Produce a normalized summary with slug, supplied facts, inferred values, unresolved material gaps, and project path.
+
+## Stop conditions and invariants
+
+Stop when meaningful story intent is absent, a slug collides, or a material gap requires user choice. Preserve supplied facts verbatim. Do not create `plans/plan-v001.md`, iteration artifacts, or images during intake.
diff --git a/skills/storyboard/workflow/storyboard-planning.md b/skills/storyboard/workflow/storyboard-planning.md
new file mode 100644
index 0000000..c5c41d0
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-planning.md
@@ -0,0 +1,23 @@
+# Storyboard Planning Workflow
+
+Turn a `proposed` project into a reviewable, image-free production plan.
+
+## Inputs
+
+Read `project.json`, project-local guides, skill-relative `resources/guides/`, `resources/templates/plan.md`, reference metadata, and the latest explicit user instruction. Apply the shared context precedence from `SKILL.md`.
+
+## Procedure
+
+1. Require status `proposed` for initial planning or `planned` for further plan edits. Edits requested while status is `review` stay in `storyboard-review` and return here only after status becomes `planned`.
+2. Load applicable story, shoot, scene, and style guides; project-local guidance overrides starter guidance.
+3. Use six frames for a general concept and eight for the soccer benchmark unless an approved frame count overrides it.
+4. Assign stable `frame_id` values such as `frame-001` and use `sequence_number` only for order.
+5. Populate every frame record with `frame_id`, `sequence_number`, `title`, `description`, `prompt`, `negative_prompt`, `metadata`, `continuity_locks`, `references`, and `image_paths`.
+6. Write the next `plans/plan-v###.md` with project summary, creative constants, assumptions, progression, frame plan, prompts, and negative constraints.
+7. Update `project.json.frames`, keep approved values distinct from inferred values, reset `review` to `decision: pending`, `plan_version: null`, `authorized_for_generation: false`, `reviewed_at: null`, and a concise `change_summary`, then set `status` to `planned` only after the plan validates.
+
+Planning does not create an iteration. The plan path is captured in the version record when generation creates that iteration.
+
+## Outputs
+
+Produce a validated plan and canonical manifest frame records for review. Stop for material ambiguity, lock conflicts, or guide conflicts that require user choice. Never create images or iteration files.
diff --git a/skills/storyboard/workflow/storyboard-references.md b/skills/storyboard/workflow/storyboard-references.md
new file mode 100644
index 0000000..3167caa
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-references.md
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
diff --git a/skills/storyboard/workflow/storyboard-refinement.md b/skills/storyboard/workflow/storyboard-refinement.md
new file mode 100644
index 0000000..a04c7f1
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-refinement.md
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
+6. Validate `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; then update `project.json.frames`, append the canonical version record including the durable five-field review snapshot inherited from its source version, increment integer `active_version`, and set `generated -> refined` or retain `refined` for later refinements.
+
+## Stop conditions and invariants
+
+Stop for lock conflicts, unresolved reference rights, unclear scope, or a missing explicit fallback choice. Never overwrite an iteration or silently regenerate the full contact sheet. On any failure, leave the prior active iteration and project state unchanged.
diff --git a/skills/storyboard/workflow/storyboard-review.md b/skills/storyboard/workflow/storyboard-review.md
new file mode 100644
index 0000000..f900eb9
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-review.md
@@ -0,0 +1,20 @@
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
+2. When entering from `planned`, set `planned -> review` after the complete review presentation exists. Keep `review.decision: pending` and authorization false until the user decides. When re-entering from `review`, show unresolved decisions without replaying the transition.
+3. Support approval; overall edits; edit/add/remove/reorder frame; alternatives; `skip review`; and proceed to creation.
+4. Handle add/remove/reorder/edit-plan requests inside this workflow. Write the revised plan and frame records, reset review to `decision: pending`, `plan_version: null`, `authorized_for_generation: false`, `reviewed_at: null`, and a concise `change_summary`, then set `review -> planned`. A later bypass is a separate explicit review decision.
+5. For approval, write `decision: approved`; for `skip review`, `generate immediately`, or `use your defaults`, write `decision: bypassed`. In both cases write the reviewed plan version to `plan_version`, an ISO 8601 UTC value to `reviewed_at`, `authorized_for_generation: true`, and a concise `change_summary`; keep status `review`.
+6. Never treat conversation-only approval as authorization. Generation sets `generated` only after re-reading the durable review object and validating its four-file iteration contract and image result.
+
+## Outputs
+
+Produce the durable review decision, plan version, changed frame IDs, remaining warnings, and generation authorization or revised plan. Stop for unresolved decisions, lock conflicts, or alternatives requiring a choice. Never generate images or iteration artifacts.
diff --git a/skills/storyboard/workflow/storyboard-versioning.md b/skills/storyboard/workflow/storyboard-versioning.md
new file mode 100644
index 0000000..ff0c0fb
--- /dev/null
+++ b/skills/storyboard/workflow/storyboard-versioning.md
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
+After all four files validate, copy destination frames to `project.json.frames`, append a canonical version record with `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, the source version's durable five-field review snapshot, `created_at`, and `change_summary`, and update integer `active_version`. A new branch sets status to `generated`; a restore uses `generated` or `refined` according to the restored result and recorded change.
+
+Stop for missing versions, missing frame IDs, ambiguous targets, or unconfirmed replacement of the active choice. Never overwrite or delete history.
diff --git a/storyboard-projects/.gitkeep b/storyboard-projects/.gitkeep
new file mode 100644
index 0000000..e69de29
