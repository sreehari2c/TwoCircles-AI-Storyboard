diff --git a/skills/storyboard/SKILL.md b/skills/storyboard/SKILL.md
index 6da8889..c7e5918 100644
--- a/skills/storyboard/SKILL.md
+++ b/skills/storyboard/SKILL.md
@@ -4,23 +4,23 @@ description: Turn a creative brief, script, concept, or shot description into a
 ---
 
 # Smart Storyboard
 
 Coordinate one storyboard project from creative intent through a reviewable plan, image-generation handoff, targeted revisions, and production exports. Keep the work file-based, scope changes precisely, and preserve approved history.
 
 ## Project discovery
 
 Follow this order exactly:
 
-1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`.
-2. If the current workspace has one active storyboard project, use it.
-3. If multiple projects exist and none is named, ask the user to choose.
+1. If the user names a project, resolve `storyboard-projects/<project-slug>/project.json`; an explicitly named committed project may be reopened.
+2. Scan `storyboard-projects/*/project.json` for project manifests. A project is active when the current workspace is inside that project's folder, or when exactly one project manifest exists under `storyboard-projects/`.
+3. If multiple manifests exist and none is in the current project folder, ask the user to choose; do not guess based on file ordering or timestamps.
 4. If no project exists and the user supplied meaningful story input, create one from `templates/` with `storyboard-intake`.
 5. If no project exists and no meaningful story input was supplied, ask for one sentence of creative intent.
 
 Treat `project.json` as the source of truth. `frame_id` is stable (for example, `frame-001`); `sequence_number` may change when frames are reordered.
 
 ## Routing
 
 Load only the relevant internal skill and guides for the current request. Route requests as follows:
 
 | Request | Load |
