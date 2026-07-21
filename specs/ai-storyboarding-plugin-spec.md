# AI-Driven Storyboarding Plugin Specification

## 1. Purpose

Build a reusable company-wide Codex Plugin that converts a creative brief, script, concept, meeting transcript, shot description, or existing shot list into a consistent, production-ready storyboard package.

The Plugin must guide a user from an incomplete idea to:

- A reviewed storyboard plan
- A single low-resolution storyboard contact sheet containing all frames
- Human-readable prompts and production metadata for each frame
- Frame-specific revisions without changing unaffected panels
- A deterministic high-resolution final storyboard JPG
- A Markdown production plan
- A CSV shot list
- A JSON project manifest

The primary interface is conversational. Users define, review, refine, reset, and commit storyboards through chat.

The final deliverable is a Plugin that can be distributed across the company. During development and local testing, all Plugin files, skills, templates, guides, schemas, prompts, and sample assets must remain visible and editable in the local repository.

---

## 2. Product Goal

Validate the following core proposition:

> A creative or producer can quickly turn an idea into a consistent visual reference that another person can understand and realistically execute.

The product is not intended to generate final video footage. It communicates creative intent through storyboard frames and structured production metadata.

---

## 3. Primary Problems

Current production-planning workflows are fragmented across videos, screenshots, mood boards, slide decks, Notes, Word documents, spreadsheets, Google Sheets, PDFs, and project-management tools.

The Plugin should address these problems:

- Finding useful inspiration frames takes too long.
- Existing references rarely match the exact shot required.
- AI tools require repeated prompt engineering.
- Visual style and scene continuity drift during revisions.
- Storyboards and shot lists often live separately.
- Long chat threads lose important context.
- Existing planning formats are difficult to update on mobile.
- Client requests must be translated manually into executable production plans.
- Changing one frame can unintentionally alter others.
- Final files and iterations can be difficult to organize and share.

---

## 4. Scope

### 4.1 Hackathon and first-release scope

The first release must support:

1. Guided storyboard definition
2. AI-generated storyboard planning
3. Optional review, enabled by default
4. Low-resolution storyboard frame generation
5. One contact sheet containing all frames
6. Frame-level refinement through chat
7. Preservation of unchanged panels through compositing
8. One-level undo
9. Deterministic high-resolution upscaling
10. Versioned local project storage
11. Markdown plan generation
12. CSV shot-list generation
13. JSON manifest generation
14. JPG storyboard export
15. Local inspection and editing of all skills, templates, and guides
16. Packaging as a reusable company Plugin

### 4.2 Out of scope for the first release

- SharePoint integration
- PDF export
- PowerPoint export
- Word export
- PNG as a final user-facing format
- Live multi-user collaboration
- Shooter assignment workflows
- Production completion tracking
- A full standalone web application
- A searchable internal footage library

A lightweight local HTML frame-card viewer/editor is a nice-to-have only.

---

## 5. Product Principles

### 5.1 Story first

Story information is mandatory and is the strongest influence on the result.

Shoot and scene details are optional. The Plugin should infer them from the story when they are absent.

### 5.2 Fast first result

A user should be able to start with one sentence and receive a useful proposal quickly.

### 5.3 Guided, not restrictive

The Plugin should:

- Ask only high-impact questions
- Avoid asking for information already supplied
- Offer recommended defaults
- Give a small number of alternatives when useful
- Allow the user to accept all defaults
- Explain technical recommendations in plain language

### 5.4 Production realism

Every frame should communicate an executable shot, including:

- Framing
- Composition
- Camera angle
- Camera height and position
- Subject placement
- Camera movement
- Approximate lens or field of view
- Location
- Lighting
- Intended action
- Duration
- Transition where relevant

### 5.5 Consistency over novelty

The Plugin must preserve:

- Subject identity
- Wardrobe
- Props
- Location
- Lighting
- Color treatment
- Aspect ratio
- Image style
- Camera language
- Narrative progression

### 5.6 Controlled refinement

When the user changes one frame, unaffected frame images must be reused exactly in the next contact sheet.

### 5.7 Structured data underneath

Every storyboard frame must have structured metadata so it can later support production scheduling, assignment, tracking, and reference-library workflows.

