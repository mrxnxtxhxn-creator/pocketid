<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Natefy Pro - Desktop Zebra</title>
    <meta name="theme-color" content="#080C14"/>

    <!-- BIBLIOTECAS (XLSX para planilhas e Ícones/Fontes) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/remixicon@3.5.0/fonts/remixicon.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg:         #080C14;
  --surface-0: #0D1421;
  --surface-1: #111827;
  --surface-2: #1A2336;
  --surface-3: #243049;
  --border:     rgba(255,255,255,0.07);
  --border-hi:  rgba(255,255,255,0.14);
  --accent:     #3B82F6;
  --accent-lo:  rgba(59,130,246,0.12);
  --accent-hi:  #60A5FA;
  --green:      #10B981;
  --green-lo:   rgba(16,185,129,0.12);
  --red:        #EF4444;
  --red-lo:     rgba(239,68,68,0.12);
  --yellow:     #F59E0B;
  --yellow-lo:  rgba(245,158,11,0.12);
  --text-1:     #F1F5F9;
  --text-2:     #94A3B8;
  --text-3:     #475569;
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

/* ── LAYOUT PRINCIPAL ── */
#app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

/* ── TOP BAR ── */
#top-bar {
  padding: 10px 20px;
  display: flex; justify-content: space-between; align-items: center;
  background: var(--surface-0);
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}
.tb-left { display: flex; align-items: center; gap: 12px; }
.tb-avatar {
  width: 36px; height: 36px; border-radius: 8px;
  background: var(--accent-lo); border: 1px solid var(--accent);
  display: flex; align-items: center; justify-content: center;
  color: var(--accent); font-size: 16px; cursor: pointer;
}
.tb-op { font-size: 13px; font-weight: 600; }
.tb-zone {
  font-family: var(--mono); font-size: 10px; font-weight: 600;
  color: var(--accent-hi); background: var(--accent-lo);
  border: 1px solid rgba(59,130,246,0.25);
  border-radius: 4px; padding: 2px 6px; margin-top: 1px; display: inline-block;
}
.tb-right { display: flex; align-items: center; gap: 10px; }
.zebra-badge {
  display: flex; align-items: center; gap: 8px;
  background: var(--surface-2); border: 1px solid var(--border-hi);
  padding: 5px 10px; border-radius: 20px; font-size: 11px; font-family: var(--mono);
}
.dot-live {
  width: 8px; height: 8px; border-radius: 50%; background: var(--green);
  box-shadow: 0 0 8px var(--green); animation: pulse 2s infinite;
}
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.4} }

/* ── TELA CENTRAL DE BIPAGEM ZEBRA ── */
#zebra-viewport {
  flex: 1 1 auto;
  min-height: 180px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle at center, rgba(59,130,246,0.05) 0%, transparent 70%);
  padding: 16px;
  position: relative;
  overflow: hidden;
}

.scanner-box {
  background: var(--surface-1);
  border: 2px dashed var(--accent);
  border-radius: 18px;
  padding: 20px 30px;
  text-align: center;
  max-width: 460px;
  width: 100%;
  box-shadow: 0 10px 30px rgba(0,0,0,0.5);
  transition: all 0.2s;
}
.scanner-box.active {
  border-style: solid;
  border-color: var(--green);
  box-shadow: 0 0 20px rgba(16,185,129,0.3);
}
.scanner-box.warn {
  border-style: solid;
  border-color: var(--yellow);
  box-shadow: 0 0 20px rgba(245,158,11,0.35);
}
.scanner-box.error {
  border-style: solid;
  border-color: var(--red);
  box-shadow: 0 0 20px rgba(239,68,68,0.35);
}

.scanner-icon {
  font-size: 36px;
  color: var(--accent);
  margin-bottom: 6px;
}

.last-scan-display {
  font-family: var(--mono);
  font-size: 22px;
  font-weight: 700;
  color: var(--green);
  margin-top: 6px;
  word-break: break-all;
  min-height: 30px;
}

