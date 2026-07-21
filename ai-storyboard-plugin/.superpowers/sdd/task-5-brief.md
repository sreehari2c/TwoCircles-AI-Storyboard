# Task 5: Write the top-level smart storyboard flow

## Files

Create:
- skills/storyboard/SKILL.md

## Public skill metadata

Start with exactly:

    ---
    name: storyboard
    description: Turn a creative brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
    ---

## Project discovery order

Document and follow this exact order:
1. If the user names a project, resolve storyboard-projects/<project-slug>/project.json.
2. If the current workspace has one active storyboard project, use it.
3. If multiple projects exist and none is named, ask the user to choose.
4. If no project exists and the user supplied meaningful story input, create one from templates.
5. If no project exists and no meaningful story input was supplied, ask for one sentence of creative intent.

## Routing table

Include all of these routes:
- New brief, script, concept, or reference -> storyboard-intake.
- Missing plan, make a plan, add/remove/reorder frames -> storyboard-planning.
- Review or assumptions request -> storyboard-review.
- Approved plan plus generate/create -> storyboard-generation.
- Change frame, make wider, keep everything else -> storyboard-refinement.
- Show versions, restore, branch -> storyboard-versioning.
- Finalize, commit, export, make a shot list -> storyboard-commit-export.
- Reference attachment or reference association -> storyboard-references.

## Lifecycle/defaults

Document:
- Normal lifecycle: intake -> plan -> review -> generate -> refine/version -> commit/export.
- Review is enabled by default.
- skip review, generate immediately, and use your defaults bypass review while retaining the internal plan.
- Load only the relevant internal skill and guides for the current request.
- General concepts default to six frames; the soccer benchmark uses eight.
- Starter guides are provisional; project-local guides override them.
- Context precedence: latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, system inference.
- project.json is source of truth; frame_id is stable and sequence_number may change.

## User-visible responses

Require concise structured responses showing:
- current stage;
- changed files;
- inferred assumptions;
- warnings/conflicts;
- frame IDs affected;
- next action.

For image generation, show exact frame prompts or a readable prompt summary. For refinement, show a written change summary and untouched frame IDs.

## Stop conditions

Stop for explicit user choice when:
- a lock would change;
- a reference rights issue is unresolved;
- a high-impact ambiguity cannot be resolved with a recommendation;
- commit validation fails.

Never claim image generation, export, or commit success without the corresponding file or reference being present. Never silently regenerate an entire storyboard after a frame-specific request.

## Validation

    $content = Get-Content skills/storyboard/SKILL.md -Raw
    foreach ($name in @('storyboard-intake','storyboard-planning','storyboard-review','storyboard-generation','storyboard-refinement','storyboard-versioning','storyboard-references','storyboard-commit-export')) {
      if ($content -notmatch [regex]::Escape($name)) { throw "Top-level flow does not reference $name" }
    }
    foreach ($phrase in @('skip review','frame-001','storyboard-projects','project.json','ChatGPT image generation')) {
      if ($content -notmatch [regex]::Escape($phrase)) { throw "Top-level flow is missing required contract phrase: $phrase" }
    }

Expected: all eight internal skills and all five contract phrases are referenced.

Commit:
    git add skills/storyboard/SKILL.md
    git commit -m "feat: add smart storyboard orchestrator"

## Context

Tasks 1–4 created the plugin scaffold, guides, templates, and internal skills in the isolated worktree. This file is the only user-facing skill. Do not add runtime code or a separate user-facing skill per stage.

## Report

Write the detailed report to .superpowers/sdd/task-5-report.md. Include files changed, commit SHA, validation output, self-review, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

