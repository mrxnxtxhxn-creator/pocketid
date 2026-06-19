<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Pro</title>
    <meta name="theme-color" content="#080C14"/>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src='https://unpkg.com/tesseract.js@v2.1.0/dist/tesseract.min.js'></script>
    <link href="https://cdn.jsdelivr.net/npm/remixicon@3.5.0/fonts/remixicon.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg:        #080C14;
  --surface-0: #0D1421;
  --surface-1: #111827;
  --surface-2: #1A2336;
  --surface-3: #243049;
  --border:    rgba(255,255,255,0.07);
  --border-hi: rgba(255,255,255,0.14);
  --accent:    #3B82F6;
  --accent-lo: rgba(59,130,246,0.12);
  --accent-hi: #60A5FA;
  --green:     #10B981;
  --green-lo:  rgba(16,185,129,0.12);
  --red:       #EF4444;
  --red-lo:    rgba(239,68,68,0.12);
  --yellow:    #F59E0B;
  --yellow-lo: rgba(245,158,11,0.12);
  --text-1:    #F1F5F9;
  --text-2:    #94A3B8;
  --text-3:    #475569;
  --mono: 'JetBrains Mono', monospace;
  --sans: 'Inter', system-ui, sans-serif;
}

html, body {
  height: 100%; width: 100%;
  overflow: hidden;
  background: var(--bg);
  font-family: var(--sans);
  color: var(--text-1);
  -webkit-font-smoothing: antialiased;
}

/* ── CAMERA ─────────────────────────────── */
#reader { position: fixed; inset: 0; z-index: 0; }
#reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

/* ── VIEWFINDER ──────────────────────────── */
#viewfinder {
  position: fixed; top: 50%; left: 50%;
  transform: translate(-50%, -60%);
  z-index: 10; pointer-events: none;
  transition: width .2s, height .2s;
}
.vf-frame {
  position: relative;
  width: 100%; height: 100%;
  border: 1.5px solid rgba(255,255,255,0.15);
  border-radius: 20px;
}
/* corner marks */
.vf-corner {
  position: absolute; width: 22px; height: 22px;
  border: 2.5px solid var(--accent);
}
.vf-corner.tl { top: -1px; left: -1px; border-right: none; border-bottom: none; border-radius: 8px 0 0 0; }
.vf-corner.tr { top: -1px; right: -1px; border-left: none; border-bottom: none; border-radius: 0 8px 0 0; }
.vf-corner.bl { bottom: -1px; left: -1px; border-right: none; border-top: none; border-radius: 0 0 0 8px; }
.vf-corner.br { bottom: -1px; right: -1px; border-left: none; border-top: none; border-radius: 0 0 8px 0; }
/* scan line */
.vf-line {
  position: absolute; left: 12px; right: 12px; height: 1px;
  background: linear-gradient(90deg, transparent, var(--accent), transparent);
  top: 0;
  animation: scanLine 2s ease-in-out infinite;
  box-shadow: 0 0 8px var(--accent);
}
@keyframes scanLine {
  0%   { top: 10%; opacity: 0; }
  10%  { opacity: 1; }
  90%  { opacity: 1; }
  100% { top: 90%; opacity: 0; }
}
/* vignette overlay */
#vignette {
  position: fixed; inset: 0; z-index: 5; pointer-events: none;
  background: radial-gradient(ellipse at center, transparent 38%, rgba(8,12,20,0.75) 100%);
}

/* ── TOP BAR ─────────────────────────────── */
#top-bar {
  position: fixed; top: 0; left: 0; right: 0; z-index: 30;
  padding: max(env(safe-area-inset-top), 14px) 20px 14px;
  display: flex; justify-content: space-between; align-items: center;
  background: linear-gradient(180deg, rgba(8,12,20,0.95) 0%, transparent 100%);
  backdrop-filter: blur(2px);
}
.tb-left { display: flex; align-items: center; gap: 12px; }
.tb-avatar {
  width: 36px; height: 36px; border-radius: 10px;
  background: var(--accent-lo); border: 1px solid var(--accent);
  display: flex; align-items: center; justify-content: center;
  color: var(--accent); font-size: 16px; cursor: pointer;
  transition: background .2s;
}
.tb-avatar:active { background: var(--accent); color: white; }
.tb-op { font-size: 13px; font-weight: 600; letter-spacing: 0.01em; }
.tb-zone {
  font-family: var(--mono); font-size: 10px; font-weight: 600;
  color: var(--accent-hi); letter-spacing: 0.08em;
  background: var(--accent-lo); border: 1px solid rgba(59,130,246,0.25);
  border-radius: 5px; padding: 2px 7px; margin-top: 2px; display: inline-block;
}
.tb-right { display: flex; align-items: center; gap: 8px; }
.dot-live {
  width: 8px; height: 8px; border-radius: 50%; background: var(--green);
  box-shadow: 0 0 6px var(--green);
  animation: pulse 2s infinite;
}
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.4} }

/* ── SCAN CHIP (bottom of viewfinder) ──── */
#scan-chip {
  position: fixed; top: 55%; left: 50%; transform: translateX(-50%);
  z-index: 15; pointer-events: none;
  background: rgba(8,12,20,0.7); backdrop-filter: blur(10px);
  border: 1px solid var(--border-hi); border-radius: 100px;
  padding: 6px 16px;
  font-family: var(--mono); font-size: 11px; color: var(--text-2);
  letter-spacing: 0.06em;
  transition: top .2s;
}

/* ── BOTTOM NAV ──────────────────────────── */
#bottom-nav {
  position: fixed; bottom: 0; left: 0; right: 0; z-index: 50;
  height: calc(64px + env(safe-area-inset-bottom));
  padding-bottom: env(safe-area-inset-bottom);
  background: rgba(13,20,33,0.96);
  backdrop-filter: blur(16px);
  border-top: 1px solid var(--border);
  display: flex; justify-content: space-around; align-items: center;
}
.nav-item {
  display: flex; flex-direction: column; align-items: center; gap: 4px;
  color: var(--text-3); font-size: 9.5px; font-weight: 500;
  letter-spacing: 0.05em; width: 20%; padding: 8px 0;
  cursor: pointer; transition: color .15s; text-transform: uppercase;
}
.nav-item i { font-size: 20px; transition: transform .2s; }
.nav-item.active { color: var(--accent); }
.nav-item.active i { transform: translateY(-2px); }

