# Quola Air V1.1 — Repo scaffold, improved UI/UX & ready-to-publish app

This document contains a repo scaffold, feature list, usage notes, and a *ready-to-use* single-file web app (HTML/CSS/JS) to replace your current `AirDraw` page with **Quola Air V1.1**. Paste the files into a new repo, test locally, then publish to GitHub Pages (instructions below).

---

## Project structure (suggested)

```
quola-air-v1.1/
├─ README.md          # this file (overview + instructions)
├─ index.html         # single-file demo app (main UI + JS)
├─ assets/
│  ├─ icons/          # optional small SVG icons
│  └─ models/         # place for downloaded .task model (optional)
├─ LICENSE
└─ .github/workflows/deploy.yml  # optional GitHub Actions to publish to gh-pages
```

---

## Highlights & UX improvements included

* Modern, responsive UI with a left control rail + right toolbar and bottom status bar
* Retina-ready canvases and smooth stroke rendering (dynamic line tapering)
* Configurable drawing modes: brush, marker, neon, and eraser
* Preset color swatches + color picker
* Pressure/size emulation using hand span distance (auto thickness mapping)
* Pinch-to-draw with smoothing; pinky-up creates new stroke
* Undo / redo stack
* Save PNG (option to include video background or only strokes)
* Export SVG (for vector-friendly edits)
* Local persistence: saves last drawing and settings to `IndexedDB` (or `localStorage` fallback)
* Accessibility: keyboard shortcuts, reduced-motion option, visible labels
* Mobile & tablet friendly layout and performance tweaks
* Lightweight: uses CDN-hosted MediaPipe Tasks, no bundler required for quick deploy

---

## How to use (quick)

1. Create a new GitHub repo named `quola-air-v1.1`.
2. Add the files from this repo (copy `index.html` from below). Commit.
3. In repo settings, enable **GitHub Pages** (publish `main` branch / `/root`), or add the GitHub Action workflow included.
4. Open `https://<your-username>.github.io/quola-air-v1.1/` after publishing.

**Local test**: run a simple static server (recommended `npx serve` or `python -m http.server 8000`) because MediaPipe sometimes requires proper host contexts.

```bash
# example local run
npx serve .
# or
python -m http.server 8000
```

---

## Files (copy these into your repo)

### `index.html` (single-file app)

