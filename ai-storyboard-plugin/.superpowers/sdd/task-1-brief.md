# Task 1: Scaffold the plugin package

## Files

- Create: .codex-plugin/plugin.json
- Modify: README.md
- Create: storyboard-projects/.gitkeep

## Requirements

Create .codex-plugin/plugin.json with this exact JSON:

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

Replace README.md with:

    # Two Circles AI Storyboard

    A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.

    ## Start

    Use the top-level storyboard workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It will discover the project state, ask only high-impact questions, create a plan, offer review, and hand approved prompts to ChatGPT image generation.

    ## Project storage

    Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.

    ## Scope

    This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.

Create an empty storyboard-projects/.gitkeep file. Do not create a sample project yet.

Validate with:

    Get-Content .codex-plugin/plugin.json -Raw | ConvertFrom-Json | Out-Null
    if (-not (Test-Path .codex-plugin/plugin.json)) { throw 'Missing plugin manifest' }
    if (-not (Test-Path storyboard-projects/.gitkeep)) { throw 'Missing project root marker' }

Commit with:

    git add .codex-plugin/plugin.json README.md storyboard-projects/.gitkeep
    git commit -m "feat: scaffold storyboard plugin package"

## Context

This is the first implementation task in the isolated worktree. The approved design and implementation plan are already committed. Keep the package pure instructions/files; do not add runtime code, dependencies, or a web app.

## Report

Write the detailed report to .superpowers/sdd/task-1-report.md. Include files changed, commit SHA, validation command and output, self-review findings, and concerns. Return only status, commits, one-line test summary, concerns, and report path.

