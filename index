<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Clean</title>

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
        /* Layout Fullscreen */
        html, body { height: 100%; width: 100%; overflow: hidden; font-family: 'Inter', sans-serif; background-color: #000; }
        
        /* Scanner Limpo (Sem linhas) */
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; object-fit: cover; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

        /* Menu Inferior Fixo */
        #bottom-nav {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 70px;
            background: rgba(15, 23, 42, 0.95);
            backdrop-filter: blur(10px);
            border-top: 1px solid #334155;
            display: flex; justify-content: space-around; align-items: center;
            z-index: 60; /* Acima do painel */
            padding-bottom: env(safe-area-inset-bottom, 10px);
        }
        .nav-item {
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            color: #94a3b8; font-size: 10px; gap: 4px; width: 20%; cursor: pointer;
        }
        .nav-item.active { color: #22d3ee; }
        .nav-item i { font-size: 22px; transition: transform 0.2s; }
        .nav-item:active i { transform: scale(0.9); }

        /* Painel Deslizante (Menu que sobe) */
        #controls-panel { 
            position: absolute;
            bottom: 70px; /* Fica logo acima do menu fixo */
            left: 0; width: 100%;
            background: rgba(15, 23, 42, 0.98); 
            border-top: 1px solid #334155; 
            border-radius: 20px 20px 0 0;
            /* Escondido (Para baixo) por padrão */
            transform: translateY(110%); 
            transition: transform 0.4s cubic-bezier(0.2, 0.8, 0.2, 1); 
            z-index: 50; 
            max-height: 60vh;
            display: flex; flex-direction: column;
        }
        /* Classe para quando o menu está visível (Para cima) */
        #controls-panel.open { transform: translateY(0); }

        /* Conteúdo dentro do painel */
        .panel-content { padding: 20px; overflow-y: auto; }

        /* Elementos escondidos ou visíveis baseados na aba */
        .tab-content { display: none; }
        .tab-content.active { display: block; animation: fadeIn 0.3s; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Status Bar Superior */
        #top-bar { position: absolute; top: 0; left: 0; width: 100%; height: 60px; background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent); z-index: 30; display: flex; justify-content: space-between; align-items: center; padding: 0 20px; padding-top: env(safe-area-inset-top, 10px); }
        
        /* Feedback Overlay */
        #feedback-overlay { position: fixed; inset: 0; z-index: 70; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.85); }
        .scan-error { background: rgba(239, 68, 68, 0.85); }
        .scan-warning { background: rgba(234, 179, 8, 0.85); }

        /* Utilitários */
        .hidden { display: none !important; }
        .btn-primary { background: linear-gradient(to right, #0891b2, #2563eb); padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; color: white; }
        #undo-btn { position: absolute; top: 80px; right: 20px; z-index: 40; background: #ef4444; color: white; width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 12px rgba(0,0,0,0.3); opacity: 0; pointer-events: none; transition: opacity 0.3s; }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }
        
        /* Canvas Oculto */
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background-color: #0f172a; }
        #ocr-loading { position: fixed; inset: 0; z-index: 90; background-color: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
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

    <div id="feedback-overlay">
        <div class="text-center p-6">
            <div id="fb-icon" class="text-6xl mb-4">✅</div>
            <div id="fb-msg" class="text-4xl font-black text-white drop-shadow-md mb-2">ENCONTRADO</div>
            <div id="fb-desc" class="text-lg text-white/90 font-medium bg-black/20 p-2 rounded">Descrição</div>
            <div id="fb-id" class="text-sm text-white/60 font-mono mt-2">ID</div>
        </div>
    </div>

    <button id="undo-btn"><i class="fas fa-undo"></i></button>

    <div id="ocr-loading" class="hidden">
        <i class="fas fa-sync fa-spin text-5xl text-cyan-500 mb-4"></i>
        <p class="text-xl font-bold text-white">Lendo Imagem...</p>
    </div>

    <div id="controls-panel">
        <div class="w-full flex justify-center pt-3 pb-1 cursor-pointer" onclick="closePanel()">
            <div class="w-12 h-1.5 bg-slate-600 rounded-full"></div>
        </div>

        <div class="panel-content">
            
            <div id="view-procurar" class="tab-content active">
                <div class="space-y-4">
                    <div class="grid grid-cols-2 gap-3">
                        <button onclick="document.getElementById('file-input').click()" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                            <i class="fas fa-file-excel text-2xl text-green-500"></i><span class="text-xs font-bold text-slate-300">PLANILHA</span>
                        </button>
                        <button onclick="document.getElementById('image-input').click()" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                            <i class="fas fa-camera text-2xl text-cyan-500"></i><span class="text-xs font-bold text-slate-300">FOTO (OCR)</span>
                        </button>
                    </div>
                    <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
                    <input type="file" id="image-input" class="hidden" accept="image/*">
                    <div id="file-status" class="bg-slate-900/50 p-2 rounded text-xs text-center text-slate-400">Nenhuma lista</div>
                    
                    <div class="flex gap-2">
                        <input type="text" id="manual-input" class="flex-1 bg-slate-900 border border-slate-600 rounded-lg px-4 py-3 text-sm text-white" placeholder="Digitar ID...">
                        <button id="manual-btn" class="bg-cyan-600 px-5 rounded-lg font-bold text-white"><i class="fas fa-check"></i></button>
                    </div>
                    
                    <div class="flex items-center justify-between bg-slate-900/50 p-3 rounded-lg">
                        <span class="text-sm text-slate-300">Modo Rápido</span>
                        <input type="checkbox" id="fast-mode-toggle" class="w-5 h-5 accent-cyan-500">
                    </div>

                    <div class="p-3 bg-slate-800/50 rounded-xl border border-slate-700">
                        <div class="flex justify-between items-center mb-2"><label class="text-xs text-slate-300 font-bold uppercase">Modo Caça</label><span id="hunt-status" class="text-xs text-cyan-400 font-mono"></span></div>
                        <div class="flex gap-2"><input type="text" id="hunt-target-id" class="w-full bg-slate-900 text-white p-2 rounded-lg font-mono border border-slate-600 text-sm" placeholder="ID Alvo"><button id="hunt-toggle-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-lg text-sm text-white">Ativar</button></div>
                    </div>

                     <div class="flex gap-2 mt-2">
                         <button onclick="clearSession()" class="w-full py-2 text-xs text-red-400 border border-red-900 rounded">Limpar Tudo</button>
                     </div>
                </div>
            </div>

            <div id="view-log" class="tab-content">
                <h2 class="text-lg font-bold text-white mb-2">Histórico</h2>
                <div id="scan-log-list" class="space-y-2"></div>
            </div>

            <div id="view-dashboard" class="tab-content text-center">
                <h2 class="text-lg font-bold text-white mb-4">Performance</h2>
                <div class="flex justify-center mb-4"><div class="relative w-32 h-32"><canvas id="progressChart"></canvas><div class="absolute inset-0 flex items-center justify-center text-2xl font-bold text-white" id="progress-text">0%</div></div></div>
                <div class="grid grid-cols-2 gap-4 mb-4">
                    <div class="bg-slate-900 p-3 rounded-lg"><div class="text-2xl font-bold text-cyan-400" id="kpi-total">0</div><div class="text-[10px] text-slate-500 uppercase">Total</div></div>
                    <div class="bg-slate-900 p-3 rounded-lg"><div class="text-2xl font-bold text-red-400" id="kpi-error">0</div><div class="text-[10px] text-slate-500 uppercase">Erros</div></div>
                </div>
                 <button id="download-report-btn" class="btn-primary text-sm mb-2"><i class="fas fa-file-export mr-2"></i> Baixar Excel</button>
                 <button id="download-flow-btn" class="w-full bg-slate-700 py-3 rounded-xl text-white font-bold text-sm"><i class="fas fa-image mr-2"></i> Baixar Fluxo</button>
            </div>

            <div id="view-zonas" class="tab-content">
                <h2 class="text-lg font-bold text-white mb-2">Zonas</h2>
                <div id="zones-list" class="space-y-2"></div>
            </div>
            
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" onclick="toggleScanMenu()"><i class="fas fa-search"></i><span>Scan</span></div>
        <div class="nav-item" onclick="switchTab('log')"><i class="fas fa-list"></i><span>Log</span></div>
        <div class="nav-item" onclick="switchTab('dashboard')"><i class="fas fa-chart-pie"></i><span>Dash</span></div>
        <div class="nav-item" onclick="switchTab('zonas')"><i class="fas fa-map-marker-alt"></i><span>Zonas</span></div>
    </nav>

    <div id="flowchart-canvas-container"></div>

    <script>
        const CONFIG = {
            STORAGE_KEY: 'natefy_v7',
            WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9'
        };

        let state = {
            operator: 'Operador',
            idsToFind: new Set(), idDescriptions: new Map(), foundIds: [], logs: [], activeZone: null,
            inventoryZones: [{id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},{id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},{id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}],
            zoneData: new Map(), isFastMode: false, isPaused: false, lastScanTime: 0, lastUndo: null
        };

        document.addEventListener('DOMContentLoaded', () => { init(); });

        function init() {
            loadState();
            // Inicia automaticamente
            startScanner();
            setupActions();
            updateUI();
        }

        // --- LÓGICA DO MENU "FLIP" ---
        // Essa é a mágica que faz o menu subir e descer
        function toggleScanMenu() {
            const panel = document.getElementById('controls-panel');
            const isScanActive = document.querySelector('.nav-item.active span').innerText === 'Scan';
            
            // Se já estiver na aba Scan
            if (isScanActive) {
                // Alterna (Sobe se tiver fechado, Desce se tiver aberto)
                panel.classList.toggle('open');
            } else {
                // Se estava em outra aba, vai para Scan e ABRE o menu para configurar
                switchTab('procurar');
                panel.classList.add('open');
            }
            
            // Atualiza visual do menu
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.querySelectorAll('.nav-item')[0].classList.add('active');
        }

        function switchTab(tabName) {
            // 1. Visual do botão
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            // Encontra o botão clicado (truque simples)
            const btns = ['procurar', 'log', 'dashboard', 'zonas'];
            document.querySelectorAll('.nav-item')[btns.indexOf(tabName) === -1 ? 0 : btns.indexOf(tabName)].classList.add('active');

            // 2. Conteúdo do painel
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            const contentId = tabName === 'procurar' ? 'view-procurar' : `view-${tabName}`;
            document.getElementById(contentId).classList.add('active');

            // 3. SEMPRE ABRE O PAINEL quando troca de aba (exceto Scan que tem lógica própria)
            const panel = document.getElementById('controls-panel');
            panel.classList.add('open');

            if(tabName === 'dashboard') updateDashboard();
            if(tabName === 'zonas') renderZones();
            if(tabName === 'log') updateUI();
        }

        function closePanel() {
            document.getElementById('controls-panel').classList.remove('open');
            resumeScanner();
        }

        // --- AÇÕES ---
        function setupActions() {
            document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]);
            document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]);
            document.getElementById('manual-btn').onclick = () => { const v=document.getElementById('manual-input').value.trim(); if(v){ processScan(v); document.getElementById('manual-input').value=''; }};
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
            ).catch(err => console.log("Câmera:", err));
        }
        
        function resumeScanner() {
            // Tenta retomar foco se necessário
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

        // --- UTILS ---
        function sendToN8n(data) {
            if(CONFIG.WEBHOOK.includes("https")) {
                fetch(CONFIG.WEBHOOK,
