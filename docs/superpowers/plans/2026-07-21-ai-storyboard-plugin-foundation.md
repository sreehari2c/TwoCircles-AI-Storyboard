# AI Storyboard Codex Plugin Foundation Implementation Plan

> For agentic workers: REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox syntax for tracking.

**Goal:** Build a portable, no-code Codex plugin with one smart storyboard flow, focused internal skills, starter guide defaults, file-based project templates, and a manual acceptance runbook for ChatGPT image-generation handoff.

**Architecture:** The repository becomes an installable plugin package. The plugin manifest declares the package and points to the skills directory. The public storyboard skill routes user intent and project state to focused internal skills. Markdown skills, JSON/CSV/Markdown templates, and project-local files provide the complete runtime behavior without a server, database, or custom code.

**Tech Stack:** Codex plugin manifest JSON, Markdown SKILL.md instructions, JSON project manifests, Markdown production plans, CSV shot lists, PowerShell read-only validation commands, and ChatGPT image-generation handoff.

## Global Constraints

- The plugin is pure instructions plus Markdown/JSON/CSV files; it has no hosted API, custom runtime, database, or image-processing implementation.
- The user interacts through one top-level storyboard flow; internal skills are selected automatically from intent and project state.
- Projects are stored under storyboard-projects/<project-slug>/ inside the plugin repository.
- Review is enabled by default; skip review, generate immediately, and use your defaults bypass review while retaining the internal plan.
- Story information is mandatory; shoot and scene information may be inferred and must be labeled as inferred.
- Stable frame IDs use the format frame-001; sequence numbers may change when frames are reordered.
- Context precedence is latest explicit user instruction, approved frame instruction, approved project plan, project-local guide, plugin starter guide, then system inference.
- Starter guides are provisional, replaceable, and must not be presented as official Two Circles guidance.
- General concepts default to six frames; the soccer benchmark uses eight frames.
- Starter shoot defaults are 16:9 delivery, 23.976 fps, 18/25/35/50/85/100mm lenses, tripod for static shots, handheld for action/intimacy, gimbal or dolly for controlled movement, and 120 fps only when slow motion serves the story.
- The default low-resolution image artifact is a numbered contact sheet; separate frame images are recorded when generated or supplied.
- Approved iterations are never overwritten. Failed generations and invalid exports leave the last valid state intact.
- Commit output includes a final storyboard image reference, Markdown production plan, expanded-schema CSV shot list, final manifest snapshot, and generation metadata.

---

## File map and responsibilities

