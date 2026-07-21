# Task 6: Add the manual acceptance runbook

## Files

Create:
- docs/storyboard-acceptance-runbook.md

## Requirements

Document setup:
- Start from a clean project folder.
- Use the supplied soccer production example spanning mural, streetcar, and high-school field.
- Require access to ChatGPT image generation.
- Do not require a hosted API, custom script, or runtime.

Document these nine acceptance actions:
1. Import or describe the mural, streetcar, and high-school-field sequence.
2. Generate an eight-frame plan using practical lens, movement, frame-rate, and shot-size variations.
3. Review the plan and verify assumptions are labeled.
4. Generate a numbered low-resolution contact sheet through ChatGPT image generation.
5. Change only the streetcar frame and verify unaffected frame records and prompts remain unchanged.
6. Replace one frame with an uploaded reference and verify the reference is frame-scoped.
7. Inspect iteration history and the change summary.
8. Commit the result and validate the Markdown plan, CSV shot list, manifest, and image references.
9. Reopen the project from storyboard-projects/<project-slug>/ without the original chat thread.

For each action list expected evidence:
- stable frame IDs;
- plan file;
- contact-sheet reference;
- unchanged prompts for unaffected frames;
- frame-scoped reference metadata;
- iteration change summary;
- final Markdown, CSV, and manifest;
- successful reopen without chat history.

Document failure checks:
- failed generation leaves active version unchanged;
- lock conflict pauses for approval;
- CSV/manifest validation failure leaves prior final outputs untouched.

Validation:
    $content = Get-Content docs/storyboard-acceptance-runbook.md -Raw
    foreach ($phrase in @('streetcar','frame-001','final-plan.md','shot-list.csv','manifest.json','without the original chat')) {
      if ($content -notmatch [regex]::Escape($phrase)) { throw "Acceptance runbook is missing: $phrase" }
    }

Expected: all benchmark and evidence terms are present.

Commit:
    git add docs/storyboard-acceptance-runbook.md
    git commit -m "docs: add storyboard acceptance runbook"

## Context

Tasks 1–5 created the plugin package, guides, templates, internal skills, and public flow. Keep the runbook no-code, concise enough to follow manually, and aligned with the exact project paths and artifact names already defined.

## Report

Write the detailed report to .superpowers/sdd/task-6-report.md. Include files changed, commit SHA, validation output, self-review, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