/* ── PAINEL DE CONTROLE / TABS ── */
#main-panel {
  flex: 1 1 50%;
  min-height: 250px;
  max-height: 55vh;
  background: var(--surface-1);
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.tab-row {
  display: flex; gap: 6px; padding: 10px 16px;
  border-bottom: 1px solid var(--border);
  background: var(--surface-0);
  overflow-x: auto;
  flex-shrink: 0;
}
.tab-pill {
  padding: 6px 14px; border-radius: 100px; font-size: 12px; font-weight: 600;
  color: var(--text-2); cursor: pointer; border: 1px solid var(--border);
  background: transparent; transition: all .15s; white-space: nowrap;
}
.tab-pill.active {
  background: var(--accent); color: white; border-color: var(--accent);
}

.tab-content-container {
  flex: 1; overflow-y: auto; padding: 16px 20px;
}
.tab-content { display: none; }
.tab-content.active { display: block; animation: fadeUp .2s ease; }
@keyframes fadeUp { from{opacity:0;transform:translateY(6px)} to{opacity:1;transform:translateY(0)} }

/* ── COMPONENTES & FORMULÁRIOS ── */
.field {
  background: var(--surface-0); border: 1px solid var(--border);
  color: var(--text-1); font-family: var(--sans); font-size: 13px;
  padding: 10px 12px; border-radius: 10px; outline: none; width: 100%;
}
.field:focus { border-color: var(--accent); }

.btn { 
  border-radius: 10px; font-weight: 600; font-size: 13px;
  cursor: pointer; display: inline-flex; align-items: center; justify-content: center;
  gap: 8px; border: none; padding: 10px 16px; transition: all .15s;
}
.btn-primary { background: var(--accent); color: white; }
.btn-primary:hover { background: #2563EB; }
.btn-ghost { background: var(--surface-2); color: var(--text-2); border: 1px solid var(--border); }
.btn-danger { background: var(--red-lo); color: var(--red); border: 1px solid rgba(239,68,68,0.2); }

.card-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 12px; }
.upload-card {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 12px; padding: 14px; display: flex; flex-direction: column;
  align-items: center; gap: 6px; cursor: pointer; transition: all .15s;
}
.upload-card:hover { border-color: var(--accent); background: var(--accent-lo); }

/* KPI Grid Ajustado para Notebook */
.kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 10px; margin-bottom: 14px; }
.kpi-card { background: var(--surface-0); border: 1px solid var(--border); border-radius: 12px; padding: 12px; }
.kpi-label { font-size: 10px; font-weight: 600; color: var(--text-3); text-transform: uppercase; margin-bottom: 4px; }
.kpi-val { font-family: var(--mono); font-size: 22px; font-weight: 700; }
.kpi-val.blue { color: var(--accent-hi); }
.kpi-val.green { color: var(--green); }
.kpi-val.purple { color: #A78BFA; }
.kpi-val.red { color: var(--red); }

/* Listas / Inventário */
.inv-summary { display: flex; gap: 14px; margin-bottom: 12px; font-size: 12px; font-family: var(--mono); flex-wrap: wrap; }
.inv-section-title { font-size: 11px; font-weight: 700; color: var(--text-2); text-transform: uppercase; letter-spacing: .04em; margin: 12px 0 6px; }
.inv-section-title:first-of-type { margin-top: 0; }
.status-chip { font-size: 10px; font-weight: 700; font-family: var(--mono); padding: 2px 6px; border-radius: 4px; }
.status-chip.ok { color: var(--green); background: var(--green-lo); }
.status-chip.pending { color: var(--yellow); background: var(--yellow-lo); }
.status-chip.missort { color: var(--red); background: var(--red-lo); }
.status-chip.duplicate { color: var(--yellow); background: var(--yellow-lo); }

/* Logs */
.log-item {
  background: var(--surface-0); border: 1px solid var(--border);
  border-radius: 8px; padding: 8px 12px; display: flex;
  justify-content: space-between; align-items: center; gap: 10px; margin-bottom: 6px;
}
.log-id { font-family: var(--mono); font-size: 13px; font-weight: 600; }

/* ── MEDIA QUERIES PARA NOTEBOOK / TELA REDUZIDA ── */
@media (max-height: 768px) {
  #top-bar { padding: 8px 16px; }
  #zebra-viewport { padding: 10px; min-height: 140px; }
  .scanner-box { padding: 14px 20px; max-width: 380px; }
  .scanner-icon { font-size: 28px; margin-bottom: 2px; }
  .scanner-box h3 { font-size: 14px !important; }
  .last-scan-display { font-size: 18px; min-height: 24px; margin-top: 2px; }
  .tab-row { padding: 8px 12px; }
  .tab-content-container { padding: 12px 16px; }
}

/* ── FEEDBACK TOAST ── */
#feedback {
  position: fixed; top: 60px; right: 20px; z-index: 100;
  pointer-events: none; opacity: 0; transition: opacity .2s;
}
.fb-pill {
  background: var(--surface-2); border: 1px solid var(--accent);
  border-radius: 12px; padding: 10px 18px; box-shadow: 0 10px 25px rgba(0,0,0,0.5);
  text-align: center;
}
.fb-pill.fb-warn { border-color: var(--yellow); }
.fb-pill.fb-error { border-color: var(--red); }
.fb-status { font-size: 10px; font-weight: 700; color: var(--accent-hi); font-family: var(--mono); }
.fb-pill.fb-warn .fb-status { color: var(--yellow); }
.fb-pill.fb-error .fb-status { color: var(--red); }
.fb-id { font-family: var(--mono); font-size: 16px; font-weight: 700; margin-top: 2px; }