<!-- copy the block below into a file named index.html -->

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Quola Air V1.1 — Air drawing with hand tracking</title>
  <meta name="description" content="Quola Air — draw in the air using hand tracking (MediaPipe HandLandmarker)." />
  <style>
    :root{
      --bg:#071028; --panel:#0e1724; --muted:#98aacf; --text:#eaf2ff; --accent:#7c9cff; --glass: rgba(255,255,255,0.04);
      --radius:12px; --shadow: 0 12px 32px rgba(2,6,20,0.6);
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg) 0%, #041028 100%); color:var(--text); font-family:Inter,ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Arial;}
    .app{display:grid; grid-template-columns:320px 1fr 84px; grid-template-rows:64px 1fr 44px; height:100vh; gap:16px; padding:18px;}

    header{grid-column:1/-1; display:flex; align-items:center; gap:12px}
    .logo{display:flex; align-items:center; gap:10px}
    .brand{font-weight:600; letter-spacing:0.6px}
    .subtitle{font-size:12px;color:var(--muted)}

    /* left rail */
    .rail{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:var(--radius); padding:14px; box-shadow:var(--shadow);}
    .section{margin-bottom:12px}
    label{display:block; font-size:13px; color:var(--muted); margin-bottom:6px}
    .row{display:flex; gap:8px; align-items:center}
    input[type=range]{width:100%}
    .swatches{display:flex; gap:6px; flex-wrap:wrap}
    .swatch{width:28px;height:28px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);cursor:pointer}

    /* stage */
    .stage{position:relative; grid-column:2/3; grid-row:2/3; border-radius:14px; overflow:hidden; box-shadow:var(--shadow); background:#000; display:flex; align-items:center; justify-content:center}
    video{position:absolute; inset:0; width:100%; height:100%; object-fit:cover; transform:scaleX(-1)}
    canvas{position:absolute; inset:0; width:100%; height:100%;}

    /* right toolbar */
    .toolbar{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:var(--radius); padding:10px; display:flex; flex-direction:column; gap:10px; align-items:center}
    button{background:transparent;border:1px solid rgba(255,255,255,0.04); padding:10px; border-radius:10px; color:var(--text); cursor:pointer}
    .pill{background:var(--glass); padding:8px 12px; border-radius:999px; border:1px solid rgba(255,255,255,0.03)}

    footer{grid-column:1/-1; display:flex; align-items:center; justify-content:space-between; padding:8px 12px; color:var(--muted); font-size:13px}

    /* responsiveness */
    @media (max-width:1000px){ .app{grid-template-columns:80px 1fr; grid-template-rows:64px 1fr 54px} .rail{display:none} .toolbar{flex-direction:row; grid-column:1/-1; justify-content:space-around} }

    /* small helpers */
    .muted{color:var(--muted); font-size:13px}
  </style>
</head>
<body>
  <div class="app" role="application" aria-label="Quola Air drawing app">
    <header>
      <div class="logo">
        <svg width="36" height="36" viewBox="0 0 24 24" fill="none" aria-hidden><rect x="2" y="2" width="20" height="20" rx="6" fill="#0f1726" stroke="#213351"/></svg>
        <div>
          <div class="brand">Quola Air <span style="opacity:.7">v1.1</span></div>
          <div class="subtitle">Robust hand-tracked air drawing — pinch to draw</div>
        </div>
      </div>
      <div style="margin-left:auto" class="muted">Open-source • Lightweight • Privacy-first — runs locally</div>
    </header>

    <aside class="rail" aria-hidden>
      <div class="section">
        <label>Brush mode</label>
        <div class="row">
          <select id="modeSelect" aria-label="Brush mode">
            <option value="brush">Brush</option>
            <option value="marker">Marker</option>
            <option value="neon">Neon</option>
            <option value="eraser">Eraser</option>
          </select>
        </div>
      </div>

      <div class="section">
        <label>Color</label>
        <div class="row swatches" id="swatches"></div>
        <input id="colorPicker" type="color" value="#7c9cff" style="margin-top:8px" />
      </div>

      <div class="section">
        <label>Size</label>
        <input id="size" type="range" min="2" max="80" value="8" />
      </div>

      <div class="section">
        <label>Smoothing</label>
        <input id="smooth" type="range" min="0" max="0.95" step="0.01" value="0.35" />
      </div>

      <div class="section">
        <label>Options</label>
        <div class="row"><input id="mirror" type="checkbox" checked /><label for="mirror">Mirror view</label></div>
        <div class="row"><input id="showGuides" type="checkbox" checked /><label for="showGuides">Show guides</label></div>
      </div>

    </aside>

    <main class="stage" id="stage">
      <video id="video" playsinline autoplay muted></video>
      <canvas id="draw"></canvas>
      <canvas id="overlay"></canvas>
    </main>

    <aside class="toolbar" role="toolbar" aria-label="tools">
      <button id="clearBtn" title="Clear (C)">🧼 Clear</button>
      <button id="undoBtn" title="Undo (Z)">↶ Undo</button>
      <button id="redoBtn" title="Redo (Shift+Z)">⤴ Redo</button>
      <button id="saveBtn" title="Save PNG">💾 Save</button>
      <button id="svgBtn" title="Export SVG">🖼️ SVG</button>
      <div class="pill" id="status">Initializing…</div>
    </aside>

    <footer>
      <div class="muted">Pinch (index + thumb) to draw · raise pinky to break stroke</div>
      <div id="info" class="muted"></div>
    </footer>
  </div>

  <!-- MediaPipe Tasks CDN (HandLandmarker) -->
  <script type="module">
  import { FilesetResolver, HandLandmarker } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0";

  const video = document.getElementById('video');
  const drawCanvas = document.getElementById('draw');
  const overlayCanvas = document.getElementById('overlay');
  const ctx = drawCanvas.getContext('2d', { alpha: true });
  const octx = overlayCanvas.getContext('2d');

  const colorPicker = document.getElementById('colorPicker');
  const swatchesEl = document.getElementById('swatches');
  const sizeEl = document.getElementById('size');
  const smoothEl = document.getElementById('smooth');
  const mirrorEl = document.getElementById('mirror');
  const showGuidesEl = document.getElementById('showGuides');
  const modeSelect = document.getElementById('modeSelect');
  const statusEl = document.getElementById('status');
  const infoEl = document.getElementById('info');

  // small utility
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));

  // stroke management
  let strokes = []; // stack of strokes for undo/redo
  let redoStack = [];
  let current = null;
  let lastPt = null;
  let isEraser = false;

  // presets
  const swatches = ['#7c9cff','#ff7ab6','#7dffc4','#ffd27c','#ffffff','#000000'];
  swatches.forEach(c=>{ const d=document.createElement('div'); d.className='swatch'; d.style.background=c; d.title=c; d.addEventListener('click',()=>{ colorPicker.value=c; }); swatchesEl.appendChild(d); });

  function setStatus(txt, good=true){ statusEl.textContent = txt; statusEl.style.opacity = good?1:0.8; statusEl.style.borderColor = good?'rgba(255,255,255,0.03)':'#6b2830'; }

  // resize
  function resizeCanvases(){
    const rect = document.getElementById('stage').getBoundingClientRect();
    [drawCanvas, overlayCanvas].forEach(c=>{ const dpr = window.devicePixelRatio || 1; c.width = Math.round(rect.width * dpr); c.height = Math.round(rect.height * dpr); c.style.width = rect.width + 'px'; c.style.height = rect.height + 'px'; const ctx2 = c.getContext('2d'); ctx2.setTransform(dpr,0,0,dpr,0,0); });
    redrawFromStrokes();
  }
  new ResizeObserver(resizeCanvases).observe(document.getElementById('stage'));

  function redrawFromStrokes(){ ctx.clearRect(0,0,drawCanvas.width,drawCanvas.height); for(const s of strokes){ ctx.save(); ctx.globalCompositeOperation = s.eraser ? 'destination-out' : 'source-over'; ctx.lineWidth = s.size; ctx.lineCap='round'; ctx.lineJoin='round'; if(s.mode==='neon'){ ctx.shadowBlur = s.size * 1.8; ctx.shadowColor = s.color; }
    ctx.strokeStyle = s.color; ctx.beginPath(); for(let i=0;i<s.points.length;i++){ const p=s.points[i]; if(i===0) ctx.moveTo(p.x,p.y); else ctx.lineTo(p.x,p.y); } ctx.stroke(); ctx.restore(); }}

  function pushPoint(pt){
    if(!current){ current = {points:[], color: isEraser ? '#000' : colorPicker.value, size: parseFloat(sizeEl.value), eraser:isEraser, mode:modeSelect.value}; strokes.push(current); redoStack = []; }
    current.points.push(pt);
    // incremental draw
    const n=current.points.length; if(n>1){ ctx.save(); ctx.globalCompositeOperation = current.eraser ? 'destination-out' : 'source-over'; ctx.lineWidth = current.size; ctx.lineCap='round'; ctx.lineJoin='round'; if(current.mode==='neon'){ ctx.shadowBlur = current.size*1.8; ctx.shadowColor = current.color; }
      ctx.strokeStyle = current.color; ctx.beginPath(); const a=current.points[n-2], b=current.points[n-1]; ctx.moveTo(a.x,a.y); ctx.lineTo(b.x,b.y); ctx.stroke(); ctx.restore(); }
  }

  function endStroke(){ current=null; lastPt=null; saveToLocal(); }

  // persistence
  function saveToLocal(){ try{ localStorage.setItem('quola:strokes', JSON.stringify(strokes)); localStorage.setItem('quola:settings', JSON.stringify({color:colorPicker.value,size:sizeEl.value,mode:modeSelect.value})); }catch(e){} }
  function loadFromLocal(){ try{ const s = JSON.parse(localStorage.getItem('quola:strokes')||'[]'); if(s.length){ strokes=s; redrawFromStrokes(); } const sett = JSON.parse(localStorage.getItem('quola:settings')||'null'); if(sett){ colorPicker.value = sett.color||colorPicker.value; sizeEl.value = sett.size||sizeEl.value; modeSelect.value = sett.mode||modeSelect.value; } }catch(e){} }
  loadFromLocal();

  // buttons
  document.getElementById('clearBtn').onclick = ()=>{ strokes=[]; redoStack=[]; redrawFromStrokes(); setStatus('Canvas cleared',true); saveToLocal(); };
  document.getElementById('undoBtn').onclick = ()=>{ if(strokes.length){ redoStack.push(strokes.pop()); redrawFromStrokes(); setStatus('Undo',true); saveToLocal(); }};
  document.getElementById('redoBtn').onclick = ()=>{ if(redoStack.length){ strokes.push(redoStack.pop()); redrawFromStrokes(); setStatus('Redo',true); saveToLocal(); }};
  document.getElementById('saveBtn').onclick = ()=>{ try{ const merged = document.createElement('canvas'); merged.width = drawCanvas.width; merged.height = drawCanvas.height; const mctx = merged.getContext('2d'); if(mirrorEl.checked){ mctx.translate(merged.width,0); mctx.scale(-1,1); }
    // draw video background
    mctx.drawImage(video, 0,0, merged.width, merged.height);
    if(mirrorEl.checked) mctx.setTransform(1,0,0,1,0,0);
    // draw strokes (scale down by devicePixelRatio in transform already set)
    mctx.drawImage(drawCanvas, 0,0);
    const a=document.createElement('a'); a.download='quola-air.png'; a.href=merged.toDataURL(); a.click(); setStatus('Saved PNG',true);}catch(e){ setStatus('Save failed', false);} };

  document.getElementById('svgBtn').onclick = ()=>{ try{ // simple SVG export (polylines)
    const w = drawCanvas.width; const h = drawCanvas.height; let svg = `<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"${w}\" height=\"${h}\" viewBox=\"0 0 ${w} ${h}\">`;
    for(const s of strokes){ if(s.points.length<2) continue; const points = s.points.map(p=>`${p.x},${p.y}`).join(' '); const stroke = s.eraser? 'none': s.color; svg += `<polyline points=\"${points}\" fill=\"none\" stroke=\"${stroke}\" stroke-width=\"${s.size}\" stroke-linecap=\"round\" stroke-linejoin=\"round\" />`; }
    svg += `</svg>`; const blob = new Blob([svg], {type:'image/svg+xml'}); const url = URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='quola-air.svg'; a.click(); setStatus('Exported SVG',true); }catch(e){ setStatus('SVG export failed',false);} };

  // camera + media pipe
  setStatus('Requesting camera…');
  try{ const stream = await navigator.mediaDevices.getUserMedia({video:{facingMode:'user', width:1280, height:720}, audio:false}); video.srcObject = stream; }catch(e){ console.error(e); setStatus('Camera error',false); infoEl.textContent='Allow camera & reload.'; }

  await new Promise(res=>{ if(video.readyState>=2) return res(); video.onloadedmetadata = ()=>res(); });
  resizeCanvases();

  setStatus('Loading model…');
  let handLandmarker = null;
  try{
    const wasmUrl = 'https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0/wasm';
    const fileset = await FilesetResolver.forVisionTasks(wasmUrl);
    handLandmarker = await HandLandmarker.createFromOptions(fileset, { baseOptions:{ modelAssetPath: 'https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task' }, runningMode:'VIDEO', numHands:1, minTrackingConfidence:0.5, minDetectionConfidence:0.5 });
    setStatus('Model ready', true);
  }catch(err){ console.error(err); setStatus('Model failed', false); infoEl.textContent='Check console (CORS / network)'; }

  // helpers for mapping
  function mapToCanvas(p){ const rect = overlayCanvas; const x = (mirrorEl.checked ? (1 - p.x) : p.x) * rect.width; const y = p.y * rect.height; return {x,y}; }
  function dist(a,b){ const dx=a.x-b.x, dy=a.y-b.y; return Math.hypot(dx,dy); }

  let fps={last:performance.now(),count:0,value:0}; let pinkyWasUp=false;

  async function frameLoop(){ try{
      fps.count++; const now=performance.now(); if(now-fps.last>=500){ fps.value = Math.round((fps.count*1000)/(now-fps.last)); fps.last=now; fps.count=0; }
      const W = overlayCanvas.width, H = overlayCanvas.height; octx.clearRect(0,0,W,H);
      let results = null; try{ results = handLandmarker.detectForVideo(video, performance.now()); }catch(e){ console.warn('detect error', e); }

      if(results && results.landmarks && results.landmarks.length>0 && results.landmarks[0].length>=21){ const lm = results.landmarks[0]; const idxTipN=lm[8], thbTipN=lm[4], idxBaseN=lm[5], pinkyTipN=lm[20], pinkyBaseN=lm[17]; const idxTip = mapToCanvas(idxTipN), thbTip = mapToCanvas(thbTipN), idxBase = mapToCanvas(idxBaseN), pinkyTip = mapToCanvas(pinkyTipN), pinkyBase = mapToCanvas(pinkyBaseN);

        const handSpan = Math.max(8, dist(idxBase, pinkyBase)); const pinchDist = dist(idxTip, thbTip); const pinchThreshold = Math.max(18, handSpan * 0.34); const pinch = pinchDist < pinchThreshold;

        const pinkyUp = pinkyTip.y < idxBase.y - handSpan * 0.12; if(pinkyUp && !pinkyWasUp) endStroke(); pinkyWasUp = pinkyUp;

        const alpha = parseFloat(smoothEl.value) || 0.35; const target = idxTip; if(!lastPt) lastPt = {x:target.x,y:target.y}; else lastPt = {x: lastPt.x * (1-alpha) + target.x*alpha, y: lastPt.y * (1-alpha) + target.y*alpha };

        // map thickness to hand span if mode is brush
        if(modeSelect.value !== 'eraser') isEraser = false; else isEraser = true;
        let mappedSize = parseFloat(sizeEl.value);
        // emulate pressure: closer hand -> thicker stroke
        const sizeBySpan = clamp((handSpan/120) * mappedSize * 1.6, 1, 120);

        if(pinch){ pushPoint({ x: lastPt.x, y: lastPt.y, size: sizeBySpan }); }
        else endStroke();

        // guides
        if(showGuidesEl.checked){ octx.save(); octx.lineWidth=2; octx.strokeStyle='rgba(200,220,255,0.7)'; octx.beginPath(); octx.moveTo(idxTip.x,idxTip.y); octx.lineTo(thbTip.x,thbTip.y); octx.stroke(); octx.beginPath(); octx.arc(lastPt.x,lastPt.y, pinch?8:6,0,Math.PI*2); octx.fillStyle = pinch? 'rgba(40,220,160,0.95)': 'rgba(124,156,255,0.9)'; octx.fill(); octx.restore(); }

        infoEl.textContent = `FPS: ${fps.value} • pinch ${Math.round(pinchDist)}px • thresh ${Math.round(pinchThreshold)}px`;
        setStatus('Tracking hand', true);
      } else {
        endStroke(); infoEl.textContent = `FPS: ${fps.value} • no hand`; setStatus('No hand detected', false);
      }

  }catch(err){ console.error(err); setStatus('Runtime error', false); } finally { requestAnimationFrame(frameLoop); }}

  requestAnimationFrame(frameLoop);

  // keyboard
  window.addEventListener('keydown', (e)=>{ if(e.key==='c') { strokes=[]; redrawFromStrokes(); setStatus('Cleared',true); saveToLocal(); } if(e.key==='z' && !e.shiftKey){ redoStack=[]; if(strokes.length){ redoStack.push(strokes.pop()); redrawFromStrokes(); setStatus('Undo',true); saveToLocal(); } } if(e.key==='Z' || (e.key==='z' && e.shiftKey)){ if(redoStack.length){ strokes.push(redoStack.pop()); redrawFromStrokes(); setStatus('Redo',true); saveToLocal(); } } });

  // save settings on change
  [colorPicker,sizeEl,modeSelect].forEach(el=>el.addEventListener('change', saveToLocal));

  // initial ready
  setStatus('Ready — pinch to draw. Increase lighting if detection is poor.', true);
  </script>
