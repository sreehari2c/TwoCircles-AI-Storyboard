# Storyboard Planning Workflow

Turn a `proposed` project into a reviewable, image-free production plan.

## Inputs

Read `project.json`, project-local guides, skill-relative `resources/guides/`, `resources/templates/plan.md`, reference metadata, and the latest explicit user instruction. Apply the shared context precedence from `SKILL.md`.

## Procedure

1. Require status `proposed` for initial planning or `planned` for further plan edits. Edits requested while status is `review` stay in `storyboard-review` and return here only after status becomes `planned`.
2. Load applicable story, shoot, scene, and style guides; project-local guidance overrides starter guidance.
3. Use six frames for a general concept and eight for the soccer benchmark unless an approved frame count overrides it.
4. Assign stable `frame_id` values such as `frame-001` and use `sequence_number` only for order.
5. Populate every frame record with `frame_id`, `sequence_number`, `title`, `description`, `prompt`, `negative_prompt`, `metadata`, `continuity_locks`, `references`, and `image_paths`.
6. Write the next `plans/plan-v###.md` with project summary, creative constants, assumptions, progression, frame plan, prompts, and negative constraints.
7. Update `project.json.frames`, keep approved values distinct from inferred values, reset `review` to `decision: pending`, `plan_version: null`, `authorized_for_generation: false`, `reviewed_at: null`, and a concise `change_summary`, then set `status` to `planned` only after the plan validates.

Planning does not create an iteration. The plan path is captured in the version record when generation creates that iteration.

## Outputs

Produce a validated plan and canonical manifest frame records for review. Stop for material ambiguity, lock conflicts, or guide conflicts that require user choice. Never create images or iteration files.