/* ── LOGIN SCREEN ── */
#login-screen {
  position: fixed; inset: 0; z-index: 200; background: var(--bg);
  display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 20px;
}
.login-box {
  background: var(--surface-1); border: 1px solid var(--border);
  padding: 30px; border-radius: 20px; width: 100%; max-width: 360px; text-align: center;
}
.hidden { display: none !important; }
</style>
</head>
<body>

<!-- LOGIN SCREEN -->
<div id="login-screen">
  <div class="login-box">
    <div style="font-size:36px; color:var(--accent); margin-bottom:8px"><i class="ri-barcode-box-line"></i></div>
    <h2 style="font-size:20px; margin-bottom:4px">Natefy Pro</h2>
    <p style="font-size:12px; color:var(--text-2); margin-bottom:20px">Estação de Bipagem Zebra DSS22</p>
    <div style="display:flex; flex-direction:column; gap:10px">
      <input type="text" id="op-input" class="field" style="text-align:center;" placeholder="Nome do Operador">
      <button class="btn btn-primary" onclick="doLogin()">ENTRAR <i class="ri-arrow-right-line"></i></button>
    </div>
  </div>
</div>

<!-- APPLICATION LAYOUT -->
<div id="app-container">

  <!-- TOP BAR -->
  <div id="top-bar">
    <div class="tb-left">
      <div class="tb-avatar" onclick="switchTab('perfil')"><i class="ri-user-3-line"></i></div>
      <div>
        <div class="tb-op" id="status-operator">Operador</div>
        <div class="tb-zone" id="status-zone">SEM ZONA</div>
      </div>
    </div>
    <div class="tb-right">
      <div class="zebra-badge">
        <div class="dot-live"></div>
        <span>ZEBRA DSS22 PRONTO</span>
      </div>
    </div>
  </div>

  <!-- ZEBRA MAIN VIEWPORT -->
  <div id="zebra-viewport">
    <div class="scanner-box" id="scan-box">
      <i class="ri-barcode-line scanner-icon"></i>
      <h3 style="font-size:15px; font-weight:600">Aguardando Bipagem</h3>
      <p style="font-size:11px; color:var(--text-3); margin-top:2px">Use o leitor físico Zebra DSS22 a qualquer momento</p>
      <div class="last-scan-display" id="last-scan-display">---</div>
    </div>
  </div>

  <!-- PANELS & TABS -->
  <div id="main-panel">
    <div class="tab-row">
      <button class="tab-pill active" onclick="switchTab('scan', this)">Operação</button>
      <button class="tab-pill" onclick="switchTab('listas', this)">Listas</button>
      <button class="tab-pill" onclick="switchTab('zonas', this)">Zonas</button>
      <button class="tab-pill" onclick="switchTab('dash', this)">Dashboard KPI</button>
      <button class="tab-pill" onclick="switchTab('log', this)">Log Histórico</button>
      <button class="tab-pill" onclick="switchTab('perfil', this)">Perfil</button>
    </div>

    <div class="tab-content-container">

      <!-- SCAN / OPERAÇÃO TAB -->
      <div id="view-scan" class="tab-content active">
        <div class="card-grid" style="margin-bottom:12px">
          <div class="upload-card" onclick="document.getElementById('file-input').click()">
            <i class="ri-file-excel-2-fill" style="color:#10B981; font-size:24px"></i>
            <span style="font-size:12px; font-weight:600">Carregar Planilha Excel</span>
          </div>
          <div style="display:flex; gap:8px; align-items:center; background:var(--surface-0); padding:10px; border-radius:12px; border:1px solid var(--border);">
            <input type="text" id="manual-input" class="field" placeholder="Ou digite o ID manualmente...">
            <button class="btn btn-primary" onclick="scanManual()"><i class="ri-check-line"></i></button>
          </div>
        </div>

        <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">

        <div id="file-status" class="hidden" style="margin-bottom:10px; font-size:12px; color:var(--green)">
          <i class="ri-checkbox-circle-fill"></i> Lista ativa carregada: <span id="file-count">0 itens</span>
        </div>

        <button class="btn btn-danger" onclick="clearSession()">
          <i class="ri-delete-bin-6-line"></i> Limpar Sessão Atual
        </button>
      </div>

      <!-- LISTAS TAB -->
      <div id="view-listas" class="tab-content">
        <p style="font-size:12px; color:var(--text-2); margin-bottom:8px">Itens Pendentes vs Bipados:</p>
        <div id="inventory-list"></div>
      </div>

      <!-- ZONAS TAB -->
      <div id="view-zonas" class="tab-content">
        <p style="font-size:12px; color:var(--text-2); margin-bottom:8px">Selecione a Zona de Bipagem Atual:</p>
        <div id="zones-list" style="display:flex; gap:8px; flex-wrap:wrap"></div>
      </div>

      <!-- DASHBOARD TAB -->
      <div id="view-dash" class="tab-content">
        <div class="kpi-grid">
          <div class="kpi-card">
            <div class="kpi-label">Total Bipados</div>
            <div class="kpi-val blue" id="kpi-total">0</div>
          </div>
          <div class="kpi-card">
            <div class="kpi-label">Acuracidade</div>
            <div class="kpi-val green" id="kpi-acc">100%</div>
          </div>
          <div class="kpi-card">
            <div class="kpi-label">Produtividade/h</div>
            <div class="kpi-val purple" id="kpi-sph">0</div>
          </div>
          <div class="kpi-card">
            <div class="kpi-label">Missorts</div>
            <div class="kpi-val red" id="kpi-miss">0</div>
          </div>
        </div>
        <button class="btn btn-ghost" onclick="downloadExcel()">
          <i class="ri-file-excel-2-line" style="color:var(--green)"></i> Exportar Relatório Excel
        </button>
      </div>

      <!-- LOG TAB -->
      <div id="view-log" class="tab-content">
        <div id="log-list"></div>
      </div>

      <!-- PERFIL TAB -->
      <div id="view-perfil" class="tab-content">
        <h3 id="p-name" style="margin-bottom:2px; font-size:16px">—</h3>
        <p style="font-size:12px; color:var(--text-3); margin-bottom:14px">Operador Conectado</p>
        <button class="btn btn-danger" onclick="logout()"><i class="ri-logout-box-r-line"></i> Sair da Conta</button>
      </div>

    </div>
  </div>

