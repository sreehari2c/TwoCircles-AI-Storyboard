# AI Storyboard Codex Plugin Foundation

**Status:** Design approved, including the second final-review architecture amendment
**Date:** 2026-07-21
**Scope:** Hackathon foundation for a portable, no-code Codex plugin paired with ChatGPT image generation

## 1. Product goal

Create a portable Codex plugin that turns a brief, script, concept, or shot description into a consistent, production-readable storyboard workflow. The user experiences one smart `storyboard` flow. That public skill routes to private workflow references and persists all operational state in a file-based project model.

The foundation validates intake, planning, durable review, image-generation handoff, scoped refinement, versioning, and transactional export. It contains no hosted service, custom runtime, database, or image-processing implementation.

## 2. Design principles

- Story information is the only mandatory creative input; shoot and scene details may be inferred when reasonable.
- Questions must materially affect the storyboard. Defaults and inferred values are labeled.
- Project-wide constants and frame locks preserve consistency.
- A refinement preserves unaffected frame records, prompts, metadata, references, and assets.
- `project.json`, plans, iteration snapshots, final manifests, and CSV exports are durable truth; chat history is not.
- Failed generation or export never mutates the last valid project or final state.

## 3. Architecture amendment: one registered skill

Only `skills/storyboard/SKILL.md` is a registered skill. It owns project discovery, routing, shared invariants, and user-visible response shape.

Eight stage instruction sets live under `skills/storyboard/workflow/`:

1. `storyboard-intake.md`
2. `storyboard-planning.md`
3. `storyboard-review.md`
4. `storyboard-generation.md`
5. `storyboard-refinement.md`
6. `storyboard-versioning.md`
7. `storyboard-references.md`
8. `storyboard-commit-export.md`

Starter guides and templates live under `skills/storyboard/resources/`. Workflow and resource files are loaded relative to `skills/storyboard/`; they are non-discoverable files, not internal skills or independent commands.

The portable project store remains at plugin-root `storyboard-projects/<project-slug>/`. Project-local `guides/` override the skill's starter guides, below explicit user instructions and approved project/frame decisions.

The starter guides remain provisional and must not be represented as official Two Circles guidance. Defaults include six frames for a general short concept, eight for the soccer benchmark, 16:9 delivery, 23.976 fps, the 18/25/35/50/85/100mm lens family, and camera movement/support chosen to serve the story. Generated sheets use `resources/templates/Storyboard_Template.jpg` as the 12-frame master board and `resources/templates/Storyboard Example.png` as the required art-style reference.

## 4. Routing and lifecycle

The public skill discovers and validates the project first, then routes by valid intent and durable state:

| Request or state | Workflow |
| --- | --- |
| New brief, script, concept, or import | Intake |
| Missing plan or make-plan request from `proposed`/`planned` | Planning |
| Review, assumptions, approval, or bypass | Review |
| Add/remove/reorder/edit-plan while status is `review` | Review; write edit, reset authorization, return to `planned` |
| Generate/create with durable authorization | Generation |
| Scoped frame change | Refinement |
| List, compare, restore, or branch versions | Versioning |
| Attach or associate a reference | References |
| Finalize, commit, export, or shot list | Commit/export |

The six lifecycle states are:

`proposed -> planned -> review -> generated -> refined -> committed`

Without an explicit stage verb, route `proposed` to planning, `planned` to review, and `review` to generation only when durable authorization is true and the decision is approved or bypassed. Otherwise `review` routes back to review. Generated/refined projects offer refinement or commit. Committed projects offer inspect, branch, restore, or export.

## 5. Durable review authorization

Every manifest contains:

```json
"review": {
  "decision": "pending",
  "plan_version": null,
  "authorized_for_generation": false,
  "reviewed_at": null,
  "change_summary": ""
}
```

Allowed decisions are `pending`, `approved`, and `bypassed`.

- Initial planning and every plan edit reset review to pending, clear plan version/timestamp, and set authorization false.
- Approval writes `approved`; an explicit `skip review`, `generate immediately`, or `use your defaults` request writes `bypassed`.
- Approval and bypass both record the reviewed plan version, ISO 8601 UTC timestamp, authorization true, and a concise change summary while status remains `review`.
- Generation requires status `review`, authorization true, an approved/bypassed decision, and a matching plan version. Otherwise it routes to review.
- Every generated, refined, restored, or branched version record snapshots all five review fields.

## 6. Canonical project and frame model

`project.json` is the source of truth. It includes project/storyboard identity, six-state lifecycle, integer active version, durable review, story/shoot/scene/style context, creative constants, references, canonical frames, immutable versions, and export records.

Stable frame IDs (`frame-001`) identify frames across revisions. `sequence_number` alone changes for reordering. Every canonical frame includes:

