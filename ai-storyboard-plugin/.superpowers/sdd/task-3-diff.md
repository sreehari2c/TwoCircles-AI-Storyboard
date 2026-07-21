diff --git a/templates/plan.md b/templates/plan.md
new file mode 100644
index 0000000..1cd982d
--- /dev/null
+++ b/templates/plan.md
@@ -0,0 +1,29 @@
+# [Project Title] Storyboard Plan
+## Project Summary
+## Creative Constants
+## Production Assumptions
+## Story Progression
+## Frame Plan
+
+Duplicate the Frame section for every stable frame ID. Final plans use machine-readable Frame ID fields.
+
+### Frame 001 — [Frame Title]
+- Frame ID: frame-001
+- Purpose:
+- Description:
+- Subject:
+- Action:
+- Shot size:
+- Camera angle:
+- Camera position:
+- Lens:
+- Depth of field:
+- Movement:
+- Location:
+- Lighting:
+- Duration:
+- Transition:
+- Continuity locks:
+- Reference source:
+- Prompt:
+- Negative constraints:
diff --git a/templates/project-readme.md b/templates/project-readme.md
new file mode 100644
index 0000000..aadccac
--- /dev/null
+++ b/templates/project-readme.md
@@ -0,0 +1,20 @@
+# Storyboard Project
+
+`project.json` is the source of truth for the storyboard project.
+
+## Project folders
+
+- `guides/` contains project guidance, conventions, and creative direction.
+- `references/` contains source references used to inform the storyboard.
+- `plans/` contains planning documents and frame-level shot plans.
+- `iterations/` contains working generations and revision history.
+- `final/` contains approved final storyboard assets.
+- `exports/` contains deliverables prepared for downstream use.
+
+## Frame identity
+
+`frame_id` is stable and identifies the same storyboard frame across revisions. `sequence_number` describes its current order and may change as the story is reorganized.
+
+## Lifecycle
+
+The project lifecycle is: intake -> plan -> review -> generate -> refine/version -> commit/export.
diff --git a/templates/project.json b/templates/project.json
new file mode 100644
index 0000000..a6a354c
--- /dev/null
+++ b/templates/project.json
@@ -0,0 +1,19 @@
+{
+  "project_id": "project-slug",
+  "title": "Untitled Storyboard",
+  "schema_version": "0.1.0",
+  "status": "proposed",
+  "active_version": 0,
+  "story": {},
+  "shoot": {},
+  "scene": {},
+  "style": {},
+  "creative_constants": {
+    "locks": [],
+    "prohibited_elements": []
+  },
+  "references": [],
+  "frames": [],
+  "versions": [],
+  "exports": []
+}
diff --git a/templates/shot-list.csv b/templates/shot-list.csv
new file mode 100644
index 0000000..419b858
--- /dev/null
+++ b/templates/shot-list.csv
@@ -0,0 +1 @@
+project_id,storyboard_id,storyboard_version,frame_id,sequence_number,shot_name,scene_name,description,narrative_purpose,subject,action,location,time_of_day,shot_size,camera_angle,camera_height,camera_position,focal_length_mm,lens_type,aperture_intent,depth_of_field,camera_movement,camera_support,frame_rate_fps,playback_intent,duration_seconds,transition_to_next,lighting,weather,wardrobe,props,priority,capture_type,assigned_shooter,scheduled_time,reference_type,reference_path,image_path,prompt,negative_prompt,continuity_notes,production_notes,status,created_at,updated_at