### 5.8 Local transparency during development

For local testing, the implementation must not hide skills, prompts, templates, schemas, style guides, or generated project state inside opaque build artifacts.

Developers must be able to inspect and edit these files directly.

---

## 6. Primary Users

### Creative or Creative Director

Needs to visualize a concept or sequence without drawing frames manually or repeatedly engineering prompts.

### Producer

Needs to turn creative and client requirements into a clear, actionable production plan.

### Shooter or Camera Operator

Needs to understand what must be captured and how.

### Client Stakeholder

Needs a clear visual representation of the proposed creative direction.

### Freelancer or Event Crew Member

Needs a concise, mobile-friendly visual and production reference.

---

## 7. Key Concepts

### 7.1 Storyboard

A storyboard visualizes how a shot or sequence could look. It communicates creative intention, action, sequence, framing, and tone.

### 7.2 Shot list

A shot list is an operational checklist defining what must be captured. It may include priority, location, time, lens, movement, frame rate, duration, assigned shooter, and completion status.

Not every shot-list item requires an image.

### 7.3 Story Guide

Defines narrative intent, structure, pacing, visual progression, and emotional progression.

### 7.4 Shoot Guide

Defines production constraints, equipment assumptions, capture approach, and cinematography recommendations.

### 7.5 Scene Guide

Defines environment, location, time, weather, subjects, wardrobe, props, atmosphere, and lighting.

### 7.6 Style Guide

Defines the approved visual language used for image generation and contact-sheet presentation.

---

## 8. End-to-End Workflow

## 8.1 Define

### Objective

Collect enough context to build a storyboard plan without forcing the user to write detailed prompts for every frame.

### Accepted inputs

- Brief
- Script
- Concept
- Client request
- Shot description
- Meeting transcript
- Schedule
- Existing storyboard
- Existing shot list
- Reference image
- External reference URL
- Mood board
- Production constraints

### Required information

- Meaningful story or creative-intent statement
- Total project duration

### Optional Story fields

- Project title
- Purpose
- Audience
- Core message
- Narrative arc
- Beginning
- Middle
- End
- Key moments
- Emotional progression
- Call to action
- Distribution channel
- Aspect ratio
- Desired frame count override

### Optional Shoot fields

- Camera type
- Available lenses
- Camera support
- Frame rate
- Slow-motion requirements
- Crew size
- Drone availability
- Talent limitations
- Safety considerations
- Capture format
- Delivery requirements

### Optional Scene fields

- Location
- Time of day
- Weather
- Lighting
- Subjects
- Wardrobe
- Props
- Background elements
- Crowd density
- Signage
- Atmosphere
- Color palette

### Define behavior

The Plugin must:

1. Extract known details from supplied material.
2. Avoid asking for details already present.
3. Infer missing shoot and scene details where reasonable.
4. Label inferred details clearly.
5. Ask only questions that materially affect the output.
6. Offer recommended defaults.
7. Allow the user to accept all recommendations.
8. Warn when a request is physically or logistically implausible.
9. Require a total duration before planning is finalized.
10. Infer a recommended frame count and allow human override.

Example:

> Recommended: 8 frames for a 30-second sequence. This allows roughly 3–4 seconds per visual beat. Reply with another frame count to override.

---

## 8.2 Plan

### Objective

Convert all context, guides, defaults, and references into a concise plan that is AI-friendly and human-readable.

### Required plan sections

#### Project Summary

- Project title
- Creative objective
- Audience
- Total duration
- Aspect ratio
- Intended use
- Selected style
- Recommended frame count
- User-overridden frame count, if applicable

#### Creative Constants

Attributes that should remain stable across all frames:

- Subject definitions
- Wardrobe
- Props
- Location continuity
- Lighting continuity
- Color treatment
- Rendering style
- Image texture
- Aspect ratio
- Brand requirements
- Prohibited elements

#### Production Assumptions

- Camera system
- Lens family
- Base frame rate
- Slow-motion assumptions
- Camera-support assumptions
- Available equipment
- Inferred constraints

#### Story Progression

A concise summary of the complete visual sequence.

#### Frame Plan

Each frame must contain:

