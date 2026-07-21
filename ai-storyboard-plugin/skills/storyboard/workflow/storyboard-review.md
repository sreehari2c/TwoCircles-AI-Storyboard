# Storyboard Review Workflow

Make review the default gate between a planned storyboard and image generation.

## Inputs

Read `project.json`, the active `plans/plan-v###.md`, project-local overrides, reference warnings, and the latest explicit user instruction. Require status `planned` or `review`.

## Procedure

1. Present the summary, constants, ordered stable frame IDs, approved values, inferred assumptions, warnings, and conflicts.
2. When entering from `planned`, set `planned -> review` after the complete review presentation exists. Keep `review.decision: pending` and authorization false until the user decides. When re-entering from `review`, show unresolved decisions without replaying the transition.
3. Support approval; overall edits; edit/add/remove/reorder frame; alternatives; `skip review`; and proceed to creation.
4. Handle add/remove/reorder/edit-plan requests inside this workflow. Write the revised plan and frame records, reset review to `decision: pending`, `plan_version: null`, `authorized_for_generation: false`, `reviewed_at: null`, and a concise `change_summary`, then set `review -> planned`. A later bypass is a separate explicit review decision.
5. For approval, write `decision: approved`; for `skip review`, `generate immediately`, or `use your defaults`, write `decision: bypassed`. In both cases write the reviewed plan version to `plan_version`, an ISO 8601 UTC value to `reviewed_at`, `authorized_for_generation: true`, and a concise `change_summary`; keep status `review`.
6. Never treat conversation-only approval as authorization. Generation sets `generated` only after re-reading the durable review object and validating its four-file iteration contract and image result.

## Outputs

Produce the durable review decision, plan version, changed frame IDs, remaining warnings, and generation authorization or revised plan. Stop for unresolved decisions, lock conflicts, or alternatives requiring a choice. Never generate images or iteration artifacts.
