# How to Use and Integrate the AI Skills

This guide explains how to install and trigger the custom AI skills (`image_to_animated_html` and `multi_scene_motion_studio`) in your projects.

---

## 1. How to Integrate the Skills into another Project
To make these skills available in a different repository:
1. Copy the `.agents/` directory from this repository.
2. Paste the `.agents/` directory into the root folder of your target project repository.
3. Commit the `.agents/` folder to Git so it is tracked and shared across team members and AI sessions:
   ```bash
   git add .agents/
   git commit -m "Add custom AI agent skills suite"
   git push
   ```

---

## 2. How the AI Agent Discovers the Skills
* When a compatible agentic AI coding assistant (like Antigravity or Gemini) loads your repository workspace, it automatically scans the root directory for `.agents/skills/`.
* The assistant parses the YAML frontmatter inside each `SKILL.md` file (such as `name: image_to_animated_html` and `name: multi_scene_motion_studio`).
* No manual configuration or registration is required. The skills are automatically loaded into the agent's context.

---

## 3. How to Trigger the AI Skills

### Single Scene Animation (`image_to_animated_html`)
Use this when you have **one static image or design** and want to turn it into an animated canvas tool:
* *"I have attached a reference image. Replicate it as an interactive, animated web application using the **`image_to_animated_html`** skill."*
* *"Build a canvas motion graphic app based on this screenshot. Follow the WebM export and alpha transparency guidelines inside **`image_to_animated_html`**."*

### Multi-Scene Motion Studio (`multi_scene_motion_studio`)
Use this when you want to build a **multi-card explainer video or promo studio** with consistent styling across multiple scenes:
* *"Build a multi-scene motion graphics studio app for a 4-scene SaaS explainer using the **`multi_scene_motion_studio`** skill. Include a global theme engine and individual scene WebM exports."*
* *"Create an interactive 3-card presentation sequence. Use the **`multi_scene_motion_studio`** skill so colors and typography remain unified, and add a batch export button for all scenes."*

---

## 4. Key Guidelines Enforced by `multi_scene_motion_studio`
1. **Global Theme Engine:** Uses a master `GlobalTheme` token state (`colors`, `typography`, `assets`, `geometry`) so editing a color or font in the master controls instantly updates all scenes across the video.
2. **Individual Scene WebM Exports:** Provides single-scene WebM clip recording (with native alpha transparency & 0.5s padding) so creators can drop individual scene files onto their editing timeline in CapCut, Premiere, or DaVinci Resolve and align them precisely to audio voiceovers.
3. **Batch Export All Scenes:** Renders and downloads every scene in the project sequentially in one click.
4. **Decoupled 60fps Frame Pacing:** Renders frames using deterministic time steps (`16.67ms`) decoupled from screen refresh rates.
