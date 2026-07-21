# Manual Acceptance Runbook

Use this runbook to verify the complete storyboard flow from a clean project folder. This is a manual acceptance test: it requires access to ChatGPT image generation, but does not require a hosted API, custom script, or runtime.

## Setup

1. Start from a clean project folder with no prior project outputs or chat-dependent state.
2. Have the supplied soccer production example ready: a sequence spanning a mural, a streetcar, and a high-school field.
3. Confirm that ChatGPT image generation is available for the contact-sheet request.

Record the project slug and use durable files under `storyboard-projects/<project-slug>/` as the source of truth throughout the run.

Confirm the installed package exposes only `skills/storyboard/SKILL.md`; its eight stage references are under `skills/storyboard/workflow/`, and starter files are under `skills/storyboard/resources/`.

## Acceptance actions

### 1. Import or describe the sequence

Import or describe the soccer production sequence in order: mural, streetcar, and high-school field. This action is intake/import only; do not create the plan yet.

Expected evidence: `project.json` has status `proposed`, active version `0`, normalized story input covering all three locations, and empty `frames` and `versions` arrays. Project-local `guides/`, `references/`, `plans/`, `iterations/`, `final/`, and `exports/` directories exist, with no planning artifact created yet.

### 2. Generate an eight-frame plan

Create an eight-frame plan covering the sequence. Use practical variations in lens, camera movement, frame rate, and shot size; keep production constraints and continuity constants consistent across frames.

Expected evidence: this is the first action that creates `plans/plan-v001.md`. It contains exactly eight identifiable frame records, each with a stable frame ID such as `frame-001` and explicit lens, movement, frame-rate, and shot-size values or intentional defaults; project status is `planned`, and review is `pending` with authorization false.

### 3. Review the plan and verify assumptions

Review every frame before generation. Check that approved values are distinguishable from inferred assumptions. Resolve or accept assumptions before proceeding.

Expected evidence: the reviewed `plans/plan-v001.md`, labeled assumptions, and a durable `project.json.review` object. Approval records `decision: approved`; explicit bypass records `decision: bypassed`. Both record `plan_version: 1`, a non-null `reviewed_at`, `authorized_for_generation: true`, and a `change_summary` while status is `review`. Plan edits reset the five fields to pending/unauthorized values and return status to `planned`.

### 4. Generate the contact sheet

Request a numbered, low-resolution contact sheet through ChatGPT image generation using the approved eight-frame plan.

Expected evidence: initial generation creates all four files: `iterations/v001/prompt-packet.md`, `iterations/v001/frames.json`, `iterations/v001/result.json`, and `iterations/v001/changes.md`. `result.json` contains the numbered contact-sheet reference, while `project.json.frames`, `project.json.versions`, integer `active_version`, and status `generated` agree with v001. The v001 version record contains the same five review fields that authorized generation.

### 5. Change only the streetcar frame

Request a scoped change to the streetcar frame only, such as adjusting its movement while preserving approved project constants. Compare the new iteration with the prior one.

If frame-level editing is unavailable and there is no compositor, verify the workflow asks for one explicit fallback:

- (a) regenerate the whole contact sheet with unchanged frame prompts locked and a continuity/drift warning;
- (b) create a frame-only revision artifact while retaining the previous contact sheet; or
- (c) update prompts/metadata only with no new image.

Expected evidence: the streetcar frame is the only changed frame; unaffected frame records and prompts remain unchanged. The new immutable `iterations/v###/` folder contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; both `result.json` and `changes.md` record the capability path or fallback and retained image references.

### 6. Replace one frame with an uploaded reference

Upload a reference and replace one selected frame with it. Confirm the reference is attached to the selected frame, not applied project-wide or copied into unrelated frame prompts.

Expected evidence: frame-scoped reference metadata names the target frame ID. The same new `iterations/v###/` folder contains `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`, with unchanged reference metadata and prompts for unaffected frames.

### 7. Inspect iteration history

List iterations and inspect change summaries for the scoped edit and reference replacement. Confirm earlier approved iterations remain available.

Expected evidence: iteration history, selected source and destination versions, affected and untouched frame IDs, and complete `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md` files in every generated, refined, restored, or branched `iterations/v###/` folder.

### 8. Commit and validate exports

Commit the selected iteration. Validate the final Markdown plan, CSV shot list, manifest, and image references.

Expected evidence: source validation occurs first. Candidate `final-plan.md`, `shot-list.csv`, image/image-reference metadata, and a prospective `manifest.json` with status `committed`, selected `active_version`, and appended export records are built under `iterations/v###/commit-candidate/`. Validate the candidate plan, all 45 CSV columns, image, and manifest together. Only then promote the set and write the same candidate manifest bytes to both `project.json` and `final/manifest.json`; verify those files are byte-for-byte identical.

### 9. Reopen from disk

Close or leave the original conversation, then reopen the project from `storyboard-projects/<project-slug>/` without the original chat thread. Inspect the project, active iteration, plan, and final artifacts.

Expected evidence: successful reopen without the original chat, with the same project slug, stable frame IDs, iteration history, and final Markdown/CSV/manifest available from disk. A named committed project offers inspect, branch, restore, or export. Choose export once and confirm the transactional candidate flow runs from status `committed`, appends validated export records, and leaves status `committed` after promotion.

## Failure checks

- Force or observe a failed generation and verify that the active version, status, prior plan, and outputs remain unchanged.
- Create a lock conflict and verify the workflow pauses for approval.
- Introduce a CSV or manifest validation failure and verify that prior final outputs remain untouched and no partial replacement appears in `final/`.
- Verify a failed commit leaves the candidate isolated under `iterations/v###/commit-candidate/` and leaves prior final outputs, status, active version, and export records unchanged.
- Clear or corrupt durable review authorization and verify a generate request routes back to review without creating an iteration.
- While status is `review`, request an add/remove/reorder/edit-plan change and verify the review workflow handles it, resets authorization, and returns status to `planned`.

Record pass/fail results, observed artifact paths, and deviations before accepting the task.
