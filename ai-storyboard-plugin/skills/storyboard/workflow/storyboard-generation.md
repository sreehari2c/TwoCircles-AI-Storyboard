# Storyboard Generation Workflow

Create an immutable first or subsequent generated iteration from an approved or explicitly review-bypassed plan.

## Inputs

Read `project.json`, the selected `plans/plan-v###.md`, all canonical frame records, skill-relative style guidance, constants, negative constraints, aspect ratio, and references. Require status `review`, `review.authorized_for_generation: true`, `review.decision` equal to `approved` or `bypassed`, and `review.plan_version` equal to the selected plan version. Otherwise route to `storyboard-review` without creating generation artifacts.

## Four-file iteration contract

Initial generation writes exactly these required artifacts under `iterations/v001/`:

- `iterations/v001/prompt-packet.md`
- `iterations/v001/frames.json`
- `iterations/v001/result.json`
- `iterations/v001/changes.md`

Every later generation uses the same names under its next immutable `iterations/v###/` folder.

## Procedure

1. Build `prompt-packet.md` from the approved plan, constants, negative constraints, selected style, aspect ratio, references, and numbered frame prompts.
2. Request a coherent numbered low-resolution contact sheet through ChatGPT image generation.
3. After a returned image reference exists, write `frames.json` as the complete array of canonical frame records, including image paths; write `result.json` with the contact-sheet reference, settings, validation result, and timestamp; and write `changes.md` with the generation scope and summary.
4. Validate all four files and their frame IDs before mutating project state.
5. Copy the complete frame array to `project.json.frames`; append a version record containing `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, a complete snapshot of the five review fields, `created_at`, and `change_summary`; set integer `active_version`; then set status to `generated`.

## Failure behavior

If image generation or artifact validation fails, do not update `project.json.frames`, `project.json.versions`, `active_version`, or status. Do not claim generation success until the image reference and all four files exist and validate.
