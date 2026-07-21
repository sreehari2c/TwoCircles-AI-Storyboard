# Task 3: Add project and export templates

## Files

Create:
- templates/project.json
- templates/project-readme.md
- templates/plan.md
- templates/shot-list.csv

## Requirements

Create templates/project.json as valid JSON with exactly this representative structure:

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

Create templates/project-readme.md explaining:
- project.json is the source of truth.
- What guides, references, plans, iterations, final, and exports contain.
- frame_id is stable while sequence_number may change.
- Lifecycle: intake -> plan -> review -> generate -> refine/version -> commit/export.

Create templates/plan.md with these headings and fields:

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

The plan must say the Frame section is duplicated for every stable frame ID and final plans use machine-readable Frame ID fields.

Create templates/shot-list.csv with exactly this single header row, preserving order:

    project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at

Validation:
    Get-Content templates/project.json -Raw | ConvertFrom-Json | Out-Null
    $header = (Get-Content templates/shot-list.csv -First 1).Split(',')
    if ($header.Count -ne 45) { throw "Expected 45 CSV columns, found $($header.Count)" }
    if ($header[3] -ne 'frame_id' -or $header[4] -ne 'sequence_number') { throw 'Frame identity columns are not in the required positions' }

Expected: JSON parses and CSV has 45 columns with frame_id before sequence_number.

Commit:
    git add templates
    git commit -m "feat: add storyboard project and export templates"

## Context

Tasks 1 and 2 created the plugin scaffold and provisional guide library in the isolated worktree. Do not add runtime code, dependencies, or image processing.

## Report

Write the detailed report to .superpowers/sdd/task-3-report.md. Include files changed, commit SHA, validation output, self-review, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