</div>

<!-- FEEDBACK TOAST -->
<div id="feedback">
  <div class="fb-pill" id="fb-pill">
    <div class="fb-status" id="fb-status-label">CÓDIGO BIPADO</div>
    <div class="fb-id" id="fb-id">—</div>
  </div>
</div>

<script>
/* ═══════════════════════════════════════════
   ESTADO DA APLICAÇÃO
═══════════════════════════════════════════ */
const KEY = 'natefy_pro_desktop_v1';

let S = {
  operator: null,
  idsToFind: [],
  found: [],
  logs: [],
  activeZone: 'Sorting',
  zones: ['Buffered','Sorting','Fraude','Missort','Returns','Bulky'],
  startTime: Date.now()
};

let audioCtx = null;

/* ═══════════════════════════════════════════
   INICIALIZAÇÃO & PERSISTÊNCIA
═══════════════════════════════════════════ */
function save() {
  localStorage.setItem(KEY, JSON.stringify(S));
}

function load() {
  const raw = localStorage.getItem(KEY);
  if (!raw) return;
  const d = JSON.parse(raw);
  S = { ...S, ...d };
}

document.addEventListener('DOMContentLoaded', () => {
  load();
  if (S.operator) bootApp();

  document.getElementById('file-input').onchange = e => handleFile(e.target.files[0]);
  document.getElementById('manual-input').addEventListener('keydown', e => { 
    if (e.key === 'Enter') scanManual(); 
  });

  // Inicializa o Áudio do Navegador para os bippes
  document.body.addEventListener('click', () => {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }, { once: true });
});

