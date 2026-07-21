# Codex Build Prompt: AI-Driven Storyboarding Plugin

You are working in a local repository. Build the application described in `ai-storyboarding-plugin-spec.md` as a reusable Codex Plugin that can later be distributed across the company.

The final implementation must be Plugin-ready, but the local development experience is equally important. Developers must be able to inspect, edit, test, and iterate on all skills, guides, prompt templates, schemas, sample files, and project state directly in the repository.

Do not hide the core behavior inside opaque generated assets. Keep the source of truth readable and version-controlled.

## Source materials available in the working environment

Use these files as product and visual references where relevant:

- `ai-storyboarding-plugin-spec.md`
- `/mnt/data/Hackathon.docx`
- `/mnt/data/Raised on Soccer Shot List (1).pdf`
- `/mnt/data/Storyboard Example.png`
- `/mnt/data/Screenshot 2026-06-24 at 1.18.49 PM 1.png`

If the exact screenshot filename contains a Unicode spacing character in the local environment, locate it safely rather than assuming the ASCII form above.

## Primary objective

Create a working local implementation that supports this workflow:

1. Define
2. Plan
3. Optional Review, enabled by default
4. Create a low-resolution storyboard
5. Refine selected frames only
6. Undo one iteration
7. Commit a deterministic high-resolution JPG and structured project files

The storyboard visual output must always be one contact sheet containing all frames.

## Critical implementation constraints

### Plugin and local source structure

- Build a reusable Plugin, not just a one-off script.
- Keep every skill in a visible local directory with a readable `SKILL.md` or `skill.md`.
- Keep style guides, scene guides, shoot guides, story guides, contact-sheet templates, schemas, and prompt templates as normal editable files.
- Include a Plugin manifest and clear packaging instructions.
- Include commands for local development, validation, testing, and packaging.
- The same source tree used locally should be packaged into the final Plugin.

### Runtime

- The target runtime is Codex.
- The primary user interface is chat.
- The implementation must persist project state to disk and must not rely on conversation history alone.
- Use the current repository directory as the root.
- Store generated projects under `./storyboard-projects/`.

### Image generation

- Create an image-provider abstraction.
- Use the best OpenAI image-generation model available in the active Codex or ChatGPT runtime.
- Do not hard-code a historical model name.
- If direct image generation is unavailable in the local test environment, implement a clean adapter, a mock provider, and a fixture-based demo path so the rest of the workflow can be tested.
- Document exactly how the real runtime provider should be connected.

### Contact-sheet generation

- Generate each frame as a separate low-resolution image first.
- Composite those frame images into one low-resolution contact-sheet JPG.
- Use a generic blue-themed design.
- Include project title, total duration, version, date, frame numbers, and frame titles.
- Use individual frame files internally even though the user-facing visual export is one JPG.

### Refinement and non-drift behavior

This is a hard requirement:

- When the user changes one or more frames, generate only those replacement frame images.
- Reuse the exact image files for all unaffected frames.
- Rebuild the contact sheet by compositing unchanged and replacement panels.
- A change to frame 4 must not modify the image bytes used for frames 1–3 or 5 onward.
- Include automated tests proving that unaffected panel files are reused.

### Undo

- Support exactly one working undo level using `current` and `previous` states.
- Before refinement, copy or move `current` to `previous`.
- “Undo” restores `previous` as `current`.
- Committed versions remain versioned permanently.

### High-resolution Commit

- Do not perform a new creative generation at Commit.
- Deterministically upscale the approved low-resolution contact-sheet JPG.
- Record source dimensions, final dimensions, method, factor, and version.
- The final user-facing storyboard format is JPG only.

### Planning

- Require a total duration for every project.
- Infer a recommended frame count from story complexity and duration.
- Allow human override.
- Allocate duration to every frame.
- Validate that frame durations sum exactly to the project duration.
- Generate stable frame IDs that survive revisions and reordering.

### Human subjects

- Use generic fictional subjects unless approved talent references are supplied.
- Keep recurring fictional subject attributes in continuity locks.

### Exports

On Commit, create:

- High-resolution storyboard JPG
- Final Markdown plan
- Final CSV shot list using the approved expanded schema
- Final JSON manifest

Also create versioned files and stable `*-final.*` aliases.

## Preferred repository structure

Use the structure in the specification unless there is a compelling technical reason to adjust it. At minimum, create:

```text
plugin/
  plugin.json
  README.md
  orchestrator/
  skills/
  guides/
  templates/
  schemas/
  examples/
src/
tests/
scripts/
storyboard-projects/
README.md
```

Recommended skills:

- storyboard-intake
- storyboard-planning
- cinematography-planning
- storyboard-review
- storyboard-generation
- storyboard-refinement
- storyboard-versioning
- storyboard-commit
- shot-list-export
- reference-management

## Technology selection

Choose a simple, maintainable implementation language appropriate for local file operations, image compositing, structured data, CLI commands, and tests.