/* ── SHEET ───────────────────────────────── */
#sheet {
  position: fixed; bottom: 64px; left: 0; right: 0; z-index: 40;
  background: var(--surface-1);
  border-radius: 20px 20px 0 0;
  box-shadow: 0 -2px 40px rgba(0,0,0,0.6);
  transform: translateY(102%);
  transition: transform .3s cubic-bezier(.2,.8,.2,1);
  max-height: 82vh;
  display: flex; flex-direction: column;
}
#sheet.open { transform: translateY(0); }
.sheet-handle {
  padding: 12px 0 8px; display: flex; justify-content: center;
  flex-shrink: 0; cursor: pointer;
}
.sheet-handle-bar {
  width: 36px; height: 4px; border-radius: 2px;
  background: var(--surface-3);
}
.sheet-scroll { overflow-y: auto; flex: 1; padding: 0 16px 20px; }

/* ── TAB ROW ──────────────────────────────── */
.tab-row {
  display: flex; overflow-x: auto; gap: 6px; padding: 8px 16px 10px;
  flex-shrink: 0; scrollbar-width: none;
  border-bottom: 1px solid var(--border);
}
.tab-row::-webkit-scrollbar { display: none; }
.tab-pill {
  padding: 6px 14px; border-radius: 100px; font-size: 12px; font-weight: 500;
  color: var(--text-2); white-space: nowrap; cursor: pointer;
  border: 1px solid var(--border); background: transparent;
  transition: all .15s; letter-spacing: 0.02em;
}
.tab-pill.active {
  background: var(--accent); color: white; border-color: var(--accent);
  box-shadow: 0 2px 12px rgba(59,130,246,0.3);
}
.tab-content { display: none; padding-top: 16px; }
.tab-content.active { display: block; animation: fadeUp .2s ease; }
@keyframes fadeUp { from{opacity:0;transform:translateY(8px)} to{opacity:1;transform:translateY(0)} }

/* ── FORM ELEMENTS ───────────────────────── */
.field {
  background: var(--surface-0); border: 1px solid var(--border);
  color: var(--text-1); font-family: var(--sans); font-size: 14px;
  padding: 12px 14px; border-radius: 12px; width: 100%; outline: none;
  transition: border-color .15s;
}
.field:focus { border-color: var(--accent); }
.field::placeholder { color: var(--text-3); }

