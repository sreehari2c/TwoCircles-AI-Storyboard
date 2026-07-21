# Two Circles AI Storyboard

A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.

## Start

Use the single registered `storyboard` workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It discovers project state, asks only high-impact questions, creates a plan, offers review, and hands approved prompts to ChatGPT image generation.

Only `skills/storyboard/SKILL.md` is registered. Its eight stage instructions live in `skills/storyboard/workflow/`, and its starter guides and templates live in `skills/storyboard/resources/`. These are ordinary skill-relative resources, not separately discoverable skills.

## Project storage

Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.

## Scope

This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.