- Stable frame ID
- Sequence number
- Frame title
- Narrative purpose
- Visual description
- Subject
- Action
- Shot size
- Camera angle
- Camera height
- Camera position
- Lens or focal length
- Depth of field
- Camera movement
- Camera support
- Location
- Lighting
- Frame rate
- Duration
- Transition to next frame
- Continuity requirements
- Reference source
- Image-generation prompt
- Negative prompt or constraints

### Duration validation

The total of all frame durations must equal the required project duration.

```text
sum(frame.duration_seconds) = project.total_duration_seconds
```

If the sequence is not realistic within the requested duration, the Plugin must warn the user and propose an adjustment.

### Plan storage

Write every plan to a versioned Markdown file before image generation.

---

## 8.3 Review

### Objective

Allow the user to approve or correct the plan before image-generation resources are used.

### Default behavior

Review is enabled by default.

Present a concise review containing:

- Project summary
- Creative constants
- Frame sequence
- Frame durations
- Major assumptions
- Warnings or conflicts

### Supported actions

- Approve
- Edit the overall plan
- Edit one frame
- Add a frame
- Remove a frame
- Reorder frames
- Override frame count
- Request alternatives
- Skip review
- Proceed to creation

### Review bypass

The user may say:

- “Skip review”
- “Generate immediately”
- “Use the recommended plan”
- “Go straight to the storyboard”

The internal plan must still be written to disk before image generation.

---

## 8.4 Create

### Objective

Generate a low-resolution storyboard for fast review and iteration.

### Image output

Create one contact-sheet JPG containing all frames.

The contact sheet must include:

- Clearly numbered panels
- Consistent panel dimensions
- Consistent aspect ratio
- Project title
- Total duration
- Storyboard version
- Generation date
- Basic blue-themed branding
- Optional client name
- Optional confidentiality label

Each panel should include:

- Frame number
- Frame title
- Generated image
- Optional duration
- Optional shot-size abbreviation

Detailed prompts and metadata should remain outside the contact sheet in chat, Markdown, CSV, and JSON.

### Frame generation

Generate each frame as an individual low-resolution image first, then composite all frame images into the contact sheet.

This internal separation is required to support targeted frame replacement later.

### Generation requirements

- Use the active style guide.
- Use sample images and references as grounding where appropriate.
- Maintain subject and scene continuity.
- Avoid unrequested logos, text, watermarks, and signage.
- Preserve intended framing and camera direction.
- Use realistic camera placement.
- Reflect lens and field-of-view differences visibly.
- Keep recurring generic subjects visually consistent.

### Post-generation chat output

For every frame, show:

- Frame number and title
- Description
- Image prompt
- Negative prompt
- Lens
- Camera angle
- Camera movement
- Frame rate
- Duration
- Transition
- Reference source
- Assumptions

---

## 8.5 Refine

### Objective

Allow targeted visual and metadata changes without changing unaffected panels.

### Example commands

- “Change frame 3 to a low-angle 35mm shot.”
- “Make frame 6 wider.”
- “Use the uploaded reference for frame 2.”
- “Keep everything the same except make the ball yellow.”
- “Regenerate frame 4 with the same prompt.”
- “Replace frames 7–9 with a quieter ending.”

### Required behavior

1. Identify affected frame IDs.
2. Determine whether the change affects a frame, several frames, creative constants, or the overall story.
3. Warn if the request could break continuity.
4. Preserve all unaffected frame prompts and metadata.
5. Reuse the exact stored image files for unaffected panels.
6. Generate replacement images only for requested frames.
7. Rebuild the contact sheet by compositing old and new panels.
8. Preserve numbering, layout, titles, and dimensions.
9. Save the previous working state before replacing the current state.
10. Show a written change summary.

### Non-drift invariant

A request to modify frame 4 must not change the image pixels used for frames 1–3 or 5 onward.

### Change summary example

```text
Iteration 4 changes:
- Frame 03: Camera angle changed from eye-level to low angle.
- Frame 03: Lens changed from 50mm to 35mm.
- Frames 01–02 and 04–08: Reused without modification.
```

---

## 8.6 Reset

### Objective

Support one-level undo.

### Required working states

- `current`
- `previous`

Before every refinement:

1. Copy or move `current` to `previous`.
2. Generate changed panels.
3. Rebuild the contact sheet.
4. Save the result as the new `current`.
5. Update project metadata.

Supported commands:

- “Undo the last change.”
- “Reset to the previous iteration.”
- “Restore the previous storyboard.”

Only one previous working iteration is required.

Committed versions remain permanently versioned.

---

## 8.7 Commit

### Objective

Finalize the approved storyboard and create durable project artifacts.

### Required behavior

1. Lock the selected iteration.
2. Deterministically upscale the approved low-resolution contact sheet.
3. Do not perform a new creative image generation.
4. Update the canonical plan with accepted refinements.
5. Save the final Markdown plan.
6. Save the final CSV shot list.
7. Save the final JSON manifest.
8. Save the final high-resolution JPG.
9. Return file paths and a concise completion summary.

### High-resolution rule

The committed image must be a deterministic upscale of the approved low-resolution contact sheet.

```text
Approved low-resolution JPG
            ↓
Deterministic upscale
            ↓
Final high-resolution JPG
```

Record:

- Source path
- Source dimensions
- Final dimensions
- Upscaling method
- Upscaling factor
- Storyboard version

---

## 9. Default Guides

The first release must use provisional editable guides derived from supplied examples and common production practice.

Required files:

```text
guides/
├── story-guide.md
├── shoot-guide.md
├── scene-guide.md
├── style-guide-blue.md
└── contact-sheet-layout-guide.md
```

These files must be replaceable later without changes to the core workflow.

### 9.1 Story Guide

Should cover:

- Narrative structure
- Visual pacing
- Emotional progression
- Establishing, detail, action, reaction, hero, and resolution shots
- Continuity
- Sequence variety
- Avoidance of redundant frames
- Relationship between duration and frame count
- Differences between event, advertising, social, interview, and documentary work

### 9.2 Shoot Guide

Suggested provisional defaults:

- Base frame rate: 23.976 fps
- Slow motion: 120 fps for selected action moments
- Common lenses: 18mm, 25mm, 35mm, 50mm, 85mm, 100mm
- Tripod for static establishers and details
- Handheld for energy and intimacy
- Gimbal or dolly for controlled pushes
- Default aspect ratio: 16:9 unless the intended channel suggests otherwise

### 9.3 Scene Guide

Should infer and lock:

- Time of day
- Lighting direction
- Weather
- Crowd level
- Environmental details
- Background depth
- Wardrobe continuity
- Prop continuity
- Location continuity

### 9.4 Blue Style Guide

The initial generic presentation style should use:

- Dark or medium blue title bars
- Pale blue caption or metadata areas
- White or neutral panel backgrounds
- Clear sans-serif typography
- Strong frame numbering
- Minimal decoration
- Presentation-friendly spacing

The image-generation style should be configurable and should initially support at least:

- Cinematic realism
- Documentary sports
- Graphic storyboard sketch
- Clean pitch frame

---

## 10. Human Subjects

Generated subjects must remain generic unless the user supplies approved talent references.

Without an approved reference, prompts must avoid:

- Real-person names
- Celebrity likenesses
- Claims that a generated person represents a client or employee
- Unapproved persistent identity assumptions

The Plugin may define fictional recurring subject attributes for continuity, for example:

```text
Subject A:
- Generic teenage football player
- Red training jersey
- Dark shorts
- White boots
- Short dark hair
```

These attributes become continuity locks.

---

## 11. Reference Handling

A frame may use:

- Existing internal reference
- External reference
- Uploaded reference
- AI-generated reference
- No reference

External references may be downloaded into the active project repository when needed.

Each reference record must include:

- Reference ID
- Type
- Original URL or source
- Download timestamp
- Local path
- Applicable frames
- Intended use
- Optional attribution
- Optional rights or usage note
- What to borrow: framing, lighting, movement, color, location, action
- What not to copy

Suggested structure:

```text
references/
├── external/
├── uploaded/
└── generated/
```

---

## 12. Shot-List CSV Schema

Create one row per storyboard frame.

Required columns:

