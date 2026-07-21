# Storyboard Refinement Workflow

Create a new immutable iteration for scoped changes while preserving unaffected frame records, prompts, metadata, references, and image paths.

## Inputs and required outputs

Read `project.json` and the active iteration's exact `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Require status `generated` or `refined`. Every new `iterations/v###/` refinement must write the same four files:

- `prompt-packet.md`
- `frames.json`
- `result.json`
- `changes.md`

## Procedure

1. Identify affected frame IDs, distinguish frame-only changes from project-constant changes, and list continuity risks.
2. Copy the active iteration into the next version as working data. Change only requested frame fields and keep every unaffected frame record and prompt byte-for-byte equivalent where the file format permits.
3. If image capability supports frame-level editing, regenerate only requested frame assets. Recompose the contact sheet only when the capability also provides that operation.
4. If frame-level editing is unsupported and no compositor is available, do not silently regenerate the full contact sheet. Ask the user to choose exactly one fallback:
   - **(a) Whole contact sheet:** regenerate the whole contact sheet with unchanged frame prompts locked, with an explicit continuity/drift warning.
   - **(b) Frame-only artifact:** create a frame-only revision artifact while retaining the previous contact sheet.
   - **(c) Metadata only:** update prompts/metadata only with no new image.
5. Record the selected capability path or fallback in both `changes.md` and `result.json`, including affected and untouched frame IDs, continuity warnings, prior image references retained, and every new image path.
6. Validate `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; then update `project.json.frames`, append the canonical version record including the durable five-field review snapshot inherited from its source version, increment integer `active_version`, and set `generated -> refined` or retain `refined` for later refinements.

## Stop conditions and invariants

Stop for lock conflicts, unresolved reference rights, unclear scope, or a missing explicit fallback choice. Never overwrite an iteration or silently regenerate the full contact sheet. On any failure, leave the prior active iteration and project state unchanged.
