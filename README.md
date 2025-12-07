<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Turbo</title>

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
        :root { --primary: #6366f1; --bg: #000000; --panel: #1e293b; }
        body, html { height: 100%; width: 100%; overflow: hidden; background: var(--bg); font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; }
        
        /* --- Scanner 90% (Gigante) --- */
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 90vw; height: 90vw; /* 90% da tela */
            max-width: 500px; max-height: 500px;
            border: 2px solid rgba(255, 255, 255, 0.4); border-radius: 30px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.7); /* Fundo bem escuro */
            z-index: 10; pointer-events: none;
        }
        /* Cantos da mira */
        .scan-overlay::before, .scan-overlay::after { content: ''; position: absolute; width: 60px; height: 60px; border: 6px solid #6366f1; border-radius: 8px; pointer-events: none; }
        .scan-overlay::before { top: -2px; left: -2px; border-bottom: 0; border-right: 0; }
        .scan-overlay::after { bottom: -2px; right: -2px; border-top: 0; border-left: 0; }

        /* --- Top Bar --- */
        #top-bar {
            position: absolute; top: 0; left: 0; width: 100%; z-index: 20;
            padding: env(safe-area-inset-top) 20px 10px 20px;
            background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent);
            display: flex; justify-content: space-between; align-items: start;
        }

        /* --- Painel Deslizante (Estilo V8 que funcionava) --- */
        #controls-panel { 
            position: absolute;
            bottom: 0; left: 0; width: 100%;
            background: rgba(30, 41, 59, 0.98); 
            border-top: 1px solid #475569; 
            border-radius: 24px 24px 0 0;
            /* Esconde a maior parte, deixa só a alça visível */
            transform: translateY(calc(100% - 60px)); 
            transition: transform 0.25s ease-out; /* Mais rápido */
            z-index: 50; 
            height: 85vh; /* Altura fixa para scroll */
            display: flex; flex-direction: column;
            padding-bottom: env(safe-area-inset-bottom, 20px);
        }
        #controls-panel.open { transform: translateY(0); }

        /* Alça do Painel */
        #panel-handle {
            width: 100%; height: 60px;
            display: flex; justify-content: center; align-items: center;
            cursor: pointer; flex-shrink: 0; background: transparent;
        }
        .handle-bar { width: 50px; height: 6px; background: #64748b; border-radius: 10px; }

        /* Abas dentro do painel */
        .tabs-container {
            display: flex; overflow-x: auto; padding: 10px 20px; gap: 10px;
            border-bottom: 1px solid #475569; flex-shrink: 0;
            scrollbar-width: none;
        }
        .tabs-container::-webkit-scrollbar { display: none; }

        .tab-btn { 
            padding: 10px 18px; border-radius: 12px; font-size: 13px; font-weight: bold; 
            color: #94a3b8; background: rgba(255,255,255,0.05); border: 1px solid transparent; white-space: nowrap;
        }
        .tab-btn.active { 
            background: var(--primary); color: white; box-shadow: 0 4px 10px rgba(99, 102, 241, 0.3);
        }

        /* Conteúdo das Abas */
        .panel-content { flex: 1; overflow-y: auto; padding: 20px; -webkit-overflow-scrolling: touch; }
        .tab-content { display: none; }
        .tab-content.active { display: block; animation: fadeIn 0.2s; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }

        /* --- Componentes --- */
        .card { background: rgba(255, 255, 255, 0.05); border-radius: 16px; padding: 16px; border: 1px solid rgba(255, 255, 255, 0.1); margin-bottom: 12px; }
        .btn-action { background: var(--primary); color: white; padding: 14px; border-radius: 14px; font-weight: bold; width: 100%; text-align: center; }
        .input-dark { background: #0f172a; border: 1px solid #475569; color: white; padding: 14px; border-radius: 14px; width: 100%; outline: none; }
        
        #feedback-overlay { position: fixed; inset: 0; z-index: 70; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.15s; }
        .scan-success { background: rgba(34, 197, 94, 0.9); }
        .scan-error { background: rgba(239, 68, 68, 0.9); }
        .scan-warning { background: rgba(234, 179, 8, 0.9); }

        .hidden { display: none !important; }
        #ocr-loading, #login-modal { position: fixed; inset: 0; z-index: 90; background: #0f172a; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        #undo-btn { position: absolute; top: 100px; right: 20px; z-index: 35; background: #ef4444; color: white; width: 56px; height: 56px; border-radius: 50%; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s; box-shadow: 0 4px 10px rgba(0,0,0,0.4); }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }
        #flowchart-canvas-container { position: absolute; left: -9999px; }
    </style>
</head>
<body>

    <div id="top-bar">
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-full bg-indigo-600 flex items-center justify-center text-white font-bold text-lg shadow-lg">
                <i class="ri-user-3-line"></i>
            </div>
            <div>
                <div id="status-operator" class="text-white font-bold leading-tight drop-shadow-md">--</div>
                <div id="status-zone" class="text-xs text-cyan-400 font-mono bg-black/50 px-2 py-0.5 rounded inline-block mt-0.5 backdrop-blur-sm">ZONA: --</div>
            </div>
        </div>
        <div class="flex gap-2">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_8px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay"></div>

    <div id="feedback-overlay">
        <div class="text-center p-6 bg-black/60 backdrop-blur-xl rounded-3xl border border-white/10 shadow-2xl transform scale-110">
            <i id="fb-icon" class="ri-checkbox-circle-fill text-7xl text-white mb-2 block"></i>
            <div id="fb-msg" class="text-4xl font-black text-white mb-2 tracking-tight">ENCONTRADO</div>
            <div id="fb-desc" class="text-sm text-gray-200 bg-white/10 p-3 rounded-xl mb-1 font-medium leading-snug">Descrição do produto</div>
            <code id="fb-id" class="text-xs font-mono text-gray-400 tracking-wider">ID: 123456</code>
        </div>
    </div>

    <button id="undo-btn"><i class="ri-arrow-go-back-line text-2xl"></i></button>

    <div id="login-modal">
        <div class="w-full max-w-xs text-center p-6">
            <div class="w-24 h-24 bg-indigo-600 rounded-3xl mx-auto mb-8 flex items-center justify-center text-5xl text-white shadow-2xl shadow-indigo-500/30">
                <i class="ri-box-3-fill"></i>
            </div>
            <h1 class="text-3xl font-bold text-white mb-2">Olá!</h1>
            <p class="text-slate-400 mb-8 text-lg">Vamos começar a bipar?</p>
            <input type="text" id="operator-input" class="input-dark text-center text-xl mb-4 font-bold" placeholder="Seu Nome">
            <button onclick="doLogin()" class="btn-action text-lg py-4">ENTRAR</button>
        </div>
    </div>

    <div id="controls-panel">
        <div id="panel-handle" onclick="togglePanel()">
            <div class="handle-bar"></div>
        </div>

        <div class="tabs-container">
            <button class="tab-btn active" onclick="switchTab('procurar', this)"><i class="ri-search-line mr-1"></i> Scan</button>
            <button class="tab-btn" onclick="switchTab('zonas', this)"><i class="ri-map-pin-line mr-1"></i> Zonas</button>
            <button class="tab-btn" onclick="switchTab('inventario', this)"><i class="ri-archive-line mr-1"></i> Listas</button>
            <button class="tab-btn" onclick="switchTab('dashboard', this)"><i class="ri-pie-chart-2-line mr-1"></i> Dash</button>
            <button class="tab-btn" onclick="switchTab('log', this)"><i class="ri-file-list-3-line mr-1"></i> Log</button>
        </div>

        <div class="panel-content">
            
            <div id="view-procurar" class="tab-content active">
                <div class="space-y-4">
                    <div class="flex gap-3">
                        <input type="text" id="manual-input" class="input-dark" placeholder="Digitar ID manual...">
                        <button id="manual-btn" class="bg-indigo-600 text-white px-5 rounded-xl"><i class="ri-check-line text-2xl"></i></button>
                    </div>

                    <div class="grid grid-cols-2 gap-3">
                         <div class="card flex items-center justify-between p-4 cursor-pointer" onclick="toggleFastMode()" id="fast-mode-card">
                            <span class="text-xs font-bold text-gray-300 uppercase">Modo Rápido</span>
                            <div class="w-4 h-4 rounded-full bg-gray-600 transition-colors" id="fast-mode-dot"></div>
                        </div>
                        <button onclick="toggleHuntMode()" id="hunt-btn" class="card flex items-center justify-center gap-2 p-4 active:bg-indigo-900/50 transition">
                            <i class="ri-focus-3-line text-xl text-indigo-400"></i>
                            <span class="text-xs font-bold text-gray-300 uppercase" id="hunt-text">Caçar ID</span>
                        </button>
                    </div>
                    
                    <div id="hunt-input-container" class="hidden animate-fade-in mt-2">
                         <input type="text" id="hunt-target-id" class="input-dark border-indigo-500" placeholder="Qual ID procurar?">
                    </div>

                    <div class="mt-6 border-t border-gray-700 pt-4">
                         <button onclick="clearSession()" class="w-full py-3 text-xs text-red-400 hover:text-red-300 border border-red-900/30 bg-red-900/10 rounded-xl mb-2 flex items-center justify-center gap-2">
                            <i class="ri-delete-bin-line"></i> Limpar Sessão
                         </button>
                         <button onclick="logout()" class="w-full py-3 text-xs text-gray-400 hover:text-white border border-gray-700 rounded-xl">Sair</button>
                    </div>
                </div>
            </div>

            <div id="view-zonas" class="tab-content">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-bold text-white">Zonas Ativas</h2>
                    <button class="text-xs text-red-400 border border-red-900/50 px-3 py-1 rounded-lg bg-red-900/10" onclick="window.setZone(null)">Sair da Zona</button>
                </div>
                <div id="zones-list" class="space-y-3">
                    </div>
            </div>

            <div id="view-inventario" class="tab-content">
                <h2 class="text-lg font-bold text-white mb-4">Carregar Listas</h2>
                
                <div class="grid grid-cols-2 gap-3 mb-6">
                    <button onclick="document.getElementById('file-input').click()" class="bg-gray-800 p-5 rounded-2xl border border-gray-600 flex flex-col items-center gap-2 active:bg-gray-700 transition">
                        <i class="ri-file-excel-2-fill text-3xl text-green-500"></i>
                        <span class="text-xs font-bold uppercase tracking-wider">Planilha</span>
                    </button>
                    <button onclick="document.getElementById('image-input').click()" class="bg-gray-800 p-5 rounded-2xl border border-gray-600 flex flex-col items-center gap-2 active:bg-gray-700 transition">
                        <i class="ri-camera-fill text-3xl text-blue-500"></i>
                        <span class="text-xs font-bold uppercase tracking-wider">Foto OCR</span>
                    </button>
                </div>

                <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                <input type="file" id="image-input" class="hidden" accept="image/*">
                <input type="file" id="inventory-file-input" class="hidden" accept=".xlsx,.csv,.pdf,.txt,.html">

                <div id="file-status" class="hidden bg-green-900/20 border border-green-500/30 p-4 rounded-xl flex items-center gap-3">
                    <i class="ri-check-double-line text-green-400 text-xl"></i>
                    <div>
                        <div class="text-xs text-green-400 font-bold uppercase">Carregado</div>
                        <div id="file-count" class="text-sm text-white">0 IDs prontos</div>
                    </div>
                </div>

                <div class="mt-6">
                    <h3 class="text-xs text-gray-500 font-bold uppercase mb-3 ml-1">Listas por Zona</h3>
                    <div id="inventory-zones-list" class="space-y-3">
                         </div>
                </div>
            </div>

            <div id="view-dashboard" class="tab-content text-center">
                <div class="flex justify-center mb-6"><div class="relative w-40 h-40"><canvas id="progressChart"></canvas><div class="absolute inset-0 flex items-center justify-center text-3xl font-bold" id="progress-text">0%</div></div></div>
                <div class="grid grid-cols-2 gap-4 mb-6">
                    <div class="card bg-gray-800"><div class="text-3xl font-bold text-indigo-400" id="kpi-total">0</div><div class="text-xs text-gray-400 uppercase font-bold">Total</div></div>
                    <div class="card bg-gray-800"><div class="text-3xl font-bold text-red-400" id="kpi-error">0</div><div class="text-xs text-gray-400 uppercase font-bold">Erros</div></div>
                </div>
                <button onclick="downloadExcel()" class="btn-action bg-green-600 mb-3"><i class="ri-file-excel-2-line mr-2"></i> Relatório Excel</button>
            </div>

            <div id="view-log" class="tab-content">
                <div id="scan-log-list" class="space-y-2"></div>
            </div>

        </div>
    </div>

    <div id="ocr-loading" class="hidden"><i class="ri-loader-4-line text-6xl text-indigo-500 animate-spin mb-4"></i><p class="text-xl font-bold text-white">Lendo Imagem...</p></div>
    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = {
            STORAGE_KEY: 'natefy_turbo_v12',
            WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9'
        };

        let state = {
            operator: null, idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null,
            inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}],
            zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null, html5QrCode: null
        };

        document.addEventListener('DOMContentLoaded', () => { init(); });

        function init() {
            loadState();
            checkLogin();
            setupActions();
            updateUI();
        }

        // --- PAINEL (Toggle) ---
        window.togglePanel = function() {
            const p = document.getElementById('controls-panel');
            p.classList.toggle('open');
            // Se fechou, garante que o scanner está ativo
            if(!p.classList.contains('open')) {
                if(state.html5QrCode && state.html5QrCode.getState() === 3) state.html5QrCode.resume();
            }
        };

        window.switchTab = function(tabName, btn) {
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
            
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById(`view-${tabName}`).classList.add('active');
            
            if(tabName === 'dashboard') updateDashboard();
            if(tabName === 'zonas') renderZones();
            if(tabName === 'log') updateUI();
            if(tabName === 'inventario') renderInventoryZones();
        };

        // --- LOGIN ---
        window.doLogin = function() {
            const val = document.getElementById('operator-input').value.trim();
            if(val) { state.operator = val; saveState(); checkLogin(); } else alert("Nome obrigatório");
        };

        function checkLogin() {
            if (state.operator) {
                document.getElementById('login-modal').classList.add('hidden');
                document.getElementById('status-operator').innerText = state.operator;
                startScanner();
            } else {
                document.getElementById('login-modal').classList.remove('hidden');
            }
        }
        window.logout = function() { if(confirm("Sair?")) { state.operator = null; saveState(); location.reload(); } };

        // --- SCANNER (TURBO 30FPS) ---
        function startScanner() {
            if(state.html5QrCode) return;
            state.html5QrCode = new Html5Qrcode("reader");
            // Aumentei FPS para 30 e qrbox um pouco maior
            state.html5QrCode.start({ facingMode: "environment" }, { fps: 30, qrbox: 350 }, 
                (decodedText) => processScan(decodedText)
            ).catch(err => console.log("Câmera:", err));
        }

        function processScan(id) {
            const now = Date.now();
            if (now - state.lastScanTime < 800) return; // Debounce reduzido para 800ms
            state.lastScanTime = now;
            if (state.isPaused) return;
            if (!state.isFastMode) state.isPaused = true;

            id = id.trim();

            if (state.huntMode && state.huntMode.isActive) {
                if (id === state.huntMode.targetId) {
                    showFeedback('success', 'ALVO ENCONTRADO!', '', id);
                    sendToN8n({ scannedId: id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: state.operator });
                    toggleHuntMode(); return;
                }
            }

            const desc = state.idDescriptions.get(id) || "Sem descrição";
            let status = "NÃO ENCONTRADO", type = "error", feedbackMsg = "NÃO ENCONTRADO", icon="ri-close-circle-fill";

            if (state.activeZone) {
                for (const [zId, zSet] of state.zoneData) {
                    if (zId !== state.activeZone && zSet.has(id)) {
                        status = "MISSORT"; type = "warning"; feedbackMsg = `MISSORT (${zId})`; icon="ri-error-warning-fill"; break;
                    }
                }
            }
            if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) {
                status = "DUPLICADO"; type = "warning"; feedbackMsg = "JÁ BIPADO"; icon="ri-history-line";
            }
            if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) {
                status = "SUCESSO"; type = "success"; feedbackMsg = "ENCONTRADO"; icon="ri-checkbox-circle-fill";
                state.idsToFind.delete(id);
            }

            const entry = { id, status, desc, type, time: new Date().toISOString(), operator: state.operator, zone: state.activeZone };
            state.logs.unshift(entry);
            if (status === "SUCESSO") state.foundIds.unshift(entry);
            state.lastUndo = entry;

            saveState();
            showFeedback(type, feedbackMsg, desc, id, icon);
            updateUI();
            sendToN8n(entry);
        }

        function showFeedback(type, msg, desc, id, iconClass) {
            const overlay = document.getElementById('feedback-overlay');
            const icon = document.getElementById('fb-icon');
            const txt = document.getElementById('fb-msg');
            const dsc = document.getElementById('fb-desc');
            const idtxt = document.getElementById('fb-id');
            const undo = document.getElementById('undo-btn');
            const colors = { success: '#22c55e', error: '#ef4444', warning: '#f59e0b' };

            icon.style.color = colors[type];
            icon.className = iconClass || 'ri-checkbox-circle-fill';
            txt.innerText = msg; dsc.innerText = desc; idtxt.innerText = `ID: ${id}`;
            overlay.className = `fixed inset-0 z-70 flex items-center justify-center p-6 transition-opacity duration-150 scan-${type}`;
            overlay.style.opacity = '1';
            
            undo.classList.add('visible');
            setTimeout(() => undo.classList.remove('visible'), 5000);
            if(navigator.vibrate) navigator.vibrate(200);
            
            // Tempo reduzido para feedback visual (mais rápido)
            setTimeout(() => { 
                overlay.style.opacity = '0'; 
                if (!state.isFastMode) state.isPaused = false; 
            }, state.isFastMode ? 300 : 1200);
        }

        // --- ZONAS (Restaurado) ---
        function renderZones() {
            const c = document.getElementById('zones-list');
            c.innerHTML = `<button class="btn-action bg-gray-600 mb-3 text-sm" onclick="window.setZone(null)">🚫 Sem Zona Ativa</button>`;
            state.inventoryZones.forEach(z => {
                const isActive = state.activeZone === z.id;
                c.innerHTML += `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700 ${isActive ? 'border-green-500' : ''}">
                    <span class="font-bold">${z.name}</span>
                    <button class="py-2 px-4 rounded-lg text-xs font-bold ${isActive ? 'bg-green-600 text-white' : 'bg-indigo-600 text-white'}" onclick="window.setZone('${z.id}')">${isActive ? 'ATIVA' : 'ATIVAR'}</button>
                </div>`;
            });
        }
        function renderInventoryZones() {
            const c = document.getElementById('inventory-zones-list');
            c.innerHTML = '';
            state.inventoryZones.forEach(z => {
                const count = state.zoneData.get(z.id)?.size || 0;
                c.innerHTML += `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700">
                    <div><div class="font-bold">${z.name}</div><div class="text-xs text-gray-400">${count} IDs</div></div>
                    <button class="py-2 px-4 rounded-lg text-xs font-bold bg-indigo-600 text-white" onclick="window.tempZone='${z.id}';document.getElementById('inventory-file-input').click()"><i class="ri-upload-cloud-line mr-1"></i> Carregar</button>
                </div>`;
            });
        }
        window.setZone = (id) => { state.activeZone = id; document.getElementById('status-zone').innerText = `ZONA: ${id ? id.toUpperCase() : '--'}`; saveState(); renderZones(); };

        // --- UTILS ---
        window.toggleFastMode = function() {
            state.isFastMode = !state.isFastMode;
            const btn = document.getElementById('fast-mode-btn');
            const dot = document.getElementById('fast-mode-dot');
            if(state.isFastMode) { btn.classList.add('active'); dot.classList.add('bg-green-500'); } 
            else { btn.classList.remove('active'); dot.classList.remove('bg-green-500'); }
        };
        
        window.toggleHuntMode = function() {
            const input = document.getElementById('hunt-input-container');
            const btn = document.getElementById('hunt-toggle-btn');
            const t = document.getElementById('hunt-target-id');
            
            if(state.huntMode && state.huntMode.isActive) {
                state.huntMode={isActive:false, targetId:null};
                input.classList.add('hidden');
                btn.innerText='Ativar'; btn.classList.remove('bg-red-600'); btn.classList.add('bg-indigo-600');
            } else {
                if(input.classList.contains('hidden')) {
                    input.classList.remove('hidden'); // Mostra input
                    btn.innerText='Confirmar';
                } else {
                    const v = t.value.trim(); if(!v) return;
                    state.huntMode={isActive:true, targetId:v};
                    input.classList.add('hidden');
                    btn.innerText='Cancelar Caça'; btn.classList.remove('bg-indigo-600'); btn.classList.add('bg-red-600');
                }
            }
        };

        function sendToN8n(data) { if(CONFIG.WEBHOOK.includes("https")) fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline")); }
        function saveState() {
            const s = { ...state, html5QrCode: null, chart: null };
            s.idsToFind = Array.from(s.idsToFind); s.idDescriptions = Array.from(s.idDescriptions.entries()); s.zoneData = Array.from(s.zoneData.entries()).map(([k,v])=>[k,Array.from(v)]);
            localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s));
        }
        function loadState() {
            const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY));
            if(s) { state = { ...state, ...s }; state.idsToFind = new Set(s.idsToFind); state.idDescriptions = new Map(s.idDescriptions); state.zoneData = new Map(s.zoneData.map(([k,v])=>[k,new Set(v)])); }
        }
        window.clearSession = function() { if(confirm('Apagar?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } };
        function handleFile(file) {
            if(!file) return; const reader = new FileReader();
            reader.onload = (e) => {
                const data = new Uint8Array(e.target.result); const wb = XLSX.read(data, {type: 'array'});
                const json = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], {header: 1});
                const ids = json.map(r => String(r[0])).filter(i => i && i.match(/\d+/));
                state.idsToFind = new Set(ids); state.foundIds = [];
                document.getElementById('file-status').classList.remove('hidden'); document.getElementById('file-count').innerText = `${ids.length} itens prontos`;
                saveState(); alert(`${ids.length} itens carregados.`);
            }; reader.readAsArrayBuffer(file);
        }
        async function handleOCR(file) {
            if (!file) return; document.getElementById('ocr-loading').classList.remove('hidden');
            try {
                const worker = Tesseract.createWorker(); await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                const { data: { text } } = await worker.recognize(file); await worker.terminate();
                const lines = text.split('\n'); let count = 0; const newIds = new Set();
                lines.forEach(line => {
                    const match = line.replace(/[^\w\s\>\-\.\(\)\/]/gi, '').match(/(\d{8,14})/);
                    if (match) { const id = match[0]; let desc = line.replace(id, '').replace(/^[\s\>\-\.]+/g, '').trim(); if (desc.length < 3) desc = "Sem Descrição"; newIds.add(id); state.idDescriptions.set(id, desc); count++; }
                });
                if (count > 0) { state.idsToFind = newIds; state.foundIds = []; document.getElementById('file-status').classList.remove('hidden'); document.getElementById('file-count').innerText = `${count} itens prontos`; saveState(); } else alert("Nada encontrado.");
            } catch (e) { alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); }
        }
        function handleInventoryFile(file) {
            if(!file || !window.tempZone) return; const reader = new FileReader();
            reader.onload = (e) => {
                const data = new Uint8Array(e.target.result); const wb = XLSX.read(data, {type: 'array'});
                const json = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], {header: 1});
                const ids = new Set(json.map(r => String(r[0])).filter(i => i && i.match(/\d+/)));
                state.zoneData.set(window.tempZone, ids); saveState(); renderInventoryZones(); alert("Carregado!");
            }; reader.readAsArrayBuffer(file);
        }
        function doUndo() { if(!state.lastUndo) return; state.logs.shift(); if (state.lastUndo.status === "SUCESSO") { state.foundIds.shift(); state.idsToFind.add(state.lastUndo.id); } sendToN8n({ ...state.lastUndo, status: "CANCELADO" }); state.lastUndo = null; document.getElementById('undo-btn').classList.remove('visible'); updateUI(); alert("Desfeito!"); }
        function updateUI() {
            const list = document.getElementById('scan-log-list');
            list.innerHTML = state.logs.slice(0, 50).map(l => `<div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700"><div><div class="font-bold text-sm">${l.id}</div><div class="text-xs text-gray-400 truncate w-48">${l.desc}</div></div><div class="text-right"><div class="text-xs font-bold ${l.status==='SUCESSO'?'text-green-500':(l.status==='ERRO'?'text-red-500':'text-yellow-500')}">${l.status}</div><div class="text-xs text-gray-500">${new Date(l.time).toLocaleTimeString()}</div></div></div>`).join('');
        }
        function updateDashboard() {
            const total = state.idsToFind.size + state.foundIds.length; const pct = total > 0 ? Math.round((state.foundIds.length / total) * 100) : 0;
            const ctx = document.getElementById('progressChart');
            if (ctx) { if(state.chart) state.chart.destroy(); state.chart = new Chart(ctx, { type: 'doughnut', data: { datasets: [{ data: [pct, 100-pct], backgroundColor: ['#6366F1', '#374151'], borderWidth: 0 }] }, options: { cutout: '80%', plugins: { tooltip: { enabled: false } } } }); }
            document.getElementById('progress-text').innerText = pct + "%"; document.getElementById('kpi-total').innerText = state.logs.length; document.getElementById('kpi-error').innerText = state.logs.filter(l=>l.status==='ERRO' || l.status==='MISSORT').length;
        }
        function downloadExcel() { if(state.logs.length === 0) return alert("Sem dados"); const ws = XLSX.utils.json_to_sheet(state.logs); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Logs"); XLSX.writeFile(wb, "Relatorio.xlsx"); }
        async function downloadFlow() { if(state.logs.length === 0) return alert("Sem dados"); const container = document.getElementById('flowchart-canvas-container'); container.innerHTML = `<div style="padding:50px; background:#1E1B4B; color:white; text-align:center"><h1>Relatório Visual</h1><h2>${state.operator}</h2><div style="display:flex; justify-content:center; gap:20px; margin-top:50px"><div style="border:2px solid #10B981; padding:20px">SUCESSO: ${state.foundIds.length}</div><div style="border:2px solid #EF4444; padding:20px">ERROS: ${state.logs.filter(l=>l.status!=='SUCESSO').length}</div></div></div>`; const canvas = await html2canvas(container); const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click(); }
    </script>
</body>
</html>
