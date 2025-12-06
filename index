<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Scanner Pro</title>

    <meta name="theme-color" content="#4F46E5"/> <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link rel="manifest" href="manifest.json">

    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/remixicon@3.5.0/fonts/remixicon.css">

    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
    <script src='https://unpkg.com/tesseract.js@v2.1.0/dist/tesseract.min.js'></script>
    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
    <script src="https://unpkg.com/pdfjs-dist@3.11.174/build/pdf.min.js"></script>
    <script>pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://unpkg.com/pdfjs-dist@3.11.174/build/pdf.worker.min.js';</script>

    <style>
        /* --- Variáveis de Cor --- */
        :root {
            --color-bg-main: #1E1B4B; /* Fundo escuro */
            --color-primary: #6366F1; /* Roxo principal */
            --color-primary-dark: #4F46E5; /* Roxo mais escuro */
            --color-primary-light: #818CF8; /* Roxo mais claro */
            --color-text: #F3F4F6; /* Texto claro */
            --color-text-muted: #9CA3AF; /* Texto secundário */
            --color-success: #10B981;
            --color-warning: #F59E0B;
            --color-error: #EF4444;
            --color-card-bg: #312E81; /* Fundo do cartão */
            --color-nav-bg: #1F2937; /* Fundo do menu inferior */
            --font-stack: 'Inter', sans-serif;
        }

        /* --- Reset e Layout Geral --- */
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        html, body { height: 100%; width: 100%; font-family: var(--font-stack); background-color: var(--color-bg-main); color: var(--color-text); overflow: hidden; }
        .hidden { display: none !important; }
        button { cursor: pointer; border: none; outline: none; }
        input { border: none; outline: none; }

        /* --- Câmera e Mira --- */
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }
        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 60vw; height: 60vw; /* Quadrado médio */
            border: 3px solid rgba(255, 255, 255, 0.7); border-radius: 24px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.5); /* Escurece ao redor */
            z-index: 10; pointer-events: none;
        }

        /* --- Barra Superior --- */
        #top-bar {
            position: absolute; top: 0; left: 0; width: 100%; height: 60px; z-index: 30;
            display: flex; justify-content: space-between; align-items: center;
            padding: 16px; padding-top: calc(16px + env(safe-area-inset-top));
            background: linear-gradient(to bottom, rgba(0,0,0,0.7), transparent);
        }
        .status-badge { padding: 4px 8px; border-radius: 12px; font-size: 12px; font-weight: 500; background: rgba(255,255,255,0.1); }
        .zone-badge { color: var(--color-primary-light); font-family: monospace; }

        /* --- Feedback Visual --- */
        #feedback-overlay {
            position: fixed; inset: 0; z-index: 70; display: flex; align-items: center; justify-content: center;
            opacity: 0; pointer-events: none; transition: opacity 0.2s;
        }
        .feedback-card {
            background: var(--color-card-bg); padding: 24px; border-radius: 20px; text-align: center;
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
        }
        .scan-success .feedback-card { border: 2px solid var(--color-success); }
        .scan-error .feedback-card { border: 2px solid var(--color-error); }
        .scan-warning .feedback-card { border: 2px solid var(--color-warning); }

        /* --- Botão Desfazer --- */
        #undo-btn {
            position: absolute; top: 80px; right: 20px; z-index: 40;
            background: var(--color-error); color: white; width: 56px; height: 56px; border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
            opacity: 0; pointer-events: none; transition: all 0.3s;
        }
        #undo-btn.visible { opacity: 1; pointer-events: auto; transform: translateY(0); }
        #undo-btn:active { transform: scale(0.95); }

        /* --- Menu Inferior Fixo (Redesign) --- */
        #bottom-nav {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 70px;
            background: var(--color-nav-bg);
            display: flex; justify-content: space-around; align-items: center;
            z-index: 60; padding-bottom: env(safe-area-inset-bottom);
            border-top: 1px solid #374151;
        }
        .nav-item {
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            color: var(--color-text-muted); font-size: 11px; gap: 4px; width: 20%; transition: color 0.2s;
        }
        .nav-item.active { color: var(--color-primary-light); }
        .nav-item i { font-size: 24px; transition: transform 0.2s; }
        .nav-item:active i { transform: scale(0.9); }

        /* --- Painel Deslizante (Redesign) --- */
        #controls-panel {
            position: absolute; bottom: 70px; left: 0; width: 100%;
            background: var(--color-card-bg);
            border-radius: 24px 24px 0 0;
            /* Inicia visível, mas só a alça e as abas */
            transform: translateY(calc(100% - 58px));
            transition: transform 0.4s cubic-bezier(0.33, 1, 0.68, 1); /* Efeito de mola suave */
            z-index: 50; max-height: 85vh;
            display: flex; flex-direction: column;
            box-shadow: 0 -5px 15px rgba(0,0,0,0.2);
        }
        #controls-panel.open { transform: translateY(0); }
        .panel-handle {
            width: 100%; height: 20px; display: flex; justify-content: center; align-items: center;
            flex-shrink: 0;
        }
        .handle-bar { width: 40px; height: 5px; background: #4B5563; border-radius: 10px; }

        /* --- Abas do Painel (Redesign) --- */
        .tabs-container {
            display: flex; overflow-x: auto; padding: 8px 16px; gap: 8px;
            background: var(--color-card-bg); flex-shrink: 0;
            scrollbar-width: none; /* Firefox */
        }
        .tabs-container::-webkit-scrollbar { display: none; /* Chrome/Safari */ }
        .tab-btn {
            padding: 8px 16px; border-radius: 20px; font-size: 13px; font-weight: 500;
            color: var(--color-text-muted); background: transparent;
            white-space: nowrap; transition: all 0.2s;
        }
        .tab-btn.active {
            background: var(--color-primary); color: white;
            box-shadow: 0 4px 10px rgba(99, 102, 241, 0.3);
        }

        /* --- Conteúdo do Painel --- */
        .panel-content { flex: 1; overflow-y: auto; padding: 20px; }
        .tab-content { display: none; animation: fadeIn 0.3s; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* --- Componentes Visuais (Cards, Buttons, etc.) --- */
        .card {
            background: rgba(255, 255, 255, 0.05); border-radius: 16px; padding: 16px;
            border: 1px solid rgba(255, 255, 255, 0.1); margin-bottom: 16px;
        }
        .btn {
            padding: 12px 20px; border-radius: 12px; font-weight: 600;
            display: flex; align-items: center; justify-content: center; gap: 8px;
            transition: transform 0.2s, opacity 0.2s;
        }
        .btn:active { transform: scale(0.98); opacity: 0.9; }
        .btn-primary { background: var(--color-primary); color: white; }
        .btn-danger { background: var(--color-error); color: white; }
        .btn-secondary { background: #4B5563; color: white; }
        .input-field {
            width: 100%; background: rgba(0,0,0,0.2); border: 1px solid #4B5563;
            padding: 12px; border-radius: 12px; color: var(--color-text);
        }
        .input-field::placeholder { color: #6B7280; }

        /* --- Outros --- */
        #login-modal, #ocr-loading {
            position: fixed; inset: 0; z-index: 90;
            display: flex; align-items: center; justify-content: center;
            background: var(--color-bg-main);
        }
        #ocr-loading { background: rgba(30, 27, 75, 0.9); }
        .spin { animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background: var(--color-bg-main); }

        /* --- Alerta de Upload --- */
        #upload-alert {
            background: var(--color-success); color: white; padding: 12px;
            border-radius: 12px; margin-top: 16px; text-align: center; font-weight: 500;
            display: flex; align-items: center; justify-content: center; gap: 8px;
        }
    </style>
</head>
<body>

    <div id="top-bar">
        <div>
            <div id="status-operator" class="font-bold text-lg">Operador</div>
            <div class="status-badge zone-badge" id="status-zone">ZONA: --</div>
        </div>
        <div class="flex gap-2">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay"></div>

    <div id="feedback-overlay">
        <div class="feedback-card">
            <i id="fb-icon" class="ri-checkbox-circle-fill text-6xl text-green-500 mb-2"></i>
            <h2 id="fb-msg" class="text-3xl font-black mb-1">ENCONTRADO</h2>
            <p id="fb-desc" class="text-sm text-gray-300 bg-white/10 p-2 rounded mb-2">Descrição do produto</p>
            <code id="fb-id" class="text-xs font-mono text-gray-400">ID: 123456</code>
        </div>
    </div>

    <button id="undo-btn"><i class="ri-arrow-go-back-line text-2xl"></i></button>

    <div id="ocr-loading" class="hidden">
        <i class="ri-loader-4-line text-6xl text-primary spin mb-4"></i>
        <p class="text-xl font-bold">Processando...</p>
    </div>

    <div id="login-modal">
        <div class="w-full max-w-sm text-center p-8">
            <div class="w-20 h-20 bg-primary rounded-2xl mx-auto mb-6 flex items-center justify-center shadow-lg shadow-primary/50">
                <i class="ri-box-3-fill text-4xl text-white"></i>
            </div>
            <h1 class="text-3xl font-bold mb-2">Olá!</h1>
            <p class="text-gray-400 mb-8">Vamos começar a bipar?</p>
            <input type="text" id="operator-input" class="input-field text-center text-lg mb-4" placeholder="Seu Nome">
            <button onclick="doLogin()" class="btn btn-primary w-full">ENTRAR</button>
        </div>
    </div>

    <div id="controls-panel">
        <div class="panel-handle" onclick="togglePanel()">
            <div class="handle-bar"></div>
        </div>

        <div class="tabs-container">
            <button class="tab-btn active" onclick="switchTab('procurar', this)">🔍 Procurar</button>
            <button class="tab-btn" onclick="switchTab('inventario', this)">📦 Inventário</button>
            <button class="tab-btn" onclick="switchTab('dashboard', this)">📊 Dash</button>
            <button class="tab-btn" onclick="switchTab('zonas', this)">📍 Zonas</button>
            <button class="tab-btn" onclick="switchTab('log', this)">📝 Log</button>
        </div>

        <div class="panel-content">

            <div id="view-procurar" class="tab-content active">
                <div class="card flex gap-3">
                    <button onclick="document.getElementById('file-input').click()" class="btn flex-1 bg-gray-800 border border-gray-700 flex-col p-4 h-auto">
                        <i class="ri-file-excel-2-fill text-3xl text-green-500"></i>
                        <span class="text-xs font-bold mt-2">PLANILHA</span>
                    </button>
                    <button onclick="document.getElementById('image-input').click()" class="btn flex-1 bg-gray-800 border border-gray-700 flex-col p-4 h-auto">
                        <i class="ri-camera-fill text-3xl text-blue-500"></i>
                        <span class="text-xs font-bold mt-2">FOTO (OCR)</span>
                    </button>
                </div>
                <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                <input type="file" id="image-input" class="hidden" accept="image/*">

                <div id="upload-alert" class="hidden">
                    <i class="ri-checkbox-circle-line text-xl"></i>
                    <span id="upload-alert-msg"></span>
                </div>

                <div class="card flex gap-2 mt-4">
                    <input type="text" id="manual-input" class="input-field" placeholder="Digitar ID...">
                    <button id="manual-btn" class="btn btn-primary px-4"><i class="ri-check-line text-xl"></i></button>
                </div>

                <div class="card flex items-center justify-between">
                    <span class="text-sm font-medium">Modo Rápido (Sem Pausa)</span>
                    <input type="checkbox" id="fast-mode-toggle" class="w-5 h-5 accent-primary">
                </div>

                <div class="card bg-primary/10 border-primary/30">
                    <div class="flex justify-between items-center mb-2">
                        <label class="text-xs font-bold uppercase text-primary-light">Modo Caça</label>
                        <span id="hunt-status" class="text-xs font-mono text-primary-light"></span>
                    </div>
                    <div class="flex gap-2">
                        <input type="text" id="hunt-target-id" class="input-field text-sm" placeholder="ID Alvo">
                        <button id="hunt-toggle-btn" class="btn btn-primary px-4 text-sm">Ativar</button>
                    </div>
                </div>

                 <div class="flex gap-2 mt-4">
                     <button onclick="clearSession()" class="btn btn-danger flex-1 py-3 text-sm">Limpar Tudo</button>
                     <button onclick="logout()" class="btn btn-secondary flex-1 py-3 text-sm">Sair</button>
                 </div>
            </div>

            <div id="view-inventario" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Carregar Inventário por Zona</h2>
                <div id="inventory-zones-list" class="space-y-3">
                    </div>
                <input type="file" id="inventory-file-input" class="hidden" accept=".xlsx,.csv,.pdf,.txt,.html">
            </div>

            <div id="view-dashboard" class="tab-content text-center">
                <h2 class="text-xl font-bold mb-4">Performance</h2>
                <div class="flex justify-center mb-6">
                    <div class="relative w-40 h-40">
                        <canvas id="progressChart"></canvas>
                        <div class="absolute inset-0 flex items-center justify-center text-3xl font-bold" id="progress-text">0%</div>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-4 mb-6">
                    <div class="card bg-gray-800"><div class="text-3xl font-bold text-primary-light" id="kpi-total">0</div><div class="text-xs text-gray-400 uppercase font-bold">Total Bipado</div></div>
                    <div class="card bg-gray-800"><div class="text-3xl font-bold text-red-400" id="kpi-error">0</div><div class="text-xs text-gray-400 uppercase font-bold">Erros</div></div>
                </div>
                 <button id="download-report-btn" class="btn btn-success w-full mb-3"><i class="ri-file-excel-2-line mr-2"></i> Baixar Relatório Excel</button>
                 <button id="download-flow-btn" class="btn btn-primary w-full"><i class="ri-image-line mr-2"></i> Baixar Fluxograma</button>
            </div>

            <div id="view-zonas" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Seleção de Zona Ativa</h2>
                <div id="zones-list" class="space-y-3">
                    </div>
            </div>

            <div id="view-log" class="tab-content">
                <h2 class="text-xl font-bold mb-4">Histórico</h2>
                <div id="scan-log-list" class="space-y-2">
                    </div>
            </div>
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="togglePanelFromNav('procurar', this)"><i class="ri-qr-scan-2-line"></i><span>Scan</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('inventario', this)"><i class="ri-archive-line"></i><span>Inventário</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('dashboard', this)"><i class="ri-pie-chart-2-line"></i><span>Dash</span></div>
        <div class="nav-item" onclick="togglePanelFromNav('zonas', this)"><i class="ri-map-pin-2-line"></i><span>Zonas</span></div>
    </nav>

    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = {
            STORAGE_KEY: 'natefy_pro_v9',
            WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9' // Substitua pelo seu
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

        // --- LOGIN ---
        window.doLogin = function() {
            const input = document.getElementById('operator-input');
            const val = input.value.trim();
            if(val) {
                state.operator = val;
                saveState();
                checkLogin();
            } else { alert("Por favor, digite seu nome."); }
        };

        function checkLogin() {
            const modal = document.getElementById('login-modal');
            if (state.operator) {
                modal.classList.add('hidden');
                document.getElementById('status-operator').innerText = state.operator.toUpperCase();
                if(!state.html5QrCode) startScanner(); // Inicia scanner apenas se não estiver rodando
            } else {
                modal.classList.remove('hidden');
            }
        }

        // --- NAVEGAÇÃO & PAINEL ---
        window.togglePanel = function(forceOpen = false) {
            const p = document.getElementById('controls-panel');
            const isOpen = p.classList.contains('open');

            if (forceOpen || !isOpen) {
                p.classList.add('open');
                if(state.html5QrCode && state.html5QrCode.getState() === 2) state.html5QrCode.pause(true); // Pausa câmera
            } else {
                p.classList.remove('open');
                if(state.html5QrCode && state.html5QrCode.getState() === 3) state.html5QrCode.resume(); // Retoma câmera
            }
        };

        window.togglePanelFromNav = function(tabName, btn) {
            const p = document.getElementById('controls-panel');
            const isScanTab = tabName === 'procurar';
            const isActive = btn.classList.contains('active');

            // Atualiza visual do nav
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            btn.classList.add('active');

            if (isScanTab) {
                // Lógica de "flip" para a aba Scan
                togglePanel(!p.classList.contains('open'));
            } else {
                // Outras abas sempre abrem o painel
                togglePanel(true);
            }
            // Sempre muda para a aba correta
            switchTab(tabName);
        };


        window.switchTab = function(tabName) {
            // Muda classe do botão
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            const tabBtn = Array.from(document.querySelectorAll('.tab-btn')).find(b => b.innerText.toLowerCase().includes(tabName.toLowerCase()));
            if(tabBtn) tabBtn.classList.add('active');

            // Muda conteúdo
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById(`view-${tabName}`).classList.add('active');

            if(tabName === 'dashboard') updateDashboard();
            if(tabName === 'zonas') renderZones();
            if(tabName === 'log') updateUI();
            if(tabName === 'inventario') renderInventoryZones();
        };

        // --- AÇÕES ---
        function setupActions() {
            document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]);
            document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]);
            document.getElementById('inventory-file-input').onchange = (e) => handleInventoryFile(e.target.files[0]);
            document.getElementById('manual-btn').onclick = () => { const v=document.getElementById('manual-input').value.trim(); if(v){ processScan(v); document.getElementById('manual-input').value=''; }};
            document.getElementById('fast-mode-toggle').onchange = (e) => state.isFastMode = e.target.checked;
            document.getElementById('undo-btn').onclick = doUndo;
            document.getElementById('download-report-btn').onclick = downloadExcel;
            document.getElementById('download-flow-btn').onclick = downloadFlow;
            document.getElementById('hunt-toggle-btn').onclick = toggleHuntMode;

            // Gesto de deslizar na alça
            const handle = document.querySelector('.panel-handle');
            let startY = 0;
            handle.addEventListener('touchstart', e => startY = e.touches[0].clientY);
            handle.addEventListener('touchend', e => {
                const diff = startY - e.changedTouches[0].clientY;
                if (diff > 50) togglePanel(true); // Deslizar para cima
                if (diff < -50) togglePanel(false); // Deslizar para baixo
            });
        }

        // --- SCANNER ---
        function startScanner() {
            state.html5QrCode = new Html5Qrcode("reader");
            state.html5QrCode.start({ facingMode: "environment" }, { fps: 15, qrbox: 250 },
                (decodedText) => processScan(decodedText)
            ).catch(err => console.log("Erro Câmera", err));
        }

        function processScan(id) {
            const now = Date.now();
            if (now - state.lastScanTime < 1000) return;
            state.lastScanTime = now;
            if (state.isPaused) return;
            if (!state.isFastMode) state.isPaused = true;

            id = id.trim();

            // Hunt Mode
            if (state.huntMode && state.huntMode.isActive) {
                if (id === state.huntMode.targetId) {
                    showFeedback('success', 'ALVO ENCONTRADO!', '', id);
                    sendToN8n({ scannedId: id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: state.operator });
                    toggleHuntMode(); return;
                }
            }

            const desc = state.idDescriptions.get(id) || "Sem descrição";
            let status = "NÃO ENCONTRADO";
            let type = "error";
            let feedbackMsg = "NÃO ENCONTRADO";
            let icon = "ri-close-circle-fill";

            if (state.activeZone) {
                for (const [zId, zSet] of state.zoneData) {
                    if (zId !== state.activeZone && zSet.has(id)) {
                        status = "MISSORT"; type = "warning"; feedbackMsg = `MISSORT (${zId})`; icon = "ri-alert-fill"; break;
                    }
                }
            }

            if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) {
                status = "DUPLICADO"; type = "warning"; feedbackMsg = "JÁ BIPADO"; icon = "ri-alert-fill";
            }

            if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) {
                status = "SUCESSO"; type = "success"; feedbackMsg = "ENCONTRADO"; icon = "ri-checkbox-circle-fill";
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
            const card = overlay.querySelector('.feedback-card');
            const icon = document.getElementById('fb-icon');
            const txt = document.getElementById('fb-msg');
            const dsc = document.getElementById('fb-desc');
            const idtxt = document.getElementById('fb-id');
            const undo = document.getElementById('undo-btn');

            overlay.className = `fixed inset-0 z-70 flex items-center justify-center p-6 transition-opacity duration-200 scan-${type}`;
            
            icon.className = `${iconClass} text-6xl mb-2 ${type === 'success' ? 'text-green-500' : type === 'error' ? 'text-red-500' : 'text-yellow-500'}`;
            txt.innerText = msg;
            dsc.innerText = desc;
            idtxt.innerText = `ID: ${id}`;

            overlay.style.opacity = '1';
            undo.classList.add('visible');
            setTimeout(() => undo.classList.remove('visible'), 5000);
            if(navigator.vibrate) navigator.vibrate(200);
            setTimeout(() => { overlay.style.opacity = '0'; if (!state.isFastMode) state.isPaused = false; }, state.isFastMode ? 500 : 1500);
        }

        function sendToN8n(data) {
            if(CONFIG.WEBHOOK.includes("https")) {
                fetch(CONFIG.WEBHOOK, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(data) }).catch(e => console.log("Offline"));
            }
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
            }
        }
        window.clearSession = function() { if(confirm('Apagar?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); } };
        window.logout = function() { if(confirm('Sair?')) { state.operator = null; saveState(); location.reload(); } };

        // --- HELPERS DE ARQUIVO (PROCURAR) ---
        function showUploadAlert(msg) {
            const alert = document.getElementById('upload-alert');
            const msgSpan = document.getElementById('upload-alert-msg');
            msgSpan.innerText = msg;
            alert.classList.remove('hidden');
            setTimeout(() => alert.classList.add('hidden'), 5000);
        }

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
                showUploadAlert(`"${file.name}" carregado: ${ids.length} IDs.`);
                saveState();
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
                        if (desc.length < 3) desc = "Sem Descrição";
                        newIds.add(id);
                        state.idDescriptions.set(id, desc);
                        count++;
                    }
                });
                if (count > 0) {
                    state.idsToFind = newIds;
                    state.foundIds = [];
                    showUploadAlert(`FOTO processada: ${count} IDs lidos.`);
                    saveState();
                } else { alert("Nada encontrado."); }
            } catch (e) { alert("Erro OCR"); } finally { document.getElementById('ocr-loading').classList.add('hidden'); }
        }

        // --- HELPERS DE ARQUIVO (INVENTÁRIO) ---
        let tempZoneId = null;
        window.triggerInventoryUpload = (zoneId) => { tempZoneId = zoneId; document.getElementById('inventory-file-input').click(); };

        async function handleInventoryFile(file) {
            if (!file || !tempZoneId) return;
            document.getElementById('ocr-loading').classList.remove('hidden');
            let ids = new Set();
            try {
                const ext = file.name.split('.').pop().toLowerCase();
                if (ext === 'xlsx' || ext === 'csv') ids = await processExcel(file);
                else if (ext === 'pdf') ids = await processPdf(file);
                else if (ext === 'txt' || ext === 'html') ids = await processText(file);
                
                if (ids.size > 0) {
                    state.zoneData.set(tempZoneId, ids);
                    saveState();
                    renderInventoryZones();
                    alert(`Sucesso! ${ids.size} IDs carregados para ${tempZoneId.toUpperCase()}.`);
                } else { alert("Nenhum ID encontrado no arquivo."); }
            } catch (e) { console.error(e); alert("Erro ao processar arquivo."); }
            finally { document.getElementById('ocr-loading').classList.add('hidden'); tempZoneId = null; }
        }

        function processExcel(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        const data = new Uint8Array(e.target.result);
                        const workbook = XLSX.read(data, {type: 'array'});
                        const sheet = workbook.Sheets[workbook.SheetNames[0]];
                        const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                        const ids = new Set(json.map(r => String(r[0])).filter(i => i && i.match(/\d{8,}/))); // Assume ID na 1ª coluna
                        resolve(ids);
                    } catch (e) { reject(e); }
                };
                reader.onerror = reject; reader.readAsArrayBuffer(file);
            });
        }

        async function processPdf(file) {
            const arrayBuffer = await file.arrayBuffer();
            const pdf = await pdfjsLib.getDocument(arrayBuffer).promise;
            let text = '';
            for (let i = 1; i <= pdf.numPages; i++) {
                const page = await pdf.getPage(i);
                const content = await page.getTextContent();
                text += content.items.map(item => item.str).join(' ') + '\n';
            }
            return extractIdsFromText(text);
        }

        function processText(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = (e) => resolve(extractIdsFromText(e.target.result));
                reader.onerror = reject; reader.readAsText(file);
            });
        }

        function extractIdsFromText(text) {
            const ids = new Set();
            // Regex para encontrar números com 8 ou mais dígitos (ajuste conforme seu padrão de ID)
            const matches = text.match(/\b\d{8,}\b/g);
            if (matches) matches.forEach(id => ids.add(id));
            return ids;
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
            const btn = document.getElementById('hunt-toggle-btn');
            const status = document.getElementById('hunt-status');
            
            if(state.huntMode && state.huntMode.isActive) {
                state.huntMode={isActive:false, targetId:null};
                t.disabled=false; t.value='';
                btn.innerText='Ativar'; btn.classList.remove('btn-danger'); btn.classList.add('btn-primary');
                status.innerText = '';
            } else {
                const v = t.value.trim(); if(!v) return alert("ID?");
                state.huntMode={isActive:true, targetId:v};
                t.disabled=true;
                btn.innerText='Cancelar'; btn.classList.remove('btn-primary'); btn.classList.add('btn-danger');
                status.innerText = 'ATIVO';
            }
            saveState();
        }

        // UI Updates
        function updateUI() {
            const list = document.getElementById('scan-log-list');
            list.innerHTML = state.logs.slice(0, 50).map(l => `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700">
                    <div>
                        <div class="font-bold text-sm">${l.id}</div>
                        <div class="text-xs text-gray-400 truncate w-48">${l.desc}</div>
                    </div>
                    <div class="text-right">
                        <div class="text-xs font-bold ${l.status==='SUCESSO'?'text-green-500':(l.status==='ERRO'?'text-red-500':'text-yellow-500')}">${l.status}</div>
                        <div class="text-xs text-gray-500">${new Date(l.time).toLocaleTimeString()}</div>
                    </div>
                </div>`).join('');
        }
        function updateDashboard() {
            const total = state.idsToFind.size + state.foundIds.length;
            const pct = total > 0 ? Math.round((state.foundIds.length / total) * 100) : 0;
            const ctx = document.getElementById('progressChart');
            if (ctx) {
                if(state.chart) state.chart.destroy();
                state.chart = new Chart(ctx, {
                    type: 'doughnut',
                    data: { datasets: [{ data: [pct, 100-pct], backgroundColor: ['#6366F1', '#374151'], borderWidth: 0 }] },
                    options: { cutout: '80%', plugins: { tooltip: { enabled: false } } }
                });
            }
            document.getElementById('progress-text').innerText = pct + "%";
            document.getElementById('kpi-total').innerText = state.logs.length;
            document.getElementById('kpi-error').innerText = state.logs.filter(l=>l.status==='ERRO' || l.status==='MISSORT').length;
        }
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
        function renderInventoryZones() {
            const c = document.getElementById('inventory-zones-list');
            c.innerHTML = '';
            state.inventoryZones.forEach(z => {
                const count = state.zoneData.get(z.id)?.size || 0;
                c.innerHTML += `
                <div class="card flex justify-between items-center mb-2 p-3 bg-gray-800 border-gray-700">
                    <div>
                        <div class="font-bold">${z.name}</div>
                        <div class="text-xs text-gray-400">${count} IDs carregados</div>
                    </div>
                    <button class="btn btn-primary py-2 px-4 text-xs" onclick="triggerInventoryUpload('${z.id}')"><i class="ri-upload-cloud-line mr-1"></i> Carregar</button>
                </div>`;
            });
        }
        window.setZone = (id) => { state.activeZone = id; document.getElementById('status-zone').innerText = `ZONA: ${id ? id.toUpperCase() : '--'}`; saveState(); renderZones(); };
        
        function downloadExcel() {
            if(state.logs.length === 0) return alert("Sem dados");
            const ws = XLSX.utils.json_to_sheet(state.logs); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Logs"); XLSX.writeFile(wb, "Relatorio.xlsx");
        }
        async function downloadFlow() {
             if(state.logs.length === 0) return alert("Sem dados");
             const container = document.getElementById('flowchart-canvas-container');
             container.innerHTML = `<div style="padding:50px; background:#1E1B4B; color:white; text-align:center"><h1>Relatório Visual</h1><h2>${state.operator}</h2><div style="display:flex; justify-content:center; gap:20px; margin-top:50px"><div style="border:2px solid #10B981; padding:20px">SUCESSO: ${state.foundIds.length}</div><div style="border:2px solid #EF4444; padding:20px">ERROS: ${state.logs.filter(l=>l.status!=='SUCESSO').length}</div></div></div>`;
             const canvas = await html2canvas(container);
             const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click();
        }
    </script>
</body>
</html>