function bootApp() {
  document.getElementById('login-screen').classList.add('hidden');
  document.getElementById('status-operator').textContent = S.operator;
  document.getElementById('p-name').textContent = S.operator;
  document.getElementById('status-zone').textContent = S.activeZone;
  renderZones();
  updateKPIs();
  renderLogs();
  renderInventoryList();
  if (S.idsToFind && S.idsToFind.length > 0) {
    document.getElementById('file-status').classList.remove('hidden');
    document.getElementById('file-count').textContent = `${S.idsToFind.length} itens`;
  }
}

/* ═══════════════════════════════════════════
   INTEGRAÇÃO COM LEITOR ZEBRA DSS22 (KEYBOARD WEDGE)
═══════════════════════════════════════════ */
let zebraBuffer = '';
let zebraTimer = null;

document.addEventListener('keydown', function(e) {
  const tag = e.target.tagName;
  if (tag === 'INPUT' || tag === 'TEXTAREA') return;

  if (e.key === 'Enter') {
    if (zebraBuffer.length > 1) {
      onScan(zebraBuffer.trim());
      zebraBuffer = '';
      e.preventDefault();
    }
    return;
  }

  if (e.key.length === 1) {
    zebraBuffer += e.key;
    clearTimeout(zebraTimer);
    zebraTimer = setTimeout(() => { zebraBuffer = ''; }, 50);
  }
});

/* ═══════════════════════════════════════════
   LÓGICA DE SCAN & BIPAGEM
═══════════════════════════════════════════ */
function onScan(rawId) {
  const id = rawId.trim();
  if (!id) return;

  const hasList = S.idsToFind && S.idsToFind.length > 0;
  const idsToFindSet = new Set(S.idsToFind);
  const alreadyFound = S.found.includes(id);
  const isUnexpected = hasList && !idsToFindSet.has(id);

  let status = 'ok';
  if (alreadyFound) status = 'duplicate';
  else if (isUnexpected) status = 'missort';

  playBeep(status);

  document.getElementById('last-scan-display').textContent = id;
  const scanBox = document.getElementById('scan-box');
  scanBox.classList.remove('active', 'warn', 'error');
  scanBox.classList.add(status === 'ok' ? 'active' : (status === 'duplicate' ? 'warn' : 'error'));
  setTimeout(() => scanBox.classList.remove('active', 'warn', 'error'), 450);

  showFeedback(id, status);

  if (!alreadyFound) S.found.push(id);

  S.logs.unshift({ id: id, time: new Date().toLocaleTimeString(), zone: S.activeZone, status: status });
  save();

  updateKPIs();
  renderLogs();
  renderInventoryList();
}