```csv
project_id
storyboard_id
storyboard_version
frame_id
sequence_number
shot_name
scene_name
description
narrative_purpose
subject
action
location
time_of_day
shot_size
camera_angle
camera_height
camera_position
focal_length_mm
lens_type
aperture_intent
depth_of_field
camera_movement
camera_support
frame_rate_fps
playback_intent
duration_seconds
transition_to_next
lighting
weather
wardrobe
props
priority
capture_type
assigned_shooter
scheduled_time
reference_type
reference_path
image_path
prompt
negative_prompt
continuity_notes
production_notes
status
created_at
updated_at
```

Rules:

- `frame_id` remains stable across revisions.
- `sequence_number` may change when frames are reordered.
- `prompt` contains the final committed image prompt.
- `reference_type` should be one of `internal`, `external`, `uploaded`, `AI-generated`, or `none`.
- `capture_type` should support `must-capture`, `inspiration`, `optional`, and `alternate`.
- `status` should support `proposed`, `approved`, `assigned`, `captured`, `completed`, and `omitted`.

---

## 13. Canonical Data Model

Each project must have a JSON source-of-truth manifest.

Suggested top-level structure:

```json
{
  "project_id": "soccer-campaign-001",
  "title": "Raised on Soccer",
  "active_version": 2,
  "total_duration_seconds": 30,
  "story": {},
  "shoot": {},
  "scene": {},
  "style": {},
  "creative_constants": {},
  "references": [],
  "frames": [],
  "working_state": {
    "current": {},
    "previous": {}
  },
  "versions": [],
  "exports": []
}
```

Suggested frame object:

```json
{
  "frame_id": "frame-003",
  "sequence_number": 3,
  "title": "Mural Action",
  "description": "Player volleys the ball beside the mural.",
  "prompt": "...",
  "negative_prompt": "...",
  "metadata": {
    "shot_size": "wide",
    "camera_angle": "low angle",
    "lens_mm": [18, 50],
    "movement": "handheld",
    "frame_rate_fps": 23.976,
    "duration_seconds": 3
  },
  "continuity_locks": [],
  "references": [],
  "image": {
    "low_res_path": "",
    "high_res_path": ""
  },
  "revision_history": []
}
```

---

## 14. Plugin and Skill Architecture

The application must be packaged as a reusable Plugin for company distribution.

During local development, the Plugin must use a transparent source layout with visible, editable skill entrypoints and supporting files.

### 14.1 Parent Orchestrator

Responsibilities:

- Detect user intent
- Load project state
- Route work to the correct skill
- Maintain current and previous iterations
- Enforce continuity and change scope
- Determine when approval is needed
- Coordinate image generation
- Coordinate compositing
- Coordinate deterministic upscaling
- Coordinate export

### 14.2 Recommended Skills

1. `storyboard-intake`
   - Collect and normalize input
   - Infer missing Story, Shoot, and Scene information

2. `storyboard-planning`
   - Create the AI-friendly and human-readable plan
   - Infer frame count
   - Allocate durations

3. `cinematography-planning`
   - Add lens, framing, movement, frame-rate, and lighting recommendations

4. `storyboard-review`
   - Present assumptions and accept revisions

5. `storyboard-generation`
   - Generate individual low-resolution frames
   - Build the low-resolution contact sheet

6. `storyboard-refinement`
   - Apply frame-specific edits
   - Preserve unaffected panels

7. `storyboard-versioning`
   - Manage `current`, `previous`, and committed versions

8. `storyboard-commit`
   - Deterministically upscale and finalize files

9. `shot-list-export`
   - Generate and validate CSV output

10. `reference-management`
    - Store and retrieve uploaded, external, internal, and generated references

### 14.3 Local skill requirements

Every skill should live in its own directory and include:

- A readable `SKILL.md` or `skill.md`
- Supporting templates
- Schemas where needed
- Example inputs and outputs where useful
- No hidden generated prompt bundles as the only source of truth

### 14.4 Local testing requirements

Developers must be able to:

- Run the Plugin from the repository root
- Inspect and edit every skill
- Inspect and edit guide files
- Inspect and edit contact-sheet templates
- Inspect prompt templates
- Inspect JSON schemas
- Use fixture projects
- Run tests without installing the Plugin company-wide
- Package the same source tree into the final Plugin

---

## 15. Image Generation Provider

Use the best available OpenAI image-generation model exposed by the active Codex or ChatGPT environment.

