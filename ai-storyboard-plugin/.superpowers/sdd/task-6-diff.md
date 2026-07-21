diff --git a/docs/storyboard-acceptance-runbook.md b/docs/storyboard-acceptance-runbook.md
new file mode 100644
index 0000000..d0d9e2a
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
+Expected evidence: a project folder and plan containing all three locations, with stable frame IDs such as `frame-001` and sequence numbers that express the current order.
+
+### 2. Generate an eight-frame plan
+
+Create an eight-frame plan covering the sequence. Use practical variations in lens, camera movement, frame rate, and shot size; keep the production constraints and continuity constants consistent across frames.
+
+Expected evidence: a saved plan file with exactly eight identifiable frame records, each with a stable frame ID and explicit lens, movement, frame-rate, and shot-size values or intentional defaults.
+
+### 3. Review the plan and verify assumptions
+
+Review every frame before generation. Check that user-approved values are distinguishable from inferred assumptions, and label each assumption rather than presenting it as confirmed input. Resolve or accept assumptions before proceeding.
+
+Expected evidence: the reviewed plan file, including labeled assumptions and a visible review/approval outcome.
+
+### 4. Generate the contact sheet
+
+Request a numbered, low-resolution contact sheet through ChatGPT image generation using the approved eight-frame plan. Confirm that the returned image is recorded with the active iteration only after the image result is available.
+
+Expected evidence: a numbered contact-sheet reference, the exact prompt packet used for generation, and an active iteration record that points to the returned image.
+
+### 5. Change only the streetcar frame
+
+Request a scoped change to the streetcar frame only—for example, adjust its movement while preserving the approved project constants. Compare the new iteration with the prior one.
+
+Expected evidence: the streetcar frame is the only changed frame; unaffected frame records and unchanged prompts remain unchanged; a new immutable iteration and scope/change record exist.
+
+### 6. Replace one frame with an uploaded reference
+
+Upload a reference and replace one selected frame with it. Confirm that the reference is attached to the selected frame, not applied as a project-wide reference or copied into unrelated frame prompts.
+
+Expected evidence: frame-scoped reference metadata naming the target frame ID, plus unchanged reference metadata and prompts for unaffected frames.
+
+### 7. Inspect iteration history
+
+List the iterations and inspect the change summary for the scoped edit and reference replacement. Confirm that earlier approved iterations remain available and have not been overwritten.
+
+Expected evidence: iteration history, selected source and destination versions, affected and untouched frame IDs, and an iteration change summary.
+
+### 8. Commit and validate exports
+
+Commit the selected iteration. Validate that the final Markdown plan, CSV shot list, manifest, and image references are complete and internally consistent.
+
+Expected evidence: `final/final-plan.md`, `final/shot-list.csv`, `final/manifest.json`, and final image-reference metadata. Confirm that every exported frame maps to a stable frame ID and that the manifest references the selected iteration and generated or supplied images.
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