.btn { 
  border-radius: 12px; font-weight: 600; font-size: 14px; font-family: var(--sans);
  cursor: pointer; transition: all .15s; display: flex; align-items: center;
  justify-content: center; gap: 8px; border: none; letter-spacing: 0.01em;
}
.btn-primary { background: var(--accent); color: white; padding: 13px 20px; width: 100%; }
.btn-primary:active { background: #2563EB; transform: scale(.99); }
.btn-ghost {
  background: var(--surface-2); color: var(--text-2);
  border: 1px solid var(--border); padding: 12px 16px; width: 100%;
}
.btn-ghost:active { background: var(--surface-3); }
.btn-danger { background: var(--red-lo); color: var(--red); border: 1px solid rgba(239,68,68,0.2); padding: 13px 20px; width: 100%; }

.btn-icon {
  width: 46px; height: 46px; border-radius: 12px; flex-shrink: 0;
  background: var(--accent); color: white; font-size: 20px;
  display: flex; align-items: center; justify-content: center; cursor: pointer;
  border: none;
}

/* ── CARDS ────────────────────────────────── */
.card-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.upload-card {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 14px; padding: 18px 12px;
  display: flex; flex-direction: column; align-items: center; gap: 8px;
  cursor: pointer; transition: all .15s;
}
.upload-card:active { border-color: var(--accent); background: var(--accent-lo); }
.upload-card i { font-size: 26px; }
.upload-card span { font-size: 11px; font-weight: 600; color: var(--text-2); letter-spacing: 0.05em; text-transform: uppercase; }

.status-bar {
  background: rgba(16,185,129,0.08); border: 1px solid rgba(16,185,129,0.2);
  border-radius: 12px; padding: 12px 14px;
  display: flex; align-items: center; gap: 10px;
}
.status-dot { width: 8px; height: 8px; border-radius: 50%; background: var(--green); flex-shrink: 0; }

/* Toggle row */
.toggle-row {
  display: flex; justify-content: space-between; align-items: center;
  padding: 14px 0; border-bottom: 1px solid var(--border);
}
.toggle-row:last-child { border-bottom: none; }
.toggle-row span { font-size: 14px; color: var(--text-1); }
.sw {
  width: 44px; height: 24px; border-radius: 100px;
  background: var(--surface-3); position: relative; cursor: pointer;
  transition: background .2s; flex-shrink: 0;
}
.sw::after {
  content: ''; position: absolute; top: 3px; left: 3px;
  width: 18px; height: 18px; border-radius: 50%;
  background: white; transition: transform .2s;
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}
.sw.on { background: var(--green); }
.sw.on::after { transform: translateX(20px); }

/* Fast mode row */
.feature-row {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 12px; padding: 14px;
  display: flex; justify-content: space-between; align-items: center;
  cursor: pointer; transition: border-color .15s;
}
.feature-row.on { border-color: rgba(16,185,129,0.4); }
.feature-row .label { font-size: 14px; font-weight: 500; }
.feature-row .sublabel { font-size: 11px; color: var(--text-3); margin-top: 2px; }

/* Hunt mode */
.hunt-card {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 14px; padding: 14px;
}
.hunt-card .hunt-label {
  font-family: var(--mono); font-size: 10px; font-weight: 600;
  color: var(--accent-hi); letter-spacing: 0.08em; text-transform: uppercase;
  margin-bottom: 10px; display: flex; align-items: center; gap: 6px;
}
.hunt-status-badge {
  background: var(--accent-lo); color: var(--accent);
  border-radius: 100px; padding: 2px 8px; font-size: 9px;
}

/* ── KPI GRID ────────────────────────────── */
.kpi-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 16px; }
.kpi-card {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 14px; padding: 14px 16px;
}
.kpi-label { font-size: 10px; font-weight: 600; color: var(--text-3); letter-spacing: 0.06em; text-transform: uppercase; margin-bottom: 6px; }
.kpi-val { font-family: var(--mono); font-size: 28px; font-weight: 600; line-height: 1; }
.kpi-val.blue { color: var(--accent-hi); }
.kpi-val.green { color: var(--green); }
.kpi-val.purple { color: #A78BFA; }
.kpi-val.red { color: var(--red); }

/* section headers */
.sec-label {
  font-size: 10px; font-weight: 700; color: var(--text-3);
  letter-spacing: 0.1em; text-transform: uppercase;
  margin-bottom: 10px; margin-top: 20px;
}
.sec-label:first-child { margin-top: 0; }

/* log item */
.log-item {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 12px; padding: 12px 14px;
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 8px;
}
.log-id { font-family: var(--mono); font-size: 13px; font-weight: 600; }
.log-desc { font-size: 11px; color: var(--text-3); margin-top: 2px; max-width: 200px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.badge {
  font-family: var(--mono); font-size: 9px; font-weight: 700;
  border-radius: 6px; padding: 3px 8px; letter-spacing: 0.06em;
}
.badge.ok { background: var(--green-lo); color: var(--green); }
.badge.err { background: var(--red-lo); color: var(--red); }
.badge.warn { background: var(--yellow-lo); color: var(--yellow); }

/* zone/inventory cards */
.zone-row {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 12px; padding: 14px;
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 8px; transition: border-color .15s;
}
.zone-row.active { border-color: var(--green); }
.zone-name { font-size: 14px; font-weight: 600; }
.zone-count { font-size: 11px; color: var(--text-3); margin-top: 2px; font-family: var(--mono); }

.btn-sm {
  font-size: 12px; font-weight: 600; padding: 8px 14px;
  border-radius: 8px; cursor: pointer; border: none;
  transition: all .15s; letter-spacing: 0.02em;
}
.btn-sm-primary { background: var(--accent); color: white; }
.btn-sm-primary:active { background: #2563EB; }
.btn-sm-green { background: var(--green-lo); color: var(--green); border: 1px solid rgba(16,185,129,0.25); }
.btn-sm-ghost { background: var(--surface-2); color: var(--text-2); border: 1px solid var(--border); }

/* slider */
input[type=range] {
  width: 100%; height: 4px; background: var(--surface-3);
  border-radius: 2px; outline: none; -webkit-appearance: none; margin-top: 8px;
}
input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none; width: 18px; height: 18px; border-radius: 50%;
  background: var(--accent); cursor: pointer; border: 2px solid white;
  box-shadow: 0 0 8px rgba(59,130,246,0.4);
}

/* ── FEEDBACK ─────────────────────────────── */
#feedback {
  position: fixed; inset: 0; z-index: 70; pointer-events: none;
  display: flex; align-items: flex-end; justify-content: center;
  padding-bottom: calc(80px + env(safe-area-inset-bottom) + 16px);
  opacity: 0; transition: opacity .15s;
}
.fb-pill {
  background: rgba(13,20,33,0.9); backdrop-filter: blur(16px);
  border: 1px solid var(--border-hi); border-radius: 16px;
  padding: 14px 20px; display: flex; flex-direction: column; align-items: center;
  min-width: 240px; max-width: 320px;
}
.fb-status { font-size: 11px; font-weight: 700; letter-spacing: 0.1em; text-transform: uppercase; margin-bottom: 4px; font-family: var(--mono); }
.fb-id { font-family: var(--mono); font-size: 18px; font-weight: 700; margin-bottom: 4px; }
.fb-desc { font-size: 12px; color: var(--text-2); text-align: center; }

/* ── UNDO BTN ────────────────────────────── */
#undo-btn {
  position: fixed; bottom: calc(80px + env(safe-area-inset-bottom) + 80px); right: 16px;
  z-index: 45; background: var(--surface-2); border: 1px solid var(--border-hi);
  color: var(--text-2); width: 44px; height: 44px; border-radius: 12px;
  display: flex; align-items: center; justify-content: center; font-size: 18px;
  opacity: 0; pointer-events: none; transition: opacity .2s; cursor: pointer;
  backdrop-filter: blur(8px);
}
#undo-btn.show { opacity: 1; pointer-events: auto; }

/* ── LOGIN ────────────────────────────────── */
#login-screen {
  position: fixed; inset: 0; z-index: 100;
  background: var(--bg);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  padding: 24px;
}
.login-logo {
  width: 72px; height: 72px; border-radius: 20px;
  background: var(--accent-lo); border: 1px solid rgba(59,130,246,0.3);
  display: flex; align-items: center; justify-content: center;
  font-size: 32px; color: var(--accent); margin-bottom: 28px;
  box-shadow: 0 0 40px rgba(59,130,246,0.15);
}
.login-title { font-size: 24px; font-weight: 700; margin-bottom: 6px; }
.login-sub { font-size: 14px; color: var(--text-2); margin-bottom: 32px; }
.login-form { width: 100%; max-width: 320px; display: flex; flex-direction: column; gap: 12px; }

/* ── OCR LOADING ─────────────────────────── */
#ocr-loading {
  position: fixed; inset: 0; z-index: 95; background: rgba(8,12,20,0.92);
  backdrop-filter: blur(12px); display: none;
  flex-direction: column; align-items: center; justify-content: center; gap: 14px;
}
#ocr-loading.show { display: flex; }
.spin { animation: spin 1s linear infinite; font-size: 40px; color: var(--accent); }
@keyframes spin { to { transform: rotate(360deg); } }

