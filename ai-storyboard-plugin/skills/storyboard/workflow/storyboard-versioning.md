# Storyboard Versioning Workflow

Inspect, restore, or branch immutable iteration history without deleting or overwriting prior snapshots.

## Inputs

Read `project.json` and each iteration's exact `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. Refuse restore or branch operations when any selected source artifact is missing or invalid.

## Procedure

- Listing or comparing versions reads the four-file contracts and does not mutate state.
- A whole-version restore creates a new `iterations/v###/` snapshot from the selected source.
- A one-frame restore copies the prior canonical frame record into a new iteration while preserving all unaffected records.
- A branch from any version, including a named committed project, creates a new active iteration and lineage without deleting history.

Every restore or branch writes the same four required files in the destination: `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`. `changes.md` and `result.json` record source version, operation, affected frames, retained image references, and validation outcome.

After all four files validate, copy destination frames to `project.json.frames`, append a canonical version record with `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, the source version's durable five-field review snapshot, `created_at`, and `change_summary`, and update integer `active_version`. A new branch sets status to `generated`; a restore uses `generated` or `refined` according to the restored result and recorded change.

Stop for missing versions, missing frame IDs, ambiguous targets, or unconfirmed replacement of the active choice. Never overwrite or delete history.
