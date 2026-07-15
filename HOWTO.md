# How to Use and Integrate the AI Skill

This guide explains how to install the `image_to_animated_html` custom skill into other projects and how to trigger your AI assistant to utilize it.

---

## 1. How to Integrate the Skill into another Project
To make this skill available in a different repository:
1. Copy the `.agents/` directory from this repository.
2. Paste the `.agents/` directory into the root folder of your other project's Git repository.
3. Commit the `.agents/` folder to Git so it is tracked and uploaded to GitHub:
   ```bash
   git add .agents/
   git commit -m "Add custom AI agent skills"
   git push
   ```

---

## 2. How the AI Agent Discovers the Skill
* When a compatible agentic AI coding assistant (like Antigravity or Gemini) loads your repository workspace, it automatically scans the root directory for the `.agents/` folder.
* The assistant parses the YAML frontmatter inside `.agents/skills/image_to_animated_html/SKILL.md`:
  ```yaml
  name: image_to_animated_html
  description: Guidelines and template structures for taking a static input image, replicating it using HTML/CSS/JS Canvas...
  ```
* No manual configuration or registration is required. The skill is automatically loaded into the agent's context.

---

## 3. How to Trigger the AI to Use the Skill
When chatting with your AI assistant, explicitly mention the skill name in your prompt. This helps the AI align its implementation exactly with the guidelines.

### Example Prompts:
* *"I have attached a reference image of a grid pattern. Replicate it as an interactive, animated web application using the **`image_to_animated_html`** skill."*
* *"Build a canvas motion graphic app based on this screenshot. Follow the WebM export and alpha transparency guidelines inside my **`image_to_animated_html`** skill."*

---

## 4. Key Guidelines Enforced by the Skill
When the AI assistant activates this skill, it automatically implements:
1. **Dynamic Easing and Scaling:** Animations are mathematically mapped to fit the duration slider perfectly (using a proportional scale factor).
2. **Transparent WebM Recording (VP9 with Alpha):** Captures high-bitrate (20 Mbps) WebM videos natively supporting transparent backgrounds, avoiding ugly green-screen chroma key fringes.
3. **Decoupled Recording Ticks:** Renders frames at exactly 60fps in real time (using `setTimeout` recursive loops instead of monitor refresh-rate bound `requestAnimationFrame`), resolving speedups on 144Hz+ displays.
4. **Buffer and Padding Safety:** Adds `0.5s` of static padding at the beginning and end of the recording to ensure no parts of the animation are lost due to encoder latency.