/* ── PROFILE ─────────────────────────────── */
.profile-hero {
  display: flex; flex-direction: column; align-items: center; padding: 20px 0 24px; 
}
.profile-av {
  width: 80px; height: 80px; border-radius: 22px;
  background: var(--accent-lo); border: 1.5px solid rgba(59,130,246,0.3);
  display: flex; align-items: center; justify-content: center;
  font-size: 36px; color: var(--accent); margin-bottom: 12px;
}
.profile-name { font-size: 20px; font-weight: 700; }
.profile-sub { font-size: 12px; color: var(--text-3); margin-top: 2px; }
.stat-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 20px; }
.stat-card { background: var(--surface-0); border: 1px solid var(--border); border-radius: 14px; padding: 16px; text-align: center; }
.stat-val { font-family: var(--mono); font-size: 28px; font-weight: 700; }
.stat-lbl { font-size: 10px; color: var(--text-3); text-transform: uppercase; letter-spacing: 0.06em; margin-top: 4px; }

/* scrollbars */
::-webkit-scrollbar { width: 0; }
.hidden { display: none !important; }

/* EMPTY STATE */
.empty {
  text-align: center; padding: 40px 0; color: var(--text-3);
}
.empty i { font-size: 32px; margin-bottom: 10px; display: block; }
.empty p { font-size: 13px; }

/* settings container */
.settings-section {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 14px; padding: 4px 14px; margin-bottom: 12px;
}
</style>
</head>
<body>

<!-- TOP BAR -->
<div id="top-bar">
  <div class="tb-left">
    <div class="tb-avatar" onclick="openTab('perfil')"><i class="ri-user-3-line"></i></div>
    <div>
      <div class="tb-op" id="status-operator">Operador</div>
      <div class="tb-zone" id="status-zone">SEM ZONA</div>
    </div>
  </div>
  <div class="tb-right">
    <span style="font-family:var(--mono);font-size:10px;color:var(--text-3)">LIVE</span>
    <div class="dot-live" id="conn-dot"></div>
  </div>
</div>

<!-- CAMERA -->
<div id="reader"></div>
<div id="vignette"></div>

<!-- VIEWFINDER -->
<div id="viewfinder">
  <div class="vf-frame">
    <div class="vf-corner tl"></div>
    <div class="vf-corner tr"></div>
    <div class="vf-corner bl"></div>
    <div class="vf-corner br"></div>
    <div class="vf-line"></div>
  </div>
</div>
<div id="scan-chip">APONTAR PARA CÓDIGO</div>

<!-- FEEDBACK -->
<div id="feedback">
  <div class="fb-pill">
    <div class="fb-status" id="fb-status">ENCONTRADO</div>
    <div class="fb-id" id="fb-id">—</div>
    <div class="fb-desc" id="fb-desc"></div>
  </div>
</div>

<!-- UNDO -->
<button id="undo-btn" onclick="doUndo()"><i class="ri-arrow-go-back-line"></i></button>

<!-- OCR LOADING -->
<div id="ocr-loading">
  <i class="ri-loader-4-line spin"></i>
  <span style="font-size:14px;font-weight:600;color:var(--text-2)">Lendo imagem…</span>
</div>

<!-- LOGIN -->
<div id="login-screen">
  <div class="login-logo"><i class="ri-box-3-fill"></i></div>
  <div class="login-title">Natefy Pro</div>
  <div class="login-sub">Sistema de bipagem operacional</div>
  <div class="login-form">
    <input type="text" id="op-input" class="field" style="text-align:center;font-size:16px" placeholder="Seu nome">
    <button class="btn btn-primary" onclick="doLogin()">ENTRAR <i class="ri-arrow-right-line"></i></button>
  </div>
</div>

