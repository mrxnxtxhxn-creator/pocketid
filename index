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
        /* --- Variáveis e Base --- */
        :root { --primary: #6366f1; --bg: #0f172a; --panel: #1e293b; }
        body, html { height: 100%; width: 100%; overflow: hidden; background: var(--bg); font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; }
        
        /* --- Scanner Fullscreen --- */
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

        /* --- MIRA DE SCAN (AUMENTADA) --- */
        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 80vw; height: 80vw; /* Quadrado Grande */
            max-width: 400px; max-height: 400px;
            border: 2px solid rgba(255, 255, 255, 0.6);
            border-radius: 24px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.6); /* Escurece o fundo */
            z-index: 10; pointer-events: none;
        }
        /* Cantos da mira para dar estilo */
        .scan-overlay::before, .scan-overlay::after {
            content: ''; position: absolute; width: 40px; height: 40px; border: 4px solid #6366f1;
            border-radius: 4px; pointer-events: none;
        }
        .scan-overlay::before { top: -2px; left: -2px; border-bottom: 0; border-right: 0; }
        .scan-overlay::after { bottom: -2px; right: -2px; border-top: 0; border-left: 0; }

        /* --- Top Bar --- */
        #top-bar {
            position: absolute; top: 0; left: 0; width: 100%; z-index: 20;
            padding: env(safe-area-inset-top) 20px 10px 20px;
            background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent);
            display: flex; justify-content: space-between; align-items: start;
        }

        /* --- Menu Inferior (Nav) --- */
        #bottom-nav {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 80px;
            background: rgba(30, 41, 59, 0.95);
            backdrop-filter: blur(10px);
            border-top: 1px solid rgba(255,255,255,0.1);
            display: flex; justify-content: space-around; align-items: center;
            padding-bottom: env(safe-area-inset-bottom);
            z-index: 50;
        }
        .nav-item {
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            color: #94a3b8; font-size: 11px; gap: 4px; width: 20%; transition: all 0.2s;
        }
        .nav-item i { font-size: 24px; transition: transform 0.2s; }
        .nav-item.active { color: #6366f1; }
        .nav-item.active i { transform: translateY(-2px); }

        /* --- Painel Deslizante --- */
        #controls-panel {
            position: absolute; bottom: 80px; left: 0; width: 100%;
            background: var(--panel);
            border-radius: 24px 24px 0 0;
            transform: translateY(110%); /* Escondido */
            transition: transform 0.3s cubic-bezier(0.2, 0.8, 0.2, 1);
            z-index: 40; max-height: 80vh; display: flex; flex-direction: column;
            box-shadow: 0 -5px 20px rgba(0,0,0,0.3);
        }
        #controls-panel.open { transform: translateY(0); }

        /* --- Conteúdo das Abas --- */
        .panel-content { padding: 20px; overflow-y: auto; flex: 1; }
        .tab-content { display: none; animation: fadeIn 0.3s; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* --- Botões e Cards --- */
        .btn-card {
            background: #334155; border: 1px solid #475569; border-radius: 16px;
            padding: 16px; display: flex; flex-direction: column; align-items: center; justify-content: center;
            gap: 8px; color: white; transition: background 0.2s;
        }
        .btn-card:active { background: #475569; }
        
        /* Botão Toggle Modo Rápido */
        .toggle-btn {
            background: #334155; border: 1px solid #475569; padding: 12px; border-radius: 12px;
            display: flex; align-items: center; justify-content: space-between; color: white;
        }
        .toggle-btn.active { background: rgba(16, 185, 129, 0.2); border-color: #10b981; color: #10b981; }
        .toggle-indicator { width: 12px; height: 12px; border-radius: 50%; background: #94a3b8; transition: background 0.3s; }
        .toggle-btn.active .toggle-indicator { background: #10b981; box-shadow: 0 0 10px #10b981; }

        /* Feedback Overlay */
        #feedback-overlay { position: fixed; inset: 0; z-index: 60; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.9); }
        .scan-error { background: rgba(239, 68, 68, 0.9); }
        .scan-warning { background: rgba(234, 179, 8, 0.9); }

        /* Utilitários */
        .hidden { display: none !important; }
        .input-dark { background: #0f172a; border: 1px solid #475569; color: white; padding: 12px; border-radius: 12px; width: 100%; outline: none; }
        .btn-action { background: #6366f1; color: white; padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; }
        
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
                <div id="status-zone" class="text-xs text-cyan-400 font-mono">Zona: --</div>
            </div>
        </div>
        <div class="flex gap-2">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_8px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay"></div>

    <div id="feedback-overlay">
        <div class="text-center p-6 bg-black/40 backdrop-blur-md rounded-2xl border border-white/10">
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
        <div class="w-full max-w-xs text-center">
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
        <div class="w-full flex justify-center pt-3 pb-1" onclick="togglePanel()">
            <div class="w-12 h-1.5 bg-slate-600 rounded-full"></div>
        </div>
        
        <button onclick="logout()" class="absolute top-3 right-4 text-slate-400 hover:text-red-400 text-sm flex items-center gap-1">
            <i class="ri-logout-box-r-line"></i> Sair
        </button>

        <div class="panel-content">
            
            <div id="view-procurar" class="tab-content active">
                <div class="space-y-4">
                    
                    <div class="grid grid-cols-2 gap-3">
                        <button onclick="document.getElementById('file-input').click()" class="btn-card">
                            <i class="ri-file-excel-2-fill text-2xl text-green-500"></i>
                            <span class="text-xs font-bold uppercase">Planilha</span>
                        </button>
                        <button onclick="document.getElementById('image-input').click()" class="btn-card">
                            <i class="ri-camera-fill text-2xl text-cyan-400"></i>
                            <span class="text-xs font-bold uppercase">Foto OCR</span>
                        </button>
                    </div>

                    <div id="file-status" class="hidden bg-green-900/30 border border-green-500/30 p-3 rounded-xl flex items-center gap-3">
                        <i class="ri-check-double-line text-green-400 text-xl"></i>
                        <div class="flex flex-col text-left">
                            <span class="text-xs text-green-400 font-bold uppercase">Lista Carregada</span>
                            <span id="file-count" class="text-xs text-slate-300">0 itens prontos</span>
                        </div>
                    </div>

                    <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                    <input type="file" id="image-input" class="hidden" accept="image/*">

                    <div class="flex gap-2">
                        <input type="text" id="manual-input" class="input-dark" placeholder="Digitar ID manual...">
                        <button id="manual-btn" class="bg-indigo-600 text-white px-4 rounded-xl"><i class="ri-check-line text-xl"></i></button>
                    </div>

                    <div class="toggle-btn" id="fast-mode-btn" onclick="toggleFastMode()">
                        <span class="text-sm font-medium">Modo Rápido (Sem Pausa)</span>
                        <div class="toggle-indicator"></div>
                    </div>

                    <div class="bg-slate-800/50 p-4 rounded-xl border border-slate-700">
                        <div class="flex justify-between items-center mb-3">
                            <label class="text-xs text-slate-300 font-bold uppercase flex items-center gap-2">
                                <i class="ri-focus-3-line text-indigo-400"></i> Modo Caça
                            </label>
                            <span id="hunt-status" class="text-xs text-indigo-400 font-mono"></span>
                        </div>
                        <div class="flex gap-2">
                            <input type="text" id="hunt-target-id" class="input-dark text-sm p-2" placeholder="ID Alvo">
                            <button id="hunt-toggle-btn" onclick="toggleHuntMode()" class="bg-indigo-600 px-4 rounded-lg text-xs font-bold text-white uppercase">Ativar</button>
                        </div>
                    </div>
                     
                     <button onclick="clearSession()" class="w-full py-3 text-xs text-red-400 hover:text-red-300 border border-red-900/50 rounded-xl mt-2">
                        <i class="ri-delete-bin-line mr-1"></i> Limpar Dados da Sessão
                     </button>
                </div>
            </div>

            <div id="view-log" class="tab-content">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-bold text-white">Histórico</h2>
                    <span class="text-xs bg-slate-800 px-2 py-1 rounded text-slate-400" id="log-count">0</span>
                </div>
                <div id="scan-log-list" class="space-y-2"></div>
            </div>

            <div id="view-dashboard" class="tab-content text-center">
                <h2 class="text-lg font-bold text-white mb-6">Performance</h2>
                <div class="flex justify-center mb-6">
                    <div class="relative w-40 h-40">
                        <canvas id="progressChart"></canvas>
                        <div class="absolute inset-0 flex items-center justify-center flex-col">
                            <span class="text-3xl font-bold text-white" id="progress-text">0%</span>
                            <span class="text-[10px] text-slate-400 uppercase">Concluído</span>
                        </div>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700">
                        <div class="text-3xl font-bold text-indigo-400" id="kpi-total">0</div>
                        <div class="text-[10px] text-slate-500 uppercase font-bold tracking-wider">Bips Totais</div>
                    </div>
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700">
                        <div class="text-3xl font-bold text-red-400" id="kpi-error">0</div>
                        <div class="text-[10px] text-slate-500 uppercase font-bold tracking-wider">Erros</div>
                    </div>
                </div>
                 <button onclick="downloadExcel()" class="btn-action bg-green-600 mb-2"><i class="ri-file-excel-2-line mr-2"></i> Relatório Excel</button>
                 <button onclick="downloadFlow()" class="btn-action bg-purple-600"><i class="ri-flow-chart mr-2"></i> Fluxograma</button>
            </div>

            <div id="view-zonas" class="tab-content">
                <h2 class="text-lg font-bold text-white mb-4">Zonas de Inventário</h2>
                <div id="zones-list" class="space-y-3"></div>
            </div>
            
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="navClick('procurar', this)">
            <i class="ri-qr-scan-2-line"></i>
            <span>Scan</span>
        </div>
        <div class="nav-item" onclick="navClick('log', this)">
            <i class="ri-file-list-3-line"></i>
            <span>Log</span>
        </div>
        <div class="nav-item" onclick="navClick('dashboard', this)">
            <i class="ri-pie-chart-2-line"></i>
            <span>Dash</span>
        </div>
        <div class="nav-item" onclick="navClick('zonas', this)">
            <i class="ri-map-pin-line"></i>
            <span>Zonas</span>
        </div>
    </nav>

    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = {
            STORAGE_KEY: 'natefy_v10',
            WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9'
        };

        let state = {
            operator: null, idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null,
            inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}],
            zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null, html5QrCode: null
        };

        // --- INIT ---
        document.addEventListener('DOMContentLoaded', () => {
            init();
            
            // CORREÇÃO DE CONGELAMENTO: Reinicia scanner ao voltar para a aba
            document.addEventListener("visibilitychange", () => {
                if (document.visibilityState === "visible" && state.operator) {
                    // Pequeno delay para garantir que o navegador "acordou"
                    setTimeout(() => {
                        if(!state.html5QrCode) startScanner();
                    }, 500);
                }
            });
        });

        function init() {
            loadState();
            checkLogin();
            if(state.operator) updateUI();
        }

        // --- LOGIN ---
        window.doLogin = function() {
            const val = document.getElementById('operator-input').value.trim();
            if(val) { state.operator = val; saveState(); checkLogin(); } 
            else alert("Nome obrigatório");
        };

        function checkLogin() {
            if (state.operator) {
                document.getElementById('login-modal').classList.add('hidden');
                document.getElementById('status-operator').innerText = state.operator;
                startScanner();
                updateUI();
            } else {
                document.getElementById('login-modal').classList.remove('hidden');
            }
        }

        window.logout = function() {
            if(confirm("Sair?")) { 
                state.operator = null; saveState(); location.reload(); 
            }
        };

        // --- NAVEGAÇÃO & PAINEL ---
        window.togglePanel = function(forceState) {
            const p = document.getElementById('controls-panel');
            const isOpen = forceState !== undefined ? forceState : !p.classList.contains('open');
            
            if(isOpen) p.classList.add('open');
            else p.classList.remove('open');
        };

        window.navClick = function(tab, el) {
            // Highlight Tab
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            el.classList.add('active');

            // Show Content
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById(`view-${tab}`).classList.add('active');

            const p = document.getElementById('controls-panel');
            const isScan = tab === 'procurar';

            if (isScan) {
                // Se clicou em Scan: 
                // Se o painel estava fechado -> abre (Config)
                // Se estava aberto -> fecha (Scan)
                togglePanel(!p.classList.contains('open'));
            } else {
                // Outras abas sempre abrem o painel
                togglePanel(true);
            }

            if(tab === 'dashboard') updateDashboard();
            if(tab === 'zonas') renderZones();
            if(tab === 'log') updateUI();
        };

        // --- SCANNER ---
        function startScanner() {
            if(state.html5QrCode) return; // Já rodando
            const html5QrCode = new Html5Qrcode("reader");
            state.html5QrCode = html5QrCode;
            
            html5QrCode.start({ facingMode: "environment" }, { fps: 15, qrbox: 300 }, 
                (decodedText) => processScan(decodedText)
            ).catch(err => {
                console.log("Câmera:", err);
                // Tenta recuperar se falhar na primeira
                setTimeout(() => html5QrCode.start({ facingMode: "environment" }, { fps: 15, qrbox: 300 }, (d)=>processScan(d)), 2000);
            });
        }

        function processScan(id) {
            const now = Date.now();
            if (now - state.lastScanTime < 1000) return;
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
            let status = "NÃO ENCONTRADO", type = "error", feedbackMsg = "NÃO ENCONTRADO";
            let icon = "ri-close-circle-fill";

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
            
            txt.innerText = msg;
            dsc.innerText = desc;
            idtxt.innerText = `ID: ${id}`;
            
            overlay.className = `fixed inset-0 z-70 flex items-center justify-center p-6 transition-opacity duration-200 scan-${type}`;
            overlay.style.opacity = '1';
            
            undo.classList.add('visible');
            setTimeout(() => undo.classList.remove('visible'), 5000);
            if(navigator.vibrate) navigator.vibrate(200);
            
            setTimeout(() => { 
                overlay.style.opacity = '0'; 
                if (!state.isFastMode) state.isPaused = false; 
            }, state.isFastMode ? 500 : 1500);
        }

        // --- OCR & FILES ---
        window.toggleFastMode = function() {
            state.isFastMode = !state.isFastMode;
            const btn = document.getElementById('fast-mode-btn');
            if(state.isFastMode) btn.classList.add('active'); else btn.classList.remove('active');
        };

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
                const idRegex = /(\d{8,14})/; 

                lines.forEach(line => {
                    const cleanLine = line.replace(/[^\w\s\>\-\.\(\)\/]/gi, '');
                    const match = cleanLine.match(idRegex);
                    if (match) {
                        const id = match[0];
                        let desc = cleanLine.replace(id, '').replace(/^[\s\>\-\.]+/g, '').trim();
                        if (desc.length < 3) desc = "Sem Descrição";
                        newIds.add(id);
                        state.idDescriptions.set(id, desc);
                        count++;
                    }
                });

                if (count > 0) {
                    state.idsToFind = newIds; state.foundIds = [];
                    updateFileStatus(count);
                    saveState();
                } else alert("Nada encontrado.");
            } catch (e) { alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); }
        }

        function handleFile(file) {
            if(!file) return;
            const reader = new FileReader();
            reader.onload = (e) => {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], {header: 1});
                const ids = json.map(r => String(r[0])).filter(i => i && i.match(/\d+/));
                state.idsToFind = new Set(ids);
                state.foundIds = [];
                updateFileStatus(ids.length);
                saveState();
            };
            reader.readAsArrayBuffer(file);
        }

        function updateFileStatus(count) {
            const el = document.getElementById('file-status');
            document.getElementById('file-count').innerText = `${count} itens carregados`;
            el.classList.remove('hidden');
        }

        // --- UTILS ---
        function sendToN8n(data) {
            if(CONFIG.WEBHOOK.includes("https")) fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline"));
        }
        function saveState() {
            const s = { ...state, html5QrCode: null, chart: null };
            s.idsToFind = Array.from(s.idsToFind);
            s.idDescriptions = Array.from(s.idDescriptions.entries());
            s.zoneData = Array.from(s.zoneData.entries()).map(([k,v])=>[k,Array.from(v)]);
            localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s));
        }
        function loadState() {
            const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY));
            if(s) {
                state = { ...state, ...s };
                state.idsToFind = new Set(s.idsToFind);
                state.idDescriptions = new Map(s.idDescriptions);
                state.zoneData = new Map(s.zoneData.map(([k,v])=>[k,new Set(v)]));
                if(state.isFastMode) document.getElementById('fast-mode-btn').classList.add('active');
            }
        }
        window.clearSession = function() { if(confirm('Apagar?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } };

        // --- ZONES ---
        function renderZones() {
            const c = document.getElementById('zones-list');
            c.innerHTML = `<button class="btn btn-secondary w-full mb-3 text-sm" onclick="window.setZone(null)">🚫 Sem Zona Ativa</button>`;
            state.inventoryZones.forEach(z => {
                const isActive = state.activeZone === z.id;
                c.innerHTML += `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700 ${isActive ? 'border-green-500' : ''}">
                    <span class="font-bold">${z.name}</span>
                    <button class="btn py-2 px-4 text-xs ${isActive ? 'btn-success' : 'btn-primary'}" onclick="window.setZone('${z.id}')">${isActive ? 'ATIVA' : 'ATIVAR'}</button>
                </div>`;
            });
        }
        window.setZone = (id) => { 
            state.activeZone = id; 
            document.getElementById('status-zone').innerText = `Zona: ${id ? id.toUpperCase() : '--'}`; 
            saveState(); renderZones(); 
        };

        // --- UI ---
        function updateUI() {
            document.getElementById('log-count').innerText = state.logs.length;
            const list = document.getElementById('scan-log-list');
            list.innerHTML = state.logs.slice(0, 50).map(l => `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700">
                    <div><div class="font-bold text-sm">${l.id}</div><div class="text-xs text-gray-400 truncate w-48">${l.desc}</div></div>
                    <div class="text-right"><div class="text-xs font-bold ${l.status==='SUCESSO'?'text-green-500':(l.status==='ERRO'?'text-red-500':'text-yellow-500')}">${l.status}</div><div class="text-xs text-gray-500">${new Date(l.time).toLocaleTimeString()}</div></div>
                </div>`).join('');
        }
        
        // Funções de Undo, Download, Hunt Mode iguais ao anterior...
        function doUndo() {
            if(!state.lastUndo) return;
            state.logs.shift();
            if (state.lastUndo.status === "SUCESSO") { state.foundIds.shift(); state.idsToFind.add(state.lastUndo.id); }
            sendToN8n({ ...state.lastUndo, status: "CANCELADO" });
            state.lastUndo = null;
            document.getElementById('undo-btn').classList.remove('visible');
            updateUI(); alert("Desfeito!");
        }
        window.toggleHuntMode = function() {
            const t = document.getElementById('hunt-target-id');
            const btn = document.getElementById('hunt-toggle-btn');
            const status = document.getElementById('hunt-status');
            if(state.huntMode && state.huntMode.isActive) {
                state.huntMode={isActive:false, targetId:null}; t.disabled=false; t.value='';
                btn.innerText='Ativar'; btn.classList.remove('btn-danger'); btn.classList.add('btn-primary'); status.innerText = '';
            } else {
                const v = t.value.trim(); if(!v) return alert("ID?");
                state.huntMode={isActive:true, targetId:v}; t.disabled=true;
                btn.innerText='Cancelar'; btn.classList.remove('btn-primary'); btn.classList.add('btn-danger'); status.innerText = 'ATIVO';
            }
            saveState();
        };
        window.downloadExcel = function() {
            if(state.logs.length === 0) return alert("Sem dados");
            const ws = XLSX.utils.json_to_sheet(state.logs); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Logs"); XLSX.writeFile(wb, "Relatorio.xlsx");
        };
        window.downloadFlow = async function() {
             if(state.logs.length === 0) return alert("Sem dados");
             const container = document.getElementById('flowchart-canvas-container');
             container.innerHTML = `<div style="padding:50px; background:#1E1B4B; color:white; text-align:center"><h1>Relatório Visual</h1><h2>${state.operator}</h2><div style="display:flex; justify-content:center; gap:20px; margin-top:50px"><div style="border:2px solid #10B981; padding:20px">SUCESSO: ${state.foundIds.length}</div><div style="border:2px solid #EF4444; padding:20px">ERROS: ${state.logs.filter(l=>l.status!=='SUCESSO').length}</div></div></div>`;
             const canvas = await html2canvas(container);
             const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click();
        };
        function updateDashboard() {
            const total = state.idsToFind.size + state.foundIds.length;
            const pct = total > 0 ? Math.round((state.foundIds.length / total) * 100) : 0;
            const ctx = document.getElementById('progressChart');
            if (ctx) {
                if(state.chart) state.chart.destroy();
                state.chart = new Chart(ctx, { type: 'doughnut', data: { datasets: [{ data: [pct, 100-pct], backgroundColor: ['#6366F1', '#374151'], borderWidth: 0 }] }, options: { cutout: '80%', plugins: { tooltip: { enabled: false } } } });
            }
            document.getElementById('progress-text').innerText = pct + "%";
            document.getElementById('kpi-total').innerText = state.logs.length;
            document.getElementById('kpi-error').innerText = state.logs.filter(l=>l.status==='ERRO' || l.status==='MISSORT').length;
        }
    </script>
</body>
</html>
