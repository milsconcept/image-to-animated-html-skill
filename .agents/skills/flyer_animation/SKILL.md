---
name: flyer_animation
description: Guidelines and template structures for taking a flyer design, creating placeholders for images and text, allowing image resizing/positioning (drag & drop), and applying dynamic transitions (like checkerboard sweeps, sequential zoom-ins, dynamic blurs, and text typewriter reveals) with WebM export.
---

# Flyer Animation & Alignment Customization Skill

Use this skill when a user provides a flat flyer design and wants to animate it. Unlike recreating text and graphics with HTML/CSS code, this skill instructs the assistant to detect layout layers, return an export checklist to the user, and generate a workspace pre-coded with wireframe shape placeholders, reference alignment overlays, and pre-baked animations.

---

## 1. Visual Analysis & Layer Detection Phase

When a flyer design is uploaded:
1. **Identify the Layers:** Detect all visual components that should animate (e.g. background design, guest speaker cutouts, main title text, sponsor logos, date labels).
2. **Compile the Export Checklist:** Present a numbered list of these elements to the user. Instruct the user to open their design software (e.g., Photoshop, Figma, Illustrator) and export each of these layers as a separate, transparent file (PNG or MP4 video).
3. **Draft the Placeholder Settings:** Formulate coordinates, width, height, and zoom focus targets for each layer based on the reference design.

---

## 2. Dashboard UI Layout & Styling (Scroll-Free viewport)

To prevent the user from having to scroll the page to preview animations, implement a fixed-height flex viewport:
* **Workspace Wrapper:** Set to fill the screen remaining height: `height: calc(100vh - headerHeight); overflow: hidden;`.
* **Sidebar Panels:** Lock panels to `height: 100%; overflow-y: auto;` to handle long option content independently.
* **Center Viewport:** Structured as a flex container: `display: flex; flex-direction: column; height: 100%; overflow: hidden;`.
* **Canvas Area:** Set `flex: 1; min-height: 0; overflow: hidden;` to force the canvas visual frame to shrink/expand dynamically within the available screen space.
* **Timeline controls:** Pin panel to the bottom using `flex-shrink: 0;` so it remains permanently visible.

---

## 3. Placeholders & Selection Architecture

### A. Pre-Animated Shape Placeholders
Pre-populate the layers array in JavaScript. Set the image sources (`img`) to `null` initially. Draw a styled crossed rectangle with the layer's label centered inside. All animations must run on these placeholders by default:
```javascript
function drawPlaceholder(ctx, layer) {
  // Crossed wireframe box
  ctx.fillStyle = 'rgba(30, 41, 59, 0.7)';
  ctx.fillRect(0, 0, layer.width, layer.height);
  ctx.strokeStyle = '#6366f1';
  ctx.lineWidth = 2;
  ctx.strokeRect(0, 0, layer.width, layer.height);

  ctx.strokeStyle = 'rgba(99, 102, 241, 0.2)';
  ctx.beginPath();
  ctx.moveTo(0, 0);
  ctx.lineTo(layer.width, layer.height);
  ctx.moveTo(layer.width, 0);
  ctx.lineTo(0, layer.height);
  ctx.stroke();

  // Label text centered
  ctx.fillStyle = '#a5b4fc';
  ctx.font = 'bold 20px monospace';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(layer.name.toUpperCase(), layer.width / 2, layer.height / 2);
}
```

### B. Canvas Locking vs. Sidebar Selection
To make alignment easy, support canvas layer locking:
* **Canvas Interaction:** Locked layers (like the background) must ignore hover and click drag hit-testing on the canvas so the user doesn't accidentally move them.
* **Sidebar Selection:** Do NOT block selecting locked layers from the sidebar layers list. Selecting a locked layer in the sidebar must open its properties so the user can upload files to replace the placeholder or adjust settings.

---

## 4. Reference Image Overlay & Canvas Conforming

Provide a file input selector and an opacity slider allowing the user to load the flat composite flyer image as a semi-transparent overlay. 

When the reference overlay image loads, **automatically resize the canvas dimensions** to match the guide image's natural pixel size. This ensures that the workspace scale and ratio match the flyer exactly:
```javascript
let referenceOverlay = {
  img: null,
  opacity: 0.5,
  visible: false
};

function handleOverlayUpload(file) {
  const url = URL.createObjectURL(file);
  const img = new Image();
  img.onload = () => {
    referenceOverlay.img = img;
    referenceOverlay.visible = true;
    
    // Automatically conform canvas dimensions to matching natural sizes
    state.width = img.naturalWidth;
    state.height = img.naturalHeight;
    canvas.width = state.width;
    canvas.height = state.height;

    // Resize background placeholder to fit
    const bg = state.layers.find(l => l.id === 'layer_bg');
    if (bg) {
      bg.width = state.width;
      bg.height = state.height;
    }
    
    setupCanvasDimensions(); // Re-adjust wrapper scaling
    draw();
  };
  img.src = url;
}

// Drawn on top of the editing canvas (only when not exporting)
function drawReferenceOverlay(ctx) {
  if (referenceOverlay.visible && referenceOverlay.img) {
    ctx.save();
    ctx.globalAlpha = referenceOverlay.opacity;
    ctx.drawImage(referenceOverlay.img, 0, 0, state.width, state.height);
    ctx.restore();
  }
}
```

---

## 5. Pre-Baked Cinematic Animation Pipeline

Pre-code all animation paths (zooms, blurs, dims) directly onto the placeholder layers. When the user replaces a placeholder, the asset automatically inherits these motion behaviors:

### A. Camera Viewport Matrix
Use matrix offsets to pan and zoom the camera target:
```javascript
let camera = { x: 960, y: 540, zoom: 1.0 }; // e.g. 1920x1080 viewport

function applyCamera(ctx) {
  ctx.save();
  ctx.translate(canvas.width / 2, canvas.height / 2);
  ctx.scale(camera.zoom, camera.zoom);
  ctx.translate(-camera.x, -camera.y);
}
```

### B. Sequenced Camera Cues & Focus Blurs
Chronologically coordinate camera focus points. During zoom focus:
* Apply blurs and dimming to all non-focused layers.
* Keep the focused layer sharp and at full opacity.
```javascript
function updateAnimationTimeline(t) {
  const speakerFocusStart = 1.5;
  const speakerFocusEnd = 4.5;
  
  if (t >= speakerFocusStart && t < speakerFocusEnd) {
    const elapsed = t - speakerFocusStart;
    const progress = Math.min(1.0, elapsed / 1.0); // 1.0s transition
    const ease = easeInOutQuad(progress);

    // Zoom into guest speaker coordinate
    camera.x = lerp(960, guestLayer.x + guestLayer.width / 2, ease);
    camera.y = lerp(540, guestLayer.y + guestLayer.height / 2, ease);
    camera.zoom = lerp(1.0, 1.5, ease);

    // Blur/Dim other layers
    state.layers.forEach(el => {
      if (el.id === guestLayer.id) {
        el.blur = 0;
        el.opacity = 1.0;
      } else {
        el.blur = lerp(0, 10, ease);
        el.opacity = lerp(1.0, 0.25, ease);
      }
    });
  }
}
```

---

## 6. Asset Customization & Exporter

* **Placeholder Replacement:** Clicking a shape on the canvas or selecting it in the sidebar list allows the user to upload an image/video to replace it.
* **WebM Exporter:** Implements high-quality (20 Mbps) WebM export by seeking video elements and rendering frame-by-frame.