Do not hard-code a historical model name.

Use an abstraction such as:

```text
ImageGenerationProvider
├── generate_frame()
├── regenerate_frame()
├── validate_dimensions()
├── record_generation_metadata()
└── report_capabilities()
```

Record:

- Provider
- Runtime model identifier
- Generation time
- Requested dimensions
- Returned dimensions
- Source references
- Prompt hash
- Generation ID where available

The implementation must gracefully report when the local environment cannot directly invoke image generation and provide a clear adapter boundary for the supported runtime.

---

## 16. Storage and Naming

The project root is relative to the folder in which Codex is running.

Recommended base path:

```text
./storyboard-projects/
```

Project path:

```text
./storyboard-projects/<project-slug>/
```

There is no retention or automatic deletion policy.

Use three-digit, zero-padded versions:

```text
storyboard-v001-low-res.jpg
storyboard-v001-high-res.jpg
plan-v001.md
shot-list-v001.csv
manifest-v001.json
changes-v001.md
```

Stable final aliases:

```text
storyboard-final.jpg
plan-final.md
shot-list-final.csv
manifest-final.json
```

---

## 17. Recommended Repository Structure

```text
./
├── plugin/
│   ├── plugin.json
│   ├── README.md
│   ├── orchestrator/
│   ├── skills/
│   │   ├── storyboard-intake/
│   │   ├── storyboard-planning/
│   │   ├── cinematography-planning/
│   │   ├── storyboard-review/
│   │   ├── storyboard-generation/
│   │   ├── storyboard-refinement/
│   │   ├── storyboard-versioning/
│   │   ├── storyboard-commit/
│   │   ├── shot-list-export/
│   │   └── reference-management/
│   ├── guides/
│   │   ├── story-guide.md
│   │   ├── shoot-guide.md
│   │   ├── scene-guide.md
│   │   ├── style-guide-blue.md
│   │   └── contact-sheet-layout-guide.md
│   ├── templates/
│   │   ├── plan-template.md
│   │   ├── change-summary-template.md
│   │   ├── contact-sheet-template.html
│   │   └── shot-list-columns.json
│   ├── schemas/
│   │   ├── project.schema.json
│   │   ├── frame.schema.json
│   │   └── reference.schema.json
│   └── examples/
├── src/
│   ├── image_provider/
│   ├── compositing/
│   ├── upscaling/
│   ├── storage/
│   ├── planning/
│   └── export/
├── tests/
│   ├── fixtures/
│   ├── unit/
│   └── integration/
├── storyboard-projects/
├── scripts/
│   ├── run_local.sh
│   ├── validate_plugin.sh
│   ├── package_plugin.sh
│   └── create_demo_project.sh
├── pyproject.toml or package.json
└── README.md
```

The exact language may be selected by Codex, but the implementation should favor a simple, maintainable local toolchain.

---

## 18. Project Output Structure

```text
storyboard-projects/
└── <project-slug>/
    ├── project.json
    ├── guides/
    │   ├── story-guide.md
    │   ├── shoot-guide.md
    │   ├── scene-guide.md
    │   ├── style-guide-blue.md
    │   └── contact-sheet-layout-guide.md
    ├── references/
    │   ├── external/
    │   ├── uploaded/
    │   └── generated/
    ├── plans/
    │   ├── plan-v001.md
    │   └── plan-v002.md
    ├── working/
    │   ├── current/
    │   │   ├── storyboard-low-res.jpg
    │   │   ├── frames/
    │   │   ├── prompts.json
    │   │   ├── shot-list.csv
    │   │   └── manifest.json
    │   └── previous/
    ├── versions/
    │   ├── v001/
    │   └── v002/
    └── final/
        ├── storyboard-v002-high-res.jpg
        ├── storyboard-final.jpg
        ├── plan-v002.md
        ├── plan-final.md
        ├── shot-list-v002.csv
        ├── shot-list-final.csv
        ├── manifest-v002.json
        └── manifest-final.json
```

---

## 19. Functional Requirements

### Must have

