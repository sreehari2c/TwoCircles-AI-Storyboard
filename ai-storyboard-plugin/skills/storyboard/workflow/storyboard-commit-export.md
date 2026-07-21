# Storyboard Commit and Export Workflow

Commit or re-export a selected generated, refined, or committed iteration as one validated transaction. Candidate work must never alter prior final outputs or project state before the complete promotion succeeds.

## Inputs

Require status `generated`, `refined`, or `committed`. Use `generated` or `refined` for an initial commit and `committed` only for an explicit export or re-export of a selected existing iteration. Read `project.json`; the selected iteration's `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`; references; and skill-relative `resources/templates/plan.md`, `resources/templates/project.json`, and `resources/templates/shot-list.csv`.

## Transaction order

1. **Validate source first.** Validate the selected iteration number, all four required iteration files, version record paths, complete canonical frame records, stable frame IDs, prompts, references, durable review authorization, and an existing image reference. Do not lock or mutate project state.
2. **Build an isolated artifact candidate.** Request the high-resolution output, warning when regeneration may drift. Preserve the approved `Storyboard_Template.jpg` layout and `Storyboard Example.png` art style: fewer than 12 frames leave unused slots black, while more than 12 frames use the approved multi-image or resized/reflowed layout. Build candidate `final-plan.md`, `shot-list.csv`, and the high-resolution image or image-reference metadata under `iterations/v###/commit-candidate/`, never directly under `final/`.
3. **Build the prospective committed manifest.** Starting from the unchanged project manifest, set `status` to `committed`, set integer `active_version` to the selected version, and append every prospective export record with final artifact path, selected version, `validation_result: passed`, and one transaction timestamp. Serialize this complete prospective state as candidate `manifest.json` in `commit-candidate/`.
4. **Validate the complete prospective state.** Parse candidate `manifest.json`; validate candidate `final-plan.md`, candidate `shot-list.csv`, and the candidate image/image-reference metadata; verify the CSV header and every 45-column row against the deterministic mapping below; verify Markdown frame IDs; validate the six project lifecycle states, three review decisions, shot-list enum values, final paths, selected active version, appended export records, and image references. Require the candidate manifest bytes intended for `project.json` and `final/manifest.json` to be identical.
5. **Promote and commit as one final step.** Preserve the prior `final/` and project manifest as rollback data. Promote the candidate plan, CSV, image/image-reference metadata, and manifest as one directory-level set, then write the exact same buffered candidate manifest bytes to both `project.json` and `final/manifest.json`. Verify the two manifest files are byte-for-byte identical before discarding rollback data and reporting success.

Allowed shot-list values are:

- `status`: `proposed`, `approved`, `assigned`, `captured`, `completed`, `omitted`
- `reference_type`: `internal`, `external`, `uploaded`, `AI-generated`, `none`
- `capture_type`: `must-capture`, `inspiration`, `optional`, `alternate`

## Deterministic 45-column mapping

Emit columns in the exact order below. `frame` means the canonical record from the selected iteration's `frames.json`; `manifest` means the prospective committed candidate.

| # | CSV column | Canonical source |
| ---: | --- | --- |
| 1 | `project_id` | `manifest.project_id` |
| 2 | `storyboard_id` | `manifest.storyboard_id` |
| 3 | `storyboard_version` | `manifest.active_version` |
| 4 | `frame_id` | `frame.frame_id` |
| 5 | `sequence_number` | `frame.sequence_number` |
| 6 | `shot_name` | `frame.title` |
| 7 | `scene_name` | `frame.metadata.scene_name` |
| 8 | `description` | `frame.description` |
| 9 | `narrative_purpose` | `frame.metadata.narrative_purpose` |
| 10 | `subject` | `frame.metadata.subject` |
| 11 | `action` | `frame.metadata.action` |
| 12 | `location` | `frame.metadata.location` |
| 13 | `time_of_day` | `frame.metadata.time_of_day` |
| 14 | `shot_size` | `frame.metadata.shot_size` |
| 15 | `camera_angle` | `frame.metadata.camera_angle` |
| 16 | `camera_height` | `frame.metadata.camera_height` |
| 17 | `camera_position` | `frame.metadata.camera_position` |
| 18 | `focal_length_mm` | `frame.metadata.focal_length_mm` |
| 19 | `lens_type` | `frame.metadata.lens_type` |
| 20 | `aperture_intent` | `frame.metadata.aperture_intent` |
| 21 | `depth_of_field` | `frame.metadata.depth_of_field` |
| 22 | `camera_movement` | `frame.metadata.camera_movement` |
| 23 | `camera_support` | `frame.metadata.camera_support` |
| 24 | `frame_rate_fps` | `frame.metadata.frame_rate_fps` |
| 25 | `playback_intent` | `frame.metadata.playback_intent` |
| 26 | `duration_seconds` | `frame.metadata.duration_seconds` |
| 27 | `transition_to_next` | `frame.metadata.transition_to_next` |
| 28 | `lighting` | `frame.metadata.lighting` |
| 29 | `weather` | `frame.metadata.weather` |
| 30 | `wardrobe` | `frame.metadata.wardrobe` |
| 31 | `props` | `frame.metadata.props` |
| 32 | `priority` | `frame.metadata.priority` |
| 33 | `capture_type` | `frame.metadata.capture_type` |
| 34 | `assigned_shooter` | `frame.metadata.assigned_shooter` |
| 35 | `scheduled_time` | `frame.metadata.scheduled_time` |
| 36 | `reference_type` | `frame.metadata.reference_type` |
| 37 | `reference_path` | `frame.metadata.reference_path` |
| 38 | `image_path` | `frame.image_paths.frame` |
| 39 | `prompt` | `frame.prompt` |
| 40 | `negative_prompt` | `frame.negative_prompt` |
| 41 | `continuity_notes` | `frame.metadata.continuity_notes` |
| 42 | `production_notes` | `frame.metadata.production_notes` |
| 43 | `status` | `frame.metadata.status` |
| 44 | `created_at` | `frame.metadata.created_at` |
| 45 | `updated_at` | `frame.metadata.updated_at` |

Serialize list-valued metadata such as wardrobe or props as semicolon-delimited text inside one RFC 4180-quoted field. Use an empty field for an absent optional value; never shift columns or invent a second source.

## Failure behavior

On source, generation, prospective-manifest, candidate validation, promotion, manifest-write, or byte-equivalence failure, report the exact failure, leave `commit-candidate/` isolated for inspection, restore any promotion rollback, and leave `project.json`, prior `final/`, `active_version`, `status`, and `exports` unchanged. Never report commit or export success without the promoted validated files and identical committed manifest state.
