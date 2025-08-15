<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<title>HotSpots — stream timer & markers</title>
<style>
  :root{
    --bg:#0f1220;--card:#171a2a;--text:#e7e9f4;--muted:#9aa0b5;--accent:#6ee7ff;--accent2:#a78bfa;--ok:#34d399;--warn:#f59e0b;--err:#ef4444;
    --border:#262b45;--hover:#2a3155;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0; background:radial-gradient(1200px 800px at 10% 10%, #11162b, var(--bg)); color:var(--text);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; -webkit-font-smoothing:antialiased;
    padding: env(safe-area-inset-top) 12px 12px 16px;
  }
  .app{ max-width:860px; margin:0 auto; display:flex; flex-direction:column; gap:12px; }
  header{
    display:flex; align-items:center; justify-content:space-between; gap:12px; padding:10px 14px; background:var(--card); border:1px solid var(--border); border-radius:12px;
  }
  .title{ font-weight:800; font-size:18px; letter-spacing:.3px }
  .project{
    display:flex; align-items:center; gap:10px; flex-wrap:wrap; color:var(--muted); font-size:14px;
  }
  .pill{ padding:4px 10px; border:1px solid var(--border); border-radius:999px; background:#14172a; }
  .status{ font-weight:700; color:var(--accent) }
  main{
    background:var(--card); border:1px solid var(--border); border-radius:16px; padding:16px; display:flex; flex-direction:column; gap:14px;
  }
  .timer{
    display:flex; flex-direction:column; gap:6px; align-items:center; justify-content:center; text-align:center;
    padding:14px; border:1px dashed var(--border); border-radius:12px; background:#121527;
  }
  .time{ font-size:64px; line-height:1; font-weight:900; letter-spacing:.05em; user-select:none; }
  .since{ color:var(--muted); font-size:12px; }
  .buttons{ display:grid; gap:12px; grid-template-columns: repeat(2, 1fr); }
  @media (min-width: 560px){ .buttons{ grid-template-columns: repeat(3, 1fr); } }
  button{
    font-size:18px; font-weight:800; padding:18px 14px; border-radius:14px; border:1px solid var(--border);
    background:#1a1f3a; color:var(--text); cursor:pointer; min-height:64px;
    box-shadow:0 1px 0 #0004 inset, 0 1px 0 #0006; transition:transform .04s ease, background .15s ease, border-color .15s ease;
    touch-action:manipulation;
  }
  button:hover{ background:var(--hover) }
  button:active{ transform:translateY(1px) }
  .primary{ background:linear-gradient(#1c2552,#1b2a63); border-color:#31407b }
  .ok{ background:linear-gradient(#164739,#0f3a2c); border-color:#256d52 }
  .warn{ background:linear-gradient(#4d3b0a,#3b2d07); border-color:#9b6a07 }
  .danger{ background:linear-gradient(#5a1720,#410f15); border-color:#9b1c2a }
  .ghost{ background:#121527 }
  .wide{ grid-column: span 2 }
  @media (min-width: 560px){ .wide{ grid-column: span 3 } }
  .row{ display:flex; gap:10px; flex-wrap:wrap; align-items:center; justify-content:space-between }
  .mini{ font-size:14px; padding:10px 12px; min-height:40px; border-radius:10px }
  .hidden{ display:none !important }
  .panel{
    background:#121527; border:1px solid var(--border); border-radius:12px; padding:12px; max-height:260px; overflow:auto;
  }
  .list{ list-style:none; margin:0; padding:0; display:flex; flex-direction:column; gap:6px }
  .item{
    display:flex; gap:10px; align-items:center; justify-content:space-between; padding:10px 12px; border:1px solid var(--border);
    border-radius:10px; background:#0f1430; font-variant-numeric: tabular-nums;
  }
  .empty{ color:var(--muted); text-align:center; padding:12px 6px; }

  dialog{
    border:none; border-radius:16px; padding:0; max-width:560px; width:92vw; background:var(--card); color:var(--text);
    box-shadow: 0 20px 60px #0009;
  }
  dialog::backdrop{ background:#0008 }
  .sheet{ padding:16px }
  .sheet h3{ margin:4px 0 14px; font-size:18px }
  .proj-list{ display:flex; flex-direction:column; gap:10px; max-height:50vh; overflow:auto }
  .proj-card{
    padding:12px; border:1px solid var(--border); border-radius:12px; background:#121527; display:flex; justify-content:space-between; gap:8px; align-items:center;
  }
  .proj-card .name{ font-weight:800 }
  .muted{ color:var(--muted) }
  .footer{ display:flex; justify-content:flex-end; gap:8px; padding:12px; border-top:1px solid var(--border); background:#0f1430; border-radius:0 0 16px 16px }
</style>
</head>
<body>
  <div class="app">
    <header>
      <div class="title">🔥 HotSpots</div>
      <div class="project">
        <div class="pill"><span class="muted">Project:</span> <strong id="projectName">—</strong></div>
        <div class="pill"><span class="muted">Status:</span> <span class="status" id="status">idle</span></div>
      </div>
    </header>

    <main>
      <div class="timer" aria-live="polite">
        <div class="time" id="time">00:00:00</div>
        <div class="since" id="since">Ready</div>
      </div>

      <div class="buttons">
        <button id="newBtn" class="ghost">🆕 New Project</button>
        <button id="loadBtn" class="ghost">📂 Load</button>

        <button id="startPauseBtn" class="primary">▶️ Start</button>
        <button id="hotspotBtn" class="ok">📍 Hotspot</button>

        <button id="viewTimesBtn" class="ghost">📝 View Times</button>
        <button id="finishBtn" class="danger">⏹️ Finish</button>

        <button id="copyBtn" class="warn wide">📋 Copy Times</button>
      </div>

      <div id="timesPanel" class="panel hidden" role="region" aria-label="Hotspot Times">
        <ul id="timesList" class="list"></ul>
        <div id="emptyTimes" class="empty hidden">No hotspots yet. Tap <strong>📍 Hotspot</strong> while the timer runs.</div>
      </div>

      <div class="row">
        <small class="muted">Shortcuts: Space = Start/Pause • H = Hotspot • T = Toggle Times</small>
        <button id="resumeBtn" class="mini hidden">▶️ Resume</button>
      </div>
    </main>
  </div>

  <!-- Load dialog -->
  <dialog id="loadDialog">
    <div class="sheet">
      <h3>Load Project</h3>
      <div id="projList" class="proj-list"></div>
    </div>
    <div class="footer">
      <button id="closeLoad" class="mini">Close</button>
    </div>
  </dialog>

<script>
(() => {
  // ==== Storage keys ====
  const INDEX_KEY = 'hotspots:projects:index';   // array of {id,name,createdAt,lastUsedAt,status}
  const LAST_KEY  = 'hotspots:lastProjectId';
  const PROJECT_KEY = id => `hotspots:project:${id}`;

  // ==== Storage helpers ====
  const readJSON = (k, fallback) => { try { return JSON.parse(localStorage.getItem(k)) ?? fallback; } catch { return fallback; } };
  const writeJSON = (k, v) => localStorage.setItem(k, JSON.stringify(v));

  function readIndex(){ return readJSON(INDEX_KEY, []); }
  function writeIndex(list){ writeJSON(INDEX_KEY, list); }
  function upsertIndex(meta){
    const list = readIndex();
    const i = list.findIndex(p => p.id === meta.id);
    if(i >= 0) list[i] = {...list[i], ...meta};
    else list.unshift(meta);
    writeIndex(list);
  }
  function getProject(id){ return readJSON(PROJECT_KEY(id), null); }
  function saveProject(p){
    const now = Date.now();
    p.lastUpdatedAt = now;
    writeJSON(PROJECT_KEY(p.id), p);
    upsertIndex({id:p.id, name:p.name, createdAt:p.createdAt, lastUsedAt:now, status:p.status});
    localStorage.setItem(LAST_KEY, p.id);
  }
  function allProjects(){ return readIndex().sort((a,b)=> (b.lastUsedAt||0)-(a.lastUsedAt||0)); }
  function nextDefaultName(){ return `project_${readIndex().length+1}`; }

  // ==== Time utils ====
  const pad2 = n => String(n).padStart(2,'0');
  function fmtHMS(ms){
    const total = Math.floor(ms/1000);
    const h = Math.floor(total/3600);
    const m = Math.floor((total%3600)/60);
    const s = total%60;
    return `${pad2(h)}:${pad2(m)}:${pad2(s)}`;
  }

  // ==== State ====
  let current = null; // {id,name,createdAt,status,elapsedMs,startedAt,hotspotsMs[],finishedAt?}

  // ==== Elements ====
  const $ = id => document.getElementById(id);
  const timeEl = $('time'), sinceEl = $('since'), nameEl = $('projectName'), statusEl = $('status');
  const startPauseBtn = $('startPauseBtn'), hotspotBtn = $('hotspotBtn'), finishBtn = $('finishBtn');
  const viewTimesBtn = $('viewTimesBtn'), copyBtn = $('copyBtn'), timesPanel = $('timesPanel');
  const timesList = $('timesList'), emptyTimes = $('emptyTimes'), resumeBtn = $('resumeBtn');
  const newBtn = $('newBtn'), loadBtn = $('loadBtn'), loadDialog = $('loadDialog');
  const projList = $('projList'), closeLoad = $('closeLoad');

  // ==== Render ====
  function renderTimes(){
    const list = current?.hotspotsMs || [];
    timesList.innerHTML = '';
    if(list.length === 0){
      emptyTimes.classList.remove('hidden');
      return;
    }
    emptyTimes.classList.add('hidden');
    list.forEach((ms, i) => {
      const li = document.createElement('li');
      li.className = 'item';
      li.textContent = `#${i+1}  ${fmtHMS(ms)}`;
      timesList.appendChild(li);
    });
  }

  function render(){
    if(!current){
      timeEl.textContent = '00:00:00';
      sinceEl.textContent = 'Ready';
      nameEl.textContent = '—';
      statusEl.textContent = 'idle';
      hotspotBtn.disabled = true;
      finishBtn.disabled = true;
      return;
    }
    const running = current.status === 'running';
    const now = Date.now();
    const elapsed = current.elapsedMs + (running ? (now - (current.startedAt||now)) : 0);

    timeEl.textContent = fmtHMS(elapsed);
    nameEl.textContent = current.name || '—';
    statusEl.textContent = current.status;

    startPauseBtn.textContent = running ? '⏸️ Pause' : (current.status === 'finished' ? '🔁 Restart' : '▶️ Start');
    hotspotBtn.disabled = !running;
    finishBtn.disabled = (current.status === 'finished');

    sinceEl.textContent =
      current.status === 'running' ? 'Running… press 📍 to add a hotspot' :
      current.status === 'paused'  ? 'Paused' :
      current.status === 'finished'? 'Finished — view/export your hotspots below' : 'Ready';

    resumeBtn.classList.toggle('hidden', !(current && current.status==='paused'));

    renderTimes();

    // Lightweight autosave of running elapsed UI view without spamming writes
    // (we already have a timer below for periodic saves)
  }

  // Continuous UI update
  function loop(){ render(); requestAnimationFrame(loop); }

  // ==== Actions ====
  function createProjectFlow(){
    const def = nextDefaultName();
    const name = (prompt('Name your project:', def) || def).trim() || def;
    const id = `id_${Date.now()}_${Math.random().toString(36).slice(2,7)}`;
    current = {
      id, name, createdAt: Date.now(),
      status: 'paused',
      elapsedMs: 0,
      startedAt: null,
      hotspotsMs: []
    };
    timesPanel.classList.add('hidden');
    saveProject(current);
    render();
  }

  function loadProject(id){
    const p = getProject(id);
    if(!p) return;
    if(p.status === 'running'){
      // Resume as paused (spec: if app closed -> pause/autosave)
      p.elapsedMs += (Date.now() - (p.startedAt||Date.now()));
      p.startedAt = null;
      p.status = 'paused';
    }
    current = p;
    saveProject(current);
    render();
  }

  function toggleStartPause(){
    if(!current){
      createProjectFlow();
      return;
    }
    if(current.status === 'finished'){
      // Restart into fresh paused state
      current.status = 'paused';
      current.elapsedMs = 0;
      current.hotspotsMs = [];
      current.startedAt = null;
      timesPanel.classList.add('hidden');
      saveProject(current);
      render();
      return;
    }
    if(current.status !== 'running'){
      current.status = 'running';
      current.startedAt = Date.now();
    } else {
      current.elapsedMs += Date.now() - (current.startedAt || Date.now());
      current.startedAt = null;
      current.status = 'paused';
    }
    saveProject(current);
    render();
  }

  function addHotspot(){
    if(!current || current.status !== 'running') return;
    const now = Date.now();
    const elapsed = current.elapsedMs + (now - (current.startedAt||now));
    current.hotspotsMs.push(elapsed);
    hotspotBtn.animate([{transform:'scale(1)'},{transform:'scale(0.97)'},{transform:'scale(1)'}],{duration:120});
    saveProject(current);
    if(timesPanel.classList.contains('hidden')){
      sinceEl.textContent = `Marked hotspot at ${fmtHMS(elapsed)}`;
      setTimeout(()=>render(), 900);
    } else {
      renderTimes();
      timesPanel.scrollTop = timesPanel.scrollHeight;
    }
  }

  function finish(){
    if(!current) return;
    if(current.status === 'running'){
      current.elapsedMs += Date.now() - (current.startedAt||Date.now());
      current.startedAt = null;
    }
    current.status = 'finished';
    current.finishedAt = Date.now();
    saveProject(current);
    timesPanel.classList.remove('hidden');
    render();
  }

  function toggleTimes(){ timesPanel.classList.toggle('hidden'); }

  function copyTimes(){
    if(!current) return;
    const total = current.elapsedMs + (current.status==='running' ? Date.now() - (current.startedAt||Date.now()) : 0);
    const lines = current.hotspotsMs.map((ms,i)=> `${i+1}. ${fmtHMS(ms)}`);
    const text = [`Project: ${current.name}`, `Total Time: ${fmtHMS(total)}`, `Hotspots:`, ...lines].join('\n');
    (navigator.clipboard?.writeText(text) || Promise.reject()).then(()=>{
      copyBtn.textContent = '✅ Copied';
      setTimeout(()=> copyBtn.textContent = '📋 Copy Times', 1200);
    }).catch(()=>{
      alert('Copied list:\n\n' + text);
    });
  }

  function openLoad(){
    projList.innerHTML = '';
    const projects = allProjects();
    if(projects.length === 0){
      const p = document.createElement('div');
      p.className = 'muted';
      p.textContent = 'No saved projects yet.';
      projList.appendChild(p);
    } else {
      projects.forEach(meta=>{
        const card = document.createElement('div');
        card.className = 'proj-card';

        const left = document.createElement('div');
        left.innerHTML = `<div class="name">${meta.name}</div>
                          <div class="muted">Updated ${new Date(meta.lastUsedAt||meta.createdAt).toLocaleString()}</div>`;

        const right = document.createElement('div');
        const btn = document.createElement('button');
        btn.className = 'mini';
        btn.textContent = 'Load';
        btn.addEventListener('click', ()=>{ loadProject(meta.id); loadDialog.close(); });

        right.appendChild(btn);
        card.appendChild(left);
        card.appendChild(right);
        projList.appendChild(card);
      });
    }
    loadDialog.showModal();
  }

  // ==== Lifecycle & autosave ====
  function periodicSave(){
    if(current) saveProject(current);
  }
  setInterval(periodicSave, 2000);

  window.addEventListener('beforeunload', ()=>{
    if(!current) return;
    if(current.status === 'running'){
      current.elapsedMs += Date.now() - (current.startedAt||Date.now());
      current.startedAt = null;
      current.status = 'paused';
    }
    saveProject(current);
  });

  document.addEventListener('visibilitychange', ()=>{
    if(document.hidden && current && current.status === 'running'){
      current.elapsedMs += Date.now() - (current.startedAt||Date.now());
      current.startedAt = null;
      current.status = 'paused';
      saveProject(current);
    }
  });

  // ==== Events ====
  startPauseBtn.addEventListener('click', toggleStartPause);
  hotspotBtn.addEventListener('click', addHotspot);
  finishBtn.addEventListener('click', finish);
  viewTimesBtn.addEventListener('click', toggleTimes);
  copyBtn.addEventListener('click', copyTimes);

  newBtn.addEventListener('click', createProjectFlow);
  loadBtn.addEventListener('click', openLoad);
  closeLoad.addEventListener('click', ()=> loadDialog.close());

  resumeBtn.addEventListener('click', ()=>{
    if(current && current.status==='paused') toggleStartPause();
  });

  // Keyboard shortcuts (ignore when typing in inputs)
  document.addEventListener('keydown', (e)=>{
    if(e.target && (e.target.tagName==='INPUT' || e.target.tagName==='TEXTAREA' || e.target.isContentEditable)) return;
    if(e.code === 'Space'){ e.preventDefault(); toggleStartPause(); }
    else if(e.key && e.key.toLowerCase() === 'h'){ addHotspot(); }
    else if(e.key && e.key.toLowerCase() === 't'){ toggleTimes(); }
  });

  // ==== Boot ====
  function boot(){
    const lastId = localStorage.getItem(LAST_KEY);
    if(lastId){
      loadProject(lastId);
    } else {
      // Create a default project immediately so users can just press Start
      const def = nextDefaultName();
      const id = `id_${Date.now()}_${Math.random().toString(36).slice(2,7)}`;
      current = { id, name: def, createdAt: Date.now(), status:'paused', elapsedMs:0, startedAt:null, hotspotsMs: [] };
      saveProject(current);
    }
    loop(); // start render loop
  }
  boot();
})();
</script>
</body>
</html>
