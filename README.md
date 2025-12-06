<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Direto</title>

    <meta name="theme-color" content="#0f172a"/>
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
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    
    <style>
        html, body { height: 100%; width: 100%; overflow: hidden; font-family: 'Inter', sans-serif; background-color: #000; }
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; object-fit: cover; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }
        .scan-overlay { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 80vw; height: 25vh; border: 2px solid rgba(255, 255, 255, 0.5); border-radius: 20px; box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.5); z-index: 10; pointer-events: none; }
        .scan-line { width: 100%; height: 2px; background: #06b6d4; box-shadow: 0 0 4px #06b6d4; animation: scanMove 2s infinite linear; }
        @keyframes scanMove { 0% { transform: translateY(0); } 50% { transform: translateY(25vh); } 100% { transform: translateY(0); } }
        #bottom-nav { position: absolute; bottom: 0; left: 0; width: 100%; height: 70px; background: rgba(15, 23, 42, 0.95); backdrop-filter: blur(10px); border-top: 1px solid #334155; display: flex; justify-content: space-around; align-items: center; z-index: 50; padding-bottom: env(safe-area-inset-bottom, 10px); }
        .nav-item { display: flex; flex-direction: column; align-items: center; justify-content: center; color: #94a3b8; font-size: 10px; gap: 4px; width: 20%; }
        .nav-item.active { color: #22d3ee; }
        .content-panel { position: absolute; bottom: 80px; left: 0; width: 100%; max-height: 70vh; overflow-y: auto; background: transparent; z-index: 40; display: none; padding: 0 16px; }
        .content-panel.active { display: block; animation: slideUp 0.3s ease-out; }
        .panel-card { background: rgba(30, 41, 59, 0.95); backdrop-filter: blur(12px); border: 1px solid #475569; border-radius: 16px; padding: 16px; color: white; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
        @keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        #top-bar { position: absolute; top: 0; left: 0; width: 100%; height: 60px; background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent); z-index: 30; display: flex; justify-content: space-between; align-items: center; padding: 0 20px; padding-top: env(safe-area-inset-top, 10px); }
        #feedback-overlay { position: fixed; inset: 0; z-index: 60; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.85); }
        .scan-error { background: rgba(239, 68, 68, 0.85); }
        .scan-warning { background: rgba(234, 179, 8, 0.85); }
        #undo-btn { position: absolute; top: 80px; right: 20px; z-index: 40; background: #ef4444; color: white; width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 12px rgba(0,0,0,0.3); opacity: 0; pointer-events: none; transition: opacity 0.3s; }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }
        .hidden { display: none !important; }
        .btn-primary { background: linear-gradient(to right, #0891b2, #2563eb); padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; }
        .btn-secondary { background: #334155; padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; border: 1px solid #475569; }
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background-color: #0f172a; }
        #ocr-loading { position: fixed; inset: 0; z-index: 90; background-color: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #ocr-loading.hidden { display: none; }
    </style>
</head>
<body>

    <div id="top-bar">
        <div class="flex flex-col">
            <span id="status-operator" class="text-sm font-bold text-white shadow-black drop-shadow-md">Operador</span>
            <span id="status-zone" class="text-xs text-cyan-400 font-mono bg-black/40 px-2 rounded">ZONA: --</span>
        </div>
        <div class="flex gap-3">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_10px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay"><div class="scan-line"></div></div>

    <div id="feedback-overlay">
        <div class="text-center p-6">
            <div id="fb-icon" class="text-6xl mb-4">✅</div>
            <div id="fb-msg" class="text-4xl font-black text-white drop-shadow-md mb-2">ENCONTRADO</div>
            <div id="fb-desc" class="text-lg text-white/90 font-medium bg-black/20 p-2 rounded">Descrição aqui</div>
            <div id="fb-id" class="text-sm text-white/60 font-mono mt-2">ID: 123456</div>
        </div>
    </div>

    <button id="undo-btn"><i class="fas fa-undo"></i></button>

    <div id="ocr-loading" class="fixed inset-0 z-[70] bg-black/90 flex flex-col items-center justify-center hidden">
        <i class="fas fa-sync fa-spin text-5xl text-cyan-500 mb-4"></i>
        <p class="text-xl font-bold text-white">Lendo Imagem...</p>
    </div>

    <div id="view-procurar" class="content-panel active">
        <div class="panel-card space-y-4">
            <div class="grid grid-cols-2 gap-3">
                <button onclick="document.getElementById('file-input').click()" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                    <i class="fas fa-file-excel text-2xl text-green-500"></i><span class="text-xs font-bold">PLANILHA</span>
                </button>
                <button onclick="document.getElementById('image-input').click()" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                    <i class="fas fa-camera text-2xl text-cyan-500"></i><span class="text-xs font-bold">FOTO (OCR)</span>
                </button>
            </div>
            <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
            <input type="file" id="image-input" class="hidden" accept="image/*">
            <div id="file-status" class="bg-slate-900/50 p-2 rounded text-xs text-center text-slate-400">Nenhuma lista carregada</div>
            <div class="flex gap-2">
                <input type="text" id="manual-input" class="flex-1 bg-slate-900 border border-slate-600 rounded-lg px-4 py-3 text-sm text-white" placeholder="Digitar ID...">
                <button id="manual-btn" class="bg-cyan-600 px-5 rounded-lg font-bold"><i class="fas fa-check"></i></button>
            </div>
            <div class="flex items-center justify-between bg-slate-900/50 p-3 rounded-lg">
                <span class="text-sm text-slate-300">Modo Rápido</span>
                <input type="checkbox" id="fast-mode-toggle" class="w-5 h-5 accent-cyan-500">
            </div>
            <div class="p-3 bg-slate-800/50 rounded-xl border border-slate-700">
                <div class="flex justify-between items-center mb-2"><label class="text-xs text-slate-300 font-bold uppercase">Modo Caça</label><span id="hunt-status" class="text-xs text-cyan-400 font-mono"></span></div>
                <div class="flex gap-2"><input type="text" id="hunt-target-id" class="w-full bg-slate-900 text-white p-2 rounded-lg font-mono border border-slate-600 text-sm" placeholder="ID Alvo"><button id="hunt-toggle-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-lg text-sm">Ativar</button></div>
            </div>
             <div class="flex gap-2 mt-2">
                 <button onclick="clearSession()" class="w-full py-2 text-xs text-red-400 border border-red-900 rounded">Limpar Tudo</button>
             </div>
        </div>
    </div>

    <div id="view-dashboard" class="content-panel">
        <div class="panel-card space-y-4 text-center">
            <h2 class="text-lg font-bold text-white">Performance</h2>
            <div class="flex justify-center"><div class="relative w-32 h-32"><canvas id="progressChart"></canvas><div class="absolute inset-0 flex items-center justify-center text-2xl font-bold" id="progress-text">0%</div></div></div>
            <div class="grid grid-cols-2 gap-4">
                <div class="bg-slate-900 p-3 rounded-lg"><div class="text-2xl font-bold text-cyan-400" id="kpi-total">0</div><div class="text-[10px] text-slate-500 uppercase">Total Bipado</div></div>
                <div class="bg-slate-900 p-3 rounded-lg"><div class="text-2xl font-bold text-red-400" id="kpi-error">0</div><div class="text-[10px] text-slate-500 uppercase">Erros / Missort</div></div>
            </div>
             <button id="download-report-btn" class="btn-primary text-sm"><i class="fas fa-file-export mr-2"></i> Baixar Relatório Excel</button>
             <button id="download-flow-btn" class="btn-secondary text-sm"><i class="fas fa-image mr-2"></i> Baixar Fluxograma</button>
        </div>
    </div>

    <div id="view-log" class="content-panel">
        <div class="panel-card">
            <h2 class="text-lg font-bold text-white mb-2">Histórico do Dia</h2>
            <div id="scan-log-list" class="space-y-2 max-h-[50vh] overflow-y-auto"><div class="text-center text-slate-500 text-xs py-4">Vazio</div></div>
        </div>
    </div>
    
    <div id="view-zonas" class="content-panel">
         <div class="panel-card">
            <h2 class="text-lg font-bold text-white mb-2">Zonas</h2>
            <div id="zones-list" class="space-y-2 max-h-[50vh] overflow-y-auto"></div>
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="changeView('procurar', this)"><i class="fas fa-search"></i><span>Scan</span></div>
        <div class="nav-item" onclick="changeView('log', this)"><i class="fas fa-list"></i><span>Log</span></div>
        <div class="nav-item" onclick="changeView('dashboard', this)"><i class="fas fa-chart-pie"></i><span>Dash</span></div>
        <div class="nav-item" onclick="changeView('zonas', this)"><i class="fas fa-map-marker-alt"></i><span>Zonas</span></div>
    </nav>

    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = {
            STORAGE_KEY: 'natefy_pro_v6_nologin',
            WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9'
        };

        let state = {
            operator: 'Operador', // NOME PADRÃO
            idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null,
            inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}],
            zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null
        };

        document.addEventListener('DOMContentLoaded', () => {
            init();
        });

        function init() {
            loadState();
            // INICIA DIRETO
            setupActions();
            startScanner();
            updateUI();
        }

        // --- NAVEGAÇÃO DO PAINEL ---
        window.changeView = function(target, el) {
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            el.classList.add('active');
            document.querySelectorAll('.content-panel').forEach(v => v.classList.remove('active'));
            
            if (target !== 'procurar') {
                document.getElementById(`view-${target}`).classList.add('active');
                if(target === 'dashboard') updateDashboard();
                if(target === 'zonas') renderZones();
            }
        };

        window.setZone = function(id) {
            state.activeZone = id;
            document.getElementById('status-zone').innerText = `ZONA: ${id ? id.toUpperCase() : '--'}`;
            saveState();
            renderZones();
        };

        function setupActions() {
            document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]);
            document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]);
            document.getElementById('manual-btn').onclick = () => {
                const v = document.getElementById('manual-input').value.trim();
                if(v) { processScan(v); document.getElementById('manual-input').value=''; }
            };
            document.getElementById('fast-mode-toggle').onchange = (e) => state.isFastMode = e.target.checked;
            document.getElementById('undo-btn').onclick = doUndo;
            document.getElementById('download-report-btn').onclick = downloadExcel;
            document.getElementById('download-flow-btn').onclick = downloadFlow;
            document.getElementById('hunt-toggle-btn').onclick = toggleHuntMode;
        }

        // --- SCANNER ---
        function startScanner() {
            const html5QrCode = new Html5Qrcode("reader");
            html5QrCode.start({ facingMode: "environment" }, { fps: 15, qrbox: 250 }, 
                (decodedText) => processScan(decodedText)
            ).catch(err => console.log("Erro Câmera", err));
        }

        function processScan(id) {
            const now = Date.now();
            if (now - state.lastScanTime < 1000) return;
            state.lastScanTime = now;

            if (state.isPaused) return;
            if (!state.isFastMode) state.isPaused = true;

            // HUNT MODE
            if (state.huntMode && state.huntMode.isActive) {
                if (id.trim() === state.huntMode.targetId) {
                    showFeedback('success', 'ALVO ENCONTRADO!', '', id);
                    sendToN8n({ scannedId: id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: state.operator });
                    toggleHuntMode(); // Desliga
                    return;
                }
            }

            id = id.trim();
            const desc = state.idDescriptions.get(id) || "Sem descrição";
            let status = "NÃO ENCONTRADO";
            let type = "error";
            let feedbackMsg = "NÃO ENCONTRADO";

            if (state.activeZone) {
                for (const [zId, zSet] of state.zoneData) {
                    if (zId !== state.activeZone && zSet.has(id)) {
                        status = "MISSORT"; type = "warning"; feedbackMsg = `MISSORT (${zId})`; break;
                    }
                }
            }

            if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) {
                status = "DUPLICADO"; type = "warning"; feedbackMsg = "JÁ BIPADO";
            }

            if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) {
                status = "SUCESSO"; type = "success"; feedbackMsg = "ENCONTRADO";
                state.idsToFind.delete(id);
            }

            const entry = { id, status, desc, type, time: new Date().toISOString(), operator: state.operator, zone: state.activeZone };

            state.logs.unshift(entry);
            if (status === "SUCESSO") state.foundIds.unshift(entry);
            state.lastUndo = entry;

            saveState();
            showFeedback(type, feedbackMsg, desc, id);
            updateUI();
            sendToN8n(entry);
        }

        function showFeedback(type, msg, desc, id) {
            const overlay = document.getElementById('feedback-overlay');
            const icon = document.getElementById('fb-icon');
            const txt = document.getElementById('fb-msg');
            const dsc = document.getElementById('fb-desc');
            const idtxt = document.getElementById('fb-id');
            const undo = document.getElementById('undo-btn');

            overlay.className = `fixed inset-0 z-40 flex items-center justify-center p-6 text-white font-black transition-opacity duration-200 text-center scan-${type}`;
            icon.innerText = type === 'success' ? '✅' : (type === 'error' ? '❌' : '⚠️');
            txt.innerText = msg;
            dsc.innerText = desc;
            idtxt.innerText = id;

            overlay.style.opacity = '1';
            undo.classList.add('visible');
            setTimeout(() => undo.classList.remove('visible'), 5000);
            if(navigator.vibrate) navigator.vibrate(200);
            setTimeout(() => { overlay.style.opacity = '0'; if (!state.isFastMode) state.isPaused = false; }, state.isFastMode ? 500 : 1500);
        }

        // --- DATA ---
        function sendToN8n(data) {
            if(CONFIG.WEBHOOK.includes("https")) {
                fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline"));
            }
        }

        function saveState() {
            const s = { ...state, html5QrCode: null, chart: null };
            s.idsToFind = Array.from(s.idsToFind);
            s.idDescriptions = Array.from(s.idDescriptions.entries());
            s.inventoryZoneData = Array.from(s.inventoryZoneData.entries()).map(([k,v])=>[k,Array.from(v)]);
            localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s));
        }

        function loadState() {
            const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY));
            if(s) {
                state = { ...state, ...s };
                state.idsToFind = new Set(s.idsToFind);
                state.idDescriptions = new Map(s.idDescriptions);
                state.inventoryZoneData = new Map(s.inventoryZoneData.map(([k,v])=>[k,new Set(v)]));
            }
        }
        
        window.clearSession = function() { if(confirm('Apagar tudo?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } };

        // --- FILES ---
        function handleFile(file) {
            if(!file) return;
            const reader = new FileReader();
            reader.onload = (e) => {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const sheet = workbook.Sheets[workbook.SheetNames[0]];
                const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                const ids = json.map(r => String(r[0])).filter(i => i && i.match(/\d+/));
                state.idsToFind = new Set(ids);
                state.foundIds = [];
                document.getElementById('file-status').innerText = `📂 Excel: ${ids.length} itens`;
                document.getElementById('file-status').className = "bg-green-900/50 p-2 rounded text-xs text-center text-green-400";
                saveState();
                alert(`${ids.length} itens carregados.`);
            };
            reader.readAsArrayBuffer(file);
        }

        async function handleOCR(file) {
            if (!file) return;
            document.getElementById('ocr-loading').classList.remove('hidden');
            try {
                const worker = Tesseract.createWorker();
                await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                const { data: { text } } = await worker.recognize(file);
                await worker.terminate();
                const lines = text.split('\n');
                let count = 0;
                const newIds = new Set();
                lines.forEach(line => {
                    const match = line.replace(/[^\w\s\>\-\.\(\)\/]/gi, '').match(/(\d{8,14})/);
                    if (match) {
                        const id = match[0];
                        let desc = line.replace(id, '').replace(/^[\s\>\-\.]+/g, '').trim();
                        if (desc.length < 3) desc = "Produto sem descrição";
                        newIds.add(id);
                        state.idDescriptions.set(id, desc);
                        count++;
                    }
                });
                if (count > 0) {
                    state.idsToFind = newIds;
                    state.foundIds = [];
                    document.getElementById('file-status').innerText = `📷 Foto: ${count} itens`;
                    document.getElementById('file-status').className = "bg-green-900/50 p-2 rounded text-xs text-center text-green-400";
                    alert(`${count} itens lidos da foto!`);
                    saveState();
                } else { alert("Não encontrei IDs válidos."); }
            } catch (e) { alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); }
        }

        function doUndo() {
            if(!state.lastUndo) return;
            state.logs.shift();
            if (state.lastUndo.status === "SUCESSO") { state.foundIds.shift(); state.idsToFind.add(state.lastUndo.id); }
            sendToN8n({ ...state.lastUndo, status: "CANCELADO" });
            state.lastUndo = null;
            document.getElementById('undo-btn').classList.remove('visible');
            updateUI();
            alert("Desfeito!");
        }
        
        function toggleHuntMode() {
            const t = document.getElementById('hunt-target-id');
            if(state.huntMode && state.huntMode.isActive) {
                state.huntMode={isActive:false, targetId:null}; t.disabled=false; t.value='';
                document.getElementById('hunt-toggle-btn').innerText='Ativar';
                document.getElementById('hunt-status').innerText = '';
                if(state.html5QrCode && state.html5QrCode.getState()===2) state.html5QrCode.resume();
            } else {
                const v = t.value.trim(); if(!v) return alert("ID?");
                state.huntMode={isActive:true, targetId:v}; t.disabled=true;
                document.getElementById('hunt-toggle-btn').innerText='Cancelar';
                document.getElementById('hunt-status').innerText = 'ATIVO';
                if(state.html5QrCode && state.html5QrCode.getState()===1) state.html5QrCode.pause(true);
            }
            saveState();
        }

        // --- UI UPDATES ---
        function updateUI() {
            const list = document.getElementById('scan-log-list');
            list.innerHTML = state.logs.slice(0, 50).map(l => `
                <div class="bg-slate-800 p-3 rounded-lg border border-slate-700 flex justify-between items-center mb-2">
                    <div><div class="font-bold text-white text-sm">${l.id}</div><div class="text-[10px] text-slate-400 truncate w-40">${l.desc}</div></div>
                    <div class="text-right"><div class="text-[10px] font-bold ${l.status==='SUCESSO'?'text-green-400':(l.status==='ERRO'?'text-red-400':'text-yellow-400')}">${l.status}</div><div class="text-[10px] text-slate-500">${new Date(l.time).toLocaleTimeString()}</div></div>
                </div>`).join('');
        }

        function renderZones() {
            const c = document.getElementById('zones-list');
            c.innerHTML = `<button class="w-full bg-slate-700 py-3 rounded-lg mb-3 text-sm" onclick="window.setZone(null)">🚫 Sair da Zona</button>`;
            state.inventoryZones.forEach(z => {
                const isActive = state.activeZone === z.id;
                c.innerHTML += `<div class="bg-slate-800 p-4 rounded-xl flex justify-between items-center mb-2 border ${isActive ? 'border-green-500' : 'border-slate-700'}"><span class="font-bold text-white">${z.name}</span><button class="px-4 py-2 rounded-lg text-xs font-bold ${isActive ? 'bg-green-600 text-white' : 'bg-blue-600 text-white'}" onclick="window.setZone('${z.id}')">${isActive ? 'ATIVA' : 'ATIVAR'}</button></div>`;
            });
        }

        function updateDashboard() {
            const total = state.idsToFind.size + state.foundIds.length;
            const pct = total > 0 ? Math.round((state.foundIds.length / total) * 100) : 0;
            const ctx = document.getElementById('progressChart');
            if (ctx) {
                if(state.chart) state.chart.destroy();
                state.chart = new Chart(ctx, { type: 'doughnut', data: { datasets: [{ data: [pct, 100-pct], backgroundColor: ['#0ea5e9', '#1e293b'], borderWidth: 0 }] }, options: { cutout: '80%', plugins: { tooltip: { enabled: false } } } });
            }
            document.getElementById('progress-text').innerText = pct + "%";
            document.getElementById('kpi-total').innerText = state.logs.length;
            document.getElementById('kpi-error').innerText = state.logs.filter(l=>l.status==='ERRO' || l.status==='MISSORT').length;
        }

        function downloadExcel() {
            if(state.logs.length === 0) return alert("Sem dados");
            const ws = XLSX.utils.json_to_sheet(state.logs); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Logs"); XLSX.writeFile(wb, "Relatorio_Natefy.xlsx");
        }
        
        async function downloadFlow() {
             if(state.logs.length === 0) return alert("Sem dados");
             const container = document.getElementById('flowchart-canvas-container');
             container.innerHTML = `<div style="padding:50px; background:#0f172a; color:white; text-align:center"><h1>Relatório Visual</h1><h2>${state.operator}</h2><div style="display:flex; justify-content:center; gap:20px; margin-top:50px"><div style="border:2px solid green; padding:20px">SUCESSO: ${state.foundIds.length}</div><div style="border:2px solid red; padding:20px">ERROS: ${state.logs.filter(l=>l.status!=='SUCESSO').length}</div></div></div>`;
             const canvas = await html2canvas(container);
             const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click();
        }
    </script>
</body>
</html>
