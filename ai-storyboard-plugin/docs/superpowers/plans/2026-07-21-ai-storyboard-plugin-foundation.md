# AI Storyboard Codex Plugin Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Deliver a portable, no-code storyboard plugin with one registered skill, eight private stage workflows, durable review authorization, immutable iterations, and state-consistent final exports.

**Architecture:** `.codex-plugin/plugin.json` declares the standard `./skills/` registry root, which contains only the public `storyboard` skill. That `SKILL.md` loads stage Markdown and starter resources relative to its own directory; nested supporting files are not discoverable skills. Durable projects remain at plugin-root `storyboard-projects/<project-slug>/`, and all lifecycle decisions live in `project.json` rather than chat history.

**Tech Stack:** Codex plugin manifest JSON, one Markdown skill, non-discoverable Markdown workflow/reference files, JSON project manifests, Markdown plans, CSV shot lists, and ChatGPT image-generation handoff. No custom code or runtime.

## Global Constraints

- Only `skills/storyboard/SKILL.md` is registered or discoverable.
- Exactly eight stage files live under `skills/storyboard/workflow/`.
- Starter guides and templates live under `skills/storyboard/resources/`; root `workflow/`, `guides/`, and `templates/` directories do not exist.
- Projects remain under plugin-root `storyboard-projects/<project-slug>/`.
- The six status values are `proposed`, `planned`, `review`, `generated`, `refined`, and `committed`.
- Review decisions are `pending`, `approved`, and `bypassed`; generation requires durable authorization for the selected plan.
- Every generated/refined/restored/branched iteration has `prompt-packet.md`, `frames.json`, `result.json`, and `changes.md`.
- Every version record snapshots the five review fields.
- Commit/export validates candidate plan, 45-column CSV, image, and prospective committed manifest before state changes.
- A failed generation or export leaves prior project and final state unchanged.

---

## File map and responsibilities

| Path | Responsibility |
| --- | --- |
| `.codex-plugin/plugin.json` | Plugin identity and standard `./skills/` root containing one public skill. |
| `README.md` | Public package layout, portable store, and no-code scope. |
| `skills/storyboard/SKILL.md` | The sole public skill: discovery, routing, shared state fallback, response contract. |
| `skills/storyboard/workflow/*.md` | Eight private stage contracts loaded by the public skill. |
| `skills/storyboard/resources/guides/` | Seven provisional starter guide files. |
| `skills/storyboard/resources/templates/project.json` | Canonical manifest, durable review, frame/version/export contracts. |
| `skills/storyboard/resources/templates/project-readme.md` | Project lifecycle and record semantics. |
| `skills/storyboard/resources/templates/plan.md` | Machine-readable storyboard plan shape. |
| `skills/storyboard/resources/templates/shot-list.csv` | Exact 45-column CSV header. |
| `storyboard-projects/.gitkeep` | Portable project-store root. |
| `docs/storyboard-acceptance-runbook.md` | Manual lifecycle and failure evidence. |

## Task 1: Consolidate the registered skill package

**Files:**

- Modify: `.codex-plugin/plugin.json`
- Modify: `README.md`
- Modify: `skills/storyboard/SKILL.md`
- Move: `workflow/*.md` to `skills/storyboard/workflow/*.md`
- Move: `guides/` to `skills/storyboard/resources/guides/`
- Move: `templates/` to `skills/storyboard/resources/templates/`

**Interfaces:**

- Consumes: plugin-root project paths and the existing stage contracts.
- Produces: one registered skill with skill-relative private resources.

- [ ] Move every stage, guide, and template to the canonical skill-relative destination and remove root duplicates.
- [ ] Keep exactly eight stage workflow files; orchestration belongs in the public `SKILL.md`.
- [ ] Resolve `workflow/...` and `resources/...` relative to `skills/storyboard/` in every runtime instruction.
- [ ] Keep `storyboard-projects/<project-slug>/` paths relative to the plugin root.
- [ ] Verify every path named by the public skill exists.

## Task 2: Make review authorization durable

**Files:**

- Modify: `skills/storyboard/resources/templates/project.json`
- Modify: `skills/storyboard/workflow/storyboard-planning.md`
- Modify: `skills/storyboard/workflow/storyboard-review.md`
- Modify: `skills/storyboard/workflow/storyboard-generation.md`
- Modify: `skills/storyboard/workflow/storyboard-refinement.md`
- Modify: `skills/storyboard/workflow/storyboard-versioning.md`
- Modify: `skills/storyboard/SKILL.md`

**Interfaces:**

- Consumes: selected `plans/plan-v###.md` and current manifest status.
- Produces: a five-field review object and review snapshot on every version record.

