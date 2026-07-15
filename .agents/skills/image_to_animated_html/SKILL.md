---
name: image_to_animated_html
description: Guidelines and template structures for taking a static input image, replicating it using HTML/CSS/JS Canvas, and building a premium animation engine with interactive controls and WebM exports.
---

# Replicating and Animating Images in HTML/CSS/JS

Use this skill when the user provides a reference image (such as a pattern, transition, UI design, or abstract geometry) and wants to rebuild it as an interactive, animated web application.

---

## 1. Visual Analysis Phase
When given an image, inspect and break down:
* **Geometry & Layout:** Is it a grid (checkerboard, mosaic), a flow of particles, concentric circles, or a geometric vector-like pattern?
* **Color Palette & Transparency:** Extract dominant colors. Determine if elements have glowing, anti-aliased, or semi-transparent edges. Identify if the animation is best presented on a transparent background (native alpha channel) rather than a solid chroma key green screen, as chroma keying soft borders/glows creates ugly color fringes.
* **Core Animation Concept:** How should the static elements move? (e.g. expanding tiles, cascading diagonal sweeps, rotating nodes, floating physics elements).
* **Interactive Parameters:** What controls should the user have? (e.g. transparency toggles, color pickers, grid size/rows, speed/duration, noise/glitch factors, size/scale, spacing).

---

## 2. Page Layout & UI Design
Always build a premium, state-of-the-art UI:
* **Visual Theme:** Use dark mode backgrounds (`bg-slate-950`), custom typography (like Inter/Outfit), subtle custom scrollbars, and indigo/violet glow accents (`glow-indigo`).
* **Layout Structure:**
  * **Header:** Title, subtitle, and badges detailing version and render type (e.g., Canvas 2D/WebGL).
  * **Workspace:** Two-column grid (e.g., 7-8 columns for viewport, 4-5 columns for parameters).
  * **Viewport Container:** Center the canvas visually inside a dark border wrapper. Preserve standard aspect ratios (like 16:9 or 9:16) cleanly.
  * **Control Panel:** Group sliders, inputs, toggles, and dropdowns under neat, descriptive headings. Use custom styled inputs matching the dark premium theme.

---

## 3. Rendering Engine (HTML5 Canvas)
Use the Canvas 2D Context for high-performance pixel-level rendering:
* **Resolution & Transparency Control:** Define target canvas resolutions (e.g., 1920x1080 for 16:9, 1080x1920 for 9:16) and scale them down using CSS. If transparent background mode is enabled, clear the canvas dynamically using `ctx.clearRect` instead of filling with a chroma background color:
  ```javascript
  function draw() {
    if (state.transparentBg) {
      ctx.clearRect(0, 0, state.width, state.height);
    } else {
      ctx.fillStyle = state.colorChroma;
      ctx.fillRect(0, 0, state.width, state.height);
    }
    // ... render elements ...
  }
  ```
* **Deterministic Timing & Delays:** Define animation cell structures and coordinate-based delays:
  ```javascript
  // Example delay calculation based on distance from center
  const centerX = cols / 2;
  const centerY = rows / 2;
  const dist = Math.hypot(c - centerX, r - centerY);
  const delay = dist * staggerSpread + Math.random() * noise;
  ```
* **Proportional Duration Scaling:** Scale cell delays and cell durations relative to the user's overall duration slider. The last animating element should finish *exactly* when the overall animation reaches `duration`:
  ```javascript
  const unscaledCellDuration = 0.5;
  const scaleFactor = totalUnscaledDuration > 0 ? (state.duration / totalUnscaledDuration) : 1;
  cell.delay = unscaledDelay * scaleFactor;
  cell.cellDuration = unscaledCellDuration * scaleFactor;
  ```
* **Render Loop:** Run a live loop using `requestAnimationFrame(tick)` during preview:
  ```javascript
  function tick(timestamp) {
    if (!state.isPlaying) return;
    const elapsed = (timestamp - lastFrameTime) / 1000;
    lastFrameTime = timestamp;
    state.currentTime += elapsed;
    
    if (state.currentTime >= state.duration) {
      if (state.isLooping) state.currentTime = 0;
      else {
        state.currentTime = state.duration;
        state.isPlaying = false;
      }
    }
    draw();
    if (state.isPlaying) requestAnimationFrame(tick);
  }
  ```

---

## 4. Decoupled WebM Export Loop (with Alpha Support & High Bitrate)
Never record video bound directly to `requestAnimationFrame` ticks, as this causes speed and duration corruption on high-refresh-rate displays. Instead, use a decoupled frame-by-frame recorder with start and end padding. 

To preserve soft glows, gradients, and anti-aliased borders, export with **native transparency (alpha channel)** and utilize a high bitrate (e.g. **20 Mbps**) to eliminate blocky compression artifacts around transparent edges:

```javascript
document.getElementById('btn-export').addEventListener('click', async () => {
  // Capture frames and construct video clip (includes alpha channels by default)
  const stream = canvas.captureStream(60); 
  const recorder = new MediaRecorder(stream, {
    mimeType: 'video/webm;codecs=vp9', // VP9 codec natively supports alpha transparency
    videoBitsPerSecond: 20000000 // 20 Mbps ultra-high quality to avoid glow artifacts
  });

  const recordedChunks = [];
  recorder.ondataavailable = (e) => {
    if (e.data.size > 0) recordedChunks.push(e.data);
  };

  recorder.onstop = () => {
    const blob = new Blob(recordedChunks, { type: 'video/webm' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `animation-export.webm`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };

  // Reset states
  state.isPlaying = false;
  state.currentTime = 0;
  draw();

  const frameDuration = 1000 / 60; // 16.67ms (60 FPS pacing)
  const startPaddingFrames = 30;   // 0.5s pre-animation padding
  const endPaddingFrames = 30;     // 0.5s post-animation padding
  const animationFrames = Math.ceil(state.duration * 60);
  const totalFrames = startPaddingFrames + animationFrames + endPaddingFrames;
  let frameCount = 0;

  const exportTick = () => {
    // Determine timestamp state
    if (frameCount < startPaddingFrames) {
      state.currentTime = 0; // Freeze at start state
    } else if (frameCount < startPaddingFrames + animationFrames) {
      state.currentTime = (frameCount - startPaddingFrames) / 60; // Animate
    } else {
      state.currentTime = state.duration; // Freeze at end state
    }

    draw();
    frameCount++;

    if (frameCount < totalFrames) {
      setTimeout(exportTick, frameDuration);
    } else {
      // Small buffer delay to allow the encoder to flush its queues
      setTimeout(() => recorder.stop(), 300);
    }
  };

  // Wait for the recorder to start encoding before drawing frames
  recorder.onstart = () => {
    setTimeout(exportTick, frameDuration);
  };

  recorder.start();
});
```
