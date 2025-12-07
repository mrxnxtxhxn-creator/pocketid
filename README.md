<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Analytics</title>

    <meta name="theme-color" content="#4F46E5"/>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link rel="manifest" href="manifest.json">

    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
    <script src='https://unpkg.com/tesseract.js@v2.1.0/dist/tesseract.min.js'></script>
    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/remixicon@3.5.0/fonts/remixicon.css" rel="stylesheet">
    <script src="https://cdn.tailwindcss.com"></script>

    <style>
        :root { --primary: #6366f1; --bg: #0f172a; --panel: #1e293b; }
        body, html { height: 100%; width: 100%; overflow: hidden; background: var(--bg); font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; }
        
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

        /* MIRA */
        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 80vw; height: 80vw; max-width: 400px; max-height: 400px;
            border: 2px solid rgba(255, 255, 255, 0.6); border-radius: 24px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.6); z-index: 10; pointer-events: none;
        }
        .scan-overlay::before, .scan-overlay::after { content: ''; position: absolute; width: 40px; height: 40px; border: 4px solid #6366f1; border-radius: 4px; pointer-events: none; }
        .scan-overlay::before { top: -2px; left: -2px; border-bottom: 0; border-right: 0; }
        .scan-overlay::after { bottom: -2px; right: -2px; border-top: 0; border-left: 0; }

        /* UI ELEMENTS */
        #top-bar { position: absolute; top: 0; left: 0; width: 100%; z-index: 20; padding: env(safe-area-inset-top) 20px 10px 20px; background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent); display: flex; justify-content: space-between; align-items: start; }
        
        #bottom-nav { position: absolute; bottom: 0; left: 0; width: 100%; height: 80px; background: rgba(30, 41, 59, 0.95); backdrop-filter: blur(10px); border-top: 1px solid rgba(255,255,255,0.1); display: flex; justify-content: space-around; align-items: center; padding-bottom: env(safe-area-inset-bottom); z-index: 50; }
        .nav-item { display: flex; flex-direction: column; align-items: center; justify-content: center; color: #94a3b8; font-size: 11px; gap: 4px; width: 20%; transition: all 0.2s; }
        .nav-item.active { color: #6366f1; }
        .nav-item i { font-size: 24px; }
        .nav-item.active i { transform: translateY(-2px); }

        #controls-panel { position: absolute; bottom: 80px; left: 0; width: 100%; background: var(--panel); border-radius: 24px 24px 0 0; transform: translateY(110%); transition: transform 0.3s cubic-bezier(0.2, 0.8, 0.2, 1); z-index: 40; max-height: 85vh; display: flex; flex-direction: column; box-shadow: 0 -5px 20px rgba(0,0,0,0.3); }
        #controls-panel.open { transform: translateY(0); }

        .tabs-container { display: flex; overflow-x: auto; padding: 8px 16px; gap: 8px; background: var(--panel); flex-shrink: 0; scrollbar-width: none; }
        .tab-btn { padding: 8px 16px; border-radius: 20px; font-size: 13px; font-weight: 500; color: #94a3b8; white-space: nowrap; transition: all 0.2s; }
        .tab-btn.active { background: var(--primary); color: white; box-shadow: 0 4px 10px rgba(99, 102, 241, 0.3); }
        
        .panel-content { padding: 20px; overflow-y: auto; flex: 1; }
        .tab-content { display: none; animation: fadeIn 0.3s; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .btn-card { background: #334155; border: 1px solid #475569; border-radius: 16px; padding: 16px; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; color: white; }
        .input-dark { background: #0f172a; border: 1px solid #475569; color: white; padding: 12px; border-radius: 12px; width: 100%; outline: none; }
        .btn-action { background: #6366f1; color: white; padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; }

        /* KPI Cards */
        .kpi-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
        .kpi-box { background: #0f172a; border-radius: 12px; padding: 15px; border-left: 4px solid; position: relative; overflow: hidden; }
        .kpi-label { font-size: 10px; color: #94a3b8; text-transform: uppercase; letter-spacing: 1px; font-weight: bold; margin-bottom: 4px; }
        .kpi-value { font-size: 24px; font-weight: 800; color: white; line-height: 1; }
        .kpi-sub { font-size: 10px; color: #64748b; margin-top: 4px; }

        #feedback-overlay { position: fixed; inset: 0; z-index: 60; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.9); }
        .scan-error { background: rgba(239, 68, 68, 0.9); }
        .scan-warning { background: rgba(234, 179, 8, 0.9); }
        
        .hidden { display: none !important; }
        #ocr-loading, #login-modal { position: fixed; inset: 0; z-index: 90; background: #0f172a; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        #undo-btn { position: absolute; top: 100px; right: 20px; z-index: 35; background: #ef4444; color: white; width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }
        #flowchart-canvas-container { position: absolute; left: -9999px; }
    </style>
</head>
<body>

    <div id="top-bar">
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-full bg-indigo-500 flex items-center justify-center text-white font-bold text-lg shadow-lg">
                <i class="ri-user-3-line"></i>
            </div>
            <div>
                <div id="status-operator" class="text-white font-bold leading-tight">Operador</div>
                <div id="status-zone" class="text-xs text-cyan-400 font-mono bg-black/40 px-2 rounded inline-block mt-0.5">Zona: --</div>
            </div>
        </div>
        <div class="flex gap-2">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_8px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay"></div>

    <div id="feedback-overlay">
        <div class="text-center p-6 bg-black/40 backdrop-blur-md rounded-2xl border border-white/10 shadow-2xl">
            <i id="fb-icon" class="ri-checkbox-circle-fill text-6xl text-white mb-2 block"></i>
            <div id="fb-msg" class="text-3xl font-black text-white mb-2">ENCONTRADO</div>
            <div id="fb-desc" class="text-sm text-gray-200 bg-white/10 p-2 rounded mb-1">Descrição</div>
            <div id="fb-id" class="text-xs font-mono text-gray-400">ID: 123</div>
        </div>
    </div>

    <button id="undo-btn"><i class="ri-arrow-go-back-line text-2xl"></i></button>

    <div id="ocr-loading" class="hidden">
        <i class="ri-loader-4-line text-6xl text-indigo-500 animate-spin mb-4"></i>
        <p class="text-xl font-bold text-white">Lendo Imagem...</p>
    </div>

    <div id="login-modal">
        <div class="w-full max-w-xs text-center p-6">
            <div class="w-20 h-20 bg-indigo-600 rounded-2xl mx-auto mb-6 flex items-center justify-center text-4xl text-white shadow-lg shadow-indigo-500/50">
                <i class="ri-box-3-fill"></i>
            </div>
            <h1 class="text-2xl font-bold text-white mb-2">Olá!</h1>
            <p class="text-slate-400 mb-8">Vamos começar a bipar?</p>
            <input type="text" id="operator-input" class="input-dark text-center text-lg mb-4" placeholder="Seu Nome">
            <button onclick="doLogin()" class="btn-action">ENTRAR</button>
        </div>
    </div>

    <div id="controls-panel">
        <div class="panel-handle w-full flex justify-center pt-3 pb-1" onclick="togglePanel()">
            <div class="w-12 h-1.5 bg-slate-600 rounded-full"></div>
        </div>

        <div class="tabs-container">
            <button class="tab-btn active" onclick="switchTab('procurar', this)">🔍 Scan</button>
            <button class="tab-btn" onclick="switchTab('dashboard', this)">📊 Analytics</button>
            <button class="tab-btn" onclick="switchTab('inventario', this)">📦 Listas</button>
            <button class="tab-btn" onclick="switchTab('zonas', this)">📍 Zonas</button>
            <button class="tab-btn" onclick="switchTab('log', this)">📝 Log</button>
        </div>

        <div class="panel-content">
            
            <div id="view-procurar" class="tab-content active">
                <div class="space-y-4">
                    <div class="grid grid-cols-2 gap-3">
                        <button onclick="document.getElementById('file-input').click()" class="btn-card">
                            <i class="ri-file-excel-2-fill text-2xl text-green-500"></i><span class="text-xs font-bold uppercase">Planilha</span>
                        </button>
                        <button onclick="document.getElementById('image-input').click()" class="btn-card">
                            <i class="ri-camera-fill text-2xl text-cyan-400"></i><span class="text-xs font-bold uppercase">Foto OCR</span>
                        </button>
                    </div>
                    <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                    <input type="file" id="image-input" class="hidden" accept="image/*">
                    
                    <div id="file-status" class="hidden bg-green-900/30 border border-green-500/30 p-3 rounded-xl flex items-center gap-3">
                        <i class="ri-check-double-line text-green-400 text-xl"></i>
                        <div><div class="text-xs text-green-400 font-bold uppercase">Carregado</div><div id="file-count" class="text-sm text-white">0 itens prontos</div></div>
                    </div>

                    <div class="flex gap-2">
                        <input type="text" id="manual-input" class="input-dark" placeholder="Digitar ID manual...">
                        <button id="manual-btn" class="bg-indigo-600 text-white px-4 rounded-xl"><i class="ri-check-line text-xl"></i></button>
                    </div>

                    <div class="btn-card flex-row justify-between py-3 cursor-pointer" onclick="toggleFastMode()" id="fast-mode-btn">
                        <span class="text-sm font-medium">Modo Rápido</span>
                        <div class="w-10 h-6 bg-slate-700 rounded-full relative"><div class="w-4 h-4 bg-white rounded-full absolute top-1 left-1 transition-all" id="fast-mode-dot"></div></div>
                    </div>

                    <div class="card bg-primary/10 border-primary/30 p-4 rounded-xl border">
                        <div class="flex justify-between items-center mb-2">
                            <label class="text-xs font-bold uppercase text-primary-light">Modo Caça</label>
                            <span id="hunt-status" class="text-xs font-mono text-primary-light"></span>
                        </div>
                        <div class="flex gap-2">
                            <input type="text" id="hunt-target-id" class="input-dark text-sm p-2" placeholder="ID Alvo">
                            <button id="hunt-toggle-btn" onclick="toggleHuntMode()" class="bg-indigo-600 px-4 rounded-lg text-xs font-bold text-white uppercase">Ativar</button>
                        </div>
                    </div>
                     <button onclick="clearSession()" class="w-full py-3 text-xs text-red-400 border border-red-900/30 rounded-xl mt-2">Limpar Sessão</button>
                </div>
            </div>

            <div id="view-dashboard" class="tab-content">
                <h2 class="text-lg font-bold text-white mb-4">Métricas de Hoje</h2>
                
                <div class="kpi-grid">
                    <div class="kpi-box border-blue-500">
                        <div class="kpi-label">Produtividade</div>
                        <div class="kpi-value" id="kpi-sph">0</div>
                        <div class="kpi-sub">Bips/Hora</div>
                    </div>
                    <div class="kpi-box border-green-500">
                        <div class="kpi-label">Acuracidade</div>
                        <div class="kpi-value text-green-400" id="kpi-acc">100%</div>
                        <div class="kpi-sub" id="kpi-found-total">0 Sucessos</div>
                    </div>
                    <div class="kpi-box border-purple-500">
                        <div class="kpi-label">Tempo Médio</div>
                        <div class="kpi-value text-purple-400" id="kpi-time">0s</div>
                        <div class="kpi-sub">Entre bips</div>
                    </div>
                    <div class="kpi-box border-red-500">
                        <div class="kpi-label">Missorts</div>
                        <div class="kpi-value text-red-400" id="kpi-missort">0</div>
                        <div class="kpi-sub text-red-500/70">Zonas Erradas</div>
                    </div>
                </div>

                <div class="bg-slate-900 p-4 rounded-xl border border-slate-700 mb-4">
                    <h3 class="text-xs font-bold text-gray-400 uppercase mb-3 flex items-center gap-2"><i class="ri-pulse-line"></i> Ritmo (Linha do Tempo)</h3>
                    <div style="height: 150px;"><canvas id="rhythmChart"></canvas></div>
                </div>

                <div class="bg-slate-900 p-4 rounded-xl border border-slate-700 mb-4">
                    <h3 class="text-xs font-bold text-gray-400 uppercase mb-3 flex items-center gap-2"><i class="ri-trophy-line"></i> Ranking Operadores</h3>
                    <div style="height: 150px;"><canvas id="rankingChart"></canvas></div>
                </div>

                <div class="bg-slate-900 p-4 rounded-xl border border-slate-700 mb-6">
                    <h3 class="text-xs font-bold text-gray-400 uppercase mb-3 flex items-center gap-2"><i class="ri-layout-masonry-line"></i> Mapa de Zonas</h3>
                    <div style="height: 150px;"><canvas id="zoneMapChart"></canvas></div>
                </div>

                 <button onclick="downloadExcel()" class="btn-action bg-green-600 mb-3"><i class="ri-file-excel-2-line mr-2"></i> Relatório Completo</button>
            </div>

            <div id="view-inventario" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Carregar Inventário</h2>
                <div id="inventory-zones-list" class="space-y-3"></div>
                <input type="file" id="inventory-file-input" class="hidden" accept=".xlsx,.csv,.pdf,.txt,.html">
            </div>
            <div id="view-zonas" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Selecionar Zona</h2>
                <div id="zones-list" class="space-y-3"></div>
            </div>
            <div id="view-log" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Histórico</h2>
                <div id="scan-log-list" class="space-y-2"></div>
            </div>
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="togglePanelFromNav('procurar', this)"><i class="ri-qr-scan-2-line"></i><span>Scan</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('inventario', this)"><i class="ri-archive-line"></i><span>Listas</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('dashboard', this)"><i class="ri-pie-chart-2-line"></i><span>Dash</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('zonas', this)"><i class="ri-map-pin-line"></i><span>Zonas</span></div>
    </nav>
    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = { STORAGE_KEY: 'natefy_v14_intel', WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9' };
        let state = { operator: null, idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null, inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}], zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null, html5QrCode: null, charts: {} };

        document.addEventListener('DOMContentLoaded', () => { init(); document.addEventListener("visibilitychange", () => { if(document.visibilityState === "visible" && state.operator) setTimeout(() => { if(!state.html5QrCode) startScanner(); }, 500); }); });
        function init() { loadState(); checkLogin(); setupActions(); updateUI(); }
        window.doLogin = function() { const val = document.getElementById('operator-input').value.trim(); if(val) { state.operator = val; saveState(); checkLogin(); } else alert("Nome obrigatório"); };
        function checkLogin() { if(state.operator) { document.getElementById('login-modal').classList.add('hidden'); document.getElementById('status-operator').innerText = state.operator; startScanner(); } else { document.getElementById('login-modal').classList.remove('hidden'); } }
        
        window.togglePanel = function(forceOpen) { const p = document.getElementById('controls-panel'); const isOpen = forceOpen !== undefined ? forceOpen : !p.classList.contains('open'); if(isOpen) p.classList.add('open'); else p.classList.remove('open'); };
        window.togglePanelFromNav = function(tabName, btn) { document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active')); btn.classList.add('active'); if(tabName === 'procurar') { const p = document.getElementById('controls-panel'); togglePanel(!p.classList.contains('open')); } else { togglePanel(true); } switchTab(tabName); };
        window.switchTab = function(tabName, btn) { if(btn) { document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active')); btn.classList.add('active'); } document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active')); document.getElementById(`view-${tabName}`).classList.add('active'); if(tabName === 'dashboard') updateDashboard(); if(tabName === 'zonas') renderZones(); if(tabName === 'log') updateUI(); if(tabName === 'inventario') renderInventoryZones(); };
        
        function setupActions() {
            document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]);
            document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]);
            document.getElementById('inventory-file-input').onchange = (e) => handleInventoryFile(e.target.files[0]);
            document.getElementById('manual-btn').onclick = () => { const v=document.getElementById('manual-input').value.trim(); if(v){ processScan(v); document.getElementById('manual-input').value=''; }};
            document.getElementById('undo-btn').onclick = doUndo;
            document.getElementById('download-report-btn').onclick = downloadExcel;
            document.getElementById('hunt-toggle-btn').onclick = toggleHuntMode;
            const handle = document.querySelector('.panel-handle'); let startY = 0;
            handle.addEventListener('touchstart', e => startY = e.touches[0].clientY); handle.addEventListener('touchend', e => { const diff = startY - e.changedTouches[0].clientY; if(diff > 50) togglePanel(true); if(diff < -50) togglePanel(false); });
        }
        
        window.toggleFastMode = function() { state.isFastMode = !state.isFastMode; const dot = document.getElementById('fast-mode-dot'); const btn = document.getElementById('fast-mode-btn'); if(state.isFastMode) { btn.classList.add('active'); btn.style.borderColor = '#10b981'; dot.style.transform = 'translateX(100%)'; } else { btn.classList.remove('active'); btn.style.borderColor = '#475569'; dot.style.transform = 'translateX(0)'; } };

        function startScanner() { if(state.html5QrCode) return; state.html5QrCode = new Html5Qrcode("reader"); state.html5QrCode.start({ facingMode: "environment" }, { fps: 30, qrbox: 300 }, (d) => processScan(d)).catch(err => console.log("Câmera:", err)); }
        function processScan(id) {
            const now = Date.now(); if (now - state.lastScanTime < 800) return; state.lastScanTime = now;
            if (state.isPaused) return; if (!state.isFastMode) state.isPaused = true;
            id = id.trim();
            if (state.huntMode && state.huntMode.isActive) { if (id === state.huntMode.targetId) { showFeedback('success', 'ALVO ENCONTRADO!', '', id); sendToN8n({ scannedId: id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: state.operator }); toggleHuntMode(); return; } }
            const desc = state.idDescriptions.get(id) || "Sem descrição";
            let status = "NÃO ENCONTRADO", type = "error", feedbackMsg = "NÃO ENCONTRADO", icon="ri-close-circle-fill";
            if (state.activeZone) { for (const [zId, zSet] of state.zoneData) { if (zId !== state.activeZone && zSet.has(id)) { status = "MISSORT"; type = "warning"; feedbackMsg = `MISSORT (${zId})`; icon="ri-error-warning-fill"; break; } } }
            if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) { status = "DUPLICADO"; type = "warning"; feedbackMsg = "JÁ BIPADO"; icon="ri-history-line"; }
            if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) { status = "SUCESSO"; type = "success"; feedbackMsg = "ENCONTRADO"; icon="ri-checkbox-circle-fill"; state.idsToFind.delete(id); }
            const entry = { id, status, desc, type, time: new Date().toISOString(), operator: state.operator, zone: state.activeZone };
            state.logs.unshift(entry); if (status === "SUCESSO") state.foundIds.unshift(entry); state.lastUndo = entry;
            saveState(); showFeedback(type, feedbackMsg, desc, id, icon); updateUI(); sendToN8n(entry);
        }
        function showFeedback(type, msg, desc, id, iconClass) {
            const overlay = document.getElementById('feedback-overlay'); const icon = document.getElementById('fb-icon'); const txt = document.getElementById('fb-msg'); const dsc = document.getElementById('fb-desc'); const idtxt = document.getElementById('fb-id'); const undo = document.getElementById('undo-btn');
            const colors = { success: '#22c55e', error: '#ef4444', warning: '#f59e0b' };
            icon.style.color = colors[type]; icon.className = iconClass || 'ri-checkbox-circle-fill'; txt.innerText = msg; dsc.innerText = desc; idtxt.innerText = `ID: ${id}`;
            overlay.className = `fixed inset-0 z-70 flex items-center justify-center p-6 transition-opacity duration-150 scan-${type}`; overlay.style.opacity = '1'; undo.classList.add('visible'); setTimeout(() => undo.classList.remove('visible'), 5000); if(navigator.vibrate) navigator.vibrate(200); setTimeout(() => { overlay.style.opacity = '0'; if (!state.isFastMode) state.isPaused = false; }, state.isFastMode ? 300 : 1200);
        }

        // --- INTELLIGENCE DASHBOARD ---
        function updateDashboard() {
            // KPIs
            const total = state.logs.length;
            const success = state.logs.filter(l => l.status === 'SUCESSO').length;
            const missort = state.logs.filter(l => l.status === 'MISSORT').length;
            const error = state.logs.filter(l => l.status === 'NÃO ENCONTRADO' || l.status === 'ERRO').length;
            
            const accuracy = total > 0 ? Math.round((success / total) * 100) : 100;
            
            // SPH (Scans Per Hour) - Últimos 10 mins extrapolados
            const tenMinAgo = new Date(Date.now() - 600000);
            const recent = state.logs.filter(l => new Date(l.time) > tenMinAgo).length;
            const sph = recent * 6;

            // Tempo Médio (Busca) - Média de diferença entre sucessos
            let avgTime = 0;
            const successLogs = state.logs.filter(l => l.status === 'SUCESSO').sort((a,b) => new Date(a.time) - new Date(b.time));
            if (successLogs.length > 1) {
                let sumDiff = 0;
                for(let i=1; i<successLogs.length; i++) {
                    sumDiff += (new Date(successLogs[i].time) - new Date(successLogs[i-1].time)) / 1000;
                }
                avgTime = Math.round(sumDiff / (successLogs.length - 1));
            }

            document.getElementById('kpi-sph').innerText = sph;
            document.getElementById('kpi-acc').innerText = accuracy + '%';
            document.getElementById('kpi-found-total').innerText = success + ' Sucessos';
            document.getElementById('kpi-time').innerText = avgTime + 's';
            document.getElementById('kpi-missort').innerText = missort;

            renderCharts(success, error, missort);
        }

        function renderCharts(s, e, m) {
            // Rhythm Chart
            const rCtx = document.getElementById('rhythmChart');
            if (rCtx) {
                if(state.charts.rhythm) state.charts.rhythm.destroy();
                const hourly = {};
                state.logs.forEach(l => { const h = new Date(l.time).getHours(); hourly[h] = (hourly[h]||0)+1; });
                state.charts.rhythm = new Chart(rCtx, {
                    type: 'line',
                    data: { labels: Object.keys(hourly), datasets: [{ label: 'Bips', data: Object.values(hourly), borderColor: '#6366f1', tension: 0.4, fill: true, backgroundColor: 'rgba(99, 102, 241, 0.2)' }] },
                    options: { responsive: true, maintainAspectRatio: false, scales: { x: { display: false }, y: { display: false } }, plugins: { legend: { display: false } } }
                });
            }
            
            // Ranking (Simulado com dados locais por enquanto, idealmente viria do Global)
            const rankCtx = document.getElementById('rankingChart');
            if(rankCtx) {
                if(state.charts.rank) state.charts.rank.destroy();
                state.charts.rank = new Chart(rankCtx, {
                    type: 'bar',
                    indexAxis: 'y',
                    data: { labels: ['Você'], datasets: [{ label: 'Bips', data: [state.logs.length], backgroundColor: '#10b981', borderRadius: 4 }] },
                    options: { responsive: true, maintainAspectRatio: false, scales: { x: { display: false }, y: { ticks: { color: 'white' } } }, plugins: { legend: { display: false } } }
                });
            }

            // Zone Map
            const zCtx = document.getElementById('zoneMapChart');
            if(zCtx) {
                if(state.charts.zone) state.charts.zone.destroy();
                const zData = {};
                state.logs.forEach(l => { if(l.zone) zData[l.zone] = (zData[l.zone]||0)+1; });
                state.charts.zone = new Chart(zCtx, {
                    type: 'bar',
                    data: { labels: Object.keys(zData), datasets: [{ label: 'Bips', data: Object.values(zData), backgroundColor: '#f59e0b', borderRadius: 4 }] },
                    options: { responsive: true, maintainAspectRatio: false, scales: { x: { ticks: { color: '#94a3b8' }, grid: { display: false } }, y: { display: false } }, plugins: { legend: { display: false } } }
                });
            }
        }

        // ... (Funções auxiliares sendToN8n, saveState, loadState, handleFile, handleOCR, handleInventoryFile, doUndo, toggleHuntMode, updateUI, renderZones, renderInventoryZones, downloadExcel) ...
        // Mantendo as mesmas lógicas para economizar espaço e focar no Dashboard
        function sendToN8n(data) { if(CONFIG.WEBHOOK.includes("https")) fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline")); }
        function saveState() { const s = { ...state, html5QrCode: null, charts: {} }; s.idsToFind = Array.from(s.idsToFind); s.idDescriptions = Array.from(s.idDescriptions.entries()); s.zoneData = Array.from(s.zoneData.entries()).map(([k,v])=>[k,Array.from(v)]); localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s)); }
        function loadState() { const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY)); if(s) { state = { ...state, ...s }; state.idsToFind = new Set(s.idsToFind); state.idDescriptions = new Map(s.idDescriptions); state.zoneData = new Map(s.zoneData.map(([k,v])=>[k,new Set(v)])); } }
        window.clearSession = function() { if(confirm('Apagar?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } };
        function handleFile(file) { if(!file)return; const r=new FileReader; r.onload=e=>{ const d=new Uint8Array(e.target.result); const w=XLSX.read(d,{type:'array'}); const j=XLSX.utils.sheet_to_json(w.Sheets[w.SheetNames[0]],{header:1}); const ids=j.map(x=>String(x[0])).filter(i=>i&&i.match(/\d+/)); state.idsToFind=new Set(ids); state.foundIds=[]; document.getElementById('file-status').classList.remove('hidden'); document.getElementById('file-count').innerText=`${ids.length} itens`; saveState(); }; r.readAsArrayBuffer(file); }
        async function handleOCR(file) { if(!file)return; document.getElementById('ocr-loading').classList.remove('hidden'); try { const w=Tesseract.createWorker(); await w.load(); await w.loadLanguage('eng'); await w.initialize('eng'); const {data:{text}}=await w.recognize(file); await w.terminate(); const l=text.split('\n'); let c=0; const n=new Set(); const r=/(\d{8,14})/; l.forEach(line=>{ const m=line.replace(/[^\w\s\>\-\.\(\)\/]/gi,'').match(r); if(m){ const id=m[0]; let d=line.replace(id,'').replace(/^[\s\>\-\.]+/g,'').trim(); if(d.length<3)d="Sem Descrição"; n.add(id); state.idDescriptions.set(id,d); c++; } }); if(c>0){ state.idsToFind=n; state.foundIds=[]; document.getElementById('file-status').classList.remove('hidden'); document.getElementById('file-count').innerText=`${c} itens (OCR)`; saveState(); } else alert("Nada encontrado"); } catch(e){ alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); } }
        function handleInventoryFile(file) { if(!file||!window.tempZone)return; const r=new FileReader; r.onload=e=>{ const d=new Uint8Array(e.target.result); const w=XLSX.read(d,{type:'array'}); const j=XLSX.utils.sheet_to_json(w.Sheets[w.SheetNames[0]],{header:1}); const ids=new Set(j.map(x=>String(x[0])).filter(i=>i&&i.match(/\d+/))); state.zoneData.set(window.tempZone, ids); saveState(); renderInventoryZones(); alert("Carregado!"); }; r.readAsArrayBuffer(file); }
        function doUndo() { if(!state.lastUndo)return; state.logs.shift(); if(state.lastUndo.status==="SUCESSO"){state.foundIds.shift(); state.idsToFind.add(state.lastUndo.id);} sendToN8n({...state.lastUndo,status:"CANCELADO"}); state.lastUndo=null; document.getElementById('undo-btn').classList.remove('visible'); updateUI(); alert("Desfeito!"); }
        window.toggleHuntMode = function() { const t=document.getElementById('hunt-target-id'); const b=document.getElementById('hunt-toggle-btn'); const s=document.getElementById('hunt-status'); if(state.huntMode&&state.huntMode.isActive){ state.huntMode={isActive:false,targetId:null}; t.disabled=false; t.value=''; b.innerText='Ativar'; b.classList.remove('bg-red-600'); b.classList.add('btn-primary'); s.innerText=''; } else { const v=t.value.trim(); if(!v)return alert("ID?"); state.huntMode={isActive:true,targetId:v}; t.disabled=true; b.innerText='Cancelar'; b.classList.remove('btn-primary'); b.classList.add('bg-red-600'); s.innerText='ATIVO'; } saveState(); };
        function updateUI() { const l=document.getElementById('scan-log-list'); l.innerHTML=state.logs.slice(0,50).map(i=>`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700"><div><div class="font-bold text-sm">${i.id}</div><div class="text-xs text-gray-400 truncate w-48">${i.desc}</div></div><div class="text-right"><div class="text-xs font-bold ${i.status==='SUCESSO'?'text-green-500':(i.status==='ERRO'?'text-red-500':'text-yellow-500')}">${i.status}</div><div class="text-xs text-gray-500">${new Date(i.time).toLocaleTimeString()}</div></div></div>`).join(''); }
        function renderZones() { const c=document.getElementById('zones-list'); c.innerHTML=`<button class="btn-action bg-gray-600 mb-3 text-sm" onclick="window.setZone(null)">🚫 Sem Zona Ativa</button>`; state.inventoryZones.forEach(z=>{ const a=state.activeZone===z.id; c.innerHTML+=`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700 ${a?'border-green-500':''}"><span class="font-bold">${z.name}</span><button class="btn py-2 px-4 text-xs ${a?'bg-green-600 text-white':'bg-indigo-600 text-white'}" onclick="window.setZone('${z.id}')">${a?'ATIVA':'ATIVAR'}</button></div>`; }); }
        function renderInventoryZones() { const c=document.getElementById('inventory-zones-list'); c.innerHTML=''; state.inventoryZones.forEach(z=>{ const cnt=state.zoneData.get(z.id)?.size||0; c.innerHTML+=`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700"><div><div class="font-bold">${z.name}</div><div class="text-xs text-gray-400">${cnt} IDs</div></div><button class="btn btn-primary py-2 px-4 text-xs" onclick="window.tempZone='${z.id}';document.getElementById('inventory-file-input').click()"><i class="ri-upload-cloud-line mr-1"></i> Carregar</button></div>`; }); }
        function downloadExcel() { if(state.logs.length===0)return alert("Sem dados"); const w=XLSX.utils.json_to_sheet(state.logs); const b=XLSX.utils.book_new(); XLSX.utils.book_append_sheet(b,w,"Logs"); XLSX.writeFile(b,"Relatorio.xlsx"); }
    </script>
</body>
</html>
