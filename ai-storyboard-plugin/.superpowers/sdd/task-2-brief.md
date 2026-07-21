# Task 2: Add provisional guides with usable defaults

## Files

Create:
- guides/story-guide.md
- guides/shoot-guide.md
- guides/scene-guide.md
- guides/styles/cinematic-realism.md
- guides/styles/documentary-sports.md
- guides/styles/graphic-storyboard-sketch.md
- guides/styles/clean-pitch-frame.md

## Requirements

All files are Markdown instructions. Every style file must contain the exact phrase "Status: Provisional starter placeholder" and these headings: Rendering method, Contrast, Saturation, Lighting, Texture, Typical lenses, Camera movement, Framing tendencies, Subject treatment, Continuity locks, Prohibited traits. Do not include official logos, proprietary claims, or unverified Two Circles brand rules.

guides/story-guide.md must state:
- This is a provisional starter default, not official Two Circles guidance.
- General short concepts default to six frames; infer another count when narrative beats, duration, or event coverage require it.
- Sequence grammar: establishing, introduce subject/action, detail, reaction/emotion, progression/escalation, resolution/call to action.
- Ask only questions that materially change story, subject, location, action, duration, or delivery.
- Vary shot size and camera language while avoiding redundant frames.
- Duration is optional for non-linear event coverage and recorded when supplied or inferred.
- Inferred values are marked inferred until approved.

guides/shoot-guide.md must include these exact defaults:
- 16:9 delivery.
- 23.976 fps.
- 18/25/35/50/85/100mm lens family.
- Tripod for static shots.
- Handheld for action or intimacy.
- Gimbal or dolly for controlled movement.
- 120 fps only when slow motion serves the story.
It must also give plain-language recommendations for lens, support, frame rate, movement, and capture type, and prefer creative descriptions over unnecessary exposure or shutter technicalities.

guides/scene-guide.md must explain how to infer and lock location, time of day, lighting direction, weather, crowd level, background depth, wardrobe, props, signage, and atmosphere. Include a continuity checklist applied to every frame.

Style distinctions:
- cinematic-realism.md: photorealistic, premium commercial realism, controlled contrast, intentional depth of field, executable dynamic camera positions.
- documentary-sports.md: natural light, observational framing, handheld energy, authentic expressions, less polished composition.
- graphic-storyboard-sketch.md: restrained monochrome or limited color, hand-drawn appearance, clear blocking, strong silhouettes.
- clean-pitch-frame.md: polished presentation composition, controlled backgrounds, strong visual hierarchy, client-review readability.

Validation:
    $guideFiles = Get-ChildItem guides -Recurse -Filter *.md
    if ($guideFiles.Count -ne 7) { throw "Expected 7 guide files, found $($guideFiles.Count)" }
    Select-String -Path $guideFiles.FullName -Pattern 'Provisional|Status' | Out-Null

Expected: seven Markdown guide files; every style file contains the provisional status marker.

Commit:
    git add guides
    git commit -m "feat: add provisional storyboard guides"

## Context

Task 1 created the plugin scaffold in the isolated worktree. Do not add runtime code or official brand guidance. Keep the content useful as a default and explicit about its provisional status.

## Report

Write the detailed report to .superpowers/sdd/task-2-report.md. Include files changed, commit SHA, validation output, self-review, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

