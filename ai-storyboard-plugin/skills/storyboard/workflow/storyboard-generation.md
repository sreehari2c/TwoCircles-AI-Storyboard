# Storyboard Generation Workflow

Create an immutable first or subsequent generated iteration from an approved or explicitly review-bypassed plan.

## Inputs

Read `project.json`, the selected `plans/plan-v###.md`, all canonical frame records, skill-relative style guidance, `resources/templates/Storyboard_Template.jpg`, `resources/templates/Storyboard Example.png`, constants, negative constraints, aspect ratio, and references. Require status `review`, `review.authorized_for_generation: true`, `review.decision` equal to `approved` or `bypassed`, and `review.plan_version` equal to the selected plan version. Otherwise route to `storyboard-review` without creating generation artifacts.

## Four-file iteration contract

Initial generation writes exactly these required artifacts under `iterations/v001/`:

- `iterations/v001/prompt-packet.md`
- `iterations/v001/frames.json`
- `iterations/v001/result.json`
- `iterations/v001/changes.md`

Every later generation uses the same names under its next immutable `iterations/v###/` folder.

## Procedure

1. Build `prompt-packet.md` from the approved plan, constants, negative constraints, selected style, aspect ratio, references, template/layout decision, and numbered frame prompts.
2. Request a coherent low-resolution storyboard sheet through ChatGPT image generation using `Storyboard_Template.jpg` as the layout reference and `Storyboard Example.png` as the art-style reference.
3. The generated sheet must use the black presentation-board layout, grayscale pencil/ink panels on subtle paper texture, bold white title/project text, and concise white captions under filled frames. For fewer than 12 planned frames, unused template slots remain black. For more than 12 frames, follow the approved multi-image or resized/reflowed layout decision.
4. After a returned image reference exists, write `frames.json` as the complete array of canonical frame records, including image paths; write `result.json` with the storyboard sheet reference, template assets used, layout decision, settings, validation result, and timestamp; and write `changes.md` with the generation scope and summary.
5. Validate all four files and their frame IDs before mutating project state.
6. Copy the complete frame array to `project.json.frames`; append a version record containing `version_id`, `status`, `plan_path`, `prompt_packet_path`, `frames_path`, `result_path`, `changes_path`, a complete snapshot of the five review fields, `created_at`, and `change_summary`; set integer `active_version`; then set status to `generated`.

## Failure behavior

If image generation or artifact validation fails, do not update `project.json.frames`, `project.json.versions`, `active_version`, or status. Do not claim generation success until the image reference and all four files exist and validate.