- identity, title, description, prompts, continuity locks, reference IDs, image paths, and `revision_history`;
- exact metadata keys for narrative and production values;
- exact export fields needed for the 45-column shot list, including `scene_name`, `narrative_purpose`, `time_of_day`, `camera_height`, `aperture_intent`, `playback_intent`, `priority`, `capture_type`, `assigned_shooter`, `scheduled_time`, `reference_type`, `production_notes`, `status`, `created_at`, and `updated_at`.

The template defines every metadata key and the commit/export workflow defines the deterministic mapping for all 45 CSV columns.

Context precedence is:

1. Latest explicit user instruction.
2. Approved frame-specific instruction.
3. Approved project plan.
4. Project-local guide.
5. Skill starter guide.
6. System inference.

Locked values cannot change silently.

## 7. Image-generation and iteration contract

The plugin prepares a prompt packet for ChatGPT image generation; it does not implement an image provider. Every generated, refined, restored, or branched `iterations/v###/` folder contains exactly these required artifacts:

- `prompt-packet.md`
- `frames.json`
- `result.json`
- `changes.md`

Initial generation requests a coherent low-resolution storyboard sheet that follows the supplied 12-frame board and art-style reference. For fewer than 12 planned frames, unused slots stay black instead of being filled with invented frames. For more than 12 planned frames, the approved plan chooses multiple storyboard images or a deliberate resized/reflowed layout before generation. Refinement identifies affected and untouched frame IDs. When frame-only generation is unavailable and no compositor exists, the user chooses whole-sheet regeneration with drift warning, a frame-only artifact retaining the previous sheet, or metadata-only revision. No workflow silently regenerates the whole sequence.

## 8. Portable file layout

```text
<plugin-repo>/
|-- .codex-plugin/
|   `-- plugin.json
|-- skills/
|   `-- storyboard/
|       |-- SKILL.md
|       |-- workflow/
|       |   |-- storyboard-intake.md
|       |   |-- storyboard-planning.md
|       |   |-- storyboard-review.md
|       |   |-- storyboard-generation.md
|       |   |-- storyboard-refinement.md
|       |   |-- storyboard-versioning.md
|       |   |-- storyboard-references.md
|       |   `-- storyboard-commit-export.md
|       `-- resources/
|           |-- guides/
|           `-- templates/
`-- storyboard-projects/
    `-- <project-slug>/
        |-- project.json
        |-- README.md
        |-- guides/
        |-- references/
        |-- plans/
        |-- iterations/
        |-- final/
        `-- exports/
```

Package validation counts one public `SKILL.md`, eight workflow Markdown files, and the guide/template resources. Root `workflow/`, `guides/`, and `templates/` copies are invalid.

The template resources must include `Storyboard_Template.jpg` and `Storyboard Example.png` so the portable plugin can reproduce the required storyboard layout and sketch style without relying on the original workspace folder.

## 9. State-consistent commit and export

Commit/export is one transaction:

1. Validate the selected source iteration and its durable review evidence.
2. Build candidate final plan, 45-column CSV, and high-resolution image/image-reference metadata under `iterations/v###/commit-candidate/`.
3. Build a prospective committed manifest with status `committed`, selected integer `active_version`, and all export records already appended.
4. Validate candidate manifest, plan, CSV, and image together, including enum values, frame IDs, all paths, and all 45 deterministic mappings.
5. Promote the artifact set and write the same buffered candidate manifest bytes to both project-root `project.json` and `final/manifest.json` as the final commit step.

The two committed manifests must be byte-for-byte identical. Promotion keeps rollback data until both manifest writes and equivalence validation succeed. On any failure, prior `project.json`, `final/`, active version, status, and exports remain unchanged; the isolated candidate may remain for inspection.

## 10. Failure handling and safety

- Ask for the smallest useful creative-intent statement when story input is absent.
- Explain implausible production choices and recommend executable alternatives.
- Stop for lock conflicts, unresolved rights, invalid manifests, and high-impact ambiguity.
- Preserve prior plan/state after generation failure.
- Preserve prior final/state after candidate or promotion failure.
- Never imply that reference rights are approved when uncertain.

## 11. Acceptance runbook

The soccer benchmark imports the mural, streetcar, and high-school-field sequence; builds an eight-frame plan; verifies durable review evidence; generates a storyboard sheet using the supplied template and example style; confirms the four unused slots remain black; changes only the streetcar frame; applies one frame-scoped uploaded reference; inspects immutable history; commits and validates plan, all 45 CSV columns, identical manifests, and image references; then reopens from `storyboard-projects/<project-slug>/` without chat history.

## 12. Out of scope

Hosted APIs, SharePoint integration, multi-user collaboration, live production tracking, client portals, mobile UI, Office/PDF export, custom image compositing, and a standalone web application are deferred.
