# AI Storyboard Codex Plugin Foundation

**Status:** Design approved for written-spec review  
**Date:** 2026-07-21  
**Scope:** Hackathon foundation for a portable, no-code Codex plugin paired with ChatGPT image generation

## 1. Product goal

Create a portable Codex plugin that turns a brief, script, concept, or shot description into a consistent, production-readable storyboard workflow. The plugin should make the user experience feel like one smart `storyboard` flow while internally routing work through focused skills and a file-based project model.

The first milestone validates the end-to-end planning workflow rather than building a hosted application. A user should be able to start from one meaningful creative-intent statement, review a generated plan, hand it to ChatGPT image generation, refine individual frames, and commit durable Markdown, CSV, JSON, and image references inside the plugin repository.

## 2. Design principles

- Story information is the only mandatory creative input. Shoot and scene details may be inferred when reasonable.
- Fast first value comes before a long questionnaire. Questions must materially affect the storyboard.
- Defaults are recommendations, not hidden decisions. Inferred values are labeled clearly.
- Consistency is prioritized over novelty. Project-wide constants and frame-level locks are explicit.
- A frame refinement must preserve unaffected frame records, prompts, metadata, and assets as closely as the image-generation capability permits.
- The manifest, Markdown plan, CSV shot list, and iteration snapshots are the durable source of truth; the chat thread is not.
- The plugin contains no hosted service, custom runtime, database, or image-processing implementation.

## 3. Plugin architecture

### User-facing entry point

The plugin exposes one top-level `storyboard` flow. It determines the user’s intent and current project stage from the request and the active project manifest. Users do not need to know the names of internal skills.

### Internal associated skills

The top-level flow delegates to focused instruction sets:

1. **Intake** — normalize briefs, scripts, concepts, schedules, shot lists, and references; extract known facts; identify only high-impact gaps.
2. **Planning** — create the project summary, creative constants, production assumptions, story progression, frame plan, prompts, and negative constraints.
3. **Review** — present assumptions, warnings, frame sequence, and editable plan decisions; support approval, edits, reordering, adding/removing frames, alternatives, and review bypass.
4. **Generation handoff** — build a complete prompt packet for ChatGPT image generation and record the returned image reference and settings.
5. **Refinement** — identify affected frame IDs, preserve unaffected records, update only requested changes, warn about continuity impact, and create a new iteration.
6. **Versioning** — list, compare, restore, and branch storyboard iterations without destroying prior snapshots.
7. **Reference management** — store uploaded, external, internal, and AI-generated reference metadata and associate references with the project or specific frames.
8. **Commit and export** — lock the selected iteration, validate output completeness, write the final plan and CSV, and record final artifact metadata.

### Project-local overrides

Each project may override the starter guides by placing replacements in its own `guides/` directory. Project-local instructions take precedence over plugin defaults but remain below explicit user instructions and approved frame changes.

### Starter guide placeholders

The plugin will include non-official, replaceable starter guide files with usable baseline values:

- **Story:** six frames for a general short concept, covering establishing, action, detail, reaction, progression, and resolution. Frame count may be changed when the narrative or duration warrants it.
- **Shoot:** 16:9 delivery, 23.976 fps, a general 18/25/35/50/85/100mm lens family, tripod for static shots, handheld for action or intimacy, and gimbal/dolly for controlled movement. 120 fps is recommended only when slow motion serves the story.
- **Scene:** preserve one coherent location, time of day, lighting direction, wardrobe, and prop set across frames unless the approved plan intentionally changes them.
- **Styles:** placeholder definitions for cinematic realism, documentary sports, graphic storyboard sketch, and clean pitch-frame. These are provisional and must not be presented as official Two Circles brand guidance.

## 4. Smart flow and state routing

The flow first resolves the project context, then classifies intent, then loads only the relevant internal skill and guides.

| User request or state | Routed action |
|---|---|
| New brief, script, concept, or reference | Intake |
| Missing plan, “make a plan,” add/remove/reorder frames | Planning |
| “Review,” “what assumptions did you make?” | Review |
| Approved plan plus “generate/create” | Generation handoff |
| “Change frame 3,” “make this wider,” “keep everything else” | Refinement |
| “Show versions,” “restore,” “branch” | Versioning |
| “Finalize,” “commit,” “export,” “make a shot list” | Commit and export |

The normal lifecycle is:

`intake → plan → review → generate → refine/version → commit/export`

Review is enabled by default. The user may explicitly bypass it with requests such as “skip review,” “generate immediately,” or “use your defaults.” A bypass still stores the internal plan and its assumptions.

If a project is reopened, the manifest determines the next valid action. A committed project may be inspected, branched, restored, or exported again without relying on prior conversation history.

## 5. Canonical project model

Each project has one source-of-truth `project.json` manifest. The manifest includes:

- `project_id`, `title`, `schema_version`, `status`, and `active_version`.
- `story`, `shoot`, `scene`, and `style` context.
- `creative_constants` and explicit continuity locks.
- `references` with type, source, rights note, applicable frames, and intended borrowing constraints.
- `frames` with stable IDs, sequence numbers, descriptions, prompts, negative prompts, metadata, continuity locks, references, image paths, and revision history.
- `versions` with immutable iteration snapshots, plans, prompts, image references, settings, timestamps, and change summaries.
- `exports` with artifact type, path, version, validation result, and timestamp.