function scanManual() {
  const inp = document.getElementById('manual-input');
  if (inp.value) {
    onScan(inp.value);
    inp.value = '';
  }
}

/* ═══════════════════════════════════════════
   FEEDBACK VISUAL (TOAST)
═══════════════════════════════════════════ */
function showFeedback(id, status) {
  const pill = document.getElementById('fb-pill');
  const statusLabel = document.getElementById('fb-status-label');
  const idEl = document.getElementById('fb-id');

  idEl.textContent = id;
  pill.classList.remove('fb-warn', 'fb-error');

  if (status === 'duplicate') {
    pill.classList.add('fb-warn');
    statusLabel.textContent = 'JÁ BIPADO (DUPLICADO)';
  } else if (status === 'missort') {
    pill.classList.add('fb-error');
    statusLabel.textContent = 'FORA DA LISTA (MISSORT)';
  } else {
    statusLabel.textContent = 'CÓDIGO BIPADO';
  }

  const fb = document.getElementById('feedback');
  fb.style.opacity = '1';
  setTimeout(() => { fb.style.opacity = '0'; }, 1500);
}

/* ═══════════════════════════════════════════
   ÁUDIO & FEEDBACK
═══════════════════════════════════════════ */
function playBeep(status = 'ok') {
  if (!audioCtx) return;
  const tones = status === 'ok' ? [1000] : status === 'duplicate' ? [700, 700] : [420, 300];

  tones.forEach((freq, i) => {
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.type = 'sine';
    const start = audioCtx.currentTime + i * 0.13;
    osc.frequency.setValueAtTime(freq, start);
    gain.gain.setValueAtTime(0.1, start);
    osc.start(start);
    osc.stop(start + 0.1);
  });
}

/* ═══════════════════════════════════════════
   GERENCIAMENTO DE ARQUIVOS EXCEL
═══════════════════════════════════════════ */
function handleFile(file) {
  if (!file) return;
  const reader = new FileReader();
  reader.onload = e => {
    const data = new Uint8Array(e.target.result);
    const workbook = XLSX.read(data, { type: 'array' });
    const sheet = workbook.Sheets[workbook.SheetNames[0]];
    const json = XLSX.utils.sheet_to_json(sheet, { header: 1 });
    
    S.idsToFind = json.flat().map(x => String(x).trim()).filter(Boolean);
    save();

    document.getElementById('file-status').classList.remove('hidden');
    document.getElementById('file-count').textContent = `${S.idsToFind.length} itens`;

    updateKPIs();
    renderInventoryList();
  };
  reader.readAsArrayBuffer(file);
}

function downloadExcel() {
  if (S.logs.length === 0) { alert("Nenhum dado para exportar."); return; }
  const ws = XLSX.utils.json_to_sheet(S.logs);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Bipagens");
  XLSX.writeFile(wb, `Relatorio_Bipagem_${Date.now()}.xlsx`);
}

/* ═══════════════════════════════════════════
   INTERFACE & NAVEGAÇÃO
═══════════════════════════════════════════ */
function switchTab(tabName, el) {
  document.querySelectorAll('.tab-pill').forEach(b => b.classList.remove('active'));
  document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));

  if (el) el.classList.add('active');
  const target = document.getElementById(`view-${tabName}`);
  if (target) target.classList.add('active');
}

function renderZones() {
  const container = document.getElementById('zones-list');
  container.innerHTML = S.zones.map(z => `
    <button class="btn ${S.activeZone === z ? 'btn-primary' : 'btn-ghost'}" onclick="selectZone('${z}')">${z}</button>
  `).join('');
}

function selectZone(z) {
  S.activeZone = z;
  document.getElementById('status-zone').textContent = z;
  save();
  renderZones();
}

