---
name: storyboard
description: Use when turning a creative brief, script, concept, reference, or shot description into a storyboard plan, image-generation handoff, refinement, version, or production shot list.
---

# Smart Storyboard

Coordinate one storyboard project from creative intent through a reviewable plan, image-generation handoff, targeted revisions, and production exports. This is the plugin's only registered skill.

Resolve every `workflow/...` and `resources/...` path below relative to this `skills/storyboard/` directory. Load only the routed ordinary workflow reference; these files and all resources are non-discoverable and must not be offered as independently triggerable skills or commands.

## Project discovery

Follow this order exactly:

1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`; an explicitly named committed project may be reopened.
2. Scan `storyboard-projects/*/project.json` for project manifests. A project is active when the current workspace is inside that project's folder, or when exactly one project manifest exists under `storyboard-projects/`.
3. If multiple manifests exist and none is in the current project folder, ask the user to choose; do not guess based on file ordering or timestamps.
4. If no project exists and the user supplied meaningful story input, create one from `resources/templates/` with `storyboard-intake`.
5. If no project exists and no meaningful story input was supplied, ask for one sentence of creative intent.

Treat `project.json` as the source of truth. Before routing an existing project, require `status` to be one of `proposed`, `planned`, `review`, `generated`, `refined`, or `committed`, and require `active_version` to be an integer. Stop with a manifest error when either value is missing or invalid.

## Routing

Load only the relevant internal workflow reference and guides for the current request. Route requests as follows:

| Stage route name | Request | Load |
| --- | --- | --- |
| `storyboard-intake` | New brief, script, concept, or project import | `workflow/storyboard-intake.md` |
| `storyboard-planning` | Missing plan or make a plan from `proposed`/`planned` | `workflow/storyboard-planning.md` |
| `storyboard-review` | Review, assumptions, or add/remove/reorder/edit-plan while status is `review` | `workflow/storyboard-review.md` |
| `storyboard-generation` | Generate/create with durable review authorization | `workflow/storyboard-generation.md` |
| `storyboard-refinement` | Change frame, make wider, keep everything else | `workflow/storyboard-refinement.md` |
| `storyboard-versioning` | Show versions, restore, branch | `workflow/storyboard-versioning.md` |
| `storyboard-commit-export` | Finalize, commit, export, make a shot list | `workflow/storyboard-commit-export.md` |
| `storyboard-references` | Reference attachment or reference association | `workflow/storyboard-references.md` |

Explicit stage verbs control routing only when the requested transition is valid. When status is `review`, route add/remove/reorder/edit-plan requests to `storyboard-review`; that workflow writes the edit and returns status to `planned`. Never route that case directly to planning.

For requests without a stage verb, route from `project.json.status`: `proposed` to planning; `planned` to review; `review` to generation only when `review.authorized_for_generation` is `true` and `review.decision` is `approved` or `bypassed`, otherwise back to review; `generated` or `refined` to a current-iteration summary offering refinement or commit; and `committed` to a final-state summary offering inspect, branch, restore, or export. Never infer a route from missing or invalid state.

## Defaults and context

- Review is enabled by default. `skip review`, `generate immediately`, and `use your defaults` are explicit bypass decisions, not permission inferred from chat history. Record the durable review object before generation.
- General concepts default to six frames; the soccer benchmark uses eight unless an approved frame count overrides it.
- Starter guides under `resources/guides/` are provisional; project-local guides override them.
- Apply context precedence in this order: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, system inference.
- Keep supplied, approved, and inferred values distinct. Do not change an approved lock without explicit user choice.
- For generation, require durable authorization, hand approved prompts to ChatGPT image generation, and retain `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md` for every iteration with the returned image reference.
- Use the canonical frame record and 45-column export mapping defined by `resources/templates/project.json`, `resources/templates/project-readme.md`, and `workflow/storyboard-commit-export.md`.

## Stage handling

1. Discover and validate the project using this public skill's project discovery and routing contracts.
2. Load exactly one routed stage reference unless a valid multi-stage request requires the smallest necessary sequence.
3. Apply lifecycle transitions only after the routed stage's durable artifacts exist and pass its validation.
4. Associate references through `storyboard-references` before another stage relies on them.
5. Preserve stable `frame_id` values, unaffected records, immutable iterations, and prior valid finals.
6. Treat `review` as durable state: planning or plan edits reset it to pending and unauthorized; approval or explicit bypass records `plan_version`, `reviewed_at`, and authorization before generation.

## User-visible response format

Keep responses concise and structured. Include:

- **Current stage:** the routed workflow stage and project.
- **Changed files:** created or modified paths, or `none` before a write.
- **Inferred assumptions:** labeled defaults or recommendations.
- **Warnings/conflicts:** unresolved rights, locks, ambiguities, or validation issues.
- **Frame IDs affected:** affected stable IDs and, for refinements, untouched IDs.
- **Next action:** the required decision or the next safe stage.

For image generation, include exact frame prompts or a readable prompt summary. For refinement, include a written change summary and untouched frame IDs.

## Stop conditions

Stop for explicit user choice when:

- a lock would change;
- a reference rights issue is unresolved;
- a high-impact ambiguity cannot be resolved with a recommendation; or
- commit validation fails.

Never claim image generation, export, or commit success without the corresponding file or reference being present. Preserve prior approved records and outputs when a later operation fails.
