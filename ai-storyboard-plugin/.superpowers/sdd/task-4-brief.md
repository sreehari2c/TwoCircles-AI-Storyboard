# Task 4: Write the internal stage skills

## Files

Create:
- skills/storyboard-intake/SKILL.md
- skills/storyboard-planning/SKILL.md
- skills/storyboard-review/SKILL.md
- skills/storyboard-generation/SKILL.md
- skills/storyboard-refinement/SKILL.md
- skills/storyboard-versioning/SKILL.md
- skills/storyboard-references/SKILL.md
- skills/storyboard-commit-export/SKILL.md

## Shared requirements

Every file must begin with YAML frontmatter containing:
- a unique lowercase name matching its folder purpose;
- a one-sentence description.

Every file must state:
- Inputs it reads.
- Files it may create or update.
- Outputs it produces for the next stage.
- Conditions under which it must stop and ask the user.
- Invariants it must preserve.

The skills are internal capabilities loaded by the top-level storyboard flow. They should not instruct users to invoke separate skill names as the primary interface.

## Individual contracts

### storyboard-intake

Extract supplied story, shoot, scene, style, reference, duration, frame-count, aspect-ratio, and production constraints. Avoid asking for supplied facts. Mark inferred values. Create a project slug. Create a project folder under storyboard-projects/<project-slug>/ from templates/project.json and templates/project-readme.md. Create required subfolders. Stop with a normalized intake summary before planning.

### storyboard-planning

Load the applicable plugin guides and project-local overrides. Create stable frame IDs such as frame-001. Produce the project summary, creative constants, production assumptions, story progression, frame plan, prompts, negative constraints, and continuity requirements. Separate inferred assumptions from approved values. Write plans/plan-v001.md and update project.json without creating images.

### storyboard-review

Show project summary, creative constants, frame sequence, assumptions, warnings, and conflicts. Support approve, edit overall plan, edit/add/remove/reorder one frame, request alternatives, skip review, and proceed to creation. Review is default. Never generate images in this skill.

### storyboard-generation

Build a prompt packet containing approved plan, constants, negative constraints, selected style, aspect ratio, references, and frame prompts. Request a numbered, coherent low-resolution contact sheet through ChatGPT image generation. Record the exact packet and returned image reference under iterations/v001/. Update project.json only after the image result is available. If generation fails, preserve the prior plan and active version.

### storyboard-refinement

Identify affected frame IDs. Classify whether the request changes a project constant or only frame data. List continuity risks. Preserve unaffected frame records, prompts, metadata, and assets. Create the next iteration and write changes.md. Support one-frame changes, multi-frame changes, reference replacement, same-prompt regeneration, and sequence-ending replacement requests. Never silently regenerate the whole sequence for a frame-specific request.

### storyboard-versioning

List iteration folders and change summaries. Restore a whole version by creating a new branch snapshot. Restore one frame by copying its prior frame record into a new iteration. Branch from an existing version without deleting history. Never overwrite an approved iteration.

### storyboard-references

Support reference types internal, external, uploaded, AI-generated, and none. Store reference ID, type, source/path/URL, rights note, applicable frames, what to borrow, and what not to copy. Do not assume an uploaded reference applies to the whole project. Record unresolved rights uncertainty and stop before implying approval.

### storyboard-commit-export

Lock the selected iteration. Request high-resolution output at commit and warn when deterministic upscaling is unavailable and regeneration may differ. Write final/final-plan.md, final/shot-list.csv, and final/manifest.json. Record export metadata, generation settings, source references, timestamps, and change summary. Validate required manifest fields, stable frame IDs, prompt presence, allowed status/reference/capture values, required CSV header order/count, output paths, and Markdown frame identifiers before reporting success. Leave prior final outputs untouched if validation fails.

## Cross-skill rules

Use these project interfaces:
- project.json is the source of truth.
- frame_id is stable; sequence_number may change.
- Context precedence is latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, then system inference.
- Starter defaults are provisional and replaceable.
- General concepts default to six frames; the soccer benchmark uses eight.
- Default shoot values are 16:9, 23.976 fps, 18/25/35/50/85/100mm, tripod static, handheld action/intimacy, gimbal/dolly controlled movement, and 120 fps only when story-justified.
- The default image artifact is a numbered contact sheet.
- Approved iterations are never overwritten.
- The package is no-code and file-based.

Validation:
    $skillFiles = Get-ChildItem skills -Recurse -Filter SKILL.md | Where-Object { $_.FullName -notlike '*skills\storyboard\SKILL.md' }
    if ($skillFiles.Count -ne 8) { throw "Expected 8 internal skill files, found $($skillFiles.Count)" }
    foreach ($file in $skillFiles) {
      $content = Get-Content $file.FullName -Raw
      if ($content -notmatch '(?m)^name:') { throw "Missing name frontmatter in $($file.FullName)" }
      if ($content -notmatch 'Inputs|Reads') { throw "Missing input contract in $($file.FullName)" }
      if ($content -notmatch 'Outputs|Produces') { throw "Missing output contract in $($file.FullName)" }
    }

Expected: eight internal skill files, each with name frontmatter and input/output contracts.

Commit:
    git add skills/storyboard-intake skills/storyboard-planning skills/storyboard-review skills/storyboard-generation skills/storyboard-refinement skills/storyboard-versioning skills/storyboard-references skills/storyboard-commit-export
    git commit -m "feat: add storyboard stage skills"

## Context

Tasks 1–3 created the plugin scaffold, provisional guides, and templates in the isolated worktree. Follow the existing file names and JSON/CSV contracts. Do not create runtime code, dependencies, or a separate user-facing UI.

## Report

Write the detailed report to .superpowers/sdd/task-4-report.md. Include files changed, commit SHA, validation output, self-review, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

