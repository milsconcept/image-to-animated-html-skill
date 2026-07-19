# Motion Graphics & Image-to-HTML AI Skills Suite

This repository contains custom AI Agent Skills (`SKILL.md`) designed to teach AI coding assistants (like Gemini, Antigravity, or other agentic models) how to build motion graphics engines, replicate static reference images into interactive HTML/Canvas applications, and export high-quality WebM videos with native alpha transparency.

---

## Available AI Skills in this Repository

* **[.agents/skills/image_to_animated_html/SKILL.md](file:///c:/Users/Owner/Documents/Animation%20Studio/.agents/skills/image_to_animated_html/SKILL.md):** Recreates a single static reference image or layout into an interactive, animated Canvas application with duration-scaling, user-replaceable variables (logos, text, images), and WebM exports.
* **[.agents/skills/multi_scene_motion_studio/SKILL.md](file:///c:/Users/Owner/Documents/Animation%20Studio/.agents/skills/multi_scene_motion_studio/SKILL.md):** Guides the AI in building multi-scene motion graphics video engines (explainers, promo videos). Enforces global theme consistency across all scenes (brand colors, fonts, logos) and provides **individual WebM scene clip exports** plus batch exporting for video editors (CapCut, Premiere, DaVinci Resolve, OBS).
* **[.agents/skills/minimal_motion_graphics_scene_gen/SKILL.md](file:///c:/Users/Owner/Documents/Animation%20Studio/.agents/skills/minimal_motion_graphics_scene_gen/SKILL.md):** Converts video scripts or transcripts into beat-by-beat scene breakdowns and prompts for still-image generators.

---

## What is a Custom AI Skill?

A custom AI skill is a structured set of instructions, patterns, and code templates stored directly inside the codebase under `.agents/`. 

When an agentic AI assistant is run inside this repository or a project containing these skill folders, it automatically discovers and loads them into its system prompt. This equips the AI to design and build visual animations, enforce global design tokens, and handle WebM rendering with alpha transparency.
