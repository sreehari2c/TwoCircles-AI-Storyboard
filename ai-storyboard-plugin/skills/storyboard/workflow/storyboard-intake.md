# Storyboard Intake Workflow

Normalize supplied information into a portable project without repeating questions that the brief already answers.

## Inputs

Read the supplied story, shoot, scene, style, references, duration, frame count, aspect ratio, and production constraints. Read skill-relative `resources/templates/project.json` and `resources/templates/project-readme.md`.

## Procedure

1. Extract supplied facts first and ask only about missing facts that materially alter story, delivery, rights, or production constraints.
2. Mark derived values as `inferred` and keep them distinct from supplied or approved values.
3. Create a unique filesystem-safe project slug.
4. Copy the templates to `storyboard-projects/<project-slug>/project.json` and `README.md` without overwriting an existing project.
5. Create project-local `guides/`, `references/`, `plans/`, `iterations/`, `final/`, and `exports/` directories.
6. Initialize `frames`, `versions`, and `exports` as empty arrays, `active_version` as integer `0`, `status` as `proposed`, and `review` as pending with null `plan_version`/`reviewed_at` and `authorized_for_generation: false`.
7. Record normalized intake fields and stop before planning.

## Files and outputs

May create only the new project manifest, README, and directories during intake. Produce a normalized summary with slug, supplied facts, inferred values, unresolved material gaps, and project path.

## Stop conditions and invariants

Stop when meaningful story intent is absent, a slug collides, or a material gap requires user choice. Preserve supplied facts verbatim. Do not create `plans/plan-v001.md`, iteration artifacts, or images during intake.
