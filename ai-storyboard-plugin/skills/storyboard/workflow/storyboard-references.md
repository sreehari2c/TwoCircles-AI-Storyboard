# Storyboard References Workflow

Record reference provenance, permitted influence, and frame scope before another stage relies on the material.

## Inputs and procedure

Read supplied material, `project.json`, applicable frame IDs, and project-local reference metadata. Support `internal`, `external`, `uploaded`, `AI-generated`, and `none` types. Store reference ID, type, source/path/URL, rights note, applicable frames, what to borrow, and what not to copy.

Treat uploaded material as unscoped until the user identifies applicable frames. Use `none` when no reference is supplied. Record rights uncertainty without inferring permission.

## Files and outputs

May create `references/<reference-id>.md` and update the matching `project.json.references` record. When an iteration uses a reference, copy its stable reference ID into affected `frames.json` records; do not duplicate or broaden scope.

Stop when rights, source, permitted use, frame scope, or requested borrowing is unresolved. Do not assume an uploaded reference applies to the whole project.