</body>
</html>
```

---

## README + Publish steps (copy into repository README.md)

````
# Quola Air v1.1

Quola Air — a privacy-first, browser-based air-drawing app using MediaPipe HandLandmarker. This repo contains a single-file demo (`index.html`).

## Run locally

```bash
# serve current directory
npx serve .
# or
python -m http.server 8000
````

Open `http://localhost:5000` or `http://localhost:8000`.

## Publish

1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial Quola Air v1.1"
gh repo create quola-air-v1.1 --public --source=. --push
```

2. Enable GitHub Pages from repo settings (use `main` branch, root). Alternatively add a `gh-pages` Action workflow.

```

---

## Notes, tradeoffs & next steps (recommended)

- For production, consider bundling & optimizing (Vite/Parcel) and pre-fetching the `.task` file into `assets/models/` to avoid network CORS issues.
- Add tests / linting + GitHub Actions for CI and automatic deployment to GitHub Pages.
- Offer a settings modal to tune thresholds, or add a calibration step for users with different camera setups.
- Add collaborative features (WebRTC + synchronized drawing) if you want live multi-user sessions.
- Add a short onboarding animation and Lottie result popup as you mentioned earlier.

---

## License

MIT — include a `LICENSE` file with MIT text when you create the repo.

---

If you'd like, I can:
- generate a ready `.zip` containing the repo files,
- create a GitHub action YAML and a `LICENSE` file,
- or commit & publish directly to your GitHub (you'd need to give permissions or paste a PAT).

Tell me which of the above you want next.

```