- [ ] Add `decision`, `plan_version`, `authorized_for_generation`, `reviewed_at`, and `change_summary` to the manifest review object and version contract.
- [ ] Reset review to pending/unauthorized on initial planning and every plan edit.
- [ ] Record approval or explicit bypass with selected plan version and timestamp while status remains `review`.
- [ ] Route review-state add/remove/reorder/edit-plan through the review workflow; after editing, return status to `planned`.
- [ ] Require approved/bypassed durable authorization for the selected plan before generation; otherwise route to review.
- [ ] Route a no-verb `review` state to generation only when authorization is valid.

## Task 3: Complete the canonical frame/export contract

**Files:**

- Modify: `skills/storyboard/resources/templates/project.json`
- Modify: `skills/storyboard/resources/templates/project-readme.md`
- Modify: `skills/storyboard/workflow/storyboard-commit-export.md`

**Interfaces:**

- Consumes: canonical selected-iteration frame records.
- Produces: one deterministic RFC 4180 row per frame using the exact 45-column header.

- [ ] Add `revision_history`, image paths, and exact narrative/production metadata keys to the frame record.
- [ ] Include all contract gaps explicitly: `scene_name`, `narrative_purpose`, `time_of_day`, `camera_height`, `aperture_intent`, `playback_intent`, `priority`, `capture_type`, `assigned_shooter`, `scheduled_time`, `reference_type`, `production_notes`, `status`, `created_at`, and `updated_at`.
- [ ] Document an ordered source mapping for every CSV column from `project_id` through `updated_at`.
- [ ] Define list serialization and empty optional values without changing column count.

## Task 4: Make final manifest promotion state-consistent

**Files:**

- Modify: `skills/storyboard/workflow/storyboard-commit-export.md`
- Modify: `skills/storyboard/resources/templates/project-readme.md`

**Interfaces:**

- Consumes: a valid generated/refined iteration, or selected committed iteration for re-export.
- Produces: candidate artifacts and two byte-identical committed manifests.

- [ ] Validate the selected source before requesting or building candidate output.
- [ ] Build candidate plan, CSV, image/image-reference, and prospective committed `manifest.json` under `iterations/v###/commit-candidate/`.
- [ ] Put status `committed`, selected active version, and appended export records into the candidate manifest before validation.
- [ ] Validate the candidate manifest, plan, 45-column CSV, and image together.
- [ ] Promote the set and write the exact candidate manifest bytes to both `project.json` and `final/manifest.json` as the final step.
- [ ] Retain rollback data until byte equivalence passes; restore prior state on any failure.

## Task 5: Amend design and acceptance evidence

**Files:**

- Modify: `docs/superpowers/specs/2026-07-21-ai-storyboard-plugin-design.md`
- Modify: `docs/superpowers/plans/2026-07-21-ai-storyboard-plugin-foundation.md`
- Modify: `docs/storyboard-acceptance-runbook.md`

**Interfaces:**

- Consumes: the final file and state contracts.
- Produces: architecture and manual-test documentation that match runtime behavior.

- [ ] State explicitly that there is one public skill and eight private workflow files.
- [ ] Replace the obsolete eight-internal-skills file map and validation model.
- [ ] Add review reset, approval/bypass, fallback routing, and version-record evidence.
- [ ] Add prospective-manifest, 45-column, byte-equivalence, and rollback evidence.
- [ ] Preserve the soccer benchmark and no-chat reopen check.

## Task 6: Run package-wide validation

**Files:**

- Modify only files that fail a check.

**Interfaces:**

- Consumes: the complete package.
- Produces: fresh evidence for portability, state contracts, and clean version-control content.

- [ ] Parse every JSON file with `ConvertFrom-Json`.
- [ ] Confirm exactly one `SKILL.md` under `skills/`, at `skills/storyboard/SKILL.md`.
- [ ] Confirm exactly eight `skills/storyboard/workflow/*.md` files, seven guide Markdown files, and four template files.
- [ ] Confirm root `workflow/`, `guides/`, and `templates/` are absent.
- [ ] Confirm public routing names all eight workflow files and all runtime paths resolve.
- [ ] Confirm three review decisions, five review fields, six lifecycle states, four iteration filenames, review fallback choices, prospective candidate ordering, byte-identical manifests, and all 45 mapping terms.
- [ ] Run the package-wide placeholder/path checks and the skill/plugin validators.
- [ ] Run `git diff --check 28d60e9..HEAD` and inspect `git status --short`.
- [ ] Stage only `.codex-plugin`, `README.md`, `skills/storyboard`, the design, this plan, runbook, and `storyboard-projects`; never stage `.superpowers` scratch files.

## Spec coverage self-review

- One registered skill and private skill-relative files: Tasks 1 and 5.
- Portable project store and no-code scope: Task 1 and Global Constraints.
- Durable review and state routing: Task 2.
- Canonical frame records and 45-column mapping: Task 3.
- State-consistent manifest promotion and rollback: Task 4.
- Soccer acceptance and disk-only reopen: Task 5.
- Package-level structural and content checks: Task 6.
