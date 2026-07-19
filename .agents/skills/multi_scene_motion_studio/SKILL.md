---
name: multi_scene_motion_studio
description: Guidelines and template structures for building multi-scene motion graphics video engines in HTML/CSS/JS Canvas, featuring global theme consistency across all scenes, scene-by-scene asset management, interactive timeline sequencing, individual WebM exports per scene (with alpha transparency & batch export), and optional full video rendering.
---

# Multi-Scene Motion Graphics Studio Engine

Use this skill when building or planning an explainer video, promo sequence, or multi-card motion graphics tool that spans multiple sequential scenes. This skill ensures **100% visual and motion consistency** across all scenes while providing **granular individual WebM clip exports per scene** for complete control in video editing software (Premiere, CapCut, DaVinci Resolve, OBS).

---

## 1. Global Theme Engine (Visual Consistency)

To guarantee that colors, typography, border radius, and logos remain perfectly uniform across Scene 1 through Scene $N$, all scenes must draw from a single centralized **`GlobalTheme` state object**. 

Editing any property in the Master Theme controls must instantly update every scene in the project.

```javascript
const GlobalTheme = {
  // Brand Palette
  colors: {
    primaryAccent: '#5B5FEF',    // Hero glow / highlight color
    secondaryAccent: '#3DFF8A',  // Data viz accent
    backgroundBase: '#0A0A0A',   // Canvas base background
    cardFill: 'rgba(22, 22, 22, 0.85)',
    cardBorder: 'rgba(255, 255, 255, 0.12)',
    textPrimary: '#FFFFFF',
    textSecondary: '#94A3B8'
  },
  // Unified Typography
  typography: {
    headingFont: 'Inter',
    headingWeight: '800',
    bodyFont: 'Inter',
    letterSpacing: '-0.02em'
  },
  // Shared Brand Logo
  assets: {
    logoImage: null,           // Loaded HTMLImageElement placeholder/user file
    logoPosition: 'top-right',  // 'top-right' | 'top-left' | 'none'
    logoSize: 48
  },
  // Glassmorphism & Geometry Tokens
  geometry: {
    borderRadius: 24,          // Card corner radius in px
    glowRadius: 30,            // Radial glow blur spread
    transparentBg: false        // Global alpha transparency toggle
  }
};
```

---

## 2. Motion Language & Choreography

All scenes must share identical motion feel (easing curves and timing proportions) so the video flows naturally:

* **Easing Curves:** Use `cubic-bezier(0.16, 1, 0.3, 1)` for smooth, premium ease-out expansions.
* **Staggered Delays:** Maintain a consistent `0.1s - 0.15s` delay step between headings, data-viz elements, and body text.
* **Duration Scaling:** Every scene calculates its local animation duration dynamically based on `scene.duration`, ensuring the final element finishes *exactly* when the scene timestamp reaches `scene.duration`.

---

## 3. Multi-Scene Data Architecture

Store the multi-scene project as a clean array of scene objects. Each scene defines its layout type, custom text, numbers/data, and individual duration:

```javascript
const StudioProject = {
  activeSceneIndex: 0,
  isLoopingSequence: false,
  scenes: [
    {
      id: 'scene_01',
      title: 'Problem Hook',
      layout: 'stat_card',
      duration: 3.5,
      data: {
        numberText: '73%',
        labelText: 'DAILY WORKFLOW WASTED',
        description: 'Teams spend hours manually stitching video assets.'
      }
    },
    {
      id: 'scene_02',
      title: 'Feature Breakdown',
      layout: 'bar_chart',
      duration: 4.0,
      data: {
        chartLabel: 'SPEED INCREASE',
        bars: [
          { label: 'Manual', value: 25 },
          { label: 'Studio AI', value: 95 }
        ]
      }
    },
    {
      id: 'scene_03',
      title: 'Outro & CTA',
      layout: 'outro_card',
      duration: 3.0,
      data: {
        heading: 'AUTOMATE YOUR MOTION',
        subheading: 'Export 60fps Transparent WebM Clips Instantly',
        ctaText: 'START BUILDING TODAY'
      }
    }
  ]
};
```

