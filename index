<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Scanner Pro</title> <meta name="theme-color" content="#0f172a"/>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link rel="manifest" href="manifest.json">

    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
    <script src='https://unpkg.com/tesseract.js@v2.1.0/dist/tesseract.min.js'></script>
    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    
    <style>
        html, body { height: 100%; overflow: hidden; font-family: 'Inter', sans-serif; background-color: #0f172a; }
        #reader__scan_region { border: 4px solid rgba(6, 182, 212, 0.5) !important; border-radius: 1.5rem; background: none !important; box-shadow: 0 0 20px rgba(6, 182, 212, 0.2); }
        #controls-panel { background: rgba(15, 23, 42, 0.98); border-top: 1px solid #334155; transform: translateY(calc(100% - 65px)); transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1); padding-bottom: env(safe-area-inset-bottom, 20px); z-index: 50; }
        #controls-panel.open { transform: translateY(0); }
        .tab-btn { border-bottom: 2px solid transparent; transition: all 0.2s; white-space: nowrap; opacity: 0.6; }
        .tab-active { border-color: #06b6d4; color: white; opacity: 1; }
        .scan-success { animation: pulse-green 0.5s ease-out; }
        #undo-btn { position: fixed; bottom: 140px; right: 20px; z-index: 60; background-color: #ef4444; color: white; width: 56px; height: 56px; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 15px rgba(0,0,0,0.5); transition: transform 0.2s, opacity 0.2s; opacity: 0; pointer-events: none; transform: scale(0.8); }
        #undo-btn.visible { opacity: 1; pointer-events: auto; transform: scale(1); }
        .conn-indicator { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 5px; }
        .conn-online { background-color: #22c55e; box-shadow: 0 0 5px #22c55e; }
        .conn-offline { background-color: #ef4444; box-shadow: 0 0 5px #ef4444; }
        .conn-syncing { background-color: #eab308; animation: blink 1s infinite; }
        @keyframes blink { 50% { opacity: 0.5; } }
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background-color: #0f172a; }
        #login-modal { position: fixed; inset: 0; z-index: 100; background-color: rgba(15, 23, 42, 0.98); display: flex; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        #login-modal.hidden { display: none; }
        #ocr-loading { position: fixed; inset: 0; z-index: 90; background-color: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #ocr-loading.hidden { display: none; }
    </style>
</head>
<body class="text-slate-200">

    <div class="fixed top-0 left-0 right-0 z-30 bg-slate-900/90 backdrop-blur-md border-b border-slate-700 p-2 flex justify-between items-center px-4 h-14">
        <div class="flex items-center gap-2">
            <div id="connection-status" class="conn-online"></div>
            <div class="flex flex-col">
                <span id="status-operator" class="text-xs font-bold text-white leading-tight">--</span>
                <span id="status-zone" class="text-[10px] text-cyan-400 leading-tight">ZONA: --</span>
            </div>
        </div>
        <div class="flex items-center gap-3">
             <span id="queue-counter" class="text-[10px] text-yellow-500 font-mono hidden"><i class="fas fa-cloud-upload-alt"></i> <span id="queue-count">0</span></span>
             <button id="refresh-global-btn" class="text-slate-400 hover:text-white"><i class="fas fa-sync-alt"></i></button>
        </div>
    </div>

    <div id="reader" class="fixed top-14 bottom-0 left-0 w-full z-0"></div>
    <div id="feedback-overlay" class="fixed inset-0 z-40 flex items-center justify-center p-6 text-white font-black opacity-0 pointer-events-none transition-opacity duration-200 bg-black/80 text-center"></div>
    <button id="undo-btn"><i class="fas fa-undo fa-lg"></i></button>

    <div id="ocr-loading" class="hidden">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-cyan-500 mb-4"></div>
        <p class="text-lg font-bold">Processando...</p>
    </div>

    <div id="login-modal">
        <div class="bg-slate-800 p-8 rounded-2xl shadow-2xl w-11/12 max-w-md text-center border border-slate-600">
            <div class="mb-6 text-cyan-500 text-6xl"><i class="fas fa-box-open"></i></div>
            <input type="text" id="operator-input" class="w-full bg-slate-700 text-white p-4 rounded-xl mb-4 border border-slate-600 focus:border-cyan-500 outline-none text-center text-lg" placeholder="Nome do Operador">
            <button id="login-btn" class="w-full bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 text-white font-bold py-4 px-4 rounded-xl transition-all transform active:scale-95">ENTRAR</button>
        </div>
    </div>

    <div id="controls-panel" class="fixed bottom-0 w-full shadow-[0_-10px_40px_rgba(0,0,0,0.5)]">
        <div id="panel-handle" class="w-full h-8 flex justify-center items-center cursor-pointer"><div class="w-12 h-1.5 bg-slate-600 rounded-full"></div></div>
        <div class="w-full max-w-2xl mx-auto px-4 pb-4">
             <div class="w-full overflow-x-auto pb-2 mb-2 scrollbar-hide">
                <div class="flex justify-start space-x-2">
                    <button data-view="procurar" class="tab-btn tab-active py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">🔍 Procurar</button>
                    <button data-view="encontrados" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📋 Global</button>
                    <button data-view="dashboard" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📊 Dash</button>
                    <button data-view="analisador" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">⚖️ Comp.</button>
                    <button data-view="zonas" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📦 Zonas</button>
                    <button data-view="excecoes" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">⚠️ Erros</button>
                </div>
            </div>

            <div data-view-content="procurar" class="block text-center">
                 <div class="grid grid-cols-2 gap-3 mb-4">
                     <button id="load-file-btn" class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-xs flex items-center justify-center gap-2"><i class="fas fa-file-excel text-green-500"></i> Lista (Excel)</button>
                     <button id="load-image-btn" class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-xs flex items-center justify-center gap-2"><i class="fas fa-camera text-cyan-500"></i> Lista (Foto)</button>
                 </div>
                <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv"><input type="file" id="image-input" class="hidden" accept="image/*">
                <p id="file-info" class="text-xs text-green-400 mt-1 min-h-[1.2rem] font-mono"></p>
                 <div class="mt-3 text-left border-t border-slate-700 pt-3 space-y-3">
                    <div class="p-3 bg-slate-800/50 rounded-xl border border-slate-700">
                        <div class="flex justify-between items-center mb-2"><label class="text-xs text-slate-300 font-bold uppercase">Modo Caça</label><span id="hunt-status" class="text-xs text-cyan-400 font-mono"></span></div>
                        <div class="flex gap-2"><input type="text" id="hunt-target-id" class="w-full bg-slate-900 text-white p-2 rounded-lg font-mono border border-slate-600 text-sm" placeholder="ID Alvo"><button id="hunt-toggle-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-lg text-sm">Ativar</button></div>
                    </div>
                    <div class="flex justify-between items-center bg-slate-800/50 p-3 rounded-xl border border-slate-700">
                        <span class="text-sm text-slate-300 font-medium">Modo Rápido</span><input type="checkbox" id="fast-mode-toggle" class="w-4 h-4 accent-cyan-500">
                    </div>
                    <div class="flex gap-2"><button id="clear-session-btn" class="flex-1 bg-red-900/50 hover:bg-red-900 text-red-200 font-bold py-2 px-4 rounded-lg text-xs border border-red-800">Limpar</button><button id="change-operator-btn" class="flex-1 bg-slate-700 hover:bg-slate-600 text-slate-300 font-bold py-2 px-4 rounded-lg text-xs">Sair</button></div>
                    <div class="flex gap-2 mt-1"><input type="text" id="manual-input" class="w-full bg-slate-700 p-3 rounded-xl font-mono text-white border border-slate-600 outline-none" placeholder="ID Manual..."><button id="manual-check-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-xl">OK</button></div>
                </div>
            </div>

            <div data-view-content="encontrados" class="hidden">
                <div class="flex justify-between items-center mb-2"><h3 class="font-bold text-white">Global</h3><button id="export-btn" class="text-xs bg-green-700 px-3 py-1 rounded text-white">Baixar</button></div>
                <div id="global-list" class="space-y-2 max-h-[50vh] overflow-y-auto pb-20"><p class="text-center text-slate-500 text-sm mt-4">Aguardando sincronização...</p></div>
            </div>

            <div data-view-content="dashboard" class="hidden text-center">
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700 relative"><canvas id="progressChart"></canvas><div class="absolute inset-0 flex items-center justify-center font-bold text-xl" id="progress-text">0%</div></div>
                    <div class="flex flex-col gap-2">
                        <div class="bg-slate-800 p-3 rounded-xl border border-slate-700"><span class="text-xs text-slate-400">SEUS BIPS</span><div id="kpi-my-bips" class="text-2xl font-bold text-cyan-400">0</div></div>
                         <div class="bg-slate-800 p-3 rounded-xl border border-slate-700"><span class="text-xs text-slate-400">GLOBAL</span><div id="kpi-global-bips" class="text-2xl font-bold text-white">0</div></div>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-6"><button id="download-full-report-btn" class="bg-green-800 text-green-100 p-3 rounded-xl text-xs font-bold">📊 Excel</button><button id="download-flowchart-btn" class="bg-purple-800 text-purple-100 p-3 rounded-xl text-xs font-bold">🖼️ Fluxo</button></div>
                <div id="zone-finds-container" class="space-y-2"></div>
            </div>

            <div data-view-content="analisador" class="hidden space-y-4">
                <div class="space-y-2"><select id="analysis-list-a" class="w-full bg-slate-700 p-2 rounded-lg text-white"><option value="">Lista A...</option></select></div>
                <div class="space-y-2"><select id="analysis-list-b" class="w-full bg-slate-700 p-2 rounded-lg text-white"><option value="">Lista B...</option></select></div>
                <button id="run-analysis-btn" class="w-full bg-blue-600 font-bold py-3 rounded-lg">Comparar</button>
                <div id="analysis-results-container" class="hidden grid grid-cols-3 gap-2 text-center text-xs"><div class="bg-slate-800 p-2 rounded"><span class="block text-xl font-bold text-green-400" id="result-ok">0</span>OK</div><div class="bg-slate-800 p-2 rounded"><span class="block text-xl font-bold text-orange-400" id="result-sobra">0</span>Sobra</div><div class="bg-slate-800 p-2 rounded"><span class="block text-xl font-bold text-red-400" id="result-faltantes">0</span>Falta</div></div>
            </div>
            <div data-view-content="inventario" class="hidden"><div id="inventory-zones-container" class="space-y-3 max-h-[60vh] overflow-y-auto pr-2"></div></div>
            <div data-view-content="excecoes" class="hidden"><div id="exceptions-list" class="space-y-2 text-xs font-mono"></div></div>
            <div data-view-content="log" class="hidden"><div id="local-log-list" class="space-y-2 text-xs"></div></div>
        </div>
    </div>
    <div id="flowchart-canvas-container"></div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- CONFIGURAÇÃO ---
            const APP_CONFIG = {
                WEBHOOK_WRITE: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9', // Mantenha sua URL aqui
                WEBHOOK_READ: '',  
                STORAGE_KEY: 'natefy_v3',
                SYNC_INTERVAL: 15000
            };

            let state = {
                operator: null, idsToFind: new Set(), idDescriptions: new Map(), inventoryZoneData: new Map(),
                foundIds: [], localLog: [], globalLog: [], offlineQueue: [],
                inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}],
                activeZone: null, huntMode: { isActive: false, targetId: null },
                isScanning: false, isFastMode: false, lastScanTime: 0,
                zoneFinds: new Map()
            };

            const controlsPanel = document.getElementById('controls-panel');
            const panelHandle = document.getElementById('panel-handle');

            function init() {
                loadState();
                checkLogin();
                setupUI();
                createCharts();
                updateUI();
            }

            // --- LOGIN ---
            function checkLogin() {
                if (!state.operator) document.getElementById('login-modal').classList.remove('hidden');
                else { document.getElementById('login-modal').classList.add('hidden'); updateStatusUI(); startScanner(); }
            }
            document.getElementById('login-btn').addEventListener('click', () => {
                const name = document.getElementById('operator-input').value.trim();
                if (name) { state.operator = name; saveState(); checkLogin(); } else alert("Nome obrigatório");
            });
            document.getElementById('change-operator-btn').addEventListener('click', () => {
                if(confirm("Sair?")) { state.operator = null; saveState(); window.location.reload(); }
            });

            // --- SCANNER ---
            function processScan(id) {
                if (state.huntMode.isActive) {
                    if (id === state.huntMode.targetId) {
                        feedback('success', id, 'ALVO ENCONTRADO!', true);
                        toggleHuntMode();
                    } return;
                }
                if (state.isPaused) return;
                if (state.isFastMode) { const now = Date.now(); if (now - state.lastScanTime < 500) return; state.lastScanTime = now; } 
                else { state.isPaused = true; }

                const desc = state.idDescriptions.get(id) || '';
                let status = 'ERRO', msg = 'NÃO ENCONTRADO', type = 'error';

                if (state.activeZone) {
                    for (const [zId, ids] of state.inventoryZoneData) {
                        if (zId !== state.activeZone && ids.has(id)) { status = 'MISSORT'; msg = `MISSORT (${zId})`; type = 'warning'; break; }
                    }
                }
                if (status === 'ERRO' && state.idsToFind.has(id)) { status = 'SUCESSO'; msg = desc || 'ENCONTRADO'; type = 'success'; state.idsToFind.delete(id); }
                
                const entry = { id, status, desc, time: new Date().toISOString(), operator: state.operator, zone: state.activeZone };
                state.localLog.unshift(entry);
                if(status === 'SUCESSO') state.foundIds.unshift(entry);
                
                saveState();
                feedback(type, id, msg);
                updateUI();
                
                if(APP_CONFIG.WEBHOOK_WRITE) fetch(APP_CONFIG.WEBHOOK_WRITE, { method: 'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(entry) }).catch(e=>console.log(e));
            }

            function feedback(type, id, msg, persistent) {
                const el = document.getElementById('feedback-overlay');
                const colors = { success: '#22c55e', error: '#ef4444', warning: '#f59e0b' };
                el.style.background = `radial-gradient(circle, ${colors[type] || colors.error}, transparent)`;
                el.innerHTML = `<div class="text-center"><div class="text-4xl font-black mb-2">${msg}</div><div class="text-xl font-mono bg-black/50 p-2 rounded">${id}</div></div>`;
                el.style.opacity = '1';
                if(navigator.vibrate) navigator.vibrate(200);
                setTimeout(() => { el.style.opacity = '0'; if(!state.isFastMode && !persistent) state.isPaused = false; }, persistent ? 2000 : 1000);
            }

            // --- UI ---
            function updateUI() {
                document.getElementById('found-count').innerText = state.foundIds.length;
                document.getElementById('exceptions-count').innerText = state.localLog.filter(l=>l.status==='ERRO').length;
                renderLog();
                updateDashboard();
            }
            
            function renderLog() {
                document.getElementById('scan-log-list').innerHTML = state.localLog.slice(0, 50).map(l => 
                    `<div class="bg-slate-800 p-2 rounded border border-slate-700 flex justify-between">
                        <div><span class="font-bold text-white">${l.id}</span> <span class="text-[10px] text-cyan-400">${l.status}</span></div>
                        <span class="text-[10px] text-slate-500">${new Date(l.time).toLocaleTimeString()}</span>
                    </div>`
                ).join('');
            }

            function updateDashboard() {
                const total = state.idsToFind.size + state.foundIds.length;
                const pct = total > 0 ? Math.round((state.foundIds.length/total)*100) : 0;
                document.getElementById('progress-text').innerText = pct + '%';
                if(state.chart) { state.chart.data.datasets[0].data = [pct, 100-pct]; state.chart.update(); }
            }

            // --- SISTEMA ---
            function startScanner() {
                if(!state.html5QrCode) {
                    state.html5QrCode = new Html5Qrcode("reader");
                    state.html5QrCode.start({facingMode:"environment"}, {fps:15, qrbox:250}, processScan).catch(e=>{});
                }
            }
            
            function saveState() {
                const s = { ...state, html5QrCode: null, audioContext: null, chart: null };
                s.idsToFind = Array.from(s.idsToFind);
                s.inventoryZoneData = Array.from(s.inventoryZoneData.entries()).map(([k,v]) => [k, Array.from(v)]);
                s.idDescriptions = Array.from(s.idDescriptions.entries());
                s.zoneFinds = Array.from(s.zoneFinds.entries());
                localStorage.setItem(APP_CONFIG.STORAGE_KEY, JSON.stringify(s));
            }
            
            function loadState() {
                const s = JSON.parse(localStorage.getItem(APP_CONFIG.STORAGE_KEY));
                if(s) {
                    state = { ...state, ...s };
                    state.idsToFind = new Set(s.idsToFind);
                    state.inventoryZoneData = new Map(s.inventoryZoneData.map(([k,v]) => [k, new Set(v)]));
                    state.idDescriptions = new Map(s.idDescriptions);
                    state.zoneFinds = new Map(s.zoneFinds);
                }
            }

            // --- EVENTOS UI ---
            function setupUI() {
                // Tabs
                document.querySelectorAll('.tab-btn').forEach(b => {
                    b.addEventListener('click', () => {
                        document.querySelectorAll('.tab-btn').forEach(x=>x.classList.replace('tab-active','tab-inactive'));
                        b.classList.replace('tab-inactive','tab-active');
                        document.querySelectorAll('[data-view-content]').forEach(d => d.classList.add('hidden'));
                        document.querySelector(`[data-view-content="${b.dataset.view}"]`).classList.remove('hidden');
                    });
                });
                
                // Zonas
                const zoneContainer = document.getElementById('inventory-zones-container');
                state.inventoryZones.forEach(z => {
                    const div = document.createElement('div');
                    div.className = "bg-slate-800 p-3 rounded flex justify-between items-center";
                    div.innerHTML = `<span class="text-sm font-bold">${z.name}</span> <button class="text-xs bg-blue-600 px-2 py-1 rounded" onclick="setZone('${z.id}')">Ativar</button>`;
                    zoneContainer.appendChild(div);
                });
                
                // Manuais
                document.getElementById('manual-check-btn').addEventListener('click', () => {
                    const v = document.getElementById('manual-input').value.trim();
                    if(v) { processScan(v); document.getElementById('manual-input').value=''; }
                });
                
                // Hunt Mode
                document.getElementById('hunt-toggle-btn').addEventListener('click', toggleHuntMode);
                document.getElementById('fast-mode-toggle').addEventListener('change', (e) => state.isFastMode = e.target.checked);

                // File & OCR
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', (e) => handleFileSelect(e, 'main'));
                document.getElementById('load-image-btn').addEventListener('click', () => document.getElementById('image-input').click());
                document.getElementById('image-input').addEventListener('change', handleImageUpload);

                // Export
                document.getElementById('download-full-report-btn').addEventListener('click', downloadFullReport);
                document.getElementById('download-flowchart-btn').addEventListener('click', downloadFlowchart);
            }

            function toggleHuntMode() {
                const input = document.getElementById('hunt-target-id');
                if(state.huntMode.isActive) {
                    state.huntMode = { isActive: false, targetId: null };
                    input.disabled = false; input.value = '';
                    document.getElementById('hunt-toggle-btn').innerText = 'Ativar';
                    document.getElementById('hunt-status').innerText = '';
                } else {
                    const val = input.value.trim();
                    if(!val) return alert("Digite um ID");
                    state.huntMode = { isActive: true, targetId: val };
                    input.disabled = true;
                    document.getElementById('hunt-toggle-btn').innerText = 'Cancelar';
                    document.getElementById('hunt-status').innerText = 'ATIVO';
                }
            }

            window.setZone = (zid) => {
                state.activeZone = zid;
                updateStatusUI();
                alert(`Zona ${zid} ativa!`);
            };

            function updateStatusUI() {
                document.getElementById('status-zone').innerText = `ZONA: ${state.activeZone || '--'}`;
                document.getElementById('status-operator').innerText = `OP: ${state.operator || '--'}`;
            }

            function createCharts() {
                const ctx = document.getElementById('progressChart');
                if(ctx) {
                    state.chart = new Chart(ctx, {
                        type: 'doughnut',
                        data: { datasets: [{ data: [0, 100], backgroundColor: ['#0ea5e9', '#334155'], borderWidth: 0 }] },
                        options: { cutout: '80%', plugins: { tooltip: { enabled: false } } }
                    });
                }
            }

            function initAudio() {
                if(!state.audioContext) state.audioContext = new (window.AudioContext||window.webkitAudioContext)();
            }

            // --- OCR & Arquivos ---
            async function handleImageUpload(e) {
                const file = e.target.files[0]; if(!file) return;
                document.getElementById('ocr-loading').classList.remove('hidden');
                try {
                    const worker = Tesseract.createWorker();
                    await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                    const { data: { text } } = await worker.recognize(file);
                    await worker.terminate();
                    const ids = new Set();
                    text.split('\n').forEach(line => {
                        const m = line.match(/(\d{8,14})/);
                        if(m) { ids.add(m[0]); let d = line.replace(m[0], '').trim(); if(d) state.idDescriptions.set(m[0], d); }
                    });
                    if(ids.size > 0) { state.idsToFind = ids; updateUI(); alert(`${ids.size} itens carregados.`); }
                } catch(err) { alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); }
            }

            function handleFileSelect(e, zid) {
                const f = e.target.files[0]; if(!f)return;
                const r = new FileReader();
                const proc = (ids, name) => {
                    const s = new Set(ids);
                    if(zid==='main') { state.idsToFind=s; document.getElementById('file-info').textContent=`"${name}" (${s.size})`; }
                    else { state.inventoryZoneData.set(zid, s); }
                    saveToLocalStorage(); updateUI();
                };
                if(f.name.endsWith('.xlsx')){ r.onload=ev=>{const d=new Uint8Array(ev.target.result), w=XLSX.read(d,{type:'array'}), ws=w.Sheets[w.SheetNames[0]], j=XLSX.utils.sheet_to_json(ws,{header:1}); proc(j.map(r=>String(r[0])).filter(i=>i&&i.trim()), f.name);}; r.readAsArrayBuffer(f); }
                else { r.onload=ev=>{ proc(ev.target.result.trim().split(/[\n\r,]+/).map(i=>i.trim()).filter(i=>i), f.name); }; r.readAsText(f); }
            }

            // --- Relatórios ---
            function downloadFullReport() {
                const data = state.localLog.map(l => ({ 'Data': l.time, 'ID': l.id, 'Status': l.status, 'Operador': l.operator }));
                const ws = XLSX.utils.json_to_sheet(data); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Log"); XLSX.writeFile(wb, "Log.xlsx");
            }
            async function downloadFlowchart() {
                const c = document.getElementById('flowchart-canvas-container');
                c.innerHTML = `<div style="background:#0f172a;padding:50px;color:white;text-align:center"><h1>RELATÓRIO</h1><p>${state.operator}</p><p>Bips: ${state.localLog.length}</p></div>`;
                const canvas = await html2canvas(c);
                const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click();
            }

            init();
        });
    </script>
</body>
</html>