Python is preferred unless the local Plugin tooling strongly favors another language.

For a Python implementation, use:

- Pillow for JPG frame composition and deterministic resizing
- Pydantic or JSON Schema validation for project state
- Standard CSV and Markdown file generation
- Pytest for tests
- A small CLI using Typer or argparse

Do not add heavy infrastructure that is unnecessary for the local Plugin workflow.

## Required local commands

Provide convenient commands or scripts for:

1. Initialize or validate the environment
2. Run the local Plugin workflow
3. Create a demo project from fixtures
4. Validate all skill files and schemas
5. Run tests
6. Package the Plugin

Example names:

```text
scripts/run_local.sh
scripts/create_demo_project.sh
scripts/validate_plugin.sh
scripts/package_plugin.sh
```

Ensure scripts use portable, plain shell syntax.

## Required implementation phases

Work in the following order.

### Phase 1: Inspect and plan

- Read `ai-storyboarding-plugin-spec.md` fully.
- Inspect the uploaded examples.
- Summarize the architecture you will implement.
- Identify any environment limitations, especially image-generation availability.
- Do not stop after planning.

### Phase 2: Scaffold

- Create the complete repository structure.
- Add Plugin manifest and Plugin README.
- Add local root README.
- Add all skill directories and editable skill entrypoints.
- Add provisional guides and templates.
- Add schemas.

### Phase 3: Core models and storage

Implement:

- Project model
- Frame model
- Reference model
- Version model
- Local storage provider
- Versioned naming
- `current` and `previous` working states
- Atomic or safe writes where practical

### Phase 4: Planning workflow

Implement:

- Story, Shoot, and Scene normalization
- Required duration validation
- Frame-count recommendation
- Frame duration allocation
- Stable frame IDs
- Markdown plan generation
- Review payload generation

### Phase 5: Visual workflow

Implement:

- Image-generation provider interface
- Mock or fixture provider for local testing
- Individual frame output
- Blue-themed contact-sheet compositor
- Contact-sheet metadata header
- Frame numbering and titles
- Frame-level replacement
- Reuse of unchanged frame files
- One-level undo

### Phase 6: Commit workflow

Implement:

- Deterministic image upscaling
- Final JPG creation
- CSV shot-list export
- Final Markdown plan
- Final JSON manifest
- Versioned files and final aliases

### Phase 7: Tests

At minimum, test:

- Duration sums exactly
- Frame IDs remain stable across edits
- Reordering changes sequence number but not frame ID
- Refining one frame reuses unchanged frame files
- Contact sheet includes all expected frames
- Undo restores the previous working state
- Commit does not call creative image generation
- Commit produces valid JPG, Markdown, CSV, and JSON
- Plugin and skill files validate

### Phase 8: Demo

Create a complete local demo using the soccer example or a fixture derived from it.

The demo should:

1. Create a project
2. Generate a plan
3. Build a low-resolution contact sheet using fixture or mock frames
4. Change one frame
5. Prove unchanged frames were reused
6. Undo the change
7. Reapply or make another change
8. Commit the final storyboard
9. Print the generated file paths

### Phase 9: Package

- Add a packaging script.
- Produce a distributable Plugin package or package directory according to the available local Codex Plugin format.
- If Plugin tooling is unavailable, create a standards-compliant source package and clearly document the final packaging command that should be run in the supported environment.

## HTML interface nice-to-have

After the core workflow and tests pass, optionally add a lightweight local HTML interface that:

- Displays the current contact sheet
- Displays editable frame cards
- Reads and writes the same project manifest
- Does not create a second source of truth
- Produces structured refinement requests

Do not delay the core chat and local-file workflow for this feature.

## Quality requirements

- Use typed models where practical.
- Add helpful error messages.
- Validate inputs and output files.
- Avoid unnecessary dependencies.
- Keep code modular.
- Keep all prompts and guides editable.
- Include docstrings and concise comments for non-obvious behavior.
- Ensure local demo and tests run from a fresh checkout.

## Definition of done

The work is complete only when:

- The repository contains the Plugin source and local development tooling.
- All required skills, guides, templates, and schemas are visible and editable.
- A user can create a project with a required duration.
- The system recommends and supports overriding frame count.
- A plan can be reviewed or skipped.
- A low-resolution contact sheet can be generated.
- A selected frame can be replaced without changing unaffected frame files.
- One-level undo works.
- Commit performs deterministic upscaling rather than creative regeneration.
- JPG, Markdown, CSV, and JSON outputs are created.
- Tests pass.
- A demo project is included.
- Packaging instructions are complete.

## Final response expected from Codex

When finished, report:

1. Architecture implemented
2. Files and directories created
3. Commands to run locally
4. Commands to run tests
5. Commands to create the demo
6. Commands to package the Plugin
7. Image-generation adapter status
8. Any remaining limitations
9. Paths to the demo outputs

Do not merely provide code snippets or a plan. Create the working files in the repository.
