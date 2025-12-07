<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Pro</title>

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

        /* MIRA DINÂMICA */
        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            border: 2px solid rgba(255, 255, 255, 0.6); border-radius: 24px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.6); z-index: 10; pointer-events: none;
            transition: width 0.2s, height 0.2s;
        }
        /* Cantos */
        .scan-overlay::before, .scan-overlay::after { content: ''; position: absolute; width: 40px; height: 40px; border: 4px solid #6366f1; border-radius: 4px; pointer-events: none; }
        .scan-overlay::before { top: -2px; left: -2px; border-bottom: 0; border-right: 0; }
        .scan-overlay::after { bottom: -2px; right: -2px; border-top: 0; border-left: 0; }

        /* Top Bar */
        #top-bar { position: absolute; top: 0; left: 0; width: 100%; z-index: 20; padding: env(safe-area-inset-top) 20px 10px 20px; background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent); display: flex; justify-content: space-between; align-items: start; }
        
        /* Menu Inferior */
        #bottom-nav { position: absolute; bottom: 0; left: 0; width: 100%; height: 80px; background: rgba(30, 41, 59, 0.95); backdrop-filter: blur(10px); border-top: 1px solid rgba(255,255,255,0.1); display: flex; justify-content: space-around; align-items: center; padding-bottom: env(safe-area-inset-bottom); z-index: 50; }
        .nav-item { display: flex; flex-direction: column; align-items: center; justify-content: center; color: #94a3b8; font-size: 10px; gap: 4px; width: 20%; transition: all 0.2s; }
        .nav-item i { font-size: 22px; transition: transform 0.2s; }
        .nav-item.active { color: #6366f1; }
        .nav-item.active i { transform: translateY(-2px); }

        /* Painel Deslizante */
        #controls-panel { position: absolute; bottom: 80px; left: 0; width: 100%; background: var(--panel); border-radius: 24px 24px 0 0; transform: translateY(110%); transition: transform 0.3s cubic-bezier(0.2, 0.8, 0.2, 1); z-index: 40; max-height: 80vh; display: flex; flex-direction: column; box-shadow: 0 -5px 20px rgba(0,0,0,0.3); }
        #controls-panel.open { transform: translateY(0); }

        /* Abas Roláveis */
        .tabs-container { display: flex; overflow-x: auto; padding: 10px 16px; gap: 8px; background: var(--panel); flex-shrink: 0; scrollbar-width: none; border-bottom: 1px solid rgba(255,255,255,0.05); }
        .tab-btn { padding: 8px 16px; border-radius: 20px; font-size: 13px; font-weight: 500; color: #94a3b8; white-space: nowrap; transition: all 0.2s; border: 1px solid rgba(255,255,255,0.05); }
        .tab-btn.active { background: var(--primary); color: white; box-shadow: 0 4px 10px rgba(99, 102, 241, 0.3); border-color: var(--primary); }
        
        .panel-content { padding: 20px; overflow-y: auto; flex: 1; }
        .tab-content { display: none; animation: fadeIn 0.3s; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Estilos de Componentes */
        .btn-card { background: #334155; border: 1px solid #475569; border-radius: 16px; padding: 16px; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; color: white; transition: background 0.2s; }
        .input-dark { background: #0f172a; border: 1px solid #475569; color: white; padding: 12px; border-radius: 12px; width: 100%; outline: none; }
        .btn-action { background: #6366f1; color: white; padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; }
        
        input[type=range] { width: 100%; height: 6px; background: #475569; border-radius: 5px; outline: none; -webkit-appearance: none; }
        input[type=range]::-webkit-slider-thumb { -webkit-appearance: none; appearance: none; width: 20px; height: 20px; border-radius: 50%; background: #6366f1; cursor: pointer; border: 2px solid white; box-shadow: 0 0 10px rgba(99,102,241,0.5); }
        .toggle-switch { position: relative; width: 50px; height: 26px; background: #475569; border-radius: 20px; transition: 0.3s; cursor: pointer; }
        .toggle-switch::after { content: ''; position: absolute; top: 3px; left: 3px; width: 20px; height: 20px; background: white; border-radius: 50%; transition: 0.3s; }
        .toggle-active { background: #10b981; }
        .toggle-active::after { transform: translateX(24px); }

        #feedback-overlay { position: fixed; inset: 0; z-index: 60; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.9); }
        .scan-error { background: rgba(239, 68, 68, 0.9); }
        .scan-warning { background: rgba(234, 179, 8, 0.9); }
        .hidden { display: none !important; }
        #ocr-loading, #login-modal { position: fixed; inset: 0; z-index: 90; background: #0f172a; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        #undo-btn { position: absolute; top: 100px; right: 20px; z-index: 35; background: #ef4444; color: white; width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }
        #flowchart-canvas-container { position: absolute; left: -9999px; }

        /* KPIs */
        .kpi-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
        .kpi-box { background: #0f172a; border-radius: 12px; padding: 15px; border-left: 4px solid; position: relative; overflow: hidden; }
        .kpi-label { font-size: 10px; color: #94a3b8; text-transform: uppercase; letter-spacing: 1px; font-weight: bold; margin-bottom: 4px; }
        .kpi-value { font-size: 24px; font-weight: 800; color: white; line-height: 1; }
    </style>
</head>
<body>

    <div id="top-bar">
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-full bg-indigo-500 flex items-center justify-center text-white font-bold text-lg shadow-lg" onclick="switchTab('perfil')">
                <i class="ri-user-3-line"></i>
            </div>
            <div>
                <div id="status-operator" class="text-white font-bold leading-tight">Operador</div>
                <div id="status-zone" class="text-xs text-cyan-400 font-mono bg-black/50 px-2 py-0.5 rounded inline-block mt-0.5 backdrop-blur-sm">ZONA: --</div>
            </div>
        </div>
        <div class="flex gap-2">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_8px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay" id="scan-box"></div>

    <div id="feedback-overlay">
        <div class="text-center p-6 bg-black/40 backdrop-blur-md rounded-2xl border border-white/10 shadow-2xl">
            <i id="fb-icon" class="ri-checkbox-circle-fill text-6xl text-white mb-2 block"></i>
            <div id="fb-msg" class="text-3xl font-black text-white mb-2">ENCONTRADO</div>
            <div id="fb-desc" class="text-sm text-gray-200 bg-white/10 p-2 rounded mb-1">Descrição</div>
            <div id="fb-id" class="text-xs font-mono text-gray-400">ID: 123</div>
        </div>
    </div>

    <button id="undo-btn"><i class="ri-arrow-go-back-line text-2xl"></i></button>
    <div id="ocr-loading" class="hidden"><i class="ri-loader-4-line text-6xl text-indigo-500 animate-spin mb-4"></i><p class="text-xl font-bold text-white">Lendo Imagem...</p></div>

    <div id="login-modal">
        <div class="w-full max-w-xs text-center p-6">
            <div class="w-20 h-20 bg-indigo-600 rounded-2xl mx-auto mb-6 flex items-center justify-center text-4xl text-white shadow-lg shadow-indigo-500/50"><i class="ri-box-3-fill"></i></div>
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
            <button class="tab-btn" onclick="switchTab('inventario', this)">📦 Listas</button>
            <button class="tab-btn" onclick="switchTab('ajustes', this)">⚙️ Ajustes</button>
            <button class="tab-btn" onclick="switchTab('dashboard', this)">📊 Dash</button>
            <button class="tab-btn" onclick="switchTab('zonas', this)">📍 Zonas</button>
            <button class="tab-btn" onclick="switchTab('log', this)">📝 Log</button>
            <button class="tab-btn" onclick="switchTab('perfil', this)">👤 Perfil</button>
        </div>

        <div class="panel-content">
            
            <div id="view-procurar" class="tab-content active">
                <div class="space-y-4">
                    <div class="grid grid-cols-2 gap-3">
                        <button onclick="document.getElementById('file-input').click()" class="btn-card"><i class="ri-file-excel-2-fill text-2xl text-green-500"></i><span class="text-xs font-bold uppercase">Planilha</span></button>
                        <button onclick="document.getElementById('image-input').click()" class="btn-card"><i class="ri-camera-fill text-2xl text-cyan-400"></i><span class="text-xs font-bold uppercase">Foto OCR</span></button>
                    </div>
                    <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                    <input type="file" id="image-input" class="hidden" accept="image/*">
                    <div id="file-status" class="hidden bg-green-900/30 border border-green-500/30 p-3 rounded-xl flex items-center gap-3"><i class="ri-check-double-line text-green-400 text-xl"></i><div><div class="text-xs text-green-400 font-bold uppercase">Carregado</div><div id="file-count" class="text-sm text-white">0 itens prontos</div></div></div>
                    <div class="flex gap-2"><input type="text" id="manual-input" class="input-dark" placeholder="Digitar ID manual..."><button id="manual-btn" class="bg-indigo-600 text-white px-4 rounded-xl"><i class="ri-check-line text-xl"></i></button></div>
                    <div class="btn-card flex-row justify-between py-3 cursor-pointer" onclick="toggleFastMode()" id="fast-mode-btn"><span class="text-sm font-medium">Modo Rápido</span><div class="w-10 h-6 bg-slate-700 rounded-full relative"><div class="w-4 h-4 bg-white rounded-full absolute top-1 left-1 transition-all" id="fast-mode-dot"></div></div></div>
                    <div class="card bg-primary/10 border-primary/30 p-4 rounded-xl border"><div class="flex justify-between items-center mb-2"><label class="text-xs font-bold uppercase text-primary-light">Modo Caça</label><span id="hunt-status" class="text-xs font-mono text-primary-light"></span></div><div class="flex gap-2"><input type="text" id="hunt-target-id" class="input-dark text-sm p-2" placeholder="ID Alvo"><button id="hunt-toggle-btn" onclick="toggleHuntMode()" class="btn-action px-4 text-sm w-auto">Ativar</button></div></div>
                    <button onclick="clearSession()" class="w-full py-3 text-xs text-red-400 hover:text-red-300 border border-red-900/50 rounded-xl mt-2">Limpar Sessão</button>
                </div>
            </div>

            <div id="view-inventario" class="tab-content">
                <h2 class="text-xl font-bold mb-4 text-white">Carregar Inventário</h2>
                <div id="inventory-zones-list" class="space-y-3"></div>
                <input type="file" id="inventory-file-input" class="hidden" accept=".xlsx,.csv,.pdf,.txt,.html">
            </div>

            <div id="view-zonas" class="tab-content">
                <h2 class="text-xl font-bold mb-4 text-white">Selecionar Zona Ativa</h2>
                <div id="zones-list" class="space-y-3"></div>
            </div>

            <div id="view-ajustes" class="tab-content">
                <h2 class="text-xl font-bold mb-6 text-white">Configurações</h2>
                <div class="space-y-6">
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700">
                        <div class="flex justify-between mb-2"><label class="text-sm font-bold text-slate-300">Tamanho da Mira</label><span id="range-val" class="text-xs text-cyan-400 font-mono">80%</span></div>
                        <input type="range" id="scan-size-range" min="40" max="95" value="80" oninput="updateScanSize(this.value)">
                    </div>
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700">
                        <div class="flex justify-between mb-2"><label class="text-sm font-bold text-slate-300">Velocidade (FPS)</label><span id="fps-val" class="text-xs text-cyan-400 font-mono">30 FPS</span></div>
                        <input type="range" id="scan-fps-range" min="5" max="60" value="30" onchange="updateScanFPS(this.value)" oninput="document.getElementById('fps-val').innerText = this.value + ' FPS'">
                    </div>
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700 space-y-4">
                        <div class="flex justify-between items-center"><span class="text-sm text-slate-300">Som de Bip</span><div class="toggle-switch toggle-active" id="sound-toggle" onclick="toggleSetting('sound')"></div></div>
                        <div class="flex justify-between items-center"><span class="text-sm text-slate-300">Vibração</span><div class="toggle-switch toggle-active" id="vibrate-toggle" onclick="toggleSetting('vibrate')"></div></div>
                    </div>
                </div>
            </div>

            <div id="view-dashboard" class="tab-content text-center">
                <div class="kpi-grid">
                    <div class="kpi-box border-blue-500"><div class="kpi-label">Produtividade</div><div class="kpi-value" id="kpi-sph">0</div></div>
                    <div class="kpi-box border-green-500"><div class="kpi-label">Acuracidade</div><div class="kpi-value text-green-400" id="kpi-accuracy">100%</div></div>
                    <div class="kpi-box border-purple-500"><div class="kpi-label">Tempo Médio</div><div class="kpi-value text-purple-400" id="kpi-time">0s</div></div>
                    <div class="kpi-box border-red-500"><div class="kpi-label">Missorts</div><div class="kpi-value text-red-400" id="kpi-missort">0</div></div>
                </div>
                <button onclick="downloadExcel()" class="btn-action bg-green-600 mb-3"><i class="ri-file-excel-2-line mr-2"></i> Relatório Excel</button>
            </div>

             <div id="view-log" class="tab-content">
                <h2 class="text-xl font-bold mb-4 text-white">Histórico</h2>
                <div id="scan-log-list" class="space-y-2"></div>
            </div>
            
             <div id="view-perfil" class="tab-content">
                <div class="text-center mb-8 pt-4">
                    <div class="w-24 h-24 bg-indigo-600 rounded-full mx-auto mb-4 flex items-center justify-center text-4xl text-white shadow-xl ring-4 ring-indigo-900"><i class="ri-user-3-fill"></i></div>
                    <h2 class="text-2xl font-bold text-white" id="profile-name">Operador</h2>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-6">
                    <div class="bg-slate-800 p-4 rounded-xl text-center border border-slate-700"><div class="text-3xl font-bold text-cyan-400" id="profile-total-today">0</div><div class="text-xs text-slate-500 uppercase">Hoje</div></div>
                    <div class="bg-slate-800 p-4 rounded-xl text-center border border-slate-700"><div class="text-3xl font-bold text-purple-400" id="profile-total-life">0</div><div class="text-xs text-slate-500 uppercase">Total Geral</div></div>
                </div>
                <button onclick="logout()" class="w-full bg-red-900/20 border border-red-900/50 p-4 rounded-xl flex items-center justify-center text-red-400 font-bold hover:bg-red-900/30 transition"><i class="ri-logout-box-r-line mr-2"></i> SAIR DA CONTA</button>
            </div>

        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="togglePanelFromNav('procurar', this)"><i class="ri-qr-scan-2-line"></i><span>Scan</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('inventario', this)"><i class="ri-archive-line"></i><span>Listas</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('ajustes', this)"><i class="ri-settings-4-line"></i><span>Ajustes</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('dashboard', this)"><i class="ri-pie-chart-2-line"></i><span>Dash</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('zonas', this)"><i class="ri-map-pin-line"></i><span>Zonas</span></div>
    </nav>
    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = { STORAGE_KEY: 'natefy_ultimate_v18', WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9' };
        let state = { operator: null, idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null, inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}], zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null, html5QrCode: null, charts: {}, settings: { scanSize: 80, fps: 30, sound: true, vibrate: true }, totalBipsLife: 0 };
        document.addEventListener('DOMContentLoaded', () => { init(); document.addEventListener("visibilitychange", () => { if (document.visibilityState === "visible" && state.operator) setTimeout(() => { if(!state.html5QrCode) startScanner(); }, 500); }); });
        function init() { loadState(); applySettingsUI(); checkLogin(); setupActions(); updateUI(); }
        
        // SETTINGS & UI
        window.updateScanSize = function(val) { state.settings.scanSize = val; document.getElementById('range-val').innerText = val + '%'; const box = document.getElementById('scan-box'); box.style.width = val + 'vw'; box.style.height = val + 'vw'; saveState(); };
        window.updateScanFPS = function(val) { state.settings.fps = parseInt(val); saveState(); if(state.html5QrCode) { state.html5QrCode.stop().then(() => { state.html5QrCode = null; startScanner(); }); } };
        window.toggleSetting = function(key) { state.settings[key] = !state.settings[key]; const btn = document.getElementById(key === 'sound' ? 'sound-toggle' : 'vibrate-toggle'); if(state.settings[key]) btn.classList.add('toggle-active'); else btn.classList.remove('toggle-active'); saveState(); };
        function applySettingsUI() { document.getElementById('scan-size-range').value = state.settings.scanSize; updateScanSize(state.settings.scanSize); document.getElementById('scan-fps-range').value = state.settings.fps; document.getElementById('fps-val').innerText = state.settings.fps + ' FPS'; const sBtn = document.getElementById('sound-toggle'); if(state.settings.sound) sBtn.classList.add('toggle-active'); else sBtn.classList.remove('toggle-active'); const vBtn = document.getElementById('vibrate-toggle'); if(state.settings.vibrate) vBtn.classList.add('toggle-active'); else vBtn.classList.remove('toggle-active'); }
        
        // LOGIN & NAV
        window.doLogin = function() { const val = document.getElementById('operator-input').value.trim(); if(val) { state.operator = val; saveState(); checkLogin(); } else alert("Nome obrigatório"); };
        function checkLogin() { if (state.operator) { document.getElementById('login-modal').classList.add('hidden'); document.getElementById('status-operator').innerText = state.operator; document.getElementById('profile-name').innerText = state.operator; updateProfileStats(); startScanner(); } else { document.getElementById('login-modal').classList.remove('hidden'); } }
        function updateProfileStats() { document.getElementById('profile-total-today').innerText = state.logs.length; document.getElementById('profile-total-life').innerText = state.totalBipsLife; }
        window.togglePanel = function(forceOpen) { const p = document.getElementById('controls-panel'); const isOpen = forceOpen !== undefined ? forceOpen : !p.classList.contains('open'); if(isOpen) p.classList.add('open'); else p.classList.remove('open'); };
        window.togglePanelFromNav = function(tabName, btn) { document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active')); btn.classList.add('active'); if(tabName === 'procurar') { const p = document.getElementById('controls-panel'); togglePanel(!p.classList.contains('open')); } else { togglePanel(true); } switchTab(tabName); };
        window.switchTab = function(tabName, btn) { if(btn) { document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active')); btn.classList.add('active'); } document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active')); document.getElementById(`view-${tabName}`).classList.add('active'); if(tabName === 'dashboard') updateDashboard(); if(tabName === 'zonas') renderZones(); if(tabName === 'log') updateUI(); if(tabName === 'inventario') renderInventoryZones(); };
        
        // SCANNER
        function startScanner() { if(state.html5QrCode) return; state.html5QrCode = new Html5Qrcode("reader"); state.html5QrCode.start({ facingMode: "environment" }, { fps: state.settings.fps, qrbox: 300 }, (d) => processScan(d)).catch(err => console.log("Câmera:", err)); }
        function processScan(id) { const now = Date.now(); if (now - state.lastScanTime < 800) return; state.lastScanTime = now; if (state.isPaused) return; if (!state.isFastMode) state.isPaused = true; id = id.trim(); if (state.huntMode && state.huntMode.isActive) { if (id === state.huntMode.targetId) { showFeedback('success', 'ALVO ENCONTRADO!', '', id); sendToN8n({ scannedId: id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: state.operator }); toggleHuntMode(); return; } } const desc = state.idDescriptions.get(id) || "Sem descrição"; let status = "NÃO ENCONTRADO", type = "error", feedbackMsg = "NÃO ENCONTRADO", icon="ri-close-circle-fill"; if (state.activeZone) { for (const [zId, zSet] of state.zoneData) { if (zId !== state.activeZone && zSet.has(id)) { status = "MISSORT"; type = "warning"; feedbackMsg = `MISSORT (${zId})`; icon="ri-error-warning-fill"; break; } } } if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) { status = "DUPLICADO"; type = "warning"; feedbackMsg = "JÁ BIPADO"; icon="ri-history-line"; } if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) { status = "SUCESSO"; type = "success"; feedbackMsg = "ENCONTRADO"; icon="ri-checkbox-circle-fill"; state.idsToFind.delete(id); } const entry = { id, status, desc, type, time: new Date().toISOString(), operator: state.operator, zone: state.activeZone }; state.logs.unshift(entry); if (status === "SUCESSO") state.foundIds.unshift(entry); state.lastUndo = entry; state.totalBipsLife++; saveState(); showFeedback(type, feedbackMsg, desc, id, icon); updateUI(); sendToN8n(entry); updateProfileStats(); }
        function showFeedback(type, msg, desc, id, iconClass) { const overlay = document.getElementById('feedback-overlay'); const icon = document.getElementById('fb-icon'); const txt = document.getElementById('fb-msg'); const dsc = document.getElementById('fb-desc'); const idtxt = document.getElementById('fb-id'); const undo = document.getElementById('undo-btn'); const colors = { success: '#22c55e', error: '#ef4444', warning: '#f59e0b' }; icon.style.color = colors[type]; icon.className = iconClass || 'ri-checkbox-circle-fill'; txt.innerText = msg; dsc.innerText = desc; idtxt.innerText = `ID: ${id}`; overlay.className = `fixed inset-0 z-70 flex items-center justify-center p-6 transition-opacity duration-150 scan-${type}`; overlay.style.opacity = '1'; undo.classList.add('visible'); setTimeout(() => undo.classList.remove('visible'), 5000); if(state.settings.vibrate && navigator.vibrate) navigator.vibrate(200); if(state.settings.sound) playSound(type); setTimeout(() => { overlay.style.opacity = '0'; if (!state.isFastMode) state.isPaused = false; }, state.isFastMode ? 300 : 1200); }
        function playSound(type) { if(!state.audioContext) state.audioContext = new (window.AudioContext||window.webkitAudioContext)(); const o = state.audioContext.createOscillator(); const g = state.audioContext.createGain(); o.connect(g); g.connect(state.audioContext.destination); g.gain.value = 0.1; o.frequency.value = type==='success'?1200 : (type==='warning'?600:200); o.start(); setTimeout(()=>o.stop(), 150); }
        
        // ACTIONS
        function setupActions() { document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]); document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]); document.getElementById('inventory-file-input').onchange = (e) => handleInventoryFile(e.target.files[0]); document.getElementById('manual-btn').onclick = () => { const v=document.getElementById('manual-input').value.trim(); if(v){ processScan(v); document.getElementById('manual-input').value=''; }}; document.getElementById('undo-btn').onclick = doUndo; document.getElementById('download-report-btn').onclick = downloadExcel; document.getElementById('hunt-toggle-btn').onclick = toggleHuntMode; const handle = document.querySelector('.panel-handle'); let startY = 0; handle.addEventListener('touchstart', e => startY = e.touches[0].clientY); handle.addEventListener('touchend', e => { const diff = startY - e.changedTouches[0].clientY; if (diff > 50) togglePanel(true); if (diff < -50) togglePanel(false); }); document.body.addEventListener('click', () => { if(!state.audioContext) state.audioContext = new (window.AudioContext||window.webkitAudioContext)(); }, { once: true }); }
        window.toggleFastMode = function() { state.isFastMode = !state.isFastMode; const btn = document.getElementById('fast-mode-btn'); const dot = document.getElementById('fast-mode-dot'); if(state.isFastMode) { btn.classList.add('active'); dot.classList.add('bg-green-500'); dot.style.transform = 'translateX(100%)'; } else { btn.classList.remove('active'); dot.classList.remove('bg-green-500'); dot.style.transform = 'translateX(0)'; } };
        function sendToN8n(data) { if(CONFIG.WEBHOOK.includes("https")) fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline")); }
        function saveState() { const s = { ...state, html5QrCode: null, chart: null, audioContext: null }; s.idsToFind = Array.from(s.idsToFind); s.idDescriptions = Array.from(s.idDescriptions.entries()); s.zoneData = Array.from(s.zoneData.entries()).map(([k,v])=>[k,Array.from(v)]); localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s)); }
        function loadState() { const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY)); if(s) { state = { ...state, ...s }; state.idsToFind = new Set(s.idsToFind); state.idDescriptions = new Map(s.idDescriptions); state.zoneData = new Map(s.zoneData.map(([k,v])=>[k,new Set(v)])); if(!state.settings) state.settings = { scanSize: 80, fps: 30, sound: true, vibrate: true }; if(!state.totalBipsLife) state.totalBipsLife = 0; if(state.isFastMode) { document.getElementById('fast-mode-btn').classList.add('active'); document.getElementById('fast-mode-dot').classList.add('bg-green-500'); document.getElementById('fast-mode-dot').style.transform = 'translateX(100%)'; } } }
        window.clearSession = function() { if(confirm('Apagar?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } }; window.logout = function() { if(confirm('Sair?')) { state.operator = null; saveState(); location.reload(); } };
        function handleFile(f){if(!f)return;const r=new FileReader;r.onload=e=>{const d=new Uint8Array(e.target.result),w=XLSX.read(d,{type:'array'}),j=XLSX.utils.sheet_to_json(w.Sheets[w.SheetNames[0]],{header:1}),ids=j.map(x=>String(x[0])).filter(i=>i&&i.match(/\d+/));state.idsToFind=new Set(ids);state.foundIds=[];document.getElementById('file-status').classList.remove('hidden');document.getElementById('file-count').innerText=`${ids.length} itens`;saveState();};r.readAsArrayBuffer(f);}
        async function handleOCR(f){if(!f)return;document.getElementById('ocr-loading').classList.remove('hidden');try{const w=Tesseract.createWorker();await w.load();await w.loadLanguage('eng');await w.initialize('eng');const{data:{text}}=await w.recognize(f);await w.terminate();const l=text.split('\n');let c=0;const n=new Set();l.forEach(x=>{const m=x.replace(/[^\w\s\>\-\.\(\)\/]/gi,'').match(/(\d{8,14})/);if(m){const i=m[0];let d=x.replace(i,'').replace(/^[\s\>\-\.]+/g,'').trim();if(d.length<3)d="S/D";n.add(i);state.idDescriptions.set(i,d);c++;}});if(c>0){state.idsToFind=n;state.foundIds=[];document.getElementById('file-status').classList.remove('hidden');document.getElementById('file-count').innerText=`${c} itens (OCR)`;saveState();}else alert("Nada");}catch(e){alert("Erro OCR");}finally{document.getElementById('ocr-loading').classList.add('hidden');}}
        function handleInventoryFile(f){if(!f||!window.tempZone)return;const r=new FileReader;r.onload=e=>{const d=new Uint8Array(e.target.result),w=XLSX.read(d,{type:'array'}),j=XLSX.utils.sheet_to_json(w.Sheets[w.SheetNames[0]],{header:1}),ids=new Set(j.map(x=>String(x[0])).filter(i=>i&&i.match(/\d+/)));state.zoneData.set(window.tempZone,ids);saveState();renderInventoryZones();alert("Carregado!");};r.readAsArrayBuffer(f);}
        function doUndo(){if(!state.lastUndo)return;state.logs.shift();if(state.lastUndo.status==="SUCESSO"){state.foundIds.shift();state.idsToFind.add(state.lastUndo.id);}sendToN8n({...state.lastUndo,status:"CANCELADO"});state.lastUndo=null;document.getElementById('undo-btn').classList.remove('visible');updateUI();alert("Desfeito!");}
        window.toggleHuntMode=function(){const t=document.getElementById('hunt-target-id'),b=document.getElementById('hunt-toggle-btn'),s=document.getElementById('hunt-status');if(state.huntMode&&state.huntMode.isActive){state.huntMode={isActive:!1,targetId:null};t.disabled=!1;t.value='';b.innerText='Ativar';b.classList.remove('btn-danger');b.classList.add('btn-primary');s.innerText='';}else{const v=t.value.trim();if(!v)return alert("ID?");state.huntMode={isActive:!0,targetId:v};t.disabled=!0;b.innerText='Cancelar';b.classList.remove('btn-primary');b.classList.add('btn-danger');s.innerText='ATIVO';}saveState();};
        function updateUI(){const l=document.getElementById('scan-log-list');l.innerHTML=state.logs.slice(0,50).map(i=>`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700"><div><div class="font-bold text-sm">${i.id}</div><div class="text-xs text-gray-400 truncate w-48">${i.desc}</div></div><div class="text-right"><div class="text-xs font-bold ${i.status==='SUCESSO'?'text-green-500':(i.status==='ERRO'?'text-red-500':'text-yellow-500')}">${i.status}</div><div class="text-xs text-gray-500">${new Date(i.time).toLocaleTimeString()}</div></div></div>`).join('');}
        function renderZones(){const c=document.getElementById('zones-list');c.innerHTML=`<button class="btn-action bg-gray-600 mb-3 text-sm" onclick="window.setZone(null)">🚫 Sem Zona Ativa</button>`;state.inventoryZones.forEach(z=>{const a=state.activeZone===z.id;c.innerHTML+=`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700 ${a?'border-green-500':''}"><span class="font-bold text-white text-lg">${z.name}</span><button class="py-2 px-4 rounded-lg text-xs font-bold ${a?'bg-green-600 text-white':'bg-indigo-600 text-white'}" onclick="window.setZone('${z.id}')">${a?'ATIVA':'ATIVAR'}</button></div>`;});}
        function renderInventoryZones(){const c=document.getElementById('inventory-zones-list');c.innerHTML='';state.inventoryZones.forEach(z=>{const cnt=state.zoneData.get(z.id)?.size||0;c.innerHTML+=`<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700"><div><div class="font-bold text-white text-lg">${z.name}</div><div class="text-xs text-gray-400">${cnt} IDs</div></div><button class="py-2 px-4 rounded-lg text-xs font-bold bg-indigo-600 text-white" onclick="window.tempZone='${z.id}';document.getElementById('inventory-file-input').click()"><i class="ri-upload-cloud-line mr-1"></i> Carregar</button></div>`;});}
        window.setZone=(id)=>{state.activeZone=id;document.getElementById('status-zone').innerText=`ZONA: ${id?id.toUpperCase():'--'}`;saveState();renderZones();};
        function downloadExcel(){if(state.logs.length===0)return alert("Sem dados");const w=XLSX.utils.json_to_sheet(state.logs);const b=XLSX.utils.book_new();XLSX.utils.book_append_sheet(b,w,"Logs");XLSX.writeFile(b,"Relatorio.xlsx");}
        function updateDashboard(){const total=state.logs.length;const success=state.logs.filter(l=>l.status==='SUCESSO').length;const missort=state.logs.filter(l=>l.status==='MISSORT').length;const error=state.logs.filter(l=>l.status==='NÃO ENCONTRADO'||l.status==='ERRO').length;const accuracy=total>0?Math.round((success/total)*100):100;const tenMinAgo=new Date(Date.now()-600000);const recent=state.logs.filter(l=>new Date(l.time)>tenMinAgo).length;const sph=recent*6;let avgTime=0;const sLogs=state.logs.filter(l=>l.status==='SUCESSO').sort((a,b)=>new Date(a.time)-new Date(b.time));if(sLogs.length>1){let sum=0;for(let i=1;i<sLogs.length;i++)sum+=(new Date(sLogs[i].time)-new Date(sLogs[i-1].time))/1000;avgTime=Math.round(sum/(sLogs.length-1));}document.getElementById('kpi-sph').innerText=sph;document.getElementById('kpi-accuracy').innerText=accuracy+'%';document.getElementById('kpi-found-total').innerText=success+' Sucessos';document.getElementById('kpi-time').innerText=avgTime+'s';document.getElementById('kpi-missort').innerText=missort;}
    </script>
</body>
</html>