Frame IDs are stable (`frame-001`, `frame-002`); `sequence_number` may change when frames are reordered. User-approved values and inferred values are distinguished in the data and displayed in the workflow.

Context precedence is:

1. Latest explicit user instruction.
2. Approved frame-specific instruction.
3. Approved project plan.
4. Project-local guide.
5. Plugin starter guide.
6. System inference.

Locked values cannot be changed silently. A request that conflicts with a lock must identify the conflict and request explicit approval before changing it.

## 6. Image-generation handoff

The plugin does not implement or host an image provider. It prepares a generation packet for the ChatGPT image-generation capability.

The packet contains:

- Approved project summary and story progression.
- Project-wide creative constants and negative constraints.
- Selected provisional or project-local style guide.
- Aspect ratio and generation intent.
- Frame-by-frame descriptions, prompts, metadata, continuity requirements, and reference associations.
- A request for a numbered, coherent low-resolution storyboard contact sheet.

The default low-resolution output is a numbered contact sheet. Per-frame images are recorded when generated or supplied, but a custom compositor is not part of this no-code foundation. The exact prompt packet and returned image reference are stored in the iteration manifest.

For refinement, the flow first lists affected frames and possible continuity impacts. It regenerates only requested frames when the image-generation capability supports that behavior. Otherwise, it preserves unaffected frame assets and records the limitation and any suspected drift. It never silently regenerates the whole sequence after a frame-specific request.

## 7. File layout

The plugin repository will contain the plugin definition, skills, starter guides, templates, and portable projects:

```text
<plugin-repo>/
├── .codex-plugin/
│   └── plugin.json
├── skills/
│   ├── storyboard/
│   ├── storyboard-intake/
│   ├── storyboard-planning/
│   ├── storyboard-review/
│   ├── storyboard-generation/
│   ├── storyboard-refinement/
│   ├── storyboard-versioning/
│   ├── storyboard-references/
│   └── storyboard-commit-export/
├── guides/
│   ├── story-guide.md
│   ├── shoot-guide.md
│   ├── scene-guide.md
│   └── styles/
├── templates/
│   ├── project.json
│   ├── project-readme.md
│   ├── plan.md
│   └── shot-list.csv
└── storyboard-projects/
    └── <project-slug>/
        ├── project.json
        ├── README.md
        ├── guides/
        ├── references/
        │   ├── uploaded/
        │   ├── external/
        │   └── internal/
        ├── plans/
        ├── iterations/
        ├── final/
        └── exports/
```

Project folders are saved inside the plugin repository so the complete project can be exported, reopened, and shared as a portable file set.

## 8. Commit and export behavior

Commit locks the selected iteration and writes:

- A final storyboard image reference, with high-resolution output requested at commit where the image-generation capability supports it.
- `final-plan.md` containing the approved project summary and machine-readable frame headings.
- `shot-list.csv` using the expanded schema from the product specification.
- A final manifest snapshot.
- Generation settings, source references, timestamps, and a concise change summary.

The high-resolution request must preserve the approved composition. If exact deterministic upscaling is unavailable and regeneration is required, the artifact metadata and user-facing summary must warn that small visual differences may occur.

The plugin must validate stable frame IDs, required manifest fields, CSV column names, prompt presence, status values, output paths, and Markdown frame identifiers before marking commit complete.

## 9. Failure handling and safety

- Missing meaningful story input: ask for the smallest useful creative-intent statement.
- Ambiguous high-impact decisions: provide recommended defaults and a small set of alternatives.
- Physically or logistically implausible shot: explain the issue and suggest an executable alternative.
- Generation failure: preserve the prior plan and active iteration; do not overwrite approved outputs.
- Continuity risk: identify which constants or frames may drift and ask for confirmation when a locked value is affected.
- Invalid export: report the exact validation failure and leave the last valid export untouched.
- Reference rights uncertainty: record the uncertainty and do not imply usage approval.

## 10. Acceptance runbook

The first manual test uses the supplied soccer production example:

1. Import or describe the mural, streetcar, and high-school-field sequence.
2. Generate an eight-frame plan using the practical lens, movement, frame-rate, and shot-size variations.
3. Review the plan and verify assumptions are labeled.
4. Generate a numbered low-resolution contact sheet through ChatGPT image generation.
5. Change only the streetcar frame and verify unaffected frame records and prompts remain unchanged.
6. Replace one frame with an uploaded reference and verify the reference is frame-scoped.
7. Inspect iteration history and the change summary.
8. Commit the result and validate the Markdown plan, CSV shot list, manifest, and image references.
9. Reopen the project from `storyboard-projects/<project-slug>/` without the original chat thread.

The foundation is successful when the user can complete this flow from one top-level storyboard command while retaining readable, portable project files at every stage.

## 11. Out of scope for this foundation

Hosted APIs, SharePoint integration, multi-user collaboration, live production tracking, client approval portals, mobile-specific UI, PDF/PowerPoint/Word export, custom image compositing, and a standalone web application are deferred. The manifest and skill boundaries leave room for these later without changing the core project model.
