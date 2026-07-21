# Task 7: Run package-wide validation and perform the smoke review

## Scope

Validate the complete plugin package in:
C:\KORE\TwoCircles-AI-Storyboard\.worktrees\ai-storyboard-plugin

If a package file fails a required check, fix only the relevant package file, re-run the covering check, and commit the correction. Do not add runtime code or broaden scope.

## Required checks

1. Validate every JSON file:

    Get-ChildItem -Recurse -Filter *.json | ForEach-Object {
      Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null
    }

Expected: no JSON parse errors.

2. Validate required package paths:

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

3. Validate all seven guides exist and provisional markers remain.

4. Validate all eight internal skills and the public skill have frontmatter and required route/contracts.

5. Validate the CSV header has 45 columns and exact order.

6. Search package files for forbidden unfinished-work markers:

    rg -n "TODO|TBD|implement later|fill in details|add appropriate error handling" . --glob '!docs/superpowers/specs/**' --glob '!docs/superpowers/plans/**'

Expected: no matches in package files. Provisional guide markers are allowed and are not unfinished work.

7. Check all committed changes since the implementation base for whitespace:

    git diff --check 28d60e9..HEAD

Expected: no whitespace errors. If trailing blank lines from the guide task are reported, remove only those trailing blank lines and commit the correction.

8. Inspect git status and confirm all intended package files are committed. The ignored .superpowers execution ledger and review artifacts must not be added to git.

## Manual smoke review

Read in this order:
- README.md
- skills/storyboard/SKILL.md
- all eight internal SKILL.md files
- guides/story-guide.md, guides/shoot-guide.md, guides/scene-guide.md
- all four style files
- templates/project.json, templates/project-readme.md, templates/plan.md, templates/shot-list.csv
- docs/storyboard-acceptance-runbook.md

Confirm:
- a new project from one sentence routes intake -> plan -> review;
- review bypass keeps the internal plan;
- generation records a contact-sheet prompt packet and image reference;
- frame refinement names affected and untouched frame IDs;
- commit/export uses final/final-plan.md, final/shot-list.csv, and final/manifest.json;
- the package never claims success without corresponding files/references.

## Final commit

If validation required changes:
    git add .
    git commit -m "chore: validate storyboard plugin foundation"

If no changes were needed, do not create an empty commit.

## Report

Write the detailed report to .superpowers/sdd/task-7-report.md. Include every command run, output summary, files changed, any fix commit, self-review, and concerns. Return only status, commits, one-line validation summary, concerns, and report path.