<!-- SHEET -->
<div id="sheet">
  <div class="sheet-handle" onclick="toggleSheet()">
    <div class="sheet-handle-bar"></div>
  </div>

  <div class="tab-row" id="tab-row">
    <button class="tab-pill active" onclick="switchTab('scan',this)">Scan</button>
    <button class="tab-pill" onclick="switchTab('listas',this)">Listas</button>
    <button class="tab-pill" onclick="switchTab('zonas',this)">Zonas</button>
    <button class="tab-pill" onclick="switchTab('dash',this)">Dashboard</button>
    <button class="tab-pill" onclick="switchTab('log',this)">Log</button>
    <button class="tab-pill" onclick="switchTab('config',this)">Config</button>
    <button class="tab-pill" onclick="switchTab('perfil',this)">Perfil</button>
  </div>

  <div class="sheet-scroll">

    <!-- SCAN TAB -->
    <div id="view-scan" class="tab-content active">
      <div class="card-grid" style="margin-bottom:12px">
        <div class="upload-card" onclick="document.getElementById('file-input').click()">
          <i class="ri-file-excel-2-fill" style="color:#10B981"></i>
          <span>Planilha</span>
        </div>
        <div class="upload-card" onclick="document.getElementById('image-input').click()">
          <i class="ri-camera-fill" style="color:var(--accent)"></i>
          <span>Foto OCR</span>
        </div>
      </div>

      <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
      <input type="file" id="image-input" class="hidden" accept="image/*">

      <div id="file-status" class="status-bar hidden" style="margin-bottom:12px">
        <div class="status-dot"></div>
        <div>
          <div style="font-size:11px;font-weight:700;color:var(--green);letter-spacing:0.04em">LISTA CARREGADA</div>
          <div id="file-count" style="font-size:13px;color:var(--text-1)">0 itens</div>
        </div>
      </div>

      <div style="display:flex;gap:8px;margin-bottom:12px">
        <input type="text" id="manual-input" class="field" placeholder="Digitar ID manualmente…">
        <button class="btn-icon" onclick="scanManual()"><i class="ri-check-line"></i></button>
      </div>

      <div class="feature-row" id="fast-row" onclick="toggleFast()" style="margin-bottom:12px">
        <div>
          <div class="label">Modo Rápido</div>
          <div class="sublabel">Sem pausa entre scans</div>
        </div>
        <div class="sw" id="fast-sw"></div>
      </div>

      <div class="hunt-card" style="margin-bottom:12px">
        <div class="hunt-label">
          <i class="ri-crosshair-2-line"></i> Modo Caça
          <span class="hunt-status-badge hidden" id="hunt-badge">ATIVO</span>
        </div>
        <div style="display:flex;gap:8px">
          <input type="text" id="hunt-id" class="field" style="font-size:13px;padding:10px 12px" placeholder="ID alvo…">
          <button class="btn-sm btn-sm-primary" id="hunt-btn" onclick="toggleHunt()">Ativar</button>
        </div>
      </div>

      <button class="btn btn-danger" onclick="clearSession()" style="font-size:13px;padding:12px">
        <i class="ri-delete-bin-6-line"></i> Limpar sessão
      </button>
    </div>

    <!-- LISTAS TAB -->
    <div id="view-listas" class="tab-content">
      <div class="sec-label">Inventário por Zona</div>
      <div id="inventory-list"></div>
      <input type="file" id="inv-file-input" class="hidden" accept=".xlsx,.csv">
    </div>

    <!-- ZONAS TAB -->
    <div id="view-zonas" class="tab-content">
      <div class="sec-label">Zona Ativa</div>
      <div id="zones-list"></div>
    </div>

    <!-- DASH TAB -->
    <div id="view-dash" class="tab-content">
      <div class="sec-label">Desempenho da Sessão</div>
      <div class="kpi-grid">
        <div class="kpi-card">
          <div class="kpi-label">Produt. /h</div>
          <div class="kpi-val blue" id="kpi-sph">0</div>
        </div>
        <div class="kpi-card">
          <div class="kpi-label">Acuracidade</div>
          <div class="kpi-val green" id="kpi-acc">100%</div>
        </div>
        <div class="kpi-card">
          <div class="kpi-label">Tempo Médio</div>
          <div class="kpi-val purple" id="kpi-time">0s</div>
        </div>
        <div class="kpi-card">
          <div class="kpi-label">Missorts</div>
          <div class="kpi-val red" id="kpi-miss">0</div>
        </div>
      </div>
      <button class="btn btn-ghost" onclick="downloadExcel()">
        <i class="ri-file-excel-2-line" style="color:var(--green)"></i> Exportar Relatório
      </button>
    </div>

    <!-- LOG TAB -->
    <div id="view-log" class="tab-content">
      <div class="sec-label">Últimas bipagens</div>
      <div id="log-list"></div>
    </div>

    <!-- CONFIG TAB -->
    <div id="view-config" class="tab-content">
      <div class="sec-label">Câmera</div>
      <div class="settings-section">
        <div class="toggle-row">
          <div>
            <div style="font-size:13px;font-weight:500">Tamanho da mira</div>
            <div style="font-size:11px;color:var(--text-3);margin-top:1px">
              <span id="range-val">80%</span>
            </div>
          </div>
          <input type="range" id="size-range" min="40" max="95" value="80" oninput="updateSize(this.value)" style="width:100px">
        </div>
        <div class="toggle-row">
          <div>
            <div style="font-size:13px;font-weight:500">FPS do scanner</div>
            <div style="font-size:11px;color:var(--text-3);margin-top:1px"><span id="fps-val">30</span> fps</div>
          </div>
          <input type="range" id="fps-range" min="5" max="60" value="30" oninput="updateFPS(this.value)" style="width:100px">
        </div>
      </div>

      <div class="sec-label">Feedback</div>
      <div class="settings-section">
        <div class="toggle-row">
          <span>Som de bipagem</span>
          <div class="sw on" id="sw-sound" onclick="toggleSetting('sound')"></div>
        </div>
        <div class="toggle-row">
          <span>Vibração</span>
          <div class="sw on" id="sw-vibrate" onclick="toggleSetting('vibrate')"></div>
        </div>
      </div>
    </div>

    <!-- PERFIL TAB -->
    <div id="view-perfil" class="tab-content">
      <div class="profile-hero">
        <div class="profile-av"><i class="ri-user-3-fill"></i></div>
        <div class="profile-name" id="p-name">—</div>
        <div class="profile-sub">Operador Natefy</div>
      </div>
      <div class="stat-grid">
        <div class="stat-card">
          <div class="stat-val blue" id="p-today">0</div>
          <div class="stat-lbl">Hoje</div>
        </div>
        <div class="stat-card">
          <div class="stat-val purple" id="p-total">0</div>
          <div class="stat-lbl">Total Geral</div>
        </div>
      </div>
      <button class="btn btn-danger" onclick="logout()">
        <i class="ri-logout-box-r-line"></i> Sair da conta
      </button>
    </div>

  </div><!-- sheet-scroll -->
</div><!-- sheet -->

<!-- BOTTOM NAV -->
<nav id="bottom-nav">
  <div class="nav-item active" onclick="navTap('scan',this)"><i class="ri-qr-scan-2-line"></i><span>Scan</span></div>
  <div class="nav-item" onclick="navTap('listas',this)"><i class="ri-archive-line"></i><span>Listas</span></div>
  <div class="nav-item" onclick="navTap('zonas',this)"><i class="ri-map-pin-line"></i><span>Zonas</span></div>
  <div class="nav-item" onclick="navTap('dash',this)"><i class="ri-bar-chart-box-line"></i><span>Dash</span></div>
  <div class="nav-item" onclick="navTap('log',this)"><i class="ri-history-line"></i><span>Log</span></div>
</nav>

<script>
/* ═══════════════════════════════════════════
   STATE
═══════════════════════════════════════════ */
const KEY = 'natefy_pro_v1';
const WEBHOOK = 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9';

let S = {
  operator: null,
  idsToFind: [],
  idDescs: {},
  found: [],
  logs: [],
  activeZone: null,
  zoneData: {},       // {zoneId: [ids]}
  zones: ['Buffered','Sorting','Fraude','Missort','Returns','Bulky'],
  fastMode: false,
  hunt: { active: false, target: null },
  lastUndo: null,
  lastScan: 0,
  paused: false,
  settings: { size: 80, fps: 30, sound: true, vibrate: true },
  totalLife: 0,
};
let scanner = null;
let audioCtx = null;

/* ═══════════════════════════════════════════
   PERSIST
═══════════════════════════════════════════ */
function save() {
  const s = { ...S };
  localStorage.setItem(KEY, JSON.stringify(s));
}
function load() {
  const raw = localStorage.getItem(KEY);
  if (!raw) return;
  const d = JSON.parse(raw);
  S = { ...S, ...d };
  if (!S.settings) S.settings = { size: 80, fps: 30, sound: true, vibrate: true };
}

