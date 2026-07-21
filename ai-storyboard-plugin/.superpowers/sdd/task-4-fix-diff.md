diff --git a/skills/storyboard-commit-export/SKILL.md b/skills/storyboard-commit-export/SKILL.md
index 21ae8b0..6027728 100644
--- a/skills/storyboard-commit-export/SKILL.md
+++ b/skills/storyboard-commit-export/SKILL.md
@@ -9,21 +9,25 @@ Lock a validated iteration and publish final file artifacts without disturbing e
 
 ## Inputs
 
 Read `project.json`, the selected iteration's frame records, prompt packet, result reference, change summary, references, `templates/plan.md`, and `templates/shot-list.csv`.
 
 ## Procedure
 
 1. Lock the selected iteration and request high-resolution output at commit. Warn that deterministic upscaling may be unavailable and regeneration may differ.
 2. Build `final/final-plan.md`, `final/shot-list.csv`, and `final/manifest.json` from the locked iteration.
 3. Record export metadata, generation settings, source references, timestamps, and change summary.
-4. Validate required manifest fields; stable frame IDs; prompt presence; allowed status, reference, and capture values; required CSV header order and 45-column count; output paths; and Markdown frame identifiers.
+4. Validate required manifest fields; stable frame IDs; prompt presence; allowed status, reference, and capture values; required CSV header order and 45-column count; output paths; and Markdown frame identifiers. The allowed values are:
+   - `status`: `proposed`, `approved`, `assigned`, `captured`, `completed`, `omitted`
+   - `reference_type`: `internal`, `external`, `uploaded`, `AI-generated`, `none`
+   - `capture_type`: `must-capture`, `inspiration`, `optional`, `alternate`
+   Commit validation must reject any value outside these lists.
 5. Report success only after all validation passes. If it fails, leave prior final outputs untouched.
 
 ## Files
 
 May create or update `final/final-plan.md`, `final/shot-list.csv`, `final/manifest.json`, and export metadata in `project.json` only after a complete validation pass. May create temporary candidate outputs outside `final/` for validation.
 
 ## Outputs
 
 Produce a locked iteration record and validated final plan, shot list, manifest, high-resolution output reference or availability warning, and export metadata for delivery.
 
