diff --git a/skills/storyboard/SKILL.md b/skills/storyboard/SKILL.md
new file mode 100644
index 0000000..6da8889
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
+1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`.
+2. If the current workspace has one active storyboard project, use it.
+3. If multiple projects exist and none is named, ask the user to choose.
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
