---
name: image_to_animated_html
description: Guidelines and template structures for taking a static input image, replicating it using HTML/CSS/JS Canvas, and building a premium animation engine with interactive controls, user-replaceable assets (logos, text, images), and WebM exports.
---

# Replicating and Animating Images in HTML/CSS/JS

Use this skill when the user provides a reference image (such as a pattern, transition, UI design, or abstract geometry) and wants to rebuild it as an interactive, animated web application.

---

## 1. Visual Analysis Phase
When given a reference image, perform a precise spatial and element breakdown:
* **Spatial & Bounding Box Mapping:** Map out relative coordinates `(x, y, width, height)` in percentages (`0%` to `100%`) for every component (backgrounds, cards, logo containers, headings, subtext, icons). This ensures pixel-accurate structural placement on the canvas regardless of viewport resolution.
* **Text & Vector Deconstruction:** Never render static image text as hardcoded pixel bitmaps. Extract all text strings into dynamic text variables, estimating font family, weight, size, letter-spacing, and color so the user can edit the text and change fonts on the fly.
* **Asset Bounding Boxes:** Define bounding boxes with object-fit scaling (`contain` or `cover`) for logos and images so users can easily replace placeholders with their own PNG/SVG files.
* **Geometry & Layout:** Analyze structural rules: is it a grid, modular cards, concentric circles, particle flow, or geometric vector elements?
* **Color Palette & Transparency:** Extract dominant hex codes, gradients, and subtle drop-shadows. Identify if soft glows or semi-transparent overlays are present (requiring native alpha channel WebM rendering).
* **Core Animation Concept:** Determine how each visual element enters, transitions, and loops (e.g., staggered tile expands, typewriter text reveals, floating card hover physics, diagonal sweeps).
* **Interactive Parameters:** Expose controls for every extracted variable (text inputs, font dropdowns, file uploaders, color pickers, duration, stagger delay).

---

## 2. Page Layout & UI Design
Always build a premium, state-of-the-art UI:
* **Visual Theme:** Use dark mode backgrounds (`bg-slate-950`), custom typography (like Inter/Outfit), subtle custom scrollbars, and indigo/violet glow accents (`glow-indigo`).
* **Layout Structure:**
  * **Header:** Title, subtitle, and badges detailing version and render type (e.g., Canvas 2D/WebGL).
  * **Workspace:** Two-column grid (e.g., 7-8 columns for viewport, 4-5 columns for parameters).
  * **Viewport Container:** Center the canvas visually inside a dark border wrapper. Preserve standard aspect ratios (like 16:9 or 9:16) cleanly.
  * **Control Panel:** Group settings under clean sections:
    * **Layout & Timing Settings:** Aspect ratios, duration sliders, speed control.
    * **Colors & Background:** Transparent toggle, color pickers, chroma keys.
    * **Custom Asset Controls:** File selectors for custom logos (PNG/SVG) and background images, along with text input fields, font-family selectors (e.g. Google Fonts list), and font-size sliders.

---

## 3. Rendering Engine (HTML5 Canvas)
Use the Canvas 2D Context for high-performance pixel-level rendering:
* **Resolution & Transparency Control:** Define target canvas resolutions (e.g., 1920x1080 for 16:9, 1080x1920 for 9:16) and scale them down using CSS. If transparent background mode is enabled, clear the canvas dynamically using `ctx.clearRect` instead of filling with a chroma background color.
* **Asset Rendering & Placeholders:**
  * **Default Placeholders:** If the design includes a logo or image, render a default placeholder (e.g. drawing a clean geometric shape, initials, or writing a generic text placeholder like "YOUR LOGO HERE") until the user uploads their own file.
  * **Image Drawing:** Draw uploaded images using `ctx.drawImage()`, scaling and centering them inside their target layout bounding box (using object-fit cover/contain mathematics on the canvas).
  * **Text Drawing:** Draw text using standard canvas text APIs (`ctx.font`, `ctx.fillText`, `ctx.textAlign`, `ctx.textBaseline`). Ensure font styles scale proportionally with canvas dimensions.
* **Proportional Duration Scaling:** Scale cell delays and cell durations relative to the user's overall duration slider. The last animating element should finish *exactly* when the overall animation reaches `duration`.
* **Render Loop:** Run a live loop using `requestAnimationFrame(tick)` during preview, triggering a canvas redraw whenever settings or assets change.

---

## 4. Asynchronous Asset Loading & Security
To allow users to replace logos, text, and images dynamically without page reloads, implement local browser file readers and dynamic font loaders:

### A. Local Image Loading (FileReader)
Convert uploaded files into local data URLs to draw them on the canvas immediately:
```javascript
function handleImageUpload(file, stateProperty, callback) {
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (e) => {
    const img = new Image();
    img.onload = () => {
      state[stateProperty] = img; // Store image object in state
      callback(); // Redraw canvas
    };
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}
```

### B. Avoiding Canvas Taint (Security)
If loading external remote fallback assets, always configure the image's `crossOrigin` property before setting its source to prevent the canvas from becoming "tainted". A tainted canvas blocks the video recorder from capturing the stream:
```javascript
const img = new Image();
img.crossOrigin = "anonymous"; // Prevents CORS security taint
img.src = "https://example.com/logo.png";
```

### C. Dynamic Web Font Loading
When the user changes the text's font family, dynamically load the Google Font using the Web Font API to ensure the canvas renders the correct typeface:
```javascript
function loadWebFont(fontFamily, callback) {
  if (!fontFamily) return;
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = `https://fonts.googleapis.com/css2?family=${fontFamily.replace(/\s+/g, '+')}&display=swap`;
  document.head.appendChild(link);
  
  // Wait for font to load before redrawing
  document.fonts.load(`1em "${fontFamily}"`).then(() => {
    callback();
  });
}
```

---

## 5. Decoupled WebM Export Loop (with Alpha Support & High Bitrate)
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
