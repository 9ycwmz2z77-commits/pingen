<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Pin Studio — d.vd.psychology</title>

<!-- Tailwind CSS & html2canvas -->
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

<!-- Google Fonts CDN -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bodoni+Moda:ital,opsz,wght@0,6..96,400..700;1,6..96,400..700&family=Cinzel:wght@400;600;700&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400;1,600&family=Inter:wght@300;400;500;600&family=Lato:ital,wght@0,300;0,400;1,300&family=Montserrat:ital,wght@0,400;0,600;1,400&family=Playfair+Display:ital,wght@0,400;0,600;1,400&display=swap" rel="stylesheet">

<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          void: '#0D0D0D',
          panel: '#171717',
          rim: '#2A2A28',
          ink: '#E5E5E3',
          inkdim: '#8F8C86',
          gold: '#C5A059',
          golddim: '#A99260',
        }
      }
    }
  }
</script>

<style>
  html, body { background:#0A0A0A; font-family: 'Inter', sans-serif; }
  ::-webkit-scrollbar { width: 8px; height: 8px; }
  ::-webkit-scrollbar-track { background: #121212; }
  ::-webkit-scrollbar-thumb { background: #333; border-radius: 4px; }
  ::-webkit-scrollbar-thumb:hover { background: #C5A059; }

  .field-label {
    font-size: 11px;
    letter-spacing: .06em;
    color: #8F8C86;
    margin-bottom: 6px;
    display: block;
    text-transform: uppercase;
  }
  .field-input {
    width: 100%;
    background: #121212;
    border: 1px solid #2A2A28;
    color: #E5E5E3;
    padding: 10px 12px;
    border-radius: 6px;
    font-size: 13.5px;
    line-height: 1.4;
    outline: none;
    transition: border-color .15s ease;
  }
  .field-input:focus { border-color: #A99260; background:#141414; }

  select.field-input {
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6'%3E%3Cpath d='M0 0l5 6 5-6z' fill='%23A99260'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 12px center;
    padding-right: 30px;
  }

  .side-btn { font-size: 12.5px; }
  .side-btn:disabled { opacity:.55; cursor:not-allowed; }

  /* --- PIN CANVAS (True 1000x1500 render target) --- */
  #pinCanvas {
    width: 1000px;
    height: 1500px;
    position: relative;
    color: #E5E5E3;
    overflow: hidden;
    font-size: 10px; /* base unit */
    box-sizing: border-box;
    background: #0D0D0D;
  }
  #pinCanvas * { box-sizing: border-box; }

  .pin-noise::after {
    content: "";
    position: absolute; inset: 0;
    opacity: 0;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='200' height='200'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
    mix-blend-mode: overlay;
    pointer-events: none;
    transition: opacity .2s ease;
    z-index: 5;
  }
  .pin-noise.noise-on::after { opacity: .18; }
  .pin-vignette.shadow-on {
    box-shadow: inset 0 0 220px rgba(0,0,0,.75), inset 0 0 60px rgba(0,0,0,.55);
  }

  #viewport {
    position: relative;
    width: 100%;
    max-width: 440px;
    aspect-ratio: 2 / 3;
    margin: 0 auto;
    overflow: hidden;
    border-radius: 4px;
  }

  textarea.field-input { resize: vertical; min-height: 85px; }
  .tmpl-hint { font-size: 10.5px; color: #6B685F; margin-top: 4px; }

  .switch {
    width:34px; height:18px; border-radius:999px; background:#2A2A28;
    position:relative; cursor:pointer; transition:background .15s; flex-shrink:0;
  }
  .switch.on { background:#A99260; }
  .switch::after {
    content:""; position:absolute; top:2px; left:2px; width:14px; height:14px;
    border-radius:50%; background:#E5E5E3; transition:left .15s;
  }
  .switch.on::after { left:18px; }

  /* --- DESIGN THEMES --- */
  #pinCanvas.design-1  { --accent:#C5A059; --accent-dim:#A99260; --ink:#F2EEE4; --ink-dim:#8F8C86; --rule:#232320; background:#0D0D0D; }
  #pinCanvas.design-2  { --accent:#C98A83; --accent-dim:#A66E68; --ink:#EFE7E4; --ink-dim:#9C8D89; --rule:#2b2222; background:linear-gradient(160deg,#1a1414,#0d0a0a); }
  #pinCanvas.design-3  { --accent:#8FA8C9; --accent-dim:#6C87A6; --ink:#E7ECF2; --ink-dim:#8891A0; --rule:#1c222c; background:#0b0e14; }
  #pinCanvas.design-4  { --accent:#8FB08A; --accent-dim:#6E9169; --ink:#E7EDE6; --ink-dim:#8B9B89; --rule:#1c2620; background:#0c1210; }
  #pinCanvas.design-5  { --accent:#CB8459; --accent-dim:#A66838; --ink:#F0E4DA; --ink-dim:#A08B7C; --rule:#2c1f16; background:#150e0a; }
  #pinCanvas.design-6  { --accent:#B98AC9; --accent-dim:#9569A6; --ink:#EEE3F0; --ink-dim:#9C8AA0; --rule:#241a29; background:#120a14; }
  #pinCanvas.design-7  { --accent:#FFFFFF; --accent-dim:#C9C9C9; --ink:#F5F5F5; --ink-dim:#8A8A8A; --rule:#2A2A2A; background:#000000; }
  #pinCanvas.design-7::before { content:""; position:absolute; inset:36px; border:1px solid var(--accent); pointer-events:none; z-index:4; }
  #pinCanvas.design-8  { --accent:#8A6D3B; --accent-dim:#6B5530; --ink:#221E17; --ink-dim:#6B6154; --rule:#D8CFBE; background:#EDE7DC; }
  #pinCanvas.design-9  { --accent:#B8874B; --accent-dim:#93683A; --ink:#EDE3D2; --ink-dim:#9C8F76; --rule:#241d12; background:radial-gradient(ellipse at center,#17130c,#0f0c08); }
  #pinCanvas.design-9::before { content:""; position:absolute; inset:30px; border:1px solid var(--accent); pointer-events:none; z-index:4; }

  /* --- BLOCKS & SELECTION --- */
  .pin-block { position: absolute; cursor: grab; transform-origin: top left; user-select: none; z-index: 3; }
  .pin-block:active { cursor: grabbing; }
  .pin-block.selected { outline: 1.5px dashed var(--accent); outline-offset: 8px; }
  .resize-handle {
    position: absolute; right: -12px; bottom: -12px; width: 22px; height: 22px;
    border-radius: 50%; background: var(--accent); border: 2px solid #0D0D0D;
    cursor: nwse-resize; display: none; z-index: 6;
  }
  .pin-block.selected .resize-handle { display: block; }
  .remove-block-btn {
    position: absolute; top: -14px; right: -14px; width: 26px; height: 26px;
    border-radius: 50%; background: #1a1a1a; border: 1px solid var(--accent);
    color: var(--ink); font-size: 14px; line-height: 22px; text-align: center;
    display: none; cursor: pointer; z-index: 6;
  }
  .pin-block.selected .remove-block-btn { display: block; }
  .pin-block-img { width: 100%; height: 100%; object-fit: cover; display: block; }

  #pinCanvas.exporting .pin-block { outline: none !important; }
  #pinCanvas.exporting .resize-handle, #pinCanvas.exporting .remove-block-btn { display: none !important; }

  /* Modal Styling */
  #modalOverlay { display: none; }
  #modalOverlay.active { display: flex; }
</style>
</head>

<body class="bg-[#0A0A0A] text-ink min-h-screen">

<div class="flex flex-col lg:flex-row min-h-screen">

  <!-- ============ LEFT CONTROL PANEL ============ -->
  <aside class="lg:w-[420px] lg:h-screen lg:overflow-y-auto bg-panel border-b lg:border-b-0 lg:border-r border-rim">
    <div class="px-6 pt-6 pb-4 border-b border-rim">
      <p class="text-gold text-[11px] tracking-[.18em] font-semibold">PIN STUDIO</p>
      <h1 class="italic text-2xl mt-0.5 text-ink font-serif">d.vd.psychology</h1>
      <p class="text-[11px] text-inkdim mt-0.5">Unveiling The Shadows — Pinterest Pin Generator</p>
    </div>

    <div class="px-6 py-5 space-y-5">

      <!-- Preset & Design selectors -->
      <div class="grid grid-cols-2 gap-3">
        <div>
          <label class="field-label">Template</label>
          <select id="templateSelect" class="field-input"></select>
        </div>
        <div>
          <label class="field-label">Theme</label>
          <select id="designSelect" class="field-input"></select>
        </div>
      </div>

      <!-- FONT SELECTION SYSTEM -->
      <div class="p-3 bg-[#121212] border border-rim rounded-md space-y-3">
        <p class="text-[10px] text-gold tracking-wider uppercase font-semibold">Typography Settings</p>
        <div>
          <label class="field-label">Hook & Titles Font</label>
          <select id="fontTitleSelect" class="field-input">
            <option value="'Cormorant Garamond', serif">Cormorant Garamond (Classic Serif)</option>
            <option value="'Cinzel', serif">Cinzel (Roman Editorial)</option>
            <option value="'Playfair Display', serif">Playfair Display (High Contrast)</option>
            <option value="'Bodoni Moda', serif">Bodoni FLF / Moda (Luxury)</option>
            <option value="'Montserrat', sans-serif">Montserrat (Modern Clean)</option>
          </select>
        </div>
        <div>
          <label class="field-label">Body & Subtitles Font</label>
          <select id="fontBodySelect" class="field-input">
            <option value="'Inter', sans-serif">Inter (Minimal Sans)</option>
            <option value="'Montserrat', sans-serif">Montserrat (Geometric)</option>
            <option value="'Lato', sans-serif">Lato (Neutral)</option>
            <option value="'Cormorant Garamond', serif">Cormorant Garamond (Serif)</option>
          </select>
        </div>
      </div>

      <!-- Recommended Canva Template -->
      <div class="border border-rim bg-[#121212] rounded-md px-3.5 py-2.5">
        <p class="text-[10px] text-inkdim tracking-wider mb-0.5">RECOMMENDED CANVA BASE</p>
        <p id="canvaName" class="italic text-gold text-[14px] leading-snug font-serif"></p>
      </div>

      <!-- Inputs -->
      <div>
        <label class="field-label">Hook / Title</label>
        <input id="inputHook" class="field-input" type="text" placeholder="Main clickbait title" />
      </div>
      <div>
        <label class="field-label">Header / Attribution</label>
        <input id="inputHeader" class="field-input" type="text" placeholder="Small category kicker" />
      </div>
      <div>
        <label class="field-label">Body Text</label>
        <textarea id="inputBody" class="field-input" rows="4" placeholder="Body copy"></textarea>
        <p id="bodyHint" class="tmpl-hint"></p>
      </div>
      <div>
        <label class="field-label">Footer Tag</label>
        <input id="inputFooter" class="field-input" type="text" value="@d.vd.psychology  |  Unveiling The Shadows" />
      </div>

      <!-- Image Upload -->
      <div>
        <label class="field-label">Pin Background / Photo</label>
        <div class="flex gap-2">
          <label class="side-btn flex-1 text-center border border-rim hover:border-gold text-ink py-2 rounded-md cursor-pointer transition-colors">
            + Upload Image
            <input id="photoInput" type="file" accept="image/*" class="hidden" />
          </label>
          <button id="removePhotoBtn" class="hidden side-btn px-3 border border-rim hover:border-gold text-ink rounded-md transition-colors">✕</button>
        </div>
      </div>

      <!-- Scaling (UP TO 250%) & Effects -->
      <div class="space-y-3 pt-2">
        <div>
          <div class="flex justify-between items-center mb-1">
            <label class="field-label !mb-0">Text Scale (Max 250%)</label>
            <span id="fsVal" class="text-gold text-xs font-mono">100%</span>
          </div>
          <input id="fontScale" type="range" min="0.5" max="2.5" step="0.05" value="1" class="w-full accent-[#C5A059]" />
        </div>
        <div class="flex items-center justify-between">
          <span class="text-xs text-inkdim">Grain Overlay</span>
          <div id="noiseToggle" class="switch"></div>
        </div>
        <div class="flex items-center justify-between">
          <span class="text-xs text-inkdim">Architectural Vignette</span>
          <div id="shadowToggle" class="switch on"></div>
        </div>
      </div>

      <!-- Controls & Actions -->
      <div class="space-y-2 pt-2 pb-8">
        <button id="resetLayoutBtn" class="side-btn w-full border border-rim hover:border-gold text-ink py-2 rounded-md transition-colors">
          Reset Block Positions
        </button>
        <button id="previewModalBtn" class="side-btn w-full bg-gold hover:bg-golddim text-[#0D0D0D] font-medium py-2.5 rounded-md transition-colors">
          🔍 Full Screen Preview & Save
        </button>
        <button id="downloadBtn" class="side-btn w-full border border-gold text-gold hover:bg-gold/10 py-2.5 rounded-md transition-colors">
          Quick Download PNG
        </button>
        <button id="copyBtn" class="side-btn w-full border border-rim hover:border-gold text-ink py-2.5 rounded-md transition-colors">
          Copy Pinterest Description
        </button>
        <p id="copyStatus" class="text-[11px] text-gold min-h-4 text-center"></p>
      </div>

    </div>
  </aside>

  <!-- ============ RIGHT PREVIEW PANEL ============ -->
  <main class="flex-1 flex flex-col items-center justify-center py-8 px-4 bg-[#0A0A0A] relative">
    <div class="relative z-10 w-full flex flex-col items-center">
      <p class="text-[11px] tracking-widest text-inkdim mb-3">CANVAS PREVIEW · 1000 × 1500 RATIO</p>

      <div id="viewport" class="ring-1 ring-rim shadow-2xl">
        <div id="pinCanvas" class="pin-noise pin-vignette shadow-on design-1"></div>
      </div>

      <p class="text-[11px] text-inkdim mt-3 text-center max-w-xs">Click block to select. Drag to move, or drag golden handles to scale individually.</p>
    </div>
  </main>

</div>

<!-- ============ FULLSCREEN PREVIEW MODAL ============ -->
<div id="modalOverlay" class="fixed inset-0 bg-black/90 backdrop-blur-md z-50 flex-col items-center justify-center p-4">
  <div class="relative max-h-[85vh] aspect-[2/3] ring-1 ring-gold/40 shadow-2xl overflow-hidden rounded">
    <img id="modalImg" class="w-full h-full object-contain" src="" alt="Full Screen Pin Preview" />
  </div>
  <div class="flex gap-4 mt-5">
    <button id="modalSaveBtn" class="bg-gold hover:bg-golddim text-[#0D0D0D] font-medium px-6 py-2.5 rounded-md text-sm transition-colors">
      Save High-Res PNG
    </button>
    <button id="modalCloseBtn" class="border border-rim hover:border-gold text-ink px-6 py-2.5 rounded-md text-sm transition-colors">
      Close Preview
    </button>
  </div>
</div>

<script>
/* ============================================================
   HELPERS & PRESETS DATA
   ============================================================ */
function esc(s){ return (s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function lines(s){ return (s||'').split('\n').map(x=>x.trim()).filter(Boolean); }
function pipeRows(s){ return lines(s).map(l=>l.split('|').map(x=>x.trim())); }
function colonRows(s){ return lines(s).map(l=>l.split('::').map(x=>x.trim())); }
function cloneLayout(l){ return JSON.parse(JSON.stringify(l)); }

function footerContent(d){
  return `<div style="display:flex;justify-content:space-between;align-items:center;border-top:1px solid var(--rule);padding-top:1em;">
    <p class="dyn-body" style="font-size:1.1em;letter-spacing:.03em;color:var(--ink-dim);">${esc(d.footer)}</p>
    <span style="width:.7em;height:.7em;border-radius:50%;border:1px solid var(--accent);display:inline-block;"></span>
  </div>`;
}

const DEFAULT_LAYOUT = {
  header: { x: 100, y: 100, width: 800 },
  hook:   { x: 100, y: 180, width: 800 },
  body:   { x: 100, y: 460, width: 800 },
  footer: { x: 100, y: 1380, width: 800 },
};
const CENTER_LAYOUT = {
  header: { x: 120, y: 140, width: 760 },
  hook:   { x: 120, y: 240, width: 760 },
  body:   { x: 120, y: 560, width: 760 },
  footer: { x: 120, y: 1380, width: 760 },
};

const templates = [
  {
    id: 1,
    name: 'Diagnostic Blindspot',
    canva: 'Dark Elegant Comparison Pinterest Pin',
    hook: 'Why female psychopathy is misdiagnosed as BPD.',
    header: 'DIAGNOSTIC BLINDSPOT',
    body: 'Male aggression is physical | Female aggression is relational\nVisible red flags | Weaponized fragility & victimhood\nDirect confrontation | Calculated isolation & smear campaigns\nImmediate intervention | Plausible deniability',
    hint: 'Format per line: Overt Male Claim | Covert Female Claim',
    layout: DEFAULT_LAYOUT,
    render(d){
      const rows = pipeRows(d.body).slice(0,5);
      const rowsHtml = rows.map(r => `
        <div style="display:flex;align-items:flex-start;padding:1.4em 0;border-top:1px solid var(--rule);">
          <div style="flex:1;padding-right:1.4em;text-align:right;">
            <p class="dyn-title" style="font-size:2em;line-height:1.25;color:var(--ink-dim);">${esc(r[0]||'')}</p>
          </div>
          <div style="width:1px;align-self:stretch;background:var(--accent);opacity:.5;"></div>
          <div style="flex:1;padding-left:1.4em;">
            <p class="dyn-title" style="font-size:2em;line-height:1.25;color:var(--ink);">${esc(r[1]||'')}</p>
          </div>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.5em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: rowsHtml
      };
    }
  },
  {
    id: 2,
    name: 'Traumatic Cognitive Dissonance',
    canva: 'Minimal Clinical Definition Pinterest Pin',
    hook: 'Your body senses the threat. Your mind rationalizes their innocence.',
    header: 'TRAUMATIC COGNITIVE DISSONANCE (TCD)',
    body: 'The most exhausting part of covert abuse isn\'t the lie — it\'s the friction between your gut feeling and their "nurturing" mask. TCD occurs when your brain is forced to reconcile contradictory data: verbal cues of safety vs. visceral alarms signaling threat.',
    hint: 'Continuous paragraph for definition.',
    layout: CENTER_LAYOUT,
    render(d){
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);text-align:center;">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.2em;line-height:1.15;text-align:center;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: `<p class="dyn-title" style="font-size:2.2em;line-height:1.5;text-align:center;color:var(--ink-dim);">${esc(d.body)}</p>`
      };
    }
  },
  {
    id: 3,
    name: 'Triangulation Mechanics',
    canva: 'Editorial Numbered List Pinterest Pin',
    hook: 'Covert manipulators never fight one-on-one. They build an army first.',
    header: 'TRIANGULATION MECHANICS',
    body: 'Strategic Smear :: Seeding subtle doubts to third parties while maintaining a pristine public image.\nNarrative Control :: Chaperoning meetings so you can never speak directly or privately with enablers.\nDouble Abuse :: Framing your natural emotional reaction as the actual abuse.',
    hint: 'Format per line: Title :: Description',
    layout: DEFAULT_LAYOUT,
    render(d){
      const rows = colonRows(d.body).slice(0,3);
      const numerals = ['I','II','III'];
      const rowsHtml = rows.map((r,i)=>`
        <div style="display:flex;gap:1.5em;padding:1.6em 0;border-top:1px solid var(--rule);">
          <p class="dyn-title" style="font-style:italic;font-size:3.2em;color:var(--accent);line-height:1;">${numerals[i]}</p>
          <div>
            <p class="dyn-title" style="font-size:2.1em;color:var(--ink);line-height:1.2;">${esc(r[0]||'')}</p>
            <p class="dyn-body" style="font-size:1.25em;color:var(--ink-dim);line-height:1.5;margin-top:.4em;">${esc(r[1]||'')}</p>
          </div>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.3em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: rowsHtml
      };
    }
  },
  {
    id: 4,
    name: 'Dark Feminine Trap',
    canva: 'Split Screen Comparison Pinterest Pin',
    hook: 'Pop-psychology taught you to play games. Cold-heartedness isn\'t empowerment.',
    header: 'AUTHENTIC POWER VS TOXIC SEDUCTION',
    body: 'Forced callousness & withholding | Ironclad boundaries & shadow integration\nPlaying game playbooks | Strategic clarity & self-respect\nSeeking male validation | Reclaiming epistemic confidence',
    hint: 'Format per line: Myth / Pop Trap | Authentic Power',
    layout: DEFAULT_LAYOUT,
    render(d){
      const rows = pipeRows(d.body).slice(0,4);
      const rowsHtml = rows.map(r=>`
        <div style="border-top:1px solid var(--rule);padding:1.4em 0;">
          <p class="dyn-body" style="font-size:1.3em;color:var(--ink-dim);text-decoration:line-through;">${esc(r[0]||'')}</p>
          <p class="dyn-title" style="font-style:italic;font-size:2.2em;color:var(--accent);margin-top:.3em;">${esc(r[1]||'')}</p>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--ink-dim);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.2em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: rowsHtml
      };
    }
  },
  {
    id: 5,
    name: 'Somatic Collapse',
    canva: 'Aesthetic Psychology Infographic Pin',
    hook: 'Your brain tries to forgive them. Your nervous system forces a shutdown.',
    header: 'SIGNS OF SOMATIC COLLAPSE',
    body: 'Unexplained chronic fatigue & adrenal exhaustion\nDorsal vagal freeze (sleeping or extreme brain fog when stressed)\nHyper-vigilance to soft voice shifts or footsteps\nMemory gaps & timeline confusion after subtle arguments\nGut issues & tension with no medical cause',
    hint: 'One symptom per line (max 5)',
    layout: DEFAULT_LAYOUT,
    render(d){
      const items = lines(d.body).slice(0,5);
      const itemsHtml = items.map(t=>`
        <div style="display:flex;align-items:baseline;gap:1em;padding:1.2em 0;border-top:1px solid var(--rule);">
          <span style="width:.6em;height:1px;background:var(--accent);display:inline-block;"></span>
          <p class="dyn-title" style="font-size:2em;color:var(--ink);line-height:1.3;">${esc(t)}</p>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.2em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: itemsHtml
      };
    }
  },
  {
    id: 6,
    name: 'Boundary Scripts',
    canva: 'Quotes Carousel / Minimal Text Pin',
    hook: 'Stop defending your reality. Use these 3 boundary scripts instead.',
    header: 'GREY ROCK PROTOCOLS',
    body: '"I don\'t think I want to do that right now."\n"Your opinion is noted."\n"I\'m not comfortable with that. Let\'s talk about it later."',
    hint: '3 lines, each becomes a stylized script block',
    layout: DEFAULT_LAYOUT,
    render(d){
      const items = lines(d.body).slice(0,3);
      const bubbles = items.map((t,i)=>`
        <div style="background:rgba(0,0,0,.35);border:1px solid var(--accent);border-radius:1em;padding:1.2em 1.5em;margin-top:${i===0?0:'1.2em'};">
          <p class="dyn-title" style="font-style:italic;font-size:2em;line-height:1.35;color:var(--ink);">${esc(t)}</p>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4em;line-height:1.12;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: bubbles
      };
    }
  },
  {
    id: 7,
    name: 'R.A.D.A.R. Framework',
    canva: 'Clean Minimal Checklist Pin',
    hook: 'If you feel like you\'re losing your mind, run the R.A.D.A.R. check.',
    header: 'R.A.D.A.R. ABUSE CHECKLIST',
    body: 'Reputation attacks :: Spreading subtle doubts or damaging character to mutuals.\nAlliance manipulation :: Triangulating friends/family to build artificial rivalries.\nDenial tactics :: Gaslighting and outright rejecting observable reality.\nAmbient hostility :: Silent treatment, cold shoulders, and atmospheric pressure.\nRelationship weaponization :: Using vulnerabilities to force compliance.',
    hint: '5 lines mapping to R-A-D-A-R',
    layout: DEFAULT_LAYOUT,
    render(d){
      const rows = colonRows(d.body).slice(0,5);
      const letters = ['R','A','D','A','R'];
      const rowsHtml = rows.map((r,i)=>`
        <div style="display:flex;align-items:center;gap:1.2em;padding:1em 0;border-top:1px solid var(--rule);">
          <p class="dyn-body" style="font-size:2.4em;font-weight:600;color:var(--accent);width:1em;">${letters[i]}</p>
          <div>
            <p class="dyn-title" style="font-size:1.8em;color:var(--ink);line-height:1.2;">${esc(r[0]||'')}</p>
            <p class="dyn-body" style="font-size:1.15em;color:var(--ink-dim);margin-top:.2em;">${esc(r[1]||'')}</p>
          </div>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--ink-dim);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.1em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: rowsHtml
      };
    }
  },
  {
    id: 8,
    name: 'Book Quote & Epistemic Confidence',
    canva: 'Dark Elegance Book Quote Pin',
    hook: 'The hardest phase of healing isn\'t leaving. It\'s trusting your own memory again.',
    header: 'VICTORIA D. — UNVEILING THE SHADOWS',
    body: 'Memory becomes non-negotiable when securely recorded. Stop arguing facts with someone committed to misunderstanding you.',
    hint: 'Write quote in Hook, subtext in Body',
    layout: CENTER_LAYOUT,
    render(d){
      return {
        header: `<p class="dyn-body" style="font-size:1.2em;letter-spacing:.15em;color:var(--ink-dim);text-align:center;">${esc(d.header)}</p>`,
        hook: `<div style="text-align:center;"><p class="dyn-title" style="font-style:italic;font-size:5em;line-height:1;color:var(--accent);">&ldquo;</p><h1 class="dyn-title" style="font-style:italic;font-size:3.6em;line-height:1.25;color:var(--ink);margin-top:-.5em;">${esc(d.hook)}</h1></div>`,
        body: `<p class="dyn-body" style="font-size:1.8em;line-height:1.5;text-align:center;color:var(--accent-dim);margin-top:1em;">${esc(d.body)}</p>`
      };
    }
  },
  {
    id: 9,
    name: 'Gaslighting Breakout',
    canva: 'Numbered Step-by-Step Guide Pin',
    hook: 'How to break a gaslighting loop in 3 psychological steps.',
    header: 'GASLIGHTING BREAKOUT PROTOCOL',
    body: 'Stop interrogating your sanity :: Shift from "am I crazy?" to objective pattern recognition.\nDocument incidents objectively :: Externalize facts immediately to anchor your epistemic confidence.\nObserve behavior, stop arguing facts :: Assess what their distortion reveals rather than trying to win.',
    hint: '3 steps: Title :: Description',
    layout: DEFAULT_LAYOUT,
    render(d){
      const rows = colonRows(d.body).slice(0,3);
      const rowsHtml = rows.map((r,i)=>`
        <div style="padding:1.8em 0;${i>0?'border-top:1px solid var(--rule);':''}">
          <p class="dyn-title" style="font-style:italic;font-size:3.5em;color:var(--accent);line-height:1;">0${i+1}</p>
          <p class="dyn-title" style="font-size:2.1em;color:var(--ink);line-height:1.2;margin-top:.3em;">${esc(r[0]||'')}</p>
          <p class="dyn-body" style="font-size:1.25em;color:var(--ink-dim);line-height:1.5;margin-top:.4em;">${esc(r[1]||'')}</p>
        </div>`).join('');
      return {
        header: `<p class="dyn-body" style="font-size:1.3em;letter-spacing:.2em;color:var(--accent);">${esc(d.header)}</p>`,
        hook: `<h1 class="dyn-title" style="font-style:italic;font-size:4.1em;line-height:1.1;color:var(--ink);">${esc(d.hook)}</h1>`,
        body: rowsHtml
      };
    }
  }
];

const DESIGNS = [
  { id: 1, name: 'Obsidian Gold' },
  { id: 2, name: 'Ash Rose' },
  { id: 3, name: 'Midnight Steel' },
  { id: 4, name: 'Verdant Noir' },
  { id: 5, name: 'Terracotta Dusk' },
  { id: 6, name: 'Velvet Plum' },
  { id: 7, name: 'Monochrome Frame' },
  { id: 8, name: 'Ivory Editorial' },
  { id: 9, name: 'Bronze Vintage' },
];

/* ============================================================
   STATE & DOM REFS
   ============================================================ */
const els = {
  select: document.getElementById('templateSelect'),
  designSelect: document.getElementById('designSelect'),
  fontTitleSelect: document.getElementById('fontTitleSelect'),
  fontBodySelect: document.getElementById('fontBodySelect'),
  canvaName: document.getElementById('canvaName'),
  hook: document.getElementById('inputHook'),
  header: document.getElementById('inputHeader'),
  body: document.getElementById('inputBody'),
  footer: document.getElementById('inputFooter'),
  hint: document.getElementById('bodyHint'),
  canvas: document.getElementById('pinCanvas'),
  fontScale: document.getElementById('fontScale'),
  fsVal: document.getElementById('fsVal'),
  noiseToggle: document.getElementById('noiseToggle'),
  shadowToggle: document.getElementById('shadowToggle'),
  downloadBtn: document.getElementById('downloadBtn'),
  previewModalBtn: document.getElementById('previewModalBtn'),
  copyBtn: document.getElementById('copyBtn'),
  copyStatus: document.getElementById('copyStatus'),
  photoInput: document.getElementById('photoInput'),
  removePhotoBtn: document.getElementById('removePhotoBtn'),
  resetLayoutBtn: document.getElementById('resetLayoutBtn'),
  modalOverlay: document.getElementById('modalOverlay'),
  modalImg: document.getElementById('modalImg'),
  modalSaveBtn: document.getElementById('modalSaveBtn'),
  modalCloseBtn: document.getElementById('modalCloseBtn')
};

let currentTemplateId = 1;
let layoutState = {};
let selectedBlockId = null;
let currentViewScale = 1;
let dragCtx = null;
let resizeCtx = null;
let currentBlobUrl = null;

// Populate Dropdowns
templates.forEach(t => {
  const opt = document.createElement('option');
  opt.value = t.id;
  opt.textContent = `${String(t.id).padStart(2,'0')} — ${t.name}`;
  els.select.appendChild(opt);
});
DESIGNS.forEach(dsn => {
  const opt = document.createElement('option');
  opt.value = dsn.id;
  opt.textContent = `${String(dsn.id).padStart(2,'0')} — ${dsn.name}`;
  els.designSelect.appendChild(opt);
});

function getTemplate(id){ return templates.find(t => t.id === Number(id)); }

function loadTemplateDefaults(id){
  const t = getTemplate(id);
  els.hook.value = t.hook;
  els.header.value = t.header;
  els.body.value = t.body;
  els.hint.textContent = t.hint;
  els.canvaName.textContent = t.canva;
}

function currentData(){
  return {
    hook: els.hook.value,
    header: els.header.value,
    body: els.body.value,
    footer: els.footer.value,
  };
}

function resetLayout(templateId, keepImage){
  const t = getTemplate(templateId);
  const base = cloneLayout(t.layout);
  layoutState = {
    header: { ...base.header, scale: 1 },
    hook:   { ...base.hook,   scale: 1 },
    body:   { ...base.body,   scale: 1 },
    footer: { ...base.footer, scale: 1 },
    image: keepImage ? layoutState.image : null,
  };
  selectedBlockId = null;
}

function buildBlock(role, state, innerHtml, isImage){
  const div = document.createElement('div');
  div.className = 'pin-block' + (selectedBlockId === role ? ' selected' : '');
  div.dataset.role = role;
  div.style.left = state.x + 'px';
  div.style.top = state.y + 'px';
  div.style.width = state.width + 'px';
  if (isImage) div.style.height = state.height + 'px';
  div.style.transform = `scale(${state.scale || 1})`;
  div.style.zIndex = isImage ? '1' : '3';
  div.innerHTML = `<div class="block-inner">${innerHtml}</div>
    <div class="resize-handle"></div>
    ${isImage ? '<div class="remove-block-btn" title="Remove Photo">×</div>' : ''}`;
  return div;
}

function applyDynamicFonts(){
  const titleFont = els.fontTitleSelect.value;
  const bodyFont = els.fontBodySelect.value;
  els.canvas.querySelectorAll('.dyn-title').forEach(el => el.style.fontFamily = titleFont);
  els.canvas.querySelectorAll('.dyn-body').forEach(el => el.style.fontFamily = bodyFont);
}

function renderPin(){
  const t = getTemplate(currentTemplateId);
  const parts = t.render(currentData());
  els.canvas.innerHTML = '';

  if (layoutState.image) {
    els.canvas.appendChild(buildBlock('image', layoutState.image, `<img class="pin-block-img" src="${layoutState.image.src}" draggable="false" />`, true));
  }
  if (parts.header) els.canvas.appendChild(buildBlock('header', layoutState.header, parts.header));
  els.canvas.appendChild(buildBlock('hook', layoutState.hook, parts.hook));
  if (parts.body) els.canvas.appendChild(buildBlock('body', layoutState.body, parts.body));
  els.canvas.appendChild(buildBlock('footer', layoutState.footer, footerContent(currentData())));

  els.removePhotoBtn.classList.toggle('hidden', !layoutState.image);
  applyDynamicFonts();
}

/* ============================================================
   EVENTS & LISTENERS
   ============================================================ */
els.select.addEventListener('change', e => {
  currentTemplateId = Number(e.target.value);
  loadTemplateDefaults(currentTemplateId);
  resetLayout(currentTemplateId, true);
  renderPin();
});

els.designSelect.addEventListener('change', e => {
  els.canvas.className = els.canvas.className.replace(/design-\d+/,'').trim();
  els.canvas.classList.add('design-' + e.target.value);
});

[els.fontTitleSelect, els.fontBodySelect].forEach(sel => {
  sel.addEventListener('change', applyDynamicFonts);
});

[els.hook, els.header, els.body, els.footer].forEach(el => {
  el.addEventListener('input', renderPin);
});

els.resetLayoutBtn.addEventListener('click', () => {
  resetLayout(currentTemplateId, true);
  renderPin();
});

// Photo Upload
els.photoInput.addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(ev){
    layoutState.image = layoutState.image || { x: 200, y: 800, width: 600, height: 400, scale: 1 };
    layoutState.image.src = ev.target.result;
    renderPin();
  };
  reader.readAsDataURL(file);
  e.target.value = '';
});
els.removePhotoBtn.addEventListener('click', () => {
  layoutState.image = null;
  selectedBlockId = null;
  renderPin();
});

// Interactive Dragging & Resizing
els.canvas.addEventListener('pointerdown', onCanvasPointerDown);

function onCanvasPointerDown(e){
  const removeBtn = e.target.closest('.remove-block-btn');
  if (removeBtn) {
    e.stopPropagation();
    layoutState.image = null;
    selectedBlockId = null;
    renderPin();
    return;
  }
  const block = e.target.closest('.pin-block');
  if (!block) {
    selectedBlockId = null;
    els.canvas.querySelectorAll('.pin-block').forEach(b => b.classList.remove('selected'));
    return;
  }
  const role = block.dataset.role;
  selectedBlockId = role;
  els.canvas.querySelectorAll('.pin-block').forEach(b => b.classList.toggle('selected', b === block));

  const handle = e.target.closest('.resize-handle');
  if (handle) {
    startResize(e, role, block);
  } else {
    startDrag(e, role, block);
  }
}

function startDrag(e, role, block){
  e.preventDefault();
  dragCtx = { role, block, startX: e.clientX, startY: e.clientY, origX: layoutState[role].x, origY: layoutState[role].y };
  window.addEventListener('pointermove', onDragMove);
  window.addEventListener('pointerup', onDragEnd);
}
function onDragMove(e){
  if (!dragCtx) return;
  const scale = currentViewScale || 1;
  const nx = dragCtx.origX + (e.clientX - dragCtx.startX) / scale;
  const ny = dragCtx.origY + (e.clientY - dragCtx.startY) / scale;
  layoutState[dragCtx.role].x = nx;
  layoutState[dragCtx.role].y = ny;
  dragCtx.block.style.left = nx + 'px';
  dragCtx.block.style.top = ny + 'px';
}
function onDragEnd(){
  dragCtx = null;
  window.removeEventListener('pointermove', onDragMove);
  window.removeEventListener('pointerup', onDragEnd);
}

function startResize(e, role, block){
  e.preventDefault(); e.stopPropagation();
  resizeCtx = { role, block, startX: e.clientX, startScale: layoutState[role].scale || 1 };
  window.addEventListener('pointermove', onResizeMove);
  window.addEventListener('pointerup', onResizeEnd);
}
function onResizeMove(e){
  if (!resizeCtx) return;
  const scale = currentViewScale || 1;
  let newScale = resizeCtx.startScale + ((e.clientX - resizeCtx.startX) / scale) / 200;
  newScale = Math.max(0.3, Math.min(4.0, newScale));
  layoutState[resizeCtx.role].scale = newScale;
  resizeCtx.block.style.transform = `scale(${newScale})`;
}
function onResizeEnd(){
  resizeCtx = null;
  window.removeEventListener('pointermove', onResizeMove);
  window.removeEventListener('pointerup', onResizeEnd);
}

// Extended Text Font Scaling (Up to 250%)
els.fontScale.addEventListener('input', () => {
  const v = parseFloat(els.fontScale.value);
  els.canvas.style.fontSize = (10 * v) + 'px';
  els.fsVal.textContent = Math.round(v * 100) + '%';
});

// Effects Switches
function bindSwitch(el, targetClassEl, className, startOn){
  let on = startOn;
  const sync = () => { el.classList.toggle('on', on); targetClassEl.classList.toggle(className, on); };
  sync();
  el.addEventListener('click', () => { on = !on; sync(); });
}
bindSwitch(els.noiseToggle, els.canvas, 'noise-on', false);
bindSwitch(els.shadowToggle, els.canvas, 'shadow-on', true);

// Scale Viewport for Preview
function scaleViewport(){
  const viewport = document.getElementById('viewport');
  currentViewScale = viewport.getBoundingClientRect().width / 1000;
  els.canvas.style.transformOrigin = 'top left';
  els.canvas.style.transform = `scale(${currentViewScale})`;
}
window.addEventListener('resize', scaleViewport);

/* ============================================================
   CANVAS RENDERING & EXPORT SYSTEM
   ============================================================ */
async function generatePinCanvasBlob() {
  const originalTransform = els.canvas.style.transform;
  selectedBlockId = null;
  els.canvas.querySelectorAll('.pin-block').forEach(b => b.classList.remove('selected'));
  els.canvas.classList.add('exporting');
  els.canvas.style.transform = 'none';

  try {
    if (document.fonts && document.fonts.ready) { await document.fonts.ready; }
    await new Promise(r => setTimeout(r, 120));

    const renderedCanvas = await html2canvas(els.canvas, {
      width: 1000,
      height: 1500,
      scale: 2,
      backgroundColor: null,
      useCORS: true,
      allowTaint: true,
      imageTimeout: 15000,
    });

    return new Promise((resolve) => {
      renderedCanvas.toBlob(blob => resolve(blob), 'image/png');
    });
  } finally {
    els.canvas.classList.remove('exporting');
    els.canvas.style.transform = originalTransform;
  }
}

function triggerDownload(blob) {
  const t = getTemplate(currentTemplateId);
  const filename = `dvdpsychology-pin-${String(t.id).padStart(2,'0')}-${t.name.toLowerCase().replace(/[^a-z0-9]+/g,'-')}.png`;
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = filename;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

// Quick Download PNG
els.downloadBtn.addEventListener('click', async () => {
  els.downloadBtn.textContent = 'Rendering...';
  els.downloadBtn.disabled = true;
  try {
    const blob = await generatePinCanvasBlob();
    if (blob) triggerDownload(blob);
    els.copyStatus.textContent = 'PNG Exported Successfully.';
    setTimeout(() => els.copyStatus.textContent = '', 3000);
  } catch(err) {
    els.copyStatus.textContent = 'Export failed: ' + err.message;
  } finally {
    els.downloadBtn.textContent = 'Quick Download PNG';
    els.downloadBtn.disabled = false;
  }
});

// Full Screen Modal Preview
els.previewModalBtn.addEventListener('click', async () => {
  els.previewModalBtn.textContent = 'Generating Preview...';
  els.previewModalBtn.disabled = true;
  try {
    const blob = await generatePinCanvasBlob();
    if (blob) {
      if (currentBlobUrl) URL.revokeObjectURL(currentBlobUrl);
      currentBlobUrl = URL.createObjectURL(blob);
      els.modalImg.src = currentBlobUrl;
      els.modalOverlay.classList.add('active');
    }
  } catch(err) {
    alert('Could not render preview: ' + err.message);
  } finally {
    els.previewModalBtn.textContent = '🔍 Full Screen Preview & Save';
    els.previewModalBtn.disabled = false;
  }
});

els.modalSaveBtn.addEventListener('click', async () => {
  if (els.modalImg.src) {
    const response = await fetch(els.modalImg.src);
    const blob = await response.blob();
    triggerDownload(blob);
  }
});

els.modalCloseBtn.addEventListener('click', () => {
  els.modalOverlay.classList.remove('active');
});

// Copy Pinterest Description
els.copyBtn.addEventListener('click', () => {
  const t = getTemplate(currentTemplateId);
  const d = currentData();
  const hashtags = '#darkpsychology #femalepsychology #covertmanipulation #traumarecovery #shadowwork #emotionalabuse #gaslighting #unveilingtheshadows';

  const description = `${d.hook}\n\n${d.header ? d.header + '\n\n' : ''}A deeper dive into ${t.name.toLowerCase()} — from the "Unveiling The Shadows" framework by Victoria D.\n\nSave this pin for reference and reclaiming your epistemic confidence.\n\n${d.footer}\n\n${hashtags}`;

  navigator.clipboard.writeText(description).then(() => {
    els.copyStatus.textContent = 'Description copied to clipboard.';
    setTimeout(() => els.copyStatus.textContent = '', 2500);
  });
});

/* ============================================================
   INITIALIZATION
   ============================================================ */
els.select.value = 1;
els.designSelect.value = 1;
loadTemplateDefaults(1);
resetLayout(1, false);
renderPin();
requestAnimationFrame(scaleViewport);
window.addEventListener('load', scaleViewport);
setTimeout(scaleViewport, 300);
</script>

</body>
</html>