/* ═══════════════════════════════════════════
   INIT
═══════════════════════════════════════════ */
document.addEventListener('DOMContentLoaded', () => {
  load();
  applySettings();
  if (S.operator) bootApp();
  document.getElementById('file-input').onchange = e => handleFile(e.target.files[0]);
  document.getElementById('image-input').onchange = e => handleOCR(e.target.files[0]);
  document.getElementById('inv-file-input').onchange = e => handleInvFile(e.target.files[0]);
  document.getElementById('manual-input').addEventListener('keydown', e => { if (e.key === 'Enter') scanManual(); });
  document.body.addEventListener('click', () => { if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)(); }, { once: true });
  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'visible' && S.operator && !scanner) startScanner();
  });
});

function bootApp() {
  document.getElementById('login-screen').classList.add('hidden');
  document.getElementById('status-operator').textContent = S.operator;
  document.getElementById('p-name').textContent = S.operator;
  setZoneChip();
  applyFastUI();
  if (S.fastMode) document.getElementById('fast-row').classList.add('on');
  updateFileStatus();
  startScanner();
}

/* ═══════════════════════════════════════════
   LOGIN
═══════════════════════════════════════════ */
window.doLogin = function() {
  const v = document.getElementById('op-input').value.trim();
  if (!v) { document.getElementById('op-input').focus(); return; }
  S.operator = v;
  save();
  bootApp();
};
window.logout = function() {
  if (!confirm('Sair da conta?')) return;
  S.operator = null; save(); location.reload();
};

/* ═══════════════════════════════════════════
   SCANNER
═══════════════════════════════════════════ */
function startScanner() {
  if (scanner) return;
  const size = Math.floor(window.innerWidth * (S.settings.size / 100));
  scanner = new Html5Qrcode('reader');
  scanner.start(
    { facingMode: 'environment' },
    { fps: S.settings.fps, qrbox: size },
    id => onScan(id)
  ).catch(err => console.log('cam:', err));
  updateViewfinder();
}

function updateViewfinder() {
  const vf = document.getElementById('viewfinder');
  const size = Math.floor(window.innerWidth * (S.settings.size / 100));
  vf.style.width = size + 'px';
  vf.style.height = size + 'px';
}

/* ═══════════════════════════════════════════
   SCAN LOGIC
═══════════════════════════════════════════ */
function onScan(rawId) {
  const now = Date.now();
  if (now - S.lastScan < 700) return;
  S.lastScan = now;
  if (S.paused) return;
  if (!S.fastMode) S.paused = true;

  const id = rawId.trim();
  const desc = S.idDescs[id] || '';

  // Hunt mode
  if (S.hunt.active) {
    if (id === S.hunt.target) {
      showFeedback('ok', 'ALVO ENCONTRADO', id, '');
      toggleHunt();
      sendN8n({ id, status: 'HUNT_SUCCESS', operator: S.operator, ts: new Date().toISOString() });
    }
    setTimeout(() => { if (!S.fastMode) S.paused = false; }, 1000);
    return;
  }

  let status = 'NÃO ENCONTRADO', fbType = 'err';

  // Missort check
  if (S.activeZone) {
    for (const [zid, ids] of Object.entries(S.zoneData)) {
      if (zid !== S.activeZone && ids.includes(id)) {
        status = 'MISSORT'; fbType = 'warn'; break;
      }
    }
  }

  // Duplicate
  if (status === 'NÃO ENCONTRADO' && S.found.some(x => x.id === id)) {
    status = 'DUPLICADO'; fbType = 'warn';
  }

  // Success
  if (status === 'NÃO ENCONTRADO' && S.idsToFind.includes(id)) {
    status = 'SUCESSO'; fbType = 'ok';
    S.idsToFind = S.idsToFind.filter(x => x !== id);
  }

  const entry = { id, status, desc, type: fbType, time: new Date().toISOString(), operator: S.operator, zone: S.activeZone };
  S.logs.unshift(entry);
  if (status === 'SUCESSO') S.found.unshift(entry);
  S.lastUndo = entry;
  S.totalLife++;
  save();
  showFeedback(fbType, status, id, desc);
  showUndo();
  sendN8n(entry);
  refreshProfile();

  setTimeout(() => { if (!S.fastMode) S.paused = false; }, S.fastMode ? 250 : 1100);
}

window.scanManual = function() {
  const v = document.getElementById('manual-input').value.trim();
  if (!v) return;
  document.getElementById('manual-input').value = '';
  onScan(v);
};

/* ═══════════════════════════════════════════
   FEEDBACK
═══════════════════════════════════════════ */
let fbTimer;
function showFeedback(type, status, id, desc) {
  const el = document.getElementById('feedback');
  const pill = el.querySelector('.fb-pill');

  const colors = { ok: '#10B981', err: '#EF4444', warn: '#F59E0B' };
  const col = colors[type] || '#94A3B8';

  document.getElementById('fb-status').style.color = col;
  document.getElementById('fb-status').textContent = status;
  document.getElementById('fb-id').style.color = col;
  document.getElementById('fb-id').textContent = id;
  document.getElementById('fb-desc').textContent = desc;
  pill.style.borderColor = col + '33';

  el.style.opacity = '1';
  el.style.pointerEvents = 'none';

  if (S.settings.vibrate && navigator.vibrate) navigator.vibrate(type === 'ok' ? [80] : [40, 60, 40]);
  if (S.settings.sound) playBeep(type);

  clearTimeout(fbTimer);
  fbTimer = setTimeout(() => { el.style.opacity = '0'; }, S.fastMode ? 400 : 1400);
}

function playBeep(type) {
  if (!audioCtx) return;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.connect(g); g.connect(audioCtx.destination);
  g.gain.value = 0.08;
  o.frequency.value = type === 'ok' ? 1400 : type === 'warn' ? 700 : 250;
  o.start(); setTimeout(() => o.stop(), 120);
}