/* ═══════════════════════════════════════════
   RECONCILIAÇÃO: LISTA IMPORTADA x BIPADOS
═══════════════════════════════════════════ */
function renderInventoryList() {
  const container = document.getElementById('inventory-list');
  if (!container) return;

  if (!S.idsToFind || S.idsToFind.length === 0) {
    container.innerHTML = `<p style="font-size:12px; color:var(--text-3)">
      Nenhuma lista carregada ainda. Importe uma planilha na aba "Operação" para
      acompanhar pendentes, bipados e itens fora da lista.
    </p>`;
    return;
  }

  const foundSet = new Set(S.found);
  const idsToFindSet = new Set(S.idsToFind);

  const pending = S.idsToFind.filter(id => !foundSet.has(id));
  const matched = S.idsToFind.filter(id => foundSet.has(id));
  const extras = [...new Set(S.found)].filter(id => !idsToFindSet.has(id));

  let html = `<div class="inv-summary">
    <span style="color:var(--green)">Bipados: ${matched.length}/${S.idsToFind.length}</span>
    <span style="color:var(--yellow)">Pendentes: ${pending.length}</span>
    <span style="color:var(--red)">Fora da lista: ${extras.length}</span>
  </div>`;

  if (pending.length) {
    html += `<div class="inv-section-title">Pendentes</div>`;
    html += pending.map(id => `
      <div class="log-item">
        <span class="log-id">${id}</span>
        <span class="status-chip pending">PENDENTE</span>
      </div>`).join('');
  }

  if (matched.length) {
    html += `<div class="inv-section-title">Bipados</div>`;
    html += matched.map(id => `
      <div class="log-item">
        <span class="log-id">${id}</span>
        <span class="status-chip ok">OK</span>
      </div>`).join('');
  }

  if (extras.length) {
    html += `<div class="inv-section-title">Fora da lista (missort)</div>`;
    html += extras.map(id => `
      <div class="log-item">
        <span class="log-id">${id}</span>
        <span class="status-chip missort">MISSORT</span>
      </div>`).join('');
  }

  container.innerHTML = html;
}

function updateKPIs() {
  document.getElementById('kpi-total').textContent = S.logs.length;

  const hours = Math.max((Date.now() - S.startTime) / 3600000, 0.01);
  document.getElementById('kpi-sph').textContent = Math.round(S.logs.length / hours);

  const hasList = S.idsToFind && S.idsToFind.length > 0;
  if (hasList) {
    const foundSet = new Set(S.found);
    const idsToFindSet = new Set(S.idsToFind);
    const matchedCount = S.idsToFind.filter(id => foundSet.has(id)).length;
    const extrasCount = [...new Set(S.found)].filter(id => !idsToFindSet.has(id)).length;
    const acc = Math.round((matchedCount / S.idsToFind.length) * 100);

    document.getElementById('kpi-acc').textContent = acc + '%';
    document.getElementById('kpi-miss').textContent = extrasCount;
  } else {
    document.getElementById('kpi-acc').textContent = '100%';
    document.getElementById('kpi-miss').textContent = '0';
  }
}

function renderLogs() {
  const list = document.getElementById('log-list');
  list.innerHTML = S.logs.map(l => {
    const status = l.status || 'ok';
    const chipClass = status === 'ok' ? 'ok' : (status === 'duplicate' ? 'duplicate' : 'missort');
    const chipLabel = status === 'ok' ? 'OK' : (status === 'duplicate' ? 'DUPLICADO' : 'MISSORT');
    return `
    <div class="log-item">
      <span class="log-id">${l.id}</span>
      <span class="status-chip ${chipClass}">${chipLabel}</span>
      <span style="font-size:11px; color:var(--text-3)">${l.zone} - ${l.time}</span>
    </div>`;
  }).join('');
}

function doLogin() {
  const v = document.getElementById('op-input').value.trim();
  if (!v) return;
  S.operator = v;
  save();
  bootApp();
}

function logout() {
  if (!confirm('Deseja sair da conta?')) return;
  S.operator = null;
  save();
  location.reload();
}

function clearSession() {
  if (!confirm('Limpar histórico da sessão atual?')) return;
  S.found = [];
  S.logs = [];
  S.startTime = Date.now();
  save();
  updateKPIs();
  renderLogs();
  renderInventoryList();
  document.getElementById('last-scan-display').textContent = '---';
}
</script>
</body>
</html>

```