- Story guide
- Default style guides
- Default shoot guide
- Default scene guide
- Sample output images
- Planning skill
- Low-resolution frame generation
- One low-resolution contact sheet
- High-resolution JPG saved locally
- Plugin packaging
- Stable frame IDs
- Human-readable plan
- Prompt displayed for every frame
- Local project manifest
- Markdown plan export
- CSV shot-list export
- Frame-level replacement through compositing
- One-level undo
- Local source inspection and editing

### Should have

- Project-specific shoot guide
- Project-specific scene guide
- Optional default review step
- Multiple-frame refinement
- Change summaries
- Drift warnings
- Existing-reference support
- Reference downloading
- Validation commands
- Fixture projects

### Nice to have

- Lightweight HTML frame-card editor
- Visual comparison between current and previous iterations
- Automated drift analysis
- Multiple contact-sheet layouts
- Additional branding controls

---

## 20. Non-Functional Requirements

### Performance

- Planning must not require image generation.
- Generate only changed panels during refinement.
- High-resolution processing occurs only on Commit.

### Reliability

- Failed operations must not overwrite approved files.
- File writes should be atomic where practical.
- CSV, Markdown, and JSON must be validated before Commit completes.
- Every committed version must be recoverable from disk.

### Usability

- Users should not need technical cinematography knowledge.
- Technical recommendations should be explained plainly.
- Outputs should remain readable on mobile.
- Chat responses should use stable frame numbers and concise summaries.

### Portability

- Use standard files: Markdown, JSON, CSV, JPG.
- Do not depend on a chat thread as the only source of project state.

### Security

- Do not upload internal references to external systems without approval.
- Record the image-generation provider used.
- Avoid placing confidential information into prompts unless necessary.

### Maintainability

- Keep prompts, guides, templates, and schemas outside compiled code.
- Use clear interfaces for image generation, storage, compositing, and upscaling.
- Include tests for critical workflows.

---

## 21. Acceptance Criteria

The first release is successful when a user can:

1. Start from a short creative idea.
2. Provide or confirm a total duration.
3. Receive a recommended frame count.
4. Override that frame count.
5. Receive useful Story, Shoot, and Scene defaults.
6. Produce a concise editable plan.
7. Skip or complete Review.
8. Generate one numbered low-resolution contact sheet.
9. Read the prompt and metadata for every frame.
10. Request a change to one frame.
11. Receive a new contact sheet where unchanged panels are reused exactly.
12. See a written change summary.
13. Undo the last refinement.
14. Commit the storyboard.
15. Receive a deterministic high-resolution JPG.
16. Receive a final Markdown plan.
17. Receive a valid CSV shot list.
18. Receive a valid JSON manifest.
19. Reopen the project from files without relying on the original chat.
20. Inspect and edit all local skills, guides, templates, and schemas.
21. Package the implementation as a reusable company Plugin.

---

## 22. Suggested MVP Test Scenario

Use the supplied soccer shot-list example as a benchmark.

The test should include:

- Multiple locations
- Static establishers
- Details
- Hero frames
- Handheld action
- Celebration
- 18mm, 25mm, 50mm, and 100mm lenses
- 23.976, 120, and 144 fps examples
- Tripod, handheld, tracking, pan, and dolly movement

Test flow:

1. Import or summarize the existing shot list.
2. Build an eight-frame storyboard plan.
3. Generate one contact sheet.
4. Change only the streetcar frame.
5. Verify all other frame image files are reused exactly.
6. Replace one frame using an uploaded reference.
7. Undo the last change.
8. Reapply the change.
9. Commit the result.
10. Validate the JPG, Markdown, CSV, and JSON outputs.

---

## 23. Final Build Priority

### Priority 1: Core local architecture

- Repository structure
- Plugin manifest
- Editable skills
- Editable guides
- Editable templates
- JSON schemas
- Local run command
- Local validation command

### Priority 2: Planning workflow

- Define
- Frame-count inference
- Duration allocation
- Plan generation
- Review

### Priority 3: Visual workflow

- Individual low-resolution frame generation
- Contact-sheet compositing
- Frame-level replacement
- One-level undo

### Priority 4: Commit workflow

- Deterministic upscaling
- JPG output
- Markdown plan
- CSV shot list
- JSON manifest

### Priority 5: Packaging and quality

- Plugin packaging
- Tests
- Fixture project
- Documentation
- Optional HTML viewer/editor