---

## 4. Flexible WebM Export Engine (Individual Clips & Batching)

Creators need granular control over individual scene clips so they can align each scene to their voiceover in their video editor. Provide three distinct export options in the UI:

### A. Individual Active Scene Export (Primary Control)
Export *only* the currently selected scene as a standalone 60fps WebM clip with native alpha transparency and `0.5s` start/end padding:

```javascript
async function exportSingleScene(sceneIndex) {
  const scene = StudioProject.scenes[sceneIndex];
  const filename = `${scene.id}-${scene.title.toLowerCase().replace(/\s+/g, '-')}.webm`;
  
  await renderAndRecordScene(scene, filename);
}
```

### B. Batch Export All Scenes
Loop through every scene sequentially and trigger an automatic WebM download for each scene clip:

```javascript
async function batchExportAllScenes() {
  const exportBtn = document.getElementById('btn-batch-export');
  exportBtn.disabled = true;

  for (let i = 0; i < StudioProject.scenes.length; i++) {
    const scene = StudioProject.scenes[i];
    const filename = `scene-${i + 1}-${scene.id}.webm`;
    
    // Update status UI
    document.getElementById('export-status').textContent = 
      `Rendering Scene ${i + 1} of ${StudioProject.scenes.length}: ${scene.title}...`;

    await renderAndRecordScene(scene, filename);
  }

  exportBtn.disabled = false;
  document.getElementById('export-status').textContent = "All Scenes Exported Successfully!";
}
```

### C. Frame-by-Frame Pacing & Padding Core (`renderAndRecordScene`)
Implement a robust 60fps frame tick loop decoupled from screen refresh rates:

```javascript
function renderAndRecordScene(scene, filename) {
  return new Promise((resolve) => {
    const stream = canvas.captureStream(60);
    const recorder = new MediaRecorder(stream, {
      mimeType: 'video/webm;codecs=vp9', // VP9 native alpha transparency support
      videoBitsPerSecond: 20000000        // 20 Mbps ultra-high quality
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
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      resolve();
    };

    const frameDuration = 1000 / 60; // 16.67ms (60 FPS pacing)
    const startPaddingFrames = 30;   // 0.5s pre-animation padding
    const endPaddingFrames = 30;     // 0.5s post-animation padding
    const animationFrames = Math.ceil(scene.duration * 60);
    const totalFrames = startPaddingFrames + animationFrames + endPaddingFrames;
    let frameCount = 0;

    const exportTick = () => {
      let currentTime = 0;
      if (frameCount < startPaddingFrames) {
        currentTime = 0;
      } else if (frameCount < startPaddingFrames + animationFrames) {
        currentTime = (frameCount - startPaddingFrames) / 60;
      } else {
        currentTime = scene.duration;
      }

      // Render the specific scene at exact timestamp
      drawScene(scene, currentTime);
      frameCount++;

      if (frameCount < totalFrames) {
        setTimeout(exportTick, frameDuration);
      } else {
        setTimeout(() => recorder.stop(), 300);
      }
    };

    recorder.onstart = () => {
      setTimeout(exportTick, frameDuration);
    };

    recorder.start();
  });
}
```

---

## 5. UI Layout Guidelines for Multi-Scene Apps

Always build a structured, multi-scene master interface:
1. **Top Navigation Header:** Global theme quick-pickers (Accent color, transparent toggle, master export batch button).
2. **Scene Selector Ribbon (Top/Bottom):** Thumbnail cards for Scene 1, Scene 2, ... Scene $N$, with options to add new scenes, reorder scenes, or click to preview.
3. **Viewport Container (Center):** Shows live preview of the currently selected scene.
4. **Sidebar Controls (Right):**
   * **Global Theme Tab:** Master font selector, global accent colors, brand logo uploader.
   * **Active Scene Tab:** Controls specific to the selected scene (title, numbers, text copy, layout type selector, duration slider, **"Export This Scene WebM"** button).
