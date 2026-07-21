# Task 5 Report: Smart Storyboard Orchestrator

## Status

Implemented the single public storyboard workflow skill requested by Task 5.

## Files Changed

- `skills/storyboard/SKILL.md` (committed): Public `storyboard` skill with project discovery, stage routing, lifecycle defaults, context precedence, response contract, and stop conditions.
- `.superpowers/sdd/task-5-report.md` (this report): Task evidence and review record.

## Commit

- `9efd0b49fe6c4dcc56813a658eacf2323c1e5ee3` — `feat: add smart storyboard orchestrator`

## Validation Output

```text
PASS: 8 internal skills, 5 contract phrases, 8 route targets, and exact metadata validated.
```

The validation checked the eight required stage names, the five required phrases (`skip review`, `frame-001`, `storyboard-projects`, `project.json`, and `ChatGPT image generation`), the presence of every linked `skills/<stage>/SKILL.md` route target, and the exact three-line public metadata header. `git diff --check` also returned cleanly before commit.

## Self-Review

- The file begins with the required public metadata exactly.
- Project discovery follows the required five-step order.
- Every required request route maps to its existing internal skill path.
- The normal lifecycle, review defaults and bypass behavior, frame-count defaults, guide override behavior, context precedence, and stable frame identity rules are explicit.
- User-visible response fields, generation/refinement-specific response requirements, and all four explicit stop conditions are present.
- The flow prohibits unsupported success claims and whole-storyboard regeneration for a frame-scoped request.
- The commit changes only `skills/storyboard/SKILL.md`; no runtime code or additional public stage skill was added.

## Concerns

None. The report is intentionally not included in the task commit because the brief's prescribed commit stages only `skills/storyboard/SKILL.md`.

## Review Fix: Active Project Discovery

Updated `skills/storyboard/SKILL.md` to define deterministic active-project discovery: scan `storyboard-projects/*/project.json`; prefer a project containing the current workspace; use the sole manifest when exactly one exists; ask the user when multiple manifests exist without a containing project; never infer from ordering or timestamps; and allow explicitly named committed projects to be reopened.

### Fix Command/Results

```text
focused routing/phrase validation: PASS: all eight routes and five contract phrases present.
git diff --check: PASS
```
