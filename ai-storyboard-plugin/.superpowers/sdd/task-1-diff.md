diff --git a/.codex-plugin/plugin.json b/.codex-plugin/plugin.json
new file mode 100644
index 0000000..f119696
--- /dev/null
+++ b/.codex-plugin/plugin.json
@@ -0,0 +1,17 @@
+{
+  "name": "two-circles-ai-storyboard",
+  "version": "0.1.0",
+  "description": "A guided AI storyboard workflow for turning creative briefs into production-ready visual plans and shot lists.",
+  "author": {
+    "name": "Two Circles"
+  },
+  "license": "Proprietary",
+  "keywords": [
+    "storyboard",
+    "creative planning",
+    "cinematography",
+    "shot list",
+    "image generation"
+  ],
+  "skills": "./skills/"
+}
diff --git a/README.md b/README.md
index a352d70..17ec51c 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,15 @@
-# TwoCircles-AI-Storyboard
-Ai Storyboard generation project for hackathon
+# Two Circles AI Storyboard
+
+A portable Codex plugin for turning a brief, script, concept, or shot description into a consistent storyboard plan, image-generation handoff, and production shot list.
+
+## Start
+
+Use the top-level storyboard workflow. Give it one meaningful creative-intent statement, an existing brief, or a reference. It will discover the project state, ask only high-impact questions, create a plan, offer review, and hand approved prompts to ChatGPT image generation.
+
+## Project storage
+
+Projects live in storyboard-projects/<project-slug>/ so the plugin and its projects can be exported together.
+
+## Scope
+
+This package is instruction-and-file based. It does not include a hosted API, custom runtime, database, or standalone web UI.
diff --git a/storyboard-projects/.gitkeep b/storyboard-projects/.gitkeep
new file mode 100644
index 0000000..e69de29