| Path | Responsibility |
|---|---|
| .codex-plugin/plugin.json | Plugin identity, version, metadata, and skills directory declaration. |
| README.md | Installation/use overview and the top-level command contract. |
| skills/storyboard/SKILL.md | User-facing routing, project discovery, lifecycle, and handoff rules. |
| skills/storyboard-*/SKILL.md | One focused internal capability per stage. |
| guides/*.md | Provisional story, shoot, scene, and style defaults. |
| templates/project.json | Minimal canonical manifest shape and field examples. |
| templates/project-readme.md | Per-project operating instructions and lifecycle summary. |
| templates/plan.md | Machine-readable Markdown plan structure. |
| templates/shot-list.csv | Exact CSV header contract from the product specification. |
| storyboard-projects/.gitkeep | Keeps the portable project root present in git. |
| docs/storyboard-acceptance-runbook.md | Manual eight-frame soccer acceptance procedure and checks. |

This is one plan because all tasks produce one installable plugin package and each task can be validated by inspecting its own files plus the package contract. No hosted subsystem is being implemented.

## Task 1: Scaffold the plugin package

**Files:**
- Create: .codex-plugin/plugin.json
- Modify: README.md
- Create: storyboard-projects/.gitkeep

**Interfaces:**
- Consumes: The approved design document at docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md.
- Produces: A valid plugin manifest whose skills property points to ./skills/, plus a repository README that names storyboard as the public workflow.

- [ ] Step 1: Write the manifest

Create .codex-plugin/plugin.json with this content:

    {
      "name": "two-circles-ai-storyboard",
      "version": "0.1.0",
      "description": "A guided AI storyboard workflow for turning creative briefs into production-ready visual plans and shot lists.",
      "author": {
        "name": "Two Circles"
      },
      "license": "Proprietary",
      "keywords": [
        "storyboard",
        "creative planning",
        "cinematography",
        "shot list",
        "image generation"
      ],
      "skills": "./skills/"
    }

- [ ] Step 2: Update the README

Replace the current README with a concise package overview containing:

    # Two Circles AI Storyboard

    A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.

    ## Start

    Use the top-level storyboard workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It will discover the project state, ask only high-impact questions, create a plan, offer review, and hand approved prompts to ChatGPT image generation.

    ## Project storage

    Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.

    ## Scope

    This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.

- [ ] Step 3: Keep the project root in git

Create an empty storyboard-projects/.gitkeep file. Do not create a sample project yet; project creation is handled by the top-level flow from templates/.

- [ ] Step 4: Validate the package scaffold

Run from the repository root:

    Get-Content .codex-plugin/plugin.json -Raw | ConvertFrom-Json | Out-Null
    if (-not (Test-Path .codex-plugin/plugin.json)) { throw 'Missing plugin manifest' }
    if (-not (Test-Path storyboard-projects/.gitkeep)) { throw 'Missing project root marker' }

Expected: the command completes without an exception and produces no validation error.

- [ ] Step 5: Commit

    git add .codex-plugin/plugin.json README.md storyboard-projects/.gitkeep
    git commit -m "feat: scaffold storyboard plugin package"

## Task 2: Add provisional guides with usable defaults

**Files:**
- Create: guides/story-guide.md
- Create: guides/shoot-guide.md
- Create: guides/scene-guide.md
- Create: guides/styles/cinematic-realism.md
- Create: guides/styles/documentary-sports.md
- Create: guides/styles/graphic-storyboard-sketch.md
- Create: guides/styles/clean-pitch-frame.md

**Interfaces:**
- Consumes: The guide rules in the design document and Global Constraints.
- Produces: Replaceable guide files that internal skills can load directly and project-local guides/ files can override.

- [ ] Step 1: Write the story guide

Create guides/story-guide.md with these required sections and decisions:

- Purpose: convert incomplete creative intent into a visual sequence.
- Provisional status: clearly state that the guide is a starter default, not official Two Circles guidance.
- Default frame count: six frames for a general short concept; infer another count when duration, narrative beats, or event coverage require it.
- Sequence grammar: establish, introduce subject/action, detail, reaction/emotion, progression/escalation, resolution/call to action.
- Question policy: ask only questions that materially change story, subject, location, action, duration, or delivery.
- Coverage rules: vary shot size and camera language while avoiding redundant frames.
- Duration rules: keep duration optional for non-linear event coverage and record it when supplied or inferred.
- Inference labels: every inferred story value is marked inferred until approved.

- [ ] Step 2: Write the shoot guide

Create guides/shoot-guide.md with the exact defaults from Global Constraints, plus plain-language recommendations for choosing lens, support, frame rate, movement, and capture type. Include a rule that creative descriptions should be preferred over unnecessary exposure or shutter technicalities.

- [ ] Step 3: Write the scene guide

Create guides/scene-guide.md with rules for inferring and locking location, time of day, lighting direction, weather, crowd level, background depth, wardrobe, props, signage, and atmosphere. Include a continuity checklist applied to every frame.

- [ ] Step 4: Write four style placeholders

Each style file must contain Status: Provisional starter placeholder, Rendering method, Contrast, Saturation, Lighting, Texture, Typical lenses, Camera movement, Framing tendencies, Subject treatment, Continuity locks, and Prohibited traits.

Use these starting distinctions:

- cinematic-realism.md: photorealistic, premium commercial realism, controlled contrast, intentional depth of field, executable dynamic camera positions.
- documentary-sports.md: natural light, observational framing, handheld energy, authentic expressions, less polished composition.
- graphic-storyboard-sketch.md: restrained monochrome or limited color, hand-drawn appearance, clear blocking, strong silhouettes.
- clean-pitch-frame.md: polished presentation composition, controlled backgrounds, strong visual hierarchy, client-review readability.

Do not include official logos, proprietary claims, or unverified Two Circles brand rules.

- [ ] Step 5: Validate guide completeness

Run:

    $guideFiles = Get-ChildItem guides -Recurse -Filter *.md
    if ($guideFiles.Count -ne 7) { throw "Expected 7 guide files, found $($guideFiles.Count)" }
    Select-String -Path $guideFiles.FullName -Pattern 'Provisional|Status' | Out-Null

Expected: seven Markdown guide files are found and every style file contains a provisional-status marker.

- [ ] Step 6: Commit

    git add guides
    git commit -m "feat: add provisional storyboard guides"

## Task 3: Add project and export templates

**Files:**
- Create: templates/project.json
- Create: templates/project-readme.md
- Create: templates/plan.md
- Create: templates/shot-list.csv

**Interfaces:**
- Consumes: The canonical project model, expanded CSV schema, and lifecycle from the design document.
- Produces: Templates used by intake, planning, review, and commit/export skills. The frame_id and sequence_number fields are the shared interface between JSON, Markdown, and CSV.

- [ ] Step 1: Write the manifest template

Create templates/project.json as valid JSON with these top-level keys and representative empty values:

    {
      "project_id": "project-slug",
      "title": "Untitled Storyboard",
      "schema_version": "0.1.0",
      "status": "proposed",
      "active_version": 0,
      "story": {},
      "shoot": {},
      "scene": {},
      "style": {},
      "creative_constants": {
        "locks": [],
        "prohibited_elements": []
      },
      "references": [],
      "frames": [],
      "versions": [],
      "exports": []
    }

- [ ] Step 2: Write the project README template

Create templates/project-readme.md explaining that project.json is the source of truth, listing the folder meanings, stating that frame_id is stable, and documenting the lifecycle intake -> plan -> review -> generate -> refine/version -> commit/export.

- [ ] Step 3: Write the plan template

Create templates/plan.md with these exact headings:

    # [Project Title] Storyboard Plan

    ## Project Summary
    ## Creative Constants
    ## Production Assumptions
    ## Story Progression
    ## Frame Plan

    ### Frame 001 — [Frame Title]
    - Frame ID: frame-001
    - Purpose:
    - Description:
    - Subject:
    - Action:
    - Shot size:
    - Camera angle:
    - Camera position:
    - Lens:
    - Depth of field:
    - Movement:
    - Location:
    - Lighting:
    - Duration:
    - Transition:
    - Continuity locks:
    - Reference source:
    - Prompt:
    - Negative constraints:

The planning skill will duplicate the Frame section for every stable frame ID and use Frame 03 — Title style headings in final plans while retaining the machine-readable Frame ID field.

- [ ] Step 4: Write the exact CSV header

Create templates/shot-list.csv with this single header row, preserving column order:

    project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at

- [ ] Step 5: Validate template contracts

Run:

    Get-Content templates/project.json -Raw | ConvertFrom-Json | Out-Null
    $header = (Get-Content templates/shot-list.csv -First 1).Split(',')
    if ($header.Count -ne 45) { throw "Expected 45 CSV columns, found $($header.Count)" }
    if ($header[3] -ne 'frame_id' -or $header[4] -ne 'sequence_number') { throw 'Frame identity columns are not in the required positions' }

Expected: JSON parses successfully and the CSV has 45 columns with frame_id before sequence_number.

- [ ] Step 6: Commit

    git add templates
    git commit -m "feat: add storyboard project and export templates"

## Task 4: Write the internal stage skills

**Files:**
- Create: skills/storyboard-intake/SKILL.md
- Create: skills/storyboard-planning/SKILL.md
- Create: skills/storyboard-review/SKILL.md
- Create: skills/storyboard-generation/SKILL.md
- Create: skills/storyboard-refinement/SKILL.md
- Create: skills/storyboard-versioning/SKILL.md
- Create: skills/storyboard-references/SKILL.md
- Create: skills/storyboard-commit-export/SKILL.md

**Interfaces:**
- Consumes: The project manifest and templates from Task 3, and starter/project-local guides from Task 2.
- Produces: Focused instruction boundaries that the top-level skill can load and execute without exposing separate user-facing commands.

- [ ] Step 1: Define shared skill conventions

Every file must begin with YAML frontmatter containing a unique lowercase name and a one-sentence description. Every file must state:

- Inputs it reads.
- Files it may create or update.
- Outputs it produces for the next stage.
- Conditions under which it must stop and ask the user.
- Invariants it must preserve.

- [ ] Step 2: Write intake skill behavior

skills/storyboard-intake/SKILL.md must extract supplied story, shoot, scene, style, reference, duration, frame-count, aspect-ratio, and production constraints; avoid asking for supplied facts; mark inferred values; create a project slug; copy the project and README templates; and stop with a normalized intake summary before planning.

- [ ] Step 3: Write planning skill behavior

skills/storyboard-planning/SKILL.md must load the applicable guides, create stable frame IDs, produce the canonical plan, populate creative constants, separate assumptions from approved values, write plans/plan-v001.md, and update project.json without creating images.

- [ ] Step 4: Write review skill behavior

skills/storyboard-review/SKILL.md must show project summary, creative constants, frame sequence, assumptions, warnings, and conflicts. It must support approve, edit overall plan, edit/add/remove/reorder one frame, request alternatives, skip review, and proceed to creation. It must not generate images.

- [ ] Step 5: Write generation skill behavior

skills/storyboard-generation/SKILL.md must build a prompt packet containing the approved plan, constants, negative constraints, style, aspect ratio, references, and frame prompts; request the numbered low-resolution contact sheet through ChatGPT image generation; record the exact packet and returned image reference under iterations/v001/; and update project.json only after the image result is available.

- [ ] Step 6: Write refinement skill behavior

skills/storyboard-refinement/SKILL.md must identify affected frame IDs, classify whether a project constant is affected, list continuity risks, preserve unaffected records, create the next iteration, and write changes.md. It must support one-frame, multi-frame, reference replacement, same-prompt regeneration, and sequence-ending replacement requests.

- [ ] Step 7: Write versioning skill behavior

skills/storyboard-versioning/SKILL.md must list iteration folders, show change summaries, restore a whole version by creating a new branch snapshot, restore one frame by copying its prior frame record into a new iteration, and branch from an existing version without deleting history.

- [ ] Step 8: Write references skill behavior

skills/storyboard-references/SKILL.md must support internal, external, uploaded, AI-generated, and none reference types; store source/path/URL, rights note, applicable frames, what to borrow, and what not to copy; and avoid assuming an uploaded reference applies to the whole project.

- [ ] Step 9: Write commit/export skill behavior

skills/storyboard-commit-export/SKILL.md must lock the selected iteration, request high-resolution output while warning about regeneration differences when deterministic upscaling is unavailable, write final/final-plan.md, final/shot-list.csv, and final/manifest.json, record export metadata, and validate required fields before reporting success.

- [ ] Step 10: Validate internal skill boundaries

Run:

    $skillFiles = Get-ChildItem skills -Recurse -Filter SKILL.md | Where-Object { $_.FullName -notlike '*skills\storyboard\SKILL.md' }
    if ($skillFiles.Count -ne 8) { throw "Expected 8 internal skill files, found $($skillFiles.Count)" }
    foreach ($file in $skillFiles) {
      $content = Get-Content $file.FullName -Raw
      if ($content -notmatch '(?m)^name:') { throw "Missing name frontmatter in $($file.FullName)" }
      if ($content -notmatch 'Inputs|Reads') { throw "Missing input contract in $($file.FullName)" }
      if ($content -notmatch 'Outputs|Produces') { throw "Missing output contract in $($file.FullName)" }
    }

Expected: eight internal skill files are found, and every file exposes name, input, and output contracts.

- [ ] Step 11: Commit

    git add skills/storyboard-intake skills/storyboard-planning skills/storyboard-review skills/storyboard-generation skills/storyboard-refinement skills/storyboard-versioning skills/storyboard-references skills/storyboard-commit-export
    git commit -m "feat: add storyboard stage skills"

## Task 5: Write the top-level smart storyboard flow

**Files:**
- Create: skills/storyboard/SKILL.md

**Interfaces:**
- Consumes: The eight internal skills from Task 4, templates from Task 3, guides from Task 2, and project folders under storyboard-projects/.
- Produces: The public storyboard command contract and routing behavior used by all later manual tests.

- [ ] Step 1: Define the public skill metadata

Start the file with this frontmatter:

    ---
    name: storyboard
    description: Turn a creative brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
    ---

- [ ] Step 2: Define project discovery

Document the exact discovery order:

1. If the user names a project, resolve storyboard-projects/<project-slug>/project.json.
2. If the current workspace has one active storyboard project, use it.
3. If there are multiple projects and none is named, ask the user to choose.
4. If there is no project and the user supplied meaningful story input, create one from templates.
5. If there is no project and no meaningful story input, ask for one sentence of creative intent.

- [ ] Step 3: Define routing and defaults

Include the routing table from the design document, the lifecycle, the context precedence, default review behavior, six-frame general default, eight-frame soccer benchmark exception, and the rule to load only the relevant internal skill and guide files.

- [ ] Step 4: Define user-visible responses

Require concise, structured responses that show current stage, changed files, assumptions, warnings, frame IDs, and next action. For image generation, show the exact frame prompts or a readable prompt summary. For refinement, show the change summary and untouched frame IDs.

- [ ] Step 5: Define stop conditions

The flow must stop for explicit user choice when a lock would change, a reference rights issue is unresolved, a high-impact ambiguity cannot be resolved with a recommendation, or commit validation fails. It must never claim image generation, export, or commit success without the corresponding file/reference being present.

- [ ] Step 6: Validate top-level routing references

Run:

    $content = Get-Content skills/storyboard/SKILL.md -Raw
    foreach ($name in @('storyboard-intake','storyboard-planning','storyboard-review','storyboard-generation','storyboard-refinement','storyboard-versioning','storyboard-references','storyboard-commit-export')) {
      if ($content -notmatch [regex]::Escape($name)) { throw "Top-level flow does not reference $name" }
    }
    foreach ($phrase in @('skip review','frame-001','storyboard-projects','project.json','ChatGPT image generation')) {
      if ($content -notmatch [regex]::Escape($phrase)) { throw "Top-level flow is missing required contract phrase: $phrase" }
    }

Expected: all eight internal skills and all five contract phrases are referenced.

- [ ] Step 7: Commit

    git add skills/storyboard/SKILL.md
    git commit -m "feat: add smart storyboard orchestrator"

## Task 6: Add the manual acceptance runbook

**Files:**
- Create: docs/storyboard-acceptance-runbook.md

**Interfaces:**
- Consumes: The public flow from Task 5 and the acceptance scenario from the design document.
- Produces: A repeatable, no-code smoke test for new users and future plugin revisions.

- [ ] Step 1: Document setup

State that the test starts from a clean project folder, uses the supplied soccer production example, and requires access to the ChatGPT image-generation capability. The runbook must not require a hosted API or custom script.

- [ ] Step 2: Document the nine acceptance actions

Include the exact sequence from the design: import/describe sequence, create eight-frame plan, verify inferred labels, generate contact sheet, change only streetcar frame, replace one frame with uploaded reference, inspect history, commit and validate outputs, reopen from disk.

- [ ] Step 3: Add evidence checks

For each action, list the expected file or visible result: stable frame IDs, plan file, contact-sheet reference, unchanged prompts for unaffected frames, frame-scoped reference metadata, iteration change summary, final Markdown/CSV/manifest, and successful reopen without chat history.

- [ ] Step 4: Add failure checks

Document that a failed generation leaves the active version unchanged, a lock conflict pauses for approval, and a CSV/manifest validation failure leaves prior final outputs untouched.

- [ ] Step 5: Validate the runbook references

Run:

    $content = Get-Content docs/storyboard-acceptance-runbook.md -Raw
    foreach ($phrase in @('streetcar','frame-001','final-plan.md','shot-list.csv','manifest.json','without the original chat')) {
      if ($content -notmatch [regex]::Escape($phrase)) { throw "Acceptance runbook is missing: $phrase" }
    }

Expected: the runbook contains all benchmark and evidence terms.

- [ ] Step 6: Commit

    git add docs/storyboard-acceptance-runbook.md
    git commit -m "docs: add storyboard acceptance runbook"

## Task 7: Run package-wide validation and perform the smoke review

**Files:**
- Modify: Any file that fails the checks below.

**Interfaces:**
- Consumes: The complete plugin package from Tasks 1–6.
- Produces: A clean, self-contained package ready for plugin installation or export.

- [ ] Step 1: Validate every JSON file

Run:

    Get-ChildItem -Recurse -Filter *.json | ForEach-Object {
      Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null
    }

Expected: no JSON parse errors.

- [ ] Step 2: Validate the required package paths

Run:

    $required = @(
      '.codex-plugin/plugin.json',
      'skills/storyboard/SKILL.md',
      'guides/story-guide.md',
      'guides/shoot-guide.md',
      'guides/scene-guide.md',
      'templates/project.json',
      'templates/plan.md',
      'templates/shot-list.csv',
      'docs/storyboard-acceptance-runbook.md',
      'storyboard-projects/.gitkeep'
    )
    foreach ($path in $required) {
      if (-not (Test-Path $path)) { throw "Missing required package path: $path" }
    }

Expected: no missing-path errors.

- [ ] Step 3: Search for forbidden planning placeholders

Run:

    rg -n "TODO|TBD|implement later|fill in details|add appropriate error handling" . --glob '!docs/superpowers/specs/**' --glob '!docs/superpowers/plans/**'

Expected: no matches. Provisional guide markers are allowed because they are intentional product content, not unfinished implementation steps.

- [ ] Step 4: Check repository diff and line endings

Run:

    git diff --check
    git status --short

Expected: no whitespace errors. Only intended plugin files are modified.

- [ ] Step 5: Perform the manual smoke review

Read README.md, skills/storyboard/SKILL.md, all internal skills, the four guide styles, the templates, and the acceptance runbook in that order. Confirm that every referenced path exists and that the top-level flow can describe the next action for a new project, a reviewable plan, a generated iteration, a frame refinement, and a committed project.

- [ ] Step 6: Commit the validated package

    git add .
    git commit -m "chore: validate storyboard plugin foundation"

## Spec coverage self-review

- Product goal and no-code plugin scope: Tasks 1, 4, and 5.
- One smart top-level flow with associated skills: Tasks 4 and 5.
- Provisional story, shoot, scene, and style defaults: Task 2.
- Canonical JSON project model and stable frame IDs: Task 3.
- Project-local guide overrides and portable repository storage: Tasks 1, 2, 3, and 5.
- Review-by-default and explicit bypass: Task 5.
- ChatGPT image-generation handoff and contact sheet: Tasks 4 and 5.
- Frame-level refinement, continuity locks, and change summaries: Task 4.
- Versioning, restoration, and branching behavior: Task 4.
- Markdown plan and expanded CSV shot-list export: Tasks 3 and 4.
- Failure handling, validation, and no-overwrite rules: Tasks 4, 5, 6, and 7.
- Soccer benchmark acceptance scenario: Task 6.
- Deferred hosted API, SharePoint, collaboration, UI, and office exports: Global Constraints and Task 6 documentation.

## Execution handoff

Plan complete and saved to docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md. Two execution options:

1. Subagent-Driven (recommended) — dispatch a fresh subagent per task and review between tasks.
2. Inline Execution — execute the tasks in this session with checkpoints.

Choose one approach before implementation begins.