/* ═══════════════════════════════════════════
   UNDO
═══════════════════════════════════════════ */
let undoTimer;
function showUndo() {
  const u = document.getElementById('undo-btn');
  u.classList.add('show');
  clearTimeout(undoTimer);
  undoTimer = setTimeout(() => u.classList.remove('show'), 5000);
}
window.doUndo = function() {
  if (!S.lastUndo) return;
  S.logs = S.logs.filter((_, i) => i !== 0);
  if (S.lastUndo.status === 'SUCESSO') {
    S.found = S.found.filter((_, i) => i !== 0);
    S.idsToFind.push(S.lastUndo.id);
  }
  sendN8n({ ...S.lastUndo, status: 'CANCELADO' });
  S.lastUndo = null;
  document.getElementById('undo-btn').classList.remove('show');
  save();
};

/* ═══════════════════════════════════════════
   FILES
═══════════════════════════════════════════ */
function handleFile(f) {
  if (!f) return;
  const r = new FileReader();
  r.onload = e => {
    const d = new Uint8Array(e.target.result);
    const wb = XLSX.read(d, { type: 'array' });
    const rows = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], { header: 1 });
    const ids = rows.map(x => String(x[0])).filter(i => i && /\d+/.test(i));
    S.idsToFind = ids; S.found = [];
    updateFileStatus();
    save();
  };
  r.readAsArrayBuffer(f);
}

async function handleOCR(f) {
  if (!f) return;
  document.getElementById('ocr-loading').classList.add('show');
  try {
    const w = Tesseract.createWorker();
    await w.load(); await w.loadLanguage('eng'); await w.initialize('eng');
    const { data: { text } } = await w.recognize(f);
    await w.terminate();
    const ids = []; const descs = {};
    text.split('\n').forEach(line => {
      const m = line.match(/(\d{8,14})/);
      if (m) {
        const id = m[0];
        const desc = line.replace(id, '').replace(/^[\s>\-.]+/, '').trim();
        ids.push(id);
        descs[id] = desc.length > 2 ? desc : '';
      }
    });
    if (ids.length) {
      S.idsToFind = ids; S.idDescs = { ...S.idDescs, ...descs }; S.found = [];
      updateFileStatus(); save();
    } else alert('Nenhum código encontrado na imagem.');
  } catch { alert('Erro no OCR.'); }
  finally { document.getElementById('ocr-loading').classList.remove('show'); }
}

let tempInvZone = null;
function handleInvFile(f) {
  if (!f || !tempInvZone) return;
  const r = new FileReader();
  r.onload = e => {
    const d = new Uint8Array(e.target.result);
    const wb = XLSX.read(d, { type: 'array' });
    const rows = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], { header: 1 });
    const ids = rows.map(x => String(x[0])).filter(i => i && /\d+/.test(i));
    S.zoneData[tempInvZone] = ids;
    save(); renderInventory();
  };
  r.readAsArrayBuffer(f);
}

function updateFileStatus() {
  const el = document.getElementById('file-status');
  const cnt = document.getElementById('file-count');
  if (S.idsToFind.length > 0) {
    el.classList.remove('hidden');
    cnt.textContent = `${S.idsToFind.length} itens prontos`;
  } else el.classList.add('hidden');
}

/* ═══════════════════════════════════════════
   SETTINGS
═══════════════════════════════════════════ */
function applySettings() {
  document.getElementById('size-range').value = S.settings.size;
  document.getElementById('fps-range').value = S.settings.fps;
  document.getElementById('range-val').textContent = S.settings.size + '%';
  document.getElementById('fps-val').textContent = S.settings.fps;
  applySW('sw-sound', S.settings.sound);
  applySW('sw-vibrate', S.settings.vibrate);
}
function applySW(id, val) {
  const el = document.getElementById(id);
  if (val) el.classList.add('on'); else el.classList.remove('on');
}
window.updateSize = function(v) {
  S.settings.size = parseInt(v);
  document.getElementById('range-val').textContent = v + '%';
  updateViewfinder(); save();
};
window.updateFPS = function(v) {
  S.settings.fps = parseInt(v);
  document.getElementById('fps-val').textContent = v;
  save();
  if (scanner) { scanner.stop().then(() => { scanner = null; startScanner(); }).catch(() => {}); }
};
window.toggleSetting = function(key) {
  S.settings[key] = !S.settings[key];
  applySW(key === 'sound' ? 'sw-sound' : 'sw-vibrate', S.settings[key]);
  save();
};

/* ═══════════════════════════════════════════
   FAST MODE / HUNT
═══════════════════════════════════════════ */
window.toggleFast = function() {
  S.fastMode = !S.fastMode;
  applyFastUI(); save();
};
function applyFastUI() {
  const sw = document.getElementById('fast-sw');
  const row = document.getElementById('fast-row');
  if (S.fastMode) { sw.classList.add('on'); row.classList.add('on'); }
  else { sw.classList.remove('on'); row.classList.remove('on'); }
}

window.toggleHunt = function() {
  const inp = document.getElementById('hunt-id');
  const btn = document.getElementById('hunt-btn');
  const badge = document.getElementById('hunt-badge');
  if (S.hunt.active) {
    S.hunt = { active: false, target: null };
    inp.disabled = false; inp.value = '';
    btn.textContent = 'Ativar'; btn.className = 'btn-sm btn-sm-primary';
    badge.classList.add('hidden');
  } else {
    const v = inp.value.trim();
    if (!v) { inp.focus(); return; }
    S.hunt = { active: true, target: v };
    inp.disabled = true;
    btn.textContent = 'Parar'; btn.className = 'btn-sm btn-sm-ghost';
    badge.classList.remove('hidden');
  }
  save();
};

