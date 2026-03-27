<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <title>Gym Tracker Elite</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;900&family=Bebas+Neue&family=Oswald:wght@700&family=Roboto:wght@900&family=Montserrat:wght@800&family=Playfair+Display:ital,wght@1,900&family=Syncopate:wght@700&family=Titillium+Web:wght@900&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
  
  <style>
    :root { --primary: #ff3b30; --text: #ffffff; --glass: rgba(20, 20, 22, 0.7); --font-family: 'Inter', sans-serif; }
    * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
    body { background: #000; color: var(--text); font-family: var(--font-family); height: 100vh; overflow: hidden; }

    #app-bg { position: fixed; inset: 0; z-index: -1; background-size: cover; background-position: center; filter: brightness(0.4); transition: 0.5s; }
    #app { height: 100%; display: flex; flex-direction: column; position: relative; z-index: 1; }
    
    .header { padding: 15px 20px; padding-top: env(safe-area-inset-top); display: flex; justify-content: flex-end; align-items: center; background: rgba(0,0,0,0.3); backdrop-filter: blur(15px); min-height: 60px; }
    .menu-btn { font-size: 28px; cursor: pointer; color: var(--text); padding: 5px; }

    .settings-panel { position: fixed; top: 0; right: -100%; width: 85%; max-width: 320px; height: 100%; background: rgba(10, 10, 10, 0.98); backdrop-filter: blur(25px); z-index: 500; transition: 0.4s; padding: 30px 20px; border-left: 1px solid #333; overflow-y: auto; }
    .settings-panel.active { right: 0; }

    .page { display: none; flex: 1; overflow-y: auto; padding: 20px; padding-bottom: 140px; }
    .page.active { display: block; }

    .sticky-progress { position: sticky; top: -20px; z-index: 10; background: var(--primary); color: #fff; padding: 12px; border-radius: 0 0 20px 20px; text-align: center; font-weight: 900; font-size: 14px; margin: -20px -20px 20px -20px; box-shadow: 0 10px 20px rgba(0,0,0,0.4); transition: 0.5s; }
    .sticky-progress.all-done { background: linear-gradient(45deg, #ffd700, #ffa500); color: #000; font-size: 16px; letter-spacing: 1px; box-shadow: 0 0 25px rgba(255,215,0,0.5); }

    .day-item { background: var(--glass); backdrop-filter: blur(15px); border-radius: 22px; padding: 20px; margin-bottom: 15px; border: 1px solid rgba(255,255,255,0.1); display: flex; justify-content: space-between; align-items: center; }
    .day-info { flex: 1; cursor: pointer; }
    .day-del-btn { color: #ff3b30; font-size: 18px; padding: 10px; margin-left: 10px; opacity: 0.6; }
    
    .ex-card { background: var(--glass); backdrop-filter: blur(15px); border-radius: 22px; padding: 0; margin-bottom: 15px; border: 1px solid rgba(255,255,255,0.1); overflow: hidden; transition: 0.3s; }
    .progress-txt { font-size: 14px; opacity: 0.8; font-weight: 900; color: var(--primary); margin-top: 5px; display: block; }

    .ex-card.is-done { background: rgba(76, 217, 100, 0.15); border-color: rgba(76, 217, 100, 0.5); }
    .done-tag { display: none; color: #4cd964; font-weight: 900; font-size: 12px; letter-spacing: 1px; }
    .ex-card.is-done .done-tag { display: inline-block; }

    .ex-img-box { width: 100%; height: 160px; background: #222; }
    .ex-img-box img { width: 100%; height: 100%; object-fit: cover; }
    
    .ex-info { padding: 18px; display: flex; justify-content: space-between; align-items: center; }
    .sets-row { display: flex; gap: 10px; padding: 0 18px 18px; flex-wrap: wrap; }
    .set-circle { width: 50px; height: 50px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.15); display: flex; align-items: center; justify-content: center; font-weight: 900; transition: 0.2s; }
    .set-circle.done { background: var(--primary); border-color: var(--primary); }

    .btn { border: none; border-radius: 14px; padding: 14px; font-weight: 700; color: #fff; background: rgba(255,255,255,0.1); cursor: pointer; text-transform: uppercase; }
    .btn-primary { background: var(--primary); }
    
    .timer-display { font-size: 90px; font-weight: 900; text-align: center; margin: 15px 0; color: var(--text); }
    .nav { position: fixed; bottom: 0; width: 100%; display: flex; background: rgba(0,0,0,0.85); padding-bottom: env(safe-area-inset-bottom); border-top: 1px solid #333; backdrop-filter: blur(20px); }
    .nav-btn { flex: 1; padding: 20px; text-align: center; color: #666; font-weight: 800; font-size: 11px; }
    .nav-btn.active { color: var(--primary); }

    .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.92); display: none; align-items: center; justify-content: center; z-index: 1000; padding: 20px; }
    .modal.active { display: flex; }
    .modal-box { background: #1c1c1e; width: 100%; max-width: 400px; padding: 30px; border-radius: 28px; border: 1px solid #333; }
    input, select { width: 100%; padding: 15px; margin: 10px 0; border-radius: 14px; background: #2c2c2e; border: 1px solid #444; color: #fff; }
  </style>
</head>
<body>

<div id="app-bg"></div>

<div id="app">
  <header class="header"><div class="menu-btn" onclick="toggleSettings()">⋮</div></header>

  <div class="settings-panel" id="settings-panel">
    <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:30px;">
        <h2>Settings</h2>
        <div onclick="toggleSettings()" style="font-size:24px;">✕</div>
    </div>
    <div style="margin-bottom:20px;"><label>Theme Color</label><input type="color" id="theme-pick" oninput="updateStyle('primary', this.value)"></div>
    <div style="margin-bottom:20px;"><label>Text Color</label><input type="color" id="text-pick" oninput="updateStyle('text', this.value)"></div>
    <div style="margin-bottom:20px;"><label>Font Style</label>
        <select id="font-select" onchange="updateStyle('font', this.value)">
            <option value="'Inter', sans-serif">Modern iOS</option>
            <option value="'Bebas Neue', sans-serif">Athletic</option>
            <option value="'Oswald', sans-serif">Bold Impact</option>
            <option value="'Roboto', sans-serif">Roboto</option>
            <option value="'Montserrat', sans-serif">Montserrat</option>
            <option value="'Playfair Display', serif">Playfair</option>
            <option value="'Syncopate', sans-serif">Syncopate</option>
            <option value="'Titillium Web', sans-serif">Titillium</option>
        </select>
    </div>
    <div style="margin-bottom:20px;"><label>Wallpaper</label><input type="file" id="bg-upload" accept="image/*"></div>
    
    <div style="margin-bottom:20px;">
        <label>Custom Rest Alarm</label>
        <input type="file" id="alarm-upload" accept="audio/*">
        <button class="btn" style="font-size: 10px; width:100%; margin-top: 5px;" onclick="resetAlarm()">Reset to Default Beep</button>
    </div>

    <button class="btn btn-primary" style="width:100%; margin-top:20px;" onclick="toggleSettings()">Save</button>
  </div>

  <div class="page active" id="home-page">
    <div style="display:flex; justify-content:space-between; margin-bottom:20px; align-items:center;">
      <h2 style="font-size: 26px;">GYM TRACKER</h2>
      <button class="btn btn-primary" onclick="openM('day-m')">+ Day</button>
    </div>
    <div id="day-list"></div>
  </div>

  <div class="page" id="ex-page">
    <div class="sticky-progress" id="sticky-count">0/0 EXERCISES DONE</div>
    <div style="display:flex; justify-content:space-between; margin-bottom:15px;">
        <button class="btn" onclick="goHome()">← Back</button>
        <button class="btn" style="color: #ff4444;" onclick="resetDayProgress()">Reset Sets</button>
    </div>
    <h2 id="day-title" style="margin-bottom:15px; font-size:32px;"></h2>
    <div id="ex-list"></div>
    <button class="btn btn-primary" style="width:100%;" onclick="openExM()">+ Add Exercise</button>
  </div>

  <div class="page" id="timer-page">
    <div class="timer-display" id="t-val">00:00</div>
    <div style="display:flex; gap:12px; justify-content:center; margin-bottom:25px">
        <button class="btn" onclick="adjT(-5)">-5</button>
        <button class="btn btn-primary" id="t-go" onclick="togT()" style="padding: 12px 50px;">START</button>
        <button class="btn" onclick="adjT(5)">+5</button>
    </div>
    <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap:12px;">
      <button class="btn" onclick="setT(45)">45s</button><button class="btn" onclick="setT(60)">1m</button><button class="btn" onclick="setT(180)">3m</button>
      <button class="btn" onclick="setT(240)">4m</button><button class="btn" onclick="setT(300)">5m</button>
      <button class="btn" onclick="resT()" style="color: var(--primary); font-weight: 900;">RESET</button>
    </div>
  </div>

  <nav class="nav">
    <div class="nav-btn active" id="n-home" onclick="navH()">WORKOUTS</div>
    <div class="nav-btn" id="n-timer" onclick="showP('timer-page')">REST</div>
  </nav>
</div>

<div class="modal" id="day-m">
  <div class="modal-box">
    <h3>Plan Name</h3><input type="text" id="day-in" placeholder="Day Name">
    <button class="btn btn-primary" style="width:100%; margin-top:10px;" onclick="saveD()">Create</button>
    <button class="btn" style="width:100%; margin-top:5px;" onclick="closeM('day-m')">Cancel</button>
  </div>
</div>

<div class="modal" id="ex-m">
  <div class="modal-box">
    <h3>Exercise</h3><input type="hidden" id="edit-i"><input type="text" id="ex-n-in" placeholder="Name">
    <div style="display:flex; gap:10px;"><input type="number" id="ex-w-in" placeholder="Weight"><select id="ex-u-in" style="width:100px;"><option value="kg">kg</option><option value="lbs">lbs</option></select></div>
    <div style="display:flex; gap:10px;"><input type="number" id="ex-s-in" placeholder="Sets"><input type="number" id="ex-r-in" placeholder="Reps"></div>
    <input type="file" id="ex-f-in" accept="image/*">
    <button class="btn btn-primary" style="width:100%; margin-top:15px;" onclick="saveE()">Confirm</button>
    <button class="btn" id="del-e-btn" style="width:100%; margin-top:5px; color:#ff4444; display:none;" onclick="delE()">Delete</button>
    <button class="btn" style="width:100%; margin-top:5px;" onclick="closeM('ex-m')">Cancel</button>
  </div>
</div>

<script>
  let db = JSON.parse(localStorage.getItem('gp_v12')) || [];
  let curD = null; let tSec = 0; let lastSetTime = 45; let tInt = null; let editImg = '';

  function checkDayReset() {
    const lastDate = localStorage.getItem('last_open_date');
    const today = new Date().toDateString();
    if (lastDate && lastDate !== today) {
        db.forEach(day => day.ex.forEach(e => e.done = []));
        localStorage.removeItem('has_celebrated_today');
        sync();
    }
    localStorage.setItem('last_open_date', today);
  }

  function playAlarm() {
      const customAudioData = localStorage.getItem('custom_alarm');
      if (customAudioData) {
          const audio = new Audio(customAudioData);
          audio.play().catch(e => console.error("Playback failed:", e));
      } else {
          const ctx = new (window.AudioContext || window.webkitAudioContext)();
          const osc = ctx.createOscillator();
          const gain = ctx.createGain();
          osc.connect(gain); gain.connect(ctx.destination);
          osc.frequency.setValueAtTime(880, ctx.currentTime);
          gain.gain.setValueAtTime(0.5, ctx.currentTime);
          gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.8);
          osc.start(); osc.stop(ctx.currentTime + 0.8);
      }
  }

  document.getElementById('alarm-upload').onchange = (e) => {
    const file = e.target.files[0];
    const reader = new FileReader();
    reader.onload = () => {
        localStorage.setItem('custom_alarm', reader.result);
        alert("Custom sound saved!");
    };
    if (file) reader.readAsDataURL(file);
  };

  function resetAlarm() { localStorage.removeItem('custom_alarm'); alert("Default beep restored."); }
  function toggleSettings() { document.getElementById('settings-panel').classList.toggle('active'); }
  
  function updateStyle(t, v) { 
    const prop = (t === 'font') ? 'font-family' : t;
    document.documentElement.style.setProperty('--' + prop, v); 
    localStorage.setItem('s_' + t, v); 
  }

  document.getElementById('bg-upload').onchange = (e) => {
    const reader = new FileReader();
    reader.onload = () => { document.getElementById('app-bg').style.backgroundImage = `url(${reader.result})`; localStorage.setItem('s_b', reader.result); };
    reader.readAsDataURL(e.target.files[0]);
  };

  function showP(id) {
    document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
    document.getElementById(id).classList.add('active');
    document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
    document.getElementById(id === 'timer-page' ? 'n-timer' : 'n-home').classList.add('active');
  }

  function navH() { curD !== null ? (showP('ex-page'), renderE()) : (showP('home-page'), renderD()); }
  function goHome() { curD = null; renderD(); showP('home-page'); }
  function openM(id) { document.getElementById(id).classList.add('active'); }
  function closeM(id) { document.getElementById(id).classList.remove('active'); }

  function saveD() {
    const n = document.getElementById('day-in').value;
    if(n) { db.push({name: n, ex: []}); sync(); renderD(); closeM('day-m'); document.getElementById('day-in').value=''; }
  }

  function delDayFromDash(i) {
    if(confirm("Delete this entire workout day?")) { db.splice(i, 1); sync(); renderD(); }
  }

  function renderD() {
    const l = document.getElementById('day-list'); l.innerHTML = '';
    db.forEach((d, i) => {
      let doneCount = d.ex.filter(e => e.done.length === e.s && e.s > 0).length;
      l.innerHTML += `<div class="day-item">
        <div class="day-info" onclick="openD(${i})">
            <h3 style="font-size:22px;">${d.name}</h3>
            <span class="progress-txt">${doneCount}/${d.ex.length} Exercises Done</span>
        </div>
        <div class="day-del-btn" onclick="delDayFromDash(${i})">✕</div>
      </div>`;
    });
  }

  function openD(i) { curD = i; document.getElementById('day-title').innerText = db[i].name; renderE(); showP('ex-page'); }

  function renderE() {
    const l = document.getElementById('ex-list'); l.innerHTML = '';
    let totalSets = 0; let doneSets = 0; let doneExCount = 0;
    if(!db[curD]) return;
    db[curD].ex.forEach((e, i) => {
      totalSets += e.s; doneSets += e.done.length;
      let isDone = e.done.length === e.s && e.s > 0;
      if(isDone) doneExCount++;
      let sH = ''; for(let s=0; s<e.s; s++) { sH += `<div class="set-circle ${e.done.includes(s)?'done':''}" onclick="togS(${i}, ${s})">${e.r}</div>`; }
      l.innerHTML += `<div class="ex-card ${isDone?'is-done':''}">
        ${e.img ? `<div class="ex-img-box"><img src="${e.img}"></div>` : ''}
        <div class="ex-info"><div><h3 style="font-size:22px;">${e.name} <span class="done-tag">✓</span></h3><span style="opacity:0.7;">${e.w}${e.u}</span></div><button class="btn" onclick="openExM(${i})">EDIT</button></div>
        <div class="sets-row">${sH}</div></div>`;
    });
    const prog = document.getElementById('sticky-count');
    if (totalSets > 0 && doneSets === totalSets) {
        prog.innerText = "★ WORKOUT COMPLETE ★";
        prog.classList.add('all-done');
        if (!localStorage.getItem('has_celebrated_today')) {
            confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 }, colors: ['#ffd700', '#ffffff', '#ff3b30'] });
            localStorage.setItem('has_celebrated_today', 'true');
        }
    } else {
        prog.innerText = `${doneExCount}/${db[curD].ex.length} EXERCISES DONE`;
        prog.classList.remove('all-done');
    }
  }

  function togS(ei, si) {
    const e = db[curD].ex[ei];
    if(e.done.includes(si)) { e.done = e.done.filter(x => x !== si); }
    else { e.done.push(si); }
    sync(); renderE();
  }

  function saveE() {
    const f = document.getElementById('ex-f-in').files[0];
    const ei = document.getElementById('edit-i').value;
    const fin = (img) => {
      const o = { name: document.getElementById('ex-n-in').value || "Exercise", w: document.getElementById('ex-w-in').value || "0", u: document.getElementById('ex-u-in').value, s: parseInt(document.getElementById('ex-s-in').value) || 3, r: document.getElementById('ex-r-in').value || "10", img: img || editImg, done: (ei !== "" ? db[curD].ex[ei].done : []) };
      if(ei !== "") db[curD].ex[ei] = o; else db[curD].ex.push(o);
      sync(); renderE(); closeM('ex-m');
    };
    if(f) { const r = new FileReader(); r.onload = () => fin(r.result); r.readAsDataURL(f); } else { fin(''); }
  }

  function resetDayProgress() { if(confirm("Reset sets?")) { db[curD].ex.forEach(e => e.done = []); sync(); renderE(); } }
  function openExM(idx = null) {
    const dBtn = document.getElementById('del-e-btn');
    if(idx !== null) {
      const e = db[curD].ex[idx];
      document.getElementById('edit-i').value = idx; document.getElementById('ex-n-in').value = e.name;
      document.getElementById('ex-w-in').value = e.w; document.getElementById('ex-u-in').value = e.u;
      document.getElementById('ex-s-in').value = e.s; document.getElementById('ex-r-in').value = e.r;
      editImg = e.img; dBtn.style.display = "block";
    } else { document.getElementById('edit-i').value = ""; document.getElementById('ex-n-in').value = ""; editImg = ""; dBtn.style.display = "none"; }
    openM('ex-m');
  }
  function delE() { if(confirm("Delete?")) { db[curD].ex.splice(document.getElementById('edit-i').value, 1); sync(); renderE(); closeM('ex-m'); } }
  function setT(s) { tSec = s; lastSetTime = s; updT(); }
  function adjT(s) { tSec = Math.max(0, tSec + s); updT(); }
  function resT() { clearInterval(tInt); tInt = null; tSec = lastSetTime; updT(); document.getElementById('t-go').innerText = "START"; }
  function updT() { document.getElementById('t-val').innerText = `${Math.floor(tSec/60).toString().padStart(2,'0')}:${(tSec%60).toString().padStart(2,'0')}`; }
  function togT() {
    if(tInt) { clearInterval(tInt); tInt = null; document.getElementById('t-go').innerText = "START"; }
    else { if(tSec<=0) return; tInt = setInterval(() => { 
      if(tSec>0){ tSec--; updT(); } 
      else { clearInterval(tInt); tInt=null; document.getElementById('t-go').innerText="START"; playAlarm(); } 
    }, 1000); document.getElementById('t-go').innerText="STOP"; }
  }

  function sync() { localStorage.setItem('gp_v12', JSON.stringify(db)); }
  
  checkDayReset();
  const sp = localStorage.getItem('s_primary') || '#ff3b30', st = localStorage.getItem('s_text') || '#ffffff', sf = localStorage.getItem('s_font') || "'Inter', sans-serif", sb = localStorage.getItem('s_b');
  updateStyle('primary', sp); document.getElementById('theme-pick').value = sp;
  updateStyle('text', st); document.getElementById('text-pick').value = st;
  updateStyle('font', sf); document.getElementById('font-select').value = sf;
  if(sb) document.getElementById('app-bg').style.backgroundImage = `url(${sb})`;
  renderD();
</script>
</body>
</html># Gym-tracker
