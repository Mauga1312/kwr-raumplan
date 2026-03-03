<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>KWR · Raumverfügbarkeit</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<link href="https://fonts.googleapis.com/css2?family=Sora:wght@300;400;500;600&family=Fraunces:wght@400;600&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #f2f4f7;
  --surface: #ffffff;
  --border: #e2e6ed;
  --ink: #1c2333;
  --ink2: #4a5568;
  --free-fill: #e8f5e9;
  --free-stroke: #66bb6a;
  --free-text: #2e7d32;
  --busy-fill: #fdecea;
  --busy-stroke: #ef5350;
  --busy-text: #b71c1c;
  --nonroom-fill: #eceff1;
  --nonroom-stroke: #b0bec5;
  --nonroom-text: #607d8b;
  --accent: #3d5a99;
  --green: #43a047;
  --green-dark: #2e7d32;
  --shadow-sm: 0 1px 4px rgba(0,0,0,0.07);
  --shadow-md: 0 4px 18px rgba(0,0,0,0.10);
  --shadow-lg: 0 12px 40px rgba(0,0,0,0.15);
  --r: 9px;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: 'Sora', sans-serif; background: var(--bg); color: var(--ink); min-height: 100vh; }

header {
  background: var(--ink); height: 56px; padding: 0 28px;
  display: flex; align-items: center; justify-content: space-between;
  position: sticky; top: 0; z-index: 50;
  box-shadow: 0 2px 8px rgba(0,0,0,0.22);
}
.logo { font-family:'Fraunces',serif; color:#fff; font-size:1.2rem; display:flex; align-items:center; gap:10px; }
.logo-pill { background:var(--green); color:#fff; font-family:'Sora',sans-serif; font-size:0.62rem; font-weight:600; padding:2px 8px; border-radius:20px; letter-spacing:0.07em; text-transform:uppercase; }
.hdr-right { display:flex; align-items:center; gap:14px; }
.hdr-date { color:rgba(255,255,255,0.45); font-size:0.76rem; }
.absent-btn {
  background:var(--green); color:#fff; border:none;
  padding:7px 16px; border-radius:6px;
  font-family:'Sora',sans-serif; font-size:0.8rem; font-weight:600;
  cursor:pointer; transition:all 0.2s;
}
.absent-btn:hover { background:var(--green-dark); }
.absent-btn.undo { background:#e53935; }
.absent-btn.undo:hover { background:#c62828; }

.layout { display:grid; grid-template-columns:300px 1fr; min-height:calc(100vh - 56px); }

.sidebar { background:var(--surface); border-right:1px solid var(--border); display:flex; flex-direction:column; overflow-y:auto; }
.sb-sec { padding:18px; border-bottom:1px solid var(--border); }
.sb-lbl { font-size:0.67rem; font-weight:600; letter-spacing:0.1em; text-transform:uppercase; color:var(--ink2); margin-bottom:11px; }

.stats { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
.stat { background:var(--bg); border-radius:8px; padding:12px 10px; text-align:center; }
.stat-n { font-family:'Fraunces',serif; font-size:2rem; font-weight:600; line-height:1; }
.stat-l { font-size:0.67rem; color:var(--ink2); margin-top:3px; }
.stat.free .stat-n { color:var(--free-text); }
.stat.busy .stat-n { color:var(--busy-text); }

.free-rooms-list { display:flex; flex-direction:column; gap:5px; }
.free-room-item {
  display:flex; align-items:center; gap:9px;
  padding:8px 10px; border-radius:7px;
  background:var(--free-fill); border:1px solid var(--free-stroke);
  font-size:0.82rem; font-weight:500; color:var(--free-text);
  cursor:pointer; transition:filter 0.15s;
}
.free-room-item:hover { filter:brightness(0.95); }
.free-dot { width:8px; height:8px; border-radius:50%; background:var(--free-stroke); flex-shrink:0; }
.no-entry { font-size:0.78rem; color:var(--ink2); font-style:italic; padding:4px 0; }

.absent-list { display:flex; flex-direction:column; gap:5px; }
.absent-item { display:flex; align-items:center; gap:9px; padding:8px 10px; border-radius:7px; background:var(--bg); border:1px solid var(--border); font-size:0.82rem; }
.av { width:28px; height:28px; border-radius:50%; font-size:0.68rem; font-weight:600; display:flex; align-items:center; justify-content:center; color:#fff; flex-shrink:0; }
.absent-info { flex:1; min-width:0; }
.absent-name { font-weight:500; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.absent-room { font-size:0.7rem; color:var(--ink2); }

.main { padding:22px; overflow-y:auto; }
.floor-tabs { display:flex; gap:6px; margin-bottom:18px; }
.ftab { padding:7px 18px; border-radius:6px; border:1px solid var(--border); background:var(--surface); font-family:'Sora',sans-serif; font-size:0.8rem; font-weight:500; cursor:pointer; transition:all 0.15s; color:var(--ink2); }
.ftab.active { background:var(--ink); color:#fff; border-color:var(--ink); }
.ftab:hover:not(.active) { background:var(--bg); }

.floor-panel { display:none; }
.floor-panel.active { display:block; }
.floor-card { background:var(--surface); border-radius:var(--r); border:1px solid var(--border); padding:22px; box-shadow:var(--shadow-sm); }
.floor-ttl { font-family:'Fraunces',serif; font-size:1.05rem; margin-bottom:4px; }
.floor-sub { font-size:0.73rem; color:var(--ink2); margin-bottom:18px; }
.plan-wrap { background:#f8f9fb; border-radius:8px; border:1px solid var(--border); overflow:hidden; }
.plan-wrap svg { width:100%; height:auto; display:block; }

.room-g { cursor:pointer; }
.room-g:hover rect { filter:brightness(0.93); }
text.rlabel { font-family:'Sora',sans-serif; font-size:10px; font-weight:600; pointer-events:none; text-anchor:middle; dominant-baseline:middle; }
text.rname  { font-family:'Sora',sans-serif; font-size:7.5px; pointer-events:none; text-anchor:middle; dominant-baseline:middle; opacity:0.9; }

.legend { display:flex; gap:14px; flex-wrap:wrap; margin-top:14px; }
.leg { display:flex; align-items:center; gap:6px; font-size:0.73rem; color:var(--ink2); }
.leg-sq { width:12px; height:12px; border-radius:3px; border:1px solid; }

.overlay { display:none; position:fixed; inset:0; background:rgba(28,35,51,0.48); z-index:200; align-items:center; justify-content:center; backdrop-filter:blur(4px); }
.overlay.open { display:flex; }
.modal { background:var(--surface); border-radius:12px; width:420px; max-width:94vw; box-shadow:var(--shadow-lg); overflow:hidden; animation:popIn 0.22s cubic-bezier(0.34,1.56,0.64,1); }
@keyframes popIn { from{transform:scale(0.92) translateY(14px);opacity:0} to{transform:none;opacity:1} }
.mhdr { padding:20px 24px 16px; border-bottom:1px solid var(--border); }
.mhdr h2 { font-family:'Fraunces',serif; font-size:1.1rem; }
.mhdr p { font-size:0.78rem; color:var(--ink2); margin-top:3px; }
.mbody { padding:20px 24px; }
.fg { margin-bottom:14px; }
.fg label { display:block; font-size:0.7rem; font-weight:600; letter-spacing:0.08em; text-transform:uppercase; color:var(--ink2); margin-bottom:6px; }
.fg input, .fg select { width:100%; padding:9px 12px; border:1px solid var(--border); border-radius:7px; font-family:'Sora',sans-serif; font-size:0.87rem; color:var(--ink); background:var(--bg); outline:none; transition:border-color 0.2s; }
.fg input:focus, .fg select:focus { border-color:var(--accent); background:#fff; }
.mfoot { padding:0 24px 20px; display:flex; gap:10px; justify-content:flex-end; }
.btn { padding:8px 18px; border-radius:7px; font-family:'Sora',sans-serif; font-size:0.82rem; font-weight:600; cursor:pointer; border:none; transition:all 0.18s; }
.btn-primary { background:var(--ink); color:#fff; }
.btn-primary:hover { background:#2d3a55; }
.btn-green { background:var(--green); color:#fff; }
.btn-green:hover { background:var(--green-dark); }
.btn-ghost { background:transparent; color:var(--ink2); border:1px solid var(--border); }
.btn-ghost:hover { background:var(--bg); }
.btn-red { background:#e53935; color:#fff; }
.btn-red:hover { background:#c62828; }

.room-detail { padding:20px 24px; }
.room-detail-ttl { font-family:'Fraunces',serif; font-size:1.05rem; margin-bottom:6px; }
.status-pill { display:inline-block; font-size:0.75rem; font-weight:600; padding:3px 11px; border-radius:20px; margin-bottom:14px; }
.pill-free { background:var(--free-fill); color:var(--free-text); }
.pill-busy { background:var(--busy-fill); color:var(--busy-text); }

.toast { position:fixed; bottom:22px; left:50%; transform:translateX(-50%) translateY(70px); background:var(--ink); color:#fff; padding:10px 22px; border-radius:30px; font-size:0.82rem; font-weight:500; opacity:0; transition:all 0.28s cubic-bezier(0.34,1.56,0.64,1); z-index:999; white-space:nowrap; box-shadow:var(--shadow-md); pointer-events:none; }
.toast.show { transform:translateX(-50%) translateY(0); opacity:1; }

@media(max-width:820px){ .layout{grid-template-columns:1fr} .sidebar{border-right:none;border-bottom:1px solid var(--border)} }
</style>
</head>
<body>

<header>
  <div class="logo">KWR <span class="logo-pill">Raumplan</span></div>
  <div class="hdr-right">
    <div style="display:flex;align-items:center;gap:10px">
      <button onclick="changeViewDate(-1)" style="background:rgba(255,255,255,0.1);border:1px solid rgba(255,255,255,0.2);color:#fff;width:28px;height:28px;border-radius:50%;cursor:pointer;font-size:1rem;display:flex;align-items:center;justify-content:center" title="Vorheriger Tag">‹</button>
      <span class="hdr-date" id="hdrDate"></span>
      <button onclick="changeViewDate(1)" style="background:rgba(255,255,255,0.1);border:1px solid rgba(255,255,255,0.2);color:#fff;width:28px;height:28px;border-radius:50%;cursor:pointer;font-size:1rem;display:flex;align-items:center;justify-content:center" title="Nächster Tag">›</button>
      <button onclick="goToday()" style="background:rgba(255,255,255,0.1);border:1px solid rgba(255,255,255,0.25);color:#fff;padding:4px 11px;border-radius:5px;cursor:pointer;font-size:0.72rem;font-family:'Sora',sans-serif;letter-spacing:0.04em">HEUTE</button>
    </div>
    <button class="absent-btn" id="mainBtn">Ich bin heute weg</button>
  </div>
</header>

<div class="layout">
  <aside class="sidebar">
    <div class="sb-sec">
      <div class="sb-lbl">Heute</div>
      <div class="stats">
        <div class="stat free"><div class="stat-n" id="statFree">0</div><div class="stat-l">Zimmer frei</div></div>
        <div class="stat busy"><div class="stat-n" id="statBusy">0</div><div class="stat-l">Zimmer besetzt</div></div>
      </div>
    </div>
    <div class="sb-sec">
      <div class="sb-lbl">Freie Zimmer</div>
      <div class="free-rooms-list" id="freeRoomsList"></div>
    </div>
    <div class="sb-sec" style="flex:1">
      <div class="sb-lbl">Heute abwesend</div>
      <div class="absent-list" id="absentList"></div>
    </div>
  </aside>

  <main class="main">
    <div class="floor-tabs">
      <button class="ftab active" onclick="switchFloor('og2',this)">OG 2</button>
      <button class="ftab" onclick="switchFloor('og3',this)">OG 3</button>
    </div>

    <div class="floor-panel active" id="panel-og2">
      <div class="floor-card">
        <div class="floor-ttl">Obergeschoss 2 – Zimmer 1–18</div>
        <div class="floor-sub">🟢 Grün = frei &nbsp;|&nbsp; 🔴 Rot = besetzt &nbsp;|&nbsp; Klicke auf ein Zimmer für Details</div>
        <div class="plan-wrap"><svg viewBox="0 0 1200 900" id="svg-og2"></svg></div>
        <div class="legend">
          <div class="leg"><div class="leg-sq" style="background:var(--free-fill);border-color:var(--free-stroke)"></div>Frei</div>
          <div class="leg"><div class="leg-sq" style="background:var(--busy-fill);border-color:var(--busy-stroke)"></div>Besetzt</div>
          <div class="leg"><div class="leg-sq" style="background:var(--nonroom-fill);border-color:var(--nonroom-stroke)"></div>Nebenraum</div>
        </div>
      </div>
    </div>

    <div class="floor-panel" id="panel-og3">
      <div class="floor-card">
        <div class="floor-ttl">Obergeschoss 3 – Zimmer 1–54</div>
        <div class="floor-sub">🟢 Grün = frei &nbsp;|&nbsp; 🔴 Rot = besetzt &nbsp;|&nbsp; Klicke auf ein Zimmer für Details</div>
        <div class="plan-wrap"><svg viewBox="0 0 1400 900" id="svg-og3"></svg></div>
        <div class="legend">
          <div class="leg"><div class="leg-sq" style="background:var(--free-fill);border-color:var(--free-stroke)"></div>Frei</div>
          <div class="leg"><div class="leg-sq" style="background:var(--busy-fill);border-color:var(--busy-stroke)"></div>Besetzt</div>
          <div class="leg"><div class="leg-sq" style="background:var(--nonroom-fill);border-color:var(--nonroom-stroke)"></div>Nebenraum</div>
        </div>
      </div>
    </div>
  </main>
</div>

<!-- ABSENT MODAL -->
<div class="overlay" id="absentOverlay" onclick="if(event.target===this)closeAbsent()">
  <div class="modal">
    <div class="mhdr">
      <h2>Abwesenheit eintragen</h2>
      <p>Dein Zimmer wird als <strong style="color:var(--free-text)">frei (grün)</strong> angezeigt</p>
    </div>
    <div class="mbody">
      <div class="fg">
        <label>Dein Name</label>
        <input id="inName" type="text" placeholder="Mag. Vorname Nachname" list="staffList" autocomplete="off">
        <datalist id="staffList"></datalist>
      </div>
      <div class="fg">
        <label>Dein Zimmer</label>
        <select id="inRoom"></select>
      </div>
      <div class="fg">
        <label>Zeitraum</label>
        <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px">
          <div>
            <input id="inDateFrom" type="date" style="margin:0">
            <div style="font-size:0.68rem;color:var(--ink2);margin-top:3px">Von</div>
          </div>
          <div>
            <input id="inDateTo" type="date" style="margin:0">
            <div style="font-size:0.68rem;color:var(--ink2);margin-top:3px">Bis (inkl.)</div>
          </div>
        </div>
        <div style="font-size:0.72rem;color:var(--ink2);margin-top:6px;font-style:italic">
          💡 Für einen einzelnen Tag beide Felder gleich setzen
        </div>
      </div>
      <div class="fg">
        <label>Grund / Notiz (optional)</label>
        <input id="inNote" type="text" placeholder="z.B. Homeoffice, Urlaub, Gericht …">
      </div>
    </div>
    <div class="mfoot">
      <button class="btn btn-ghost" onclick="closeAbsent()">Abbrechen</button>
      <button class="btn btn-green" onclick="saveAbsent()">Eintragen</button>
    </div>
  </div>
</div>

<!-- ROOM DETAIL MODAL -->
<div class="overlay" id="roomOverlay" onclick="if(event.target===this)closeRoomModal()">
  <div class="modal">
    <div class="room-detail" id="roomDetailContent"></div>
    <div class="mfoot" id="roomDetailFooter"></div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
// ── ROOMS ─────────────────────────────────────────────────────────────────────
const OG2 = [
  // Bottom row (3-8)
  {id:'og2-3',  x:900,y:650,w:90,h:85, label:'3',  office:true},
  {id:'og2-4',  x:810,y:650,w:90,h:85, label:'4',  office:true},
  {id:'og2-5',  x:720,y:650,w:90,h:85, label:'5',  office:true},
  {id:'og2-6',  x:630,y:650,w:90,h:85, label:'6',  office:true},
  {id:'og2-7',  x:540,y:650,w:90,h:85, label:'7',  office:true},
  {id:'og2-8',  x:450,y:650,w:90,h:85, label:'8',  office:true},
  
  // Middle section (9-14)
  {id:'og2-9',  x:700,y:450,w:90,h:90, label:'9',  office:true},
  {id:'og2-10', x:590,y:450,w:110,h:90, label:'10', office:true},
  {id:'og2-11', x:850,y:550,w:70,h:70, label:'11', office:true},
  {id:'og2-12', x:780,y:550,w:70,h:70, label:'12', office:true},
  {id:'og2-13', x:710,y:550,w:70,h:70, label:'13', office:true},
  {id:'og2-14', x:640,y:550,w:70,h:70, label:'14', office:true},
  
  // Left side (15-18)
  {id:'og2-15', x:450,y:340,w:120,h:100, label:'15', office:true},
  {id:'og2-16', x:450,y:230,w:120,h:100, label:'16', office:true},
  {id:'og2-17', x:570,y:200,w:100,h:90,  label:'17', office:true},
  {id:'og2-18', x:670,y:180,w:100,h:110, label:'18', office:true},
  
  // Right side (1-2)
  {id:'og2-1',  x:900,y:370,w:90,h:110, label:'1',  office:true},
  {id:'og2-2',  x:900,y:480,w:90,h:110, label:'2',  office:true},
  
  // Support rooms
  {id:'og2-treppe1', x:450,y:140,w:60,h:60, label:'Treppe', office:false},
  {id:'og2-treppe2', x:780,y:300,w:70,h:70, label:'Treppe', office:false},
];

const OG3 = [
  // Top area - BZ rooms and offices 1-5
  {id:'og3-bz1',     x:920,y:50,  w:80,h:70,  label:'BZ 1',  office:false},
  {id:'og3-bz2',     x:1000,y:50, w:80,h:70,  label:'BZ 2',  office:false},
  {id:'og3-bz3',     x:1080,y:50, w:80,h:70,  label:'BZ 3',  office:false},
  {id:'og3-4',       x:1160,y:50, w:70,h:70,  label:'4',     office:true},
  {id:'og3-5',       x:1230,y:50, w:130,h:100,label:'5 (großes BZ)', office:true},
  
  // Right column (6-10)
  {id:'og3-6',       x:1290,y:150, w:70,h:75, label:'6',     office:true},
  {id:'og3-7',       x:1290,y:225, w:70,h:75, label:'7',     office:true},
  {id:'og3-8',       x:1290,y:300, w:70,h:75, label:'8',     office:true},
  {id:'og3-9',       x:1290,y:375, w:70,h:75, label:'9',     office:true},
  {id:'og3-10',      x:1290,y:450, w:70,h:75, label:'10',    office:true},
  
  // Middle section - top cluster (19-25)
  {id:'og3-21',      x:1060,y:120, w:75,h:70, label:'21',    office:true},
  {id:'og3-22',      x:1135,y:120, w:75,h:70, label:'22',    office:true},
  {id:'og3-drucker', x:1210,y:120, w:40,h:70, label:'Drucker', office:false},
  {id:'og3-20',      x:1060,y:190, w:75,h:75, label:'20',    office:true},
  {id:'og3-23',      x:1135,y:190, w:55,h:65, label:'23',    office:true},
  {id:'og3-24',      x:1190,y:190, w:55,h:65, label:'24',    office:true},
  {id:'og3-25',      x:1245,y:190, w:45,h:65, label:'25',    office:true},
  {id:'og3-19',      x:1060,y:265, w:75,h:90, label:'19',    office:true},
  
  // Middle right cluster (14-18)
  {id:'og3-14',      x:990,y:355,  w:70,h:75, label:'14',    office:true},
  {id:'og3-15',      x:1060,y:355, w:70,h:75, label:'15',    office:true},
  {id:'og3-16',      x:1130,y:355, w:70,h:75, label:'16',    office:true},
  {id:'og3-17',      x:1200,y:355, w:70,h:75, label:'17',    office:true},
  {id:'og3-18',      x:1135,y:255, w:75,h:70, label:'18',    office:true},
  
  // Bottom right cluster (11-13 + WC)
  {id:'og3-13',      x:1060,y:430, w:70,h:80, label:'13',    office:true},
  {id:'og3-12',      x:1130,y:510, w:80,h:75, label:'12',    office:true},
  {id:'og3-11',      x:1210,y:510, w:80,h:75, label:'11',    office:true},
  {id:'og3-wc',      x:1130,y:430, w:70,h:80, label:'WC',    office:false},
  
  // Server + Küche
  {id:'og3-server',  x:990,y:190, w:70,h:70, label:'Server', office:false},
  {id:'og3-kueche',  x:990,y:260, w:70,h:70, label:'Küche',  office:false},
  
  // Left top area (26-27)
  {id:'og3-26',      x:800,y:420, w:120,h:85, label:'26',    office:true},
  {id:'og3-27',      x:800,y:505, w:120,h:85, label:'27',    office:true},
  
  // Left middle cluster (28-31, 48-54)
  {id:'og3-treppe',  x:660,y:420, w:90,h:100, label:'Treppe', office:false},
  {id:'og3-28',      x:750,y:520, w:90,h:70,  label:'28',    office:true},
  {id:'og3-29',      x:660,y:520, w:90,h:70,  label:'29',    office:true},
  {id:'og3-30',      x:570,y:520, w:90,h:70,  label:'30',    office:true},
  {id:'og3-31',      x:480,y:520, w:90,h:70,  label:'31',    office:true},
  
  {id:'og3-51',      x:480,y:590, w:55,h:75,  label:'51',    office:true},
  {id:'og3-50',      x:535,y:590, w:75,h:75,  label:'50',    office:true},
  {id:'og3-49',      x:610,y:590, w:80,h:75,  label:'49',    office:true},
  {id:'og3-48',      x:690,y:665, w:80,h:70,  label:'48',    office:true},
  {id:'og3-52',      x:770,y:590, w:65,h:75,  label:'52',    office:true},
  {id:'og3-53',      x:835,y:590, w:60,h:75,  label:'53',    office:true},
  {id:'og3-54',      x:895,y:590, w:50,h:75,  label:'54',    office:true},
  
  // Bottom row (32-47)
  {id:'og3-47',      x:480,y:665, w:70,h:70,  label:'47',    office:true},
  {id:'og3-46',      x:550,y:665, w:70,h:70,  label:'46',    office:true},
  {id:'og3-45',      x:620,y:665, w:70,h:70,  label:'45',    office:true},
  {id:'og3-44',      x:770,y:665, w:70,h:70,  label:'44',    office:true},
  {id:'og3-43',      x:840,y:665, w:70,h:70,  label:'43',    office:true},
  {id:'og3-42',      x:910,y:665, w:70,h:70,  label:'42',    office:true},
  {id:'og3-41',      x:980,y:665, w:70,h:70,  label:'41',    office:true},
  {id:'og3-40',      x:1050,y:665, w:70,h:70, label:'40',    office:true},
  {id:'og3-39',      x:1120,y:665, w:70,h:70, label:'39',    office:true},
  {id:'og3-38',      x:1190,y:665, w:70,h:70, label:'38',    office:true},
  {id:'og3-37',      x:1260,y:665, w:70,h:70, label:'37',    office:true},
  {id:'og3-36',      x:910,y:735,  w:70,h:70, label:'36',    office:true},
  {id:'og3-35',      x:980,y:735,  w:70,h:70, label:'35',    office:true},
  {id:'og3-34',      x:1050,y:735, w:70,h:70, label:'34',    office:true},
  {id:'og3-33',      x:1120,y:735, w:70,h:70, label:'33',    office:true},
  {id:'og3-32',      x:1190,y:735, w:70,h:70, label:'32',    office:true},
];

const ALL_OFFICES = [...OG2.filter(r=>r.office), ...OG3.filter(r=>r.office)];

// ── SUPABASE CONFIGURATION ────────────────────────────────────────────────────
const SUPABASE_URL = 'https://zxcgmcyqsxkjhctbifdy.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inp4Y2dtY3lxc3hramhjdGJpZmR5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDk2NzI0NTIsImV4cCI6MjA2NTI0ODQ1Mn0.ey3hbGc1OTJJUzI1NlsInR5cCI6IkpXVCJ9eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inp4Y2dtY3lxc3hramhjdGJpZmR5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDk2NzI0NTIsImV4cCI6MjA2NTI0ODQ1Mn0';

const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

// ── STATE ─────────────────────────────────────────────────────────────────────
async function loadState() {
  try {
    showToast('📡 Lade Daten...');
    const { data, error } = await supabase
      .from('absences')
      .select('*');
    
    if (error) throw error;
    
    // Convert from Supabase format to app format
    return data.map(row => ({
      id: row.id,
      name: row.name,
      roomId: row.room_id,
      dateFrom: row.date_from,
      dateTo: row.date_to,
      note: row.note || '',
      occupiedBy: row.occupied_by || null
    }));
  } catch (err) {
    console.error('Supabase load error:', err);
    showToast('⚠️ Fehler beim Laden');
    return [];
  }
}

async function saveToSupabase(item) {
  try {
    const { data, error } = await supabase
      .from('absences')
      .insert([{
        name: item.name,
        room_id: item.roomId,
        date_from: item.dateFrom,
        date_to: item.dateTo,
        note: item.note || '',
        occupied_by: item.occupiedBy || null
      }])
      .select();
    
    if (error) throw error;
    return data[0];
  } catch (err) {
    console.error('Supabase save error:', err);
    showToast('⚠️ Fehler beim Speichern');
    return null;
  }
}

async function deleteFromSupabase(id) {
  try {
    const { error } = await supabase
      .from('absences')
      .delete()
      .eq('id', id);
    
    if (error) throw error;
    return true;
  } catch (err) {
    console.error('Supabase delete error:', err);
    showToast('⚠️ Fehler beim Löschen');
    return false;
  }
}

let state = []; // Will be loaded from Supabase
let currentRoom = null;
let currentViewDate = new Date();
currentViewDate.setHours(0,0,0,0);

function getTodayStr() {
  return new Date().toISOString().split('T')[0];
}

const STAFF = [
  'Daniela Altieri','Mag. Angelina Aporta','Mag. Petra Artner',
  'Vanessa Aschauer','Daniela Barton','Dr. Thomas Frad',
  'Mag. Anna Gruber','Dr. Klaus Huber','Mag. Julia Kern',
  'Dr. Stefan Lang','Mag. Maria Müller','Dr. Felix Nagl',
  'Mag. Sandra Ortner','Dr. Peter Rainer','Mag. Lisa Stein',
  'Dr. Josef Urban','Mag. Eva Vogel','Dr. Helmut Wolf',
];

// ── SVG ───────────────────────────────────────────────────────────────────────
function isFree(id, dateStr) { 
  return state.some(a => 
    a.roomId === id && 
    dateStr >= a.dateFrom && 
    dateStr <= a.dateTo
  ); 
}

function renderSVG(svgId, rooms) {
  const svg = document.getElementById(svgId);
  // Use local date string to avoid timezone issues
  const y = currentViewDate.getFullYear();
  const m = String(currentViewDate.getMonth() + 1).padStart(2, '0');
  const d = String(currentViewDate.getDate()).padStart(2, '0');
  const viewDateStr = `${y}-${m}-${d}`;
  
  console.log('renderSVG - viewDateStr:', viewDateStr, 'state:', state);
  
  svg.innerHTML = '';
  rooms.forEach(r => {
    if (!r.w || !r.h) return;
    const free   = r.office && isFree(r.id, viewDateStr);
    
    if (r.id === 'og2-2') {
      console.log('Checking og2-2:', {free, viewDateStr, state});
    }
    
    const fill   = r.office ? (free?'var(--free-fill)'   :'var(--busy-fill)')   : 'var(--nonroom-fill)';
    const stroke = r.office ? (free?'var(--free-stroke)' :'var(--busy-stroke)') : 'var(--nonroom-stroke)';
    const tcolor = r.office ? (free?'var(--free-text)'   :'var(--busy-text)')   : 'var(--nonroom-text)';

    const g = document.createElementNS('http://www.w3.org/2000/svg','g');
    if (r.office) g.setAttribute('class','room-g');

    const rect = document.createElementNS('http://www.w3.org/2000/svg','rect');
    rect.setAttribute('x',r.x); rect.setAttribute('y',r.y);
    rect.setAttribute('width',r.w); rect.setAttribute('height',r.h);
    rect.setAttribute('fill',fill); rect.setAttribute('stroke',stroke);
    rect.setAttribute('stroke-width','1.5'); rect.setAttribute('rx','3');
    g.appendChild(rect);

    const labelY = free && r.h>38 ? r.y+r.h/2-7 : r.y+r.h/2;
    const t = document.createElementNS('http://www.w3.org/2000/svg','text');
    t.setAttribute('x',r.x+r.w/2); t.setAttribute('y',labelY);
    t.setAttribute('class','rlabel'); t.setAttribute('fill',tcolor);
    t.textContent = r.label;
    g.appendChild(t);

    // Don't show names in free rooms to avoid confusion (person is NOT there)

    if (r.office) g.addEventListener('click',()=>openRoomDetail(r));
    svg.appendChild(g);
  });
}

function renderAll() {
  renderSVG('svg-og2', OG2);
  renderSVG('svg-og3', OG3);
  renderSidebar();
  updateMainBtn();
}

// ── SIDEBAR ───────────────────────────────────────────────────────────────────
const AV_COLORS=['#3d5a99','#43a047','#e53935','#8e24aa','#f4511e','#0097a7','#f9a825'];
function avColor(n){let h=0;for(const c of n)h+=c.charCodeAt(0);return AV_COLORS[h%AV_COLORS.length];}
function initials(n){return n.split(' ').filter(Boolean).slice(-2).map(w=>w[0]).join('').toUpperCase();}
function rLabel(id){return id.replace('og2-','OG2 / Zi. ').replace('og3-','OG3 / Zi. ');}

function renderSidebar() {
  const total = ALL_OFFICES.length;
  // Use local date string to avoid timezone issues
  const y = currentViewDate.getFullYear();
  const m = String(currentViewDate.getMonth() + 1).padStart(2, '0');
  const d = String(currentViewDate.getDate()).padStart(2, '0');
  const viewDateStr = `${y}-${m}-${d}`;
  
  console.log('renderSidebar - viewDateStr:', viewDateStr, 'state:', state);
  
  // Get absences active on current view date
  const activeAbsences = state.filter(a => viewDateStr >= a.dateFrom && viewDateStr <= a.dateTo);
  const freeN = activeAbsences.length;
  
  console.log('Active absences:', activeAbsences, 'count:', freeN);
  
  document.getElementById('statFree').textContent = freeN;
  document.getElementById('statBusy').textContent = total - freeN;

  const frl = document.getElementById('freeRoomsList');
  frl.innerHTML = freeN
    ? activeAbsences.map(a=>`<div class="free-room-item" onclick="jumpToRoom('${a.roomId}')"><div class="free-dot"></div><span>${rLabel(a.roomId)}</span></div>`).join('')
    : '<div class="no-entry">Alle Zimmer besetzt</div>';

  const al = document.getElementById('absentList');
  al.innerHTML = activeAbsences.length
    ? activeAbsences.map(a=>{
        const days = a.dateFrom === a.dateTo 
          ? 'Heute' 
          : `${formatShortDate(a.dateFrom)} – ${formatShortDate(a.dateTo)}`;
        return `<div class="absent-item">
          <div class="av" style="background:${avColor(a.name)}">${initials(a.name)}</div>
          <div class="absent-info">
            <div class="absent-name">${a.name}</div>
            <div class="absent-room">${rLabel(a.roomId)} · ${days}${a.note?' · '+a.note:''}</div>
          </div></div>`;
      }).join('')
    : '<div class="no-entry">Niemand abwesend gemeldet</div>';

  const all = new Set([...STAFF,...state.map(a=>a.name)]);
  document.getElementById('staffList').innerHTML = [...all].map(n=>`<option value="${n}">`).join('');
}

function formatShortDate(dateStr) {
  const d = new Date(dateStr + 'T00:00:00');
  return `${d.getDate()}.${d.getMonth()+1}.`;
}

function jumpToRoom(roomId) {
  const floor = roomId.startsWith('og2')?'og2':'og3';
  document.querySelectorAll('.ftab').forEach((t,i)=>t.classList.toggle('active',(i===0&&floor==='og2')||(i===1&&floor==='og3')));
  document.querySelectorAll('.floor-panel').forEach(p=>p.classList.remove('active'));
  document.getElementById('panel-'+floor).classList.add('active');
}

// ── ABSENT MODAL ──────────────────────────────────────────────────────────────
function buildRoomSelect(preselect) {
  const sel = document.getElementById('inRoom');
  sel.innerHTML = '<option value="">– bitte wählen –</option>';
  [{label:'OG 2', rooms:OG2},{label:'OG 3', rooms:OG3}].forEach(({label,rooms})=>{
    const grp = document.createElement('optgroup'); grp.label = label;
    rooms.filter(r=>r.office).forEach(r=>{
      const o=document.createElement('option'); o.value=r.id; o.textContent=`Zimmer ${r.label}`; grp.appendChild(o);
    });
    sel.appendChild(grp);
  });
  if (preselect) sel.value=preselect;
}

function openAbsentModal(preselect) {
  buildRoomSelect(preselect);
  const saved = localStorage.getItem('kwr_myname')||'';
  document.getElementById('inName').value = saved;
  
  // Set default dates to today
  const todayStr = getTodayStr();
  document.getElementById('inDateFrom').value = todayStr;
  document.getElementById('inDateTo').value = todayStr;
  
  if (saved && !preselect) {
    // Find most recent absence for this person
    const personAbsences = state.filter(a=>a.name===saved).sort((a,b)=>b.dateFrom.localeCompare(a.dateFrom));
    if (personAbsences.length) {
      document.getElementById('inRoom').value = personAbsences[0].roomId;
    }
  }
  document.getElementById('inNote').value='';
  document.getElementById('absentOverlay').classList.add('open');
  setTimeout(()=>document.getElementById('inName').focus(),80);
}
function closeAbsent(){ document.getElementById('absentOverlay').classList.remove('open'); }

async function saveAbsent() {
  const name     = document.getElementById('inName').value.trim();
  const roomId   = document.getElementById('inRoom').value;
  const dateFrom = document.getElementById('inDateFrom').value;
  const dateTo   = document.getElementById('inDateTo').value;
  const note     = document.getElementById('inNote').value.trim();
  
  if (!name)     { shake('inName');     return; }
  if (!roomId)   { shake('inRoom');    return; }
  if (!dateFrom) { shake('inDateFrom'); return; }
  if (!dateTo)   { shake('inDateTo');   return; }
  if (dateFrom > dateTo) { 
    shake('inDateTo'); 
    showToast('⚠ "Bis"-Datum muss nach "Von"-Datum sein'); 
    return; 
  }
  
  // Delete conflicting entries from Supabase
  const toDelete = state.filter(a => {
    return (a.name === name) || 
           (a.roomId === roomId && !(dateTo < a.dateFrom || dateFrom > a.dateTo));
  });
  
  for (const item of toDelete) {
    if (item.id) {
      await deleteFromSupabase(item.id);
    }
  }
  
  // Save new entry to Supabase
  const saved = await saveToSupabase({name, roomId, dateFrom, dateTo, note, occupiedBy: null});
  
  if (saved) {
    localStorage.setItem('kwr_myname', name);
    closeAbsent();
    
    const dayCount = Math.round((new Date(dateTo) - new Date(dateFrom)) / (1000*60*60*24)) + 1;
    const msg = dayCount === 1 
      ? `✓ ${rLabel(roomId)} ist am ${formatShortDate(dateFrom)} frei` 
      : `✓ ${rLabel(roomId)} ist vom ${formatShortDate(dateFrom)} bis ${formatShortDate(dateTo)} frei (${dayCount} Tage)`;
    showToast(msg);
    
    // Reload from Supabase and re-render
    state = await loadState();
    renderAll();
  }
}

// ── MAIN BUTTON ───────────────────────────────────────────────────────────────
function updateMainBtn() {
  const myName = localStorage.getItem('kwr_myname');
  const btn = document.getElementById('mainBtn');
  const todayStr = getTodayStr();
  const amAbsentToday = myName && state.some(a=>a.name===myName && todayStr >= a.dateFrom && todayStr <= a.dateTo);
  btn.textContent = amAbsentToday ? '↩ Zurückgemeldet (rückgängig)' : 'Ich bin heute weg';
  btn.classList.toggle('undo', !!amAbsentToday);
}
document.getElementById('mainBtn').addEventListener('click', async ()=>{
  const myName = localStorage.getItem('kwr_myname');
  const todayStr = getTodayStr();
  const myAbsence = state.find(a=>a.name===myName && todayStr >= a.dateFrom && todayStr <= a.dateTo);
  
  if (myAbsence) {
    // Remove this absence from Supabase
    if (myAbsence.id) {
      await deleteFromSupabase(myAbsence.id);
    }
    showToast(`↩ ${myName} – Abwesenheit zurückgenommen`);
    state = await loadState();
    renderAll();
  } else {
    openAbsentModal();
  }
});

// ── ROOM DETAIL ───────────────────────────────────────────────────────────────
function openRoomDetail(r) {
  currentRoom = r;
  // Use local date string to avoid timezone issues
  const y = currentViewDate.getFullYear();
  const m = String(currentViewDate.getMonth() + 1).padStart(2, '0');
  const d = String(currentViewDate.getDate()).padStart(2, '0');
  const viewDateStr = `${y}-${m}-${d}`;
  
  const free = isFree(r.id, viewDateStr);
  const absence = state.find(a=>a.roomId===r.id && viewDateStr >= a.dateFrom && viewDateStr <= a.dateTo);
  const floor = r.id.startsWith('og2')?'OG 2':'OG 3';

  let html = `<div class="room-detail-ttl">${floor} · Zimmer ${r.label}</div>
    <span class="status-pill ${free?'pill-free':'pill-busy'}">${free?'✓ Frei':'● Besetzt'}</span>`;

  if (free && absence) {
    const dateInfo = absence.dateFrom === absence.dateTo 
      ? 'heute' 
      : `vom ${formatShortDate(absence.dateFrom)} bis ${formatShortDate(absence.dateTo)}`;
    html += `<div style="font-size:0.85rem;color:var(--ink2);margin-bottom:14px">
      <strong>${absence.name}</strong> ist ${dateInfo} nicht da${absence.note?` · <em>${absence.note}</em>`:''}.
    </div>`;
  } else {
    html += `<div style="font-size:0.82rem;color:var(--ink2);margin-bottom:14px">
      Dieses Zimmer gilt als besetzt.<br>Falls der Inhaber abwesend ist, kann eine Abwesenheit eingetragen werden.
    </div>`;
  }
  document.getElementById('roomDetailContent').innerHTML = html;
  const foot = document.getElementById('roomDetailFooter');
  foot.innerHTML = free
    ? `<button class="btn btn-ghost" onclick="closeRoomModal()">Schließen</button>
       <button class="btn btn-red" onclick="removeAbsence('${r.id}','${viewDateStr}')">Abwesenheit löschen</button>`
    : `<button class="btn btn-ghost" onclick="closeRoomModal()">Schließen</button>
       <button class="btn btn-green" onclick="closeRoomModal();openAbsentModal('${r.id}')">Abwesenheit eintragen</button>`;

  document.getElementById('roomOverlay').classList.add('open');
}
function closeRoomModal(){ document.getElementById('roomOverlay').classList.remove('open'); }
async function removeAbsence(roomId, viewDateStr){
  const toDelete = state.filter(a=>a.roomId===roomId && viewDateStr >= a.dateFrom && viewDateStr <= a.dateTo);
  for (const item of toDelete) {
    if (item.id) {
      await deleteFromSupabase(item.id);
    }
  }
  closeRoomModal();
  showToast('Abwesenheit gelöscht');
  state = await loadState();
  renderAll();
}

// ── FLOOR SWITCH ──────────────────────────────────────────────────────────────
function switchFloor(floor,btn){
  document.querySelectorAll('.ftab').forEach(t=>t.classList.remove('active'));
  btn.classList.add('active');
  document.querySelectorAll('.floor-panel').forEach(p=>p.classList.remove('active'));
  document.getElementById('panel-'+floor).classList.add('active');
}

// ── HELPERS ───────────────────────────────────────────────────────────────────
function shake(id){
  const el=document.getElementById(id);
  el.style.borderColor='var(--busy-stroke)';
  el.animate([{transform:'translateX(-5px)'},{transform:'translateX(5px)'},{transform:'none'}],{duration:200});
  setTimeout(()=>el.style.borderColor='',600);
}
let toastT;
function showToast(msg){
  const t=document.getElementById('toast');
  t.textContent=msg; t.classList.add('show');
  clearTimeout(toastT); toastT=setTimeout(()=>t.classList.remove('show'),2800);
}
document.addEventListener('keydown',e=>{
  if(e.key==='Escape'){closeAbsent();closeRoomModal();}
  if(e.key==='Enter'&&document.getElementById('absentOverlay').classList.contains('open')) saveAbsent();
});

// ── INIT ──────────────────────────────────────────────────────────────────────
const DN=['Sonntag','Montag','Dienstag','Mittwoch','Donnerstag','Freitag','Samstag'];
const MN=['Januar','Februar','März','April','Mai','Juni','Juli','August','September','Oktober','November','Dezember'];

function updateDateDisplay() {
  const now = new Date();
  const today = new Date(); today.setHours(0,0,0,0);
  const isToday = currentViewDate.getTime() === today.getTime();
  
  const dateStr = isToday 
    ? `Heute · ${DN[currentViewDate.getDay()]}, ${currentViewDate.getDate()}. ${MN[currentViewDate.getMonth()]}`
    : `${DN[currentViewDate.getDay()]}, ${currentViewDate.getDate()}. ${MN[currentViewDate.getMonth()]} ${currentViewDate.getFullYear()}`;
  
  document.getElementById('hdrDate').textContent = dateStr;
}

function changeViewDate(days) {
  currentViewDate.setDate(currentViewDate.getDate() + days);
  updateDateDisplay();
  renderAll();
}

function goToday() {
  currentViewDate = new Date();
  currentViewDate.setHours(0,0,0,0);
  updateDateDisplay();
  renderAll();
}

updateDateDisplay();

// Initialize - load data from Supabase
(async function init() {
  console.log('Loading from Supabase...');
  state = await loadState();
  renderAll();
  console.log('Loaded', state.length, 'absences');
})();
</script>
</body>
</html>