/* ═══════════════════════════════════════════
   ZONES
═══════════════════════════════════════════ */
function renderZones() {
  const c = document.getElementById('zones-list');
  c.innerHTML = `
    <div class="zone-row" onclick="setZone(null)" style="margin-bottom:8px;cursor:pointer">
      <div><div class="zone-name">Sem zona ativa</div></div>
      <button class="btn-sm ${!S.activeZone ? 'btn-sm-green' : 'btn-sm-ghost'}">${!S.activeZone ? 'ATIVA' : 'ATIVAR'}</button>
    </div>`;
  S.zones.forEach(z => {
    const zid = z.toLowerCase();
    const isActive = S.activeZone === zid;
    c.innerHTML += `
      <div class="zone-row ${isActive ? 'active' : ''}" style="cursor:pointer" onclick="setZone('${zid}')">
        <div><div class="zone-name">${z}</div></div>
        <button class="btn-sm ${isActive ? 'btn-sm-green' : 'btn-sm-primary'}">${isActive ? 'ATIVA' : 'ATIVAR'}</button>
      </div>`;
  });
}
function setZone(id) {
  S.activeZone = id;
  setZoneChip();
  save(); renderZones();
}
function setZoneChip() {
  document.getElementById('status-zone').textContent = S.activeZone ? S.activeZone.toUpperCase() : 'SEM ZONA';
}

/* ═══════════════════════════════════════════
   INVENTORY
═══════════════════════════════════════════ */
function renderInventory() {
  const c = document.getElementById('inventory-list');
  c.innerHTML = '';
  S.zones.forEach(z => {
    const zid = z.toLowerCase();
    const cnt = (S.zoneData[zid] || []).length;
    c.innerHTML += `
      <div class="zone-row" style="margin-bottom:8px">
        <div>
          <div class="zone-name">${z}</div>
          <div class="zone-count">${cnt} IDs</div>
        </div>
        <button class="btn-sm btn-sm-primary" onclick="loadInvZone('${zid}')">
          <i class="ri-upload-cloud-line"></i> Carregar
        </button>
      </div>`;
  });
}
window.loadInvZone = function(zid) {
  tempInvZone = zid;
  document.getElementById('inv-file-input').click();
};

/* ═══════════════════════════════════════════
   DASHBOARD
═══════════════════════════════════════════ */
function updateDash() {
  const total = S.logs.length;
  const ok = S.logs.filter(l => l.status === 'SUCESSO').length;
  const miss = S.logs.filter(l => l.status === 'MISSORT').length;
  const acc = total > 0 ? Math.round((ok / total) * 100) : 100;
  const recent = S.logs.filter(l => Date.now() - new Date(l.time) < 600000).length;
  const sph = recent * 6;
  const okLogs = S.logs.filter(l => l.status === 'SUCESSO').sort((a,b) => new Date(a.time)-new Date(b.time));
  let avg = 0;
  if (okLogs.length > 1) {
    const diffs = okLogs.slice(1).map((x,i) => (new Date(x.time)-new Date(okLogs[i].time))/1000);
    avg = Math.round(diffs.reduce((a,b)=>a+b,0)/diffs.length);
  }
  document.getElementById('kpi-sph').textContent = sph;
  document.getElementById('kpi-acc').textContent = acc + '%';
  document.getElementById('kpi-time').textContent = avg + 's';
  document.getElementById('kpi-miss').textContent = miss;
}

function downloadExcel() {
  if (!S.logs.length) return alert('Sem dados para exportar.');
  const ws = XLSX.utils.json_to_sheet(S.logs);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Logs');
  XLSX.writeFile(wb, 'natefy_relatorio.xlsx');
}

/* ═══════════════════════════════════════════
   LOG
═══════════════════════════════════════════ */
function renderLog() {
  const c = document.getElementById('log-list');
  if (!S.logs.length) {
    c.innerHTML = '<div class="empty"><i class="ri-inbox-line"></i><p>Nenhuma bipagem ainda</p></div>'; return;
  }
  c.innerHTML = S.logs.slice(0, 60).map(l => `
    <div class="log-item">
      <div>
        <div class="log-id">${l.id}</div>
        <div class="log-desc">${l.desc || l.zone || '—'}</div>
      </div>
      <div style="text-align:right">
        <div class="badge ${l.type || 'err'}">${l.status}</div>
        <div style="font-size:10px;color:var(--text-3);margin-top:4px;font-family:var(--mono)">${new Date(l.time).toLocaleTimeString()}</div>
      </div>
    </div>`).join('');
}

/* ═══════════════════════════════════════════
   PROFILE
═══════════════════════════════════════════ */
function refreshProfile() {
  document.getElementById('p-today').textContent = S.logs.length;
  document.getElementById('p-total').textContent = S.totalLife;
}

/* ═══════════════════════════════════════════
   CLEAR SESSION
═══════════════════════════════════════════ */
window.clearSession = function() {
  if (!confirm('Limpar a sessão atual?')) return;
  S.logs = []; S.found = []; S.idsToFind = []; S.idDescs = {};
  S.lastUndo = null;
  save(); updateFileStatus();
};

/* ═══════════════════════════════════════════
   NAV / SHEET
═══════════════════════════════════════════ */
window.toggleSheet = function(forceOpen) {
  const el = document.getElementById('sheet');
  const open = forceOpen !== undefined ? forceOpen : !el.classList.contains('open');
  el.classList.toggle('open', open);
};

window.navTap = function(tab, el) {
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  el.classList.add('active');
  if (tab === 'scan') {
    toggleSheet(!document.getElementById('sheet').classList.contains('open'));
  } else {
    toggleSheet(true);
  }
  switchTab(tab);
};

window.openTab = function(tab) {
  toggleSheet(true);
  switchTab(tab);
};

window.switchTab = function(tab, btn) {
  if (btn) {
    document.querySelectorAll('.tab-pill').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
  } else {
    document.querySelectorAll('.tab-pill').forEach(b => {
      b.classList.toggle('active', b.textContent.toLowerCase().includes(tab.substring(0,3)));
    });
  }
  document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
  document.getElementById('view-' + tab).classList.add('active');

  if (tab === 'dash') updateDash();
  if (tab === 'log') renderLog();
  if (tab === 'zonas') renderZones();
  if (tab === 'listas') renderInventory();
  if (tab === 'perfil') refreshProfile();
};

/* ═══════════════════════════════════════════
   N8N
═══════════════════════════════════════════ */
function sendN8n(data) {
  fetch(WEBHOOK, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  }).catch(() => {});
}
</script>
</body>
</html>
