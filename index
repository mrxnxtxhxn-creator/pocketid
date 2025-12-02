<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Natefy Scanner Pro</title>

    <meta name="theme-color" content="#06b6d4"/>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Natefy">
    <link rel="manifest" href="manifest.json">

    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
    <script src='https://unpkg.com/tesseract.js@v2.1.0/dist/tesseract.min.js'></script>
    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        html { height: 100%; }
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; position: relative; min-height: 100%; overflow: hidden; }
        #reader__scan_region { border: 4px solid rgba(255, 255, 255, 0.5) !important; border-radius: 1.5rem; background: none !important; box-shadow: 0 0 20px rgba(0, 255, 255, 0.3); }
        .scan-line { position: absolute; left: 5%; top: 10px; width: 90%; height: 4px; background: linear-gradient(to right, transparent, #06b6d4, transparent); box-shadow: 0 0 15px #06b6d4, 0 0 5px #fff; border-radius: 4px; animation: scan-animation 2.5s infinite ease-in-out; }
        @keyframes scan-animation { 0% { transform: translateY(0); } 50% { transform: translateY(calc(100% - 20px)); } 100% { transform: translateY(0); } }
        #controls-panel { background: rgba(30, 41, 59, 0.95); backdrop-filter: blur(16px); border-top: 1px solid rgba(71, 85, 105, 0.5); transform: translateY(calc(100% - 70px)); transition: transform 0.3s ease-in-out; padding-bottom: env(safe-area-inset-bottom, 0); }
        #controls-panel.open { transform: translateY(0); }
        .feedback-pulse { animation: pulse-feedback 0.8s ease-out; }
        @keyframes pulse-feedback { from { transform: scale(0.9); opacity: 0.7; } to { transform: scale(1); opacity: 1; } }
        .hunt-success-pulse { animation: hunt-pulse 0.5s ease-out 3; }
        @keyframes hunt-pulse { 0%, 100% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.2); opacity: 0.8; } }
        .tab-btn { border-bottom: 3px solid transparent; transition: all 0.2s; white-space: nowrap; }
        .tab-active { border-color: #06b6d4; color: white; }
        .tab-inactive { color: #94a3b8; }
        .toggle-bg:after { content: ''; position: absolute; top: 2px; left: 2px; background: white; border-radius: 9999px; width: 1.25rem; height: 1.25rem; transition: all 0.2s ease; }
        input:checked + .toggle-bg:after { transform: translateX(100%); left: auto; right: 2px; }
        input:checked + .toggle-bg { background-color: #06b6d4; }
        #startup-name { position: fixed; bottom: 0; left: 0; right: 0; padding: 4px 0; text-align: center; font-size: 0.7rem; color: #475569; background-color: #0f172a; z-index: 5; padding-bottom: calc(4px + env(safe-area-inset-bottom, 0)); }
        button:disabled { opacity: 0.5; cursor: not-allowed; }
        #login-modal { position: fixed; inset: 0; z-index: 100; background-color: rgba(15, 23, 42, 0.98); display: flex; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        #login-modal.hidden { display: none; }
        #ocr-loading { position: fixed; inset: 0; z-index: 90; background-color: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #ocr-loading.hidden { display: none; }
        /* Oculto para Fluxograma */
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 800px; background-color: #0f172a; padding: 40px; border-radius: 10px; color: white; font-family: 'Inter', sans-serif; }
        .flow-box { border: 2px solid; border-radius: 12px; padding: 15px; text-align: center; font-weight: bold; background: rgba(30, 41, 59, 0.8); position: relative; }
    </style>
</head>
<body class="text-slate-200">
    <div id="global-status-bar" class="fixed top-0 left-0 right-0 z-20 p-2 text-center text-xs font-bold tracking-wider transition-all duration-300 flex justify-between px-4 bg-slate-900/80 backdrop-blur-md border-b border-slate-700">
        <span id="status-zone">ZONA: --</span>
        <span id="status-operator" class="text-cyan-400">OP: --</span>
    </div>

    <div id="ocr-loading" class="hidden">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-cyan-500 mb-4"></div>
        <p class="text-lg font-bold">Processando Imagem...</p>
        <p class="text-sm text-slate-400 mt-2" id="ocr-status-text">Iniciando...</p>
    </div>

    <div id="flowchart-canvas-container"></div>

    <div id="login-modal">
        <div class="bg-slate-800 p-8 rounded-2xl shadow-2xl w-11/12 max-w-md text-center border border-slate-600">
            <h2 class="text-3xl font-black text-white mb-2">Natefy</h2>
            <p class="text-cyan-400 text-sm font-bold mb-6 tracking-widest uppercase">Scanner Pro</p>
            <p class="text-slate-300 mb-6">Identifique-se para iniciar a sessão.</p>
            <input type="text" id="operator-input" class="w-full bg-slate-700 text-white p-4 rounded-xl mb-4 border border-slate-600 focus:border-cyan-500 outline-none text-center text-lg placeholder-slate-500" placeholder="Nome do Operador">
            <button id="login-btn" class="w-full bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 text-white font-bold py-4 px-4 rounded-xl transition-all transform active:scale-95">ACESSAR SISTEMA</button>
        </div>
    </div>

    <div id="reader" class="fixed top-0 left-0 w-full h-full z-1"><div class="scan-line"></div></div>
    <div id="feedback-overlay" class="fixed inset-0 z-50 flex items-center justify-center p-4 text-white font-black opacity-0 pointer-events-none transition-opacity duration-200"></div>

    <div id="controls-panel" class="fixed bottom-0 left-0 right-0 z-10 rounded-t-2xl shadow-2xl shadow-black">
        <div id="panel-handle" class="w-full h-8 flex justify-center items-center cursor-pointer">
              <div class="w-12 h-1.5 bg-slate-600 rounded-full"></div>
        </div>
        <div class="w-full max-w-lg mx-auto px-4 pb-4">
             <div class="w-full overflow-x-auto pb-2 mb-2 scrollbar-hide">
                <div class="flex justify-start space-x-2">
                    <button data-view="procurar" class="tab-btn tab-active py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">🔍 Procurar</button>
                    <button data-view="log" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📋 Log do Dia</button>
                    <button data-view="dashboard" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📊 Dashboard</button>
                    <button data-view="analisador" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">⚖️ Analisador</button>
                    <button data-view="inventario" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">📦 Zonas</button>
                    <button data-view="excecoes" class="tab-btn tab-inactive py-2 px-3 font-semibold text-sm rounded-lg whitespace-nowrap">⚠️ Exceções</button>
                </div>
            </div>

            <div data-view-content="procurar" class="text-center">
                 <div class="grid grid-cols-2 gap-3 mb-4">
                     <button id="load-file-btn" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-3 px-2 rounded-xl text-sm border border-slate-600 flex items-center justify-center gap-2">
                         <span>📂</span> Planilha
                     </button>
                     <button id="load-image-btn" class="bg-gradient-to-r from-cyan-700 to-blue-700 hover:from-cyan-600 hover:to-blue-600 text-white font-bold py-3 px-2 rounded-xl text-sm border border-cyan-500/30 shadow-lg shadow-cyan-500/20 flex items-center justify-center gap-2">
                         <span>📷</span> Ler Foto
                     </button>
                 </div>
                 
                <input type="file" id="file-input" class="hidden" accept=".txt,.csv,.xlsx">
                <input type="file" id="image-input" class="hidden" accept="image/*">
                
                <p id="file-info" class="text-xs text-green-400 mt-1 min-h-[1.2rem] font-mono"></p>
                
                 <div class="mt-3 text-left border-t border-slate-700 pt-3 space-y-3">
                    <div class="p-3 bg-slate-800/50 rounded-xl border border-slate-700">
                        <div class="flex justify-between items-center mb-2">
                            <label class="text-xs text-slate-300 font-bold uppercase tracking-wide">Modo Caça ao Tesouro</label>
                            <span id="hunt-status" class="text-xs text-cyan-400 font-mono"></span>
                        </div>
                        <div class="flex gap-2">
                            <input type="text" id="hunt-target-id" class="w-full bg-slate-900 text-white p-2 rounded-lg font-mono border border-slate-600 focus:border-cyan-500 outline-none text-sm" placeholder="Digite ID para encontrar...">
                            <button id="hunt-toggle-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-lg text-sm transition-colors">Ativar</button>
                        </div>
                    </div>

                    <div class="flex justify-between items-center bg-slate-800/50 p-3 rounded-xl border border-slate-700">
                        <span class="text-sm text-slate-300 font-medium">Modo Rápido (Sem Pausa)</span>
                        <div class="relative inline-block w-10 align-middle select-none transition duration-200 ease-in">
                            <input type="checkbox" name="fast-mode-toggle" id="fast-mode-toggle" class="toggle-checkbox absolute block w-5 h-5 rounded-full bg-white border-4 appearance-none cursor-pointer"/>
                            <label for="fast-mode-toggle" class="toggle-bg block overflow-hidden h-6 w-11 rounded-full bg-slate-600 cursor-pointer"></label>
                        </div>
                    </div>

                    <div class="flex gap-2">
                        <button id="clear-session-btn" class="flex-1 bg-red-900/50 hover:bg-red-900 text-red-200 font-bold py-2 px-4 rounded-lg text-xs border border-red-800">Limpar Dados</button>
                        <button id="change-operator-btn" class="flex-1 bg-slate-700 hover:bg-slate-600 text-slate-300 font-bold py-2 px-4 rounded-lg text-xs">Sair / Trocar OP</button>
                    </div>

                    <div class="pt-2">
                        <div class="flex gap-2 mt-1">
                            <input type="text" id="manual-input" class="w-full bg-slate-700 p-3 rounded-xl font-mono text-white border border-slate-600 focus:border-blue-500 outline-none" placeholder="Digitar ID manual...">
                            <button id="manual-check-btn" class="bg-blue-600 hover:bg-blue-500 font-bold px-4 rounded-xl">OK</button>
                        </div>
                    </div>
                </div>
            </div>

            <div data-view-content="log" class="hidden">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-bold text-white">Histórico do Dia</h3>
                    <span class="text-xs text-slate-400 bg-slate-800 px-2 py-1 rounded">Últimos 50</span>
                </div>
                <div class="max-h-[60vh] overflow-y-auto pr-1 space-y-2" id="scan-log-list">
                    <div class="text-center text-slate-500 py-4">Nenhum registro hoje.</div>
                </div>
            </div>

            <div data-view-content="dashboard" class="hidden">
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700 flex flex-col items-center justify-center relative overflow-hidden">
                        <div class="relative w-20 h-20 mb-2">
                            <canvas id="progressChart"></canvas>
                            <div id="progress-text" class="absolute inset-0 flex items-center justify-center text-lg font-bold text-white">0%</div>
                        </div>
                        <span class="text-xs text-slate-400 uppercase tracking-widest">Progresso</span>
                    </div>
                    <div class="space-y-3">
                        <div class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-center">
                            <span class="text-xs text-slate-400 uppercase">Velocidade</span>
                            <div id="kpi-bpm" class="text-2xl font-black text-cyan-400">--</div>
                            <span class="text-[10px] text-slate-500">bips/min</span>
                        </div>
                        <div class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-center">
                            <span class="text-xs text-slate-400 uppercase">Tempo Médio</span>
                            <div id="kpi-avg-time" class="text-xl font-bold text-white">--</div>
                        </div>
                    </div>
                </div>
                
                <h4 class="text-xs font-bold text-slate-500 uppercase mb-2 tracking-widest">Ações de Relatório</h4>
                <div class="grid grid-cols-2 gap-3 mb-6">
                    <button id="download-full-report-btn" class="bg-green-800 hover:bg-green-700 text-green-100 p-3 rounded-xl text-xs font-bold flex flex-col items-center gap-1 border border-green-700">
                        <span class="text-lg">📊</span> Baixar Excel
                    </button>
                    <button id="download-flowchart-btn" class="bg-purple-800 hover:bg-purple-700 text-purple-100 p-3 rounded-xl text-xs font-bold flex flex-col items-center gap-1 border border-purple-700">
                        <span class="text-lg">🖼️</span> Baixar Fluxo
                    </button>
                </div>

                <h4 class="text-xs font-bold text-slate-500 uppercase mb-2 tracking-widest">Performance por Zona</h4>
                <div id="zone-finds-container" class="space-y-2"></div>
            </div>

            <div data-view-content="analisador" class="hidden space-y-4">
                <div class="bg-slate-800 p-4 rounded-xl border border-slate-700">
                    <h3 class="text-lg font-bold text-center text-white mb-4">Comparador de Listas</h3>
                    <p class="text-xs text-slate-400 text-center mb-4">Descubra quais itens se repetem entre duas planilhas.</p>
                    
                    <div class="space-y-3">
                        <div>
                            <label class="text-xs text-cyan-400 font-bold uppercase">Lista A (Base)</label>
                            <select id="analysis-list-a" class="w-full bg-slate-900 border border-slate-600 p-2 rounded-lg text-white text-sm mt-1"><option value="">Carregue listas primeiro...</option></select>
                        </div>
                        <div>
                            <label class="text-xs text-cyan-400 font-bold uppercase">Lista B (Comparação)</label>
                            <select id="analysis-list-b" class="w-full bg-slate-900 border border-slate-600 p-2 rounded-lg text-white text-sm mt-1"><option value="">Carregue listas primeiro...</option></select>
                        </div>
                        <button id="run-analysis-btn" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-lg shadow-lg shadow-blue-900/50">COMPARAR AGORA</button>
                    </div>
                </div>

                <div id="analysis-results-container" class="hidden">
                    <div class="grid grid-cols-3 gap-2 text-center mb-4">
                        <div class="bg-slate-800 p-2 rounded-lg border border-green-500/30"><span class="block text-xl font-bold text-green-400" id="result-ok">--</span><span class="text-[10px] text-slate-400 uppercase">Repetidos<br>(Em Ambas)</span></div>
                        <div class="bg-slate-800 p-2 rounded-lg border border-orange-500/30"><span class="block text-xl font-bold text-orange-400" id="result-sobra">--</span><span class="text-[10px] text-slate-400 uppercase">Só na B<br>(Sobras)</span></div>
                        <div class="bg-slate-800 p-2 rounded-lg border border-red-500/30"><span class="block text-xl font-bold text-red-400" id="result-faltantes">--</span><span class="text-[10px] text-slate-400 uppercase">Só na A<br>(Faltam)</span></div>
                    </div>
                    <button id="export-analysis-btn" class="w-full bg-slate-700 hover:bg-slate-600 text-white font-bold py-3 rounded-lg border border-slate-600">Baixar Relatório da Análise (.csv)</button>
                </div>
            </div>

            <div data-view-content="inventario" class="hidden">
                 <h3 class="text-lg font-bold text-center text-white mb-4">Zonas de Inventário</h3>
                <div id="inventory-zones-container" class="space-y-3 max-h-[60vh] overflow-y-auto pr-2"></div>
            </div>
            <div data-view-content="excecoes" class="hidden">
                <div class="bg-red-900/20 p-3 rounded-lg border border-red-900/50 mb-4 text-center">
                    <p class="text-sm text-red-200">Aqui ficam os itens bipados que <strong>não estavam</strong> em nenhuma lista carregada.</p>
                </div>
                 <button id="export-exceptions-btn" class="w-full bg-red-800 hover:bg-red-700 text-white font-bold py-3 px-5 rounded-lg mb-4 shadow-lg">Baixar Lista de Exceções (.csv)</button>
                <div class="max-h-48 overflow-y-auto pr-2"><ul id="exceptions-list" class="space-y-2 text-center font-mono text-sm"></ul></div>
            </div>
            <div data-view-content="encontrados" class="hidden">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-bold text-white">Sucessos</h3>
                    <span id="found-count-display" class="bg-green-900 text-green-200 px-2 py-1 rounded text-xs font-bold">0</span>
                </div>
                 <button id="export-btn" class="w-full bg-green-700 hover:bg-green-600 text-white font-bold py-3 px-5 rounded-lg mb-4">Baixar Lista de Sucesso (.csv)</button>
                <div class="max-h-[60vh] overflow-y-auto pr-2"><ul id="found-list" class="space-y-2 text-center font-mono text-sm"></ul></div>
            </div>
        </div>
    </div>

    <div id="startup-name">Natefy &copy; 2025</div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const SCAN_DELAY = 1200;
            const STORAGE_KEY = 'scannerAppState';
            const N8N_REALTIME_WEBHOOK_URL = 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9';

            let appState = {
                currentView: 'procurar',
                idsToFind: new Set(),
                idDescriptions: new Map(), // ID -> Descrição
                inventoryZones: [
                    { id: 'buffered', name: 'Buffered' }, { id: 'sorting', name: 'Sorting' },
                    { id: 'fraude', name: 'Fraude' }, { id: 'missort', name: 'Missort' },
                    { id: 'returns', name: 'Returns' }, { id: 'bulky', name: 'Bulky' },
                    { id: 'problemsolver', name: 'Problem Solver' }
                ],
                inventoryZoneData: new Map(),
                foundIds: [],
                isPaused: false,
                audioContext: null,
                html5QrCode: null,
                scanHistory: [],
                charts: {},
                notFoundIds: [],
                scanLog: [],
                zoneFinds: new Map(),
                isFastMode: false,
                lastScanTime: 0,
                activeZoneId: null,
                huntMode: { isActive: false, targetId: null },
                analysisResult: { ok: [], sobra: [], faltantes: [] },
                operatorName: null
            };

            const controlsPanel = document.getElementById('controls-panel');
            const panelHandle = document.getElementById('panel-handle');

            // --- Inicialização ---
            function initialize() {
                loadFromLocalStorage();
                buildInventoryZoneUI();
                buildAnalyzerUI();
                checkLogin();
                setupEventListeners();
                createCharts();
                updateAllUI();
            }

            function setupEventListeners() {
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', (e) => handleFileSelect(e, 'main'));
                document.getElementById('load-image-btn').addEventListener('click', () => document.getElementById('image-input').click());
                document.getElementById('image-input').addEventListener('change', handleImageUpload);
                
                document.getElementById('export-btn').addEventListener('click', exportFoundIds);
                document.getElementById('export-exceptions-btn').addEventListener('click', exportExceptions);
                document.getElementById('clear-session-btn').addEventListener('click', clearSession);
                
                document.getElementById('fast-mode-toggle').addEventListener('change', toggleFastMode);
                document.getElementById('hunt-toggle-btn').addEventListener('click', toggleHuntMode);
                
                document.getElementById('run-analysis-btn').addEventListener('click', runListAnalysis);
                document.getElementById('export-analysis-btn').addEventListener('click', exportAnalysisResults);
                
                document.getElementById('download-full-report-btn').addEventListener('click', downloadFullReport);
                document.getElementById('download-flowchart-btn').addEventListener('click', downloadFlowchart);

                document.getElementById('login-btn').addEventListener('click', doLogin);
                document.getElementById('change-operator-btn').addEventListener('click', logout);

                document.body.addEventListener('click', initAudio, { once: true });
                setupTabs();

                const manualInput = document.getElementById('manual-input');
                const checkManual = () => { const v = manualInput.value.trim(); if(v){ processScan(v); manualInput.value=''; }};
                document.getElementById('manual-check-btn').addEventListener('click', checkManual);
                manualInput.addEventListener('keydown', (e) => { if(e.key === 'Enter'){ e.preventDefault(); checkManual(); }});
                
                // Panel gestures
                let touchStartY = 0;
                document.addEventListener('touchstart', e => { if (e.target === panelHandle || controlsPanel.contains(e.target)) touchStartY = e.touches[0].clientY; });
                document.addEventListener('touchend', e => { if(touchStartY===0)return; const touchEndY=e.changedTouches[0].clientY; if(touchStartY-touchEndY>50)controlsPanel.classList.add('open'); else if(touchEndY-touchStartY>50){controlsPanel.classList.remove('open'); resumeScannerIfNeeded();} touchStartY=0; });
                panelHandle.addEventListener('click', () => { controlsPanel.classList.toggle('open'); if(!controlsPanel.classList.contains('open')) resumeScannerIfNeeded(); });
            }

            // --- Lógica de Login ---
            function checkLogin() {
                if (!appState.operatorName) { document.getElementById('login-modal').classList.remove('hidden'); } 
                else { document.getElementById('login-modal').classList.add('hidden'); updateStatusUI(); startScanner(); }
            }
            function doLogin() {
                const name = document.getElementById('operator-input').value.trim();
                if (name) { appState.operatorName = name; saveToLocalStorage(); checkLogin(); } 
                else { alert("Nome obrigatório"); }
            }
            function logout() {
                if(confirm("Trocar operador?")) { appState.operatorName = null; document.getElementById('operator-input').value = ''; checkLogin(); }
            }

            // --- OCR (Leitura de Imagem) ---
            async function handleImageUpload(e) {
                const file = e.target.files[0];
                if (!file) return;
                const ocrOverlay = document.getElementById('ocr-loading');
                const ocrText = document.getElementById('ocr-status-text');
                ocrOverlay.classList.remove('hidden');
                ocrText.textContent = "Iniciando...";

                try {
                    const worker = Tesseract.createWorker({ logger: m => ocrText.textContent = m.status === 'recognizing text' ? `Lendo: ${Math.round(m.progress * 100)}%` : m.status });
                    await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                    const { data: { text } } = await worker.recognize(file);
                    await worker.terminate();

                    const lines = text.split('\n');
                    let count = 0;
                    const idRegex = /(\d{8,14})/; // Procura IDs numéricos longos
                    const newIds = new Set();

                    lines.forEach(line => {
                        const match = line.match(idRegex);
                        if (match) {
                            const id = match[0];
                            let description = line.replace(id, '').trim().replace(/^[\.\,\-\>\s]+/, ''); // Limpa descrição
                            newIds.add(id);
                            if(description) appState.idDescriptions.set(id, description);
                            count++;
                        }
                    });

                    if (count > 0) {
                        appState.idsToFind = newIds;
                        appState.foundIds = []; // Novo arquivo, reseta encontrados
                        updateAllUI();
                        document.getElementById('file-info').textContent = `Foto: ${count} itens.`;
                        alert(`Leitura concluída! ${count} itens identificados.`);
                    } else {
                        alert("Nenhum ID encontrado na imagem.");
                    }
                } catch (err) { console.error(err); alert("Erro na leitura: " + err.message); } 
                finally { ocrOverlay.classList.add('hidden'); saveToLocalStorage(); }
            }

            // --- Processamento de Scan ---
            function processScan(scannedId) {
                if (appState.huntMode.isActive) {
                    if (scannedId === appState.huntMode.targetId) {
                        showFeedback('hunt_success', scannedId, "ITEM-ALVO ENCONTRADO!");
                        sendLogToN8n({ scannedId, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: appState.operatorName });
                        toggleHuntMode();
                    }
                    return;
                }
                if (appState.isPaused) return;
                if (appState.isFastMode) { const now = Date.now(); if (now - appState.lastScanTime < 350) return; appState.lastScanTime = now; } 
                else { appState.isPaused = true; }

                const now = new Date();
                // Busca descrição se houver (OCR)
                const desc = appState.idDescriptions.get(scannedId) || "";
                
                let status = 'Não Encontrado';
                let activeZone = appState.activeZoneId;
                let foundZone = null;

                // 1. Verifica Missort (Prioridade)
                if (activeZone) {
                    for (const [zid, set] of appState.inventoryZoneData) {
                        if (zid !== activeZone && set.has(scannedId)) {
                            status = `Missort (${appState.inventoryZones.find(z=>z.id===zid)?.name || zid})`;
                            foundZone = zid;
                            showFeedback('warning_missort', scannedId, `ALERTA: ITEM DE ${foundZone.toUpperCase()}`);
                            break;
                        }
                    }
                }

                // 2. Verifica Duplicado
                if (status === 'Não Encontrado' && appState.foundIds.some(i => i.id === scannedId)) {
                    status = 'Duplicado';
                    showFeedback('warning', scannedId, 'JÁ ENCONTRADO');
                }

                // 3. Verifica Sucesso (Lista Principal)
                if (status === 'Não Encontrado' && appState.idsToFind.has(scannedId)) {
                    status = 'Encontrado';
                    showFeedback('success', scannedId, desc); // Mostra descrição no feedback
                    appState.idsToFind.delete(scannedId);
                    appState.foundIds.unshift({ id: scannedId, timestamp: now });
                }

                // 4. Verifica Sucesso (Outras Zonas - sem ser missort ativo)
                if (status === 'Não Encontrado') {
                    for (const [zid, set] of appState.inventoryZoneData) {
                        if (set.has(scannedId)) {
                            const zn = appState.inventoryZones.find(z=>z.id===zid)?.name || zid;
                            status = `Encontrado (${zn})`;
                            foundZone = zid;
                            showFeedback('success', scannedId, `ZONA: ${zn}`);
                            appState.zoneFinds.set(zid, (appState.zoneFinds.get(zid) || 0) + 1);
                            break;
                        }
                    }
                }

                // 5. Se ainda for erro
                if (status === 'Não Encontrado') {
                    showFeedback('error', scannedId);
                    appState.notFoundIds.unshift({ id: scannedId, timestamp: now });
                }

                // Log e Estado
                const logEntry = { 
                    id: scannedId, 
                    time: now, 
                    status: status, 
                    activeZone: activeZone,
                    description: desc 
                };
                
                appState.scanLog.unshift(logEntry);
                appState.scanHistory.push(now);
                
                updateAllUI();
                saveToLocalStorage();
                
                // Envia para N8N
                sendLogToN8n({
                    scannedId, timestamp: now.toISOString(), status, activeZoneId: activeZone, operator: appState.operatorName, description: desc
                });
            }

            // --- Relatórios ---
            function downloadFullReport() {
                if (appState.scanLog.length === 0) { alert("Sem dados."); return; }
                const data = appState.scanLog.map(l => ({
                    'Data': new Date(l.time).toLocaleDateString(),
                    'Hora': new Date(l.time).toLocaleTimeString(),
                    'ID Pacote': l.id,
                    'Descrição (OCR)': l.description || "-",
                    'Status': l.status,
                    'Zona Ativa': l.activeZone || "-",
                    'Operador': appState.operatorName
                }));
                const ws = XLSX.utils.json_to_sheet(data);
                const wb = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(wb, ws, "Log Completo");
                XLSX.writeFile(wb, `Natefy_Log_${new Date().toISOString().slice(0,10)}.xlsx`);
            }

            async function downloadFlowchart() {
                if (appState.scanLog.length === 0) { alert("Sem dados."); return; }
                const container = document.getElementById('flowchart-canvas-container');
                const total = appState.scanLog.length;
                const ok = appState.scanLog.filter(l=>l.status.includes('Encontrado')).length;
                const err = appState.scanLog.filter(l=>l.status === 'Não Encontrado').length;
                const mis = appState.scanLog.filter(l=>l.status.includes('Missort')).length;
                
                container.innerHTML = `
                    <div style="display:flex;flex-direction:column;align-items:center;gap:30px;background:#0f172a;padding:40px;width:600px;text-align:center;">
                        <h2 style="color:#06b6d4;font-size:28px;margin:0;">Fluxo de Operação</h2>
                        <p style="color:#94a3b8;margin:0;">${appState.operatorName} - ${new Date().toLocaleDateString()}</p>
                        <div class="flow-box" style="border-color:white;width:200px;font-size:18px;">TOTAL BIPADO<br><span style="font-size:32px">${total}</span></div>
                        <div style="font-size:24px;color:white;">⬇</div>
                        <div style="display:flex;gap:20px;width:100%;justify-content:center;">
                            <div class="flow-box" style="border-color:#22c55e;color:#22c55e;width:140px;">SUCESSO<br><span style="font-size:24px">${ok}</span></div>
                            <div class="flow-box" style="border-color:#f97316;color:#f97316;width:140px;">MISSORT<br><span style="font-size:24px">${mis}</span></div>
                            <div class="flow-box" style="border-color:#ef4444;color:#ef4444;width:140px;">ERRO<br><span style="font-size:24px">${err}</span></div>
                        </div>
                    </div>`;
                
                try {
                    const canvas = await html2canvas(container, {backgroundColor: '#0f172a'});
                    const link = document.createElement('a');
                    link.download = 'fluxograma.png';
                    link.href = canvas.toDataURL();
                    link.click();
                } catch(e) { console.error(e); alert("Erro ao gerar imagem"); }
            }

            // --- Analisador (Repetidos) ---
            function runListAnalysis() {
                const idA = document.getElementById("analysis-list-a").value;
                const idB = document.getElementById("analysis-list-b").value;
                if(!idA || !idB) return alert("Selecione duas listas");
                
                const setA = getListById(idA);
                const setB = getListById(idB);
                
                // Encontrar REPETIDOS (Interseção)
                const repeated = [];
                const onlyB = [];
                const onlyA = Array.from(setA);

                setB.forEach(id => {
                    if (setA.has(id)) {
                        repeated.push(id);
                        const idx = onlyA.indexOf(id);
                        if(idx > -1) onlyA.splice(idx, 1);
                    } else {
                        onlyB.push(id);
                    }
                });

                appState.analysisResult = { ok: repeated, sobra: onlyB, faltantes: onlyA };
                document.getElementById("result-ok").textContent = repeated.length;
                document.getElementById("result-sobra").textContent = onlyB.length;
                document.getElementById("result-faltantes").textContent = onlyA.length;
                document.getElementById("analysis-results-container").classList.remove("hidden");
            }

            // --- UI Updates ---
            function updateAllUI() {
                updateFoundListUI();
                updateExceptionsListUI();
                updateScanLogUI();
                updateDashboard();
                updateStatusUI();
            }

            function updateScanLogUI() {
                const list = document.getElementById("scan-log-list");
                if (appState.scanLog.length === 0) list.innerHTML = '<div class="text-center text-slate-500 py-4">Nenhum registro hoje.</div>';
                else {
                    list.innerHTML = appState.scanLog.slice(0, 50).map(l => {
                        let color = "text-white";
                        if(l.status.includes('Missort')) color = "text-orange-400 font-bold";
                        else if(l.status === 'Não Encontrado') color = "text-red-400";
                        else if(l.status.includes('Encontrado')) color = "text-green-400";
                        else if(l.status === 'Duplicado') color = "text-yellow-400";

                        return `
                        <div class="bg-slate-800 p-3 rounded-lg border border-slate-700 flex justify-between items-center">
                            <div class="flex flex-col text-left">
                                <span class="font-mono font-bold text-white text-sm">${l.id}</span>
                                ${l.description ? `<span class="text-[10px] text-cyan-200/70 truncate w-48">${l.description}</span>` : ''}
                                <span class="text-[10px] ${color} uppercase tracking-wider">${l.status}</span>
                            </div>
                            <span class="text-xs text-slate-500 font-mono">${new Date(l.time).toLocaleTimeString('pt-BR', {hour:'2-digit', minute:'2-digit'})}</span>
                        </div>`;
                    }).join("");
                }
            }

            // --- Funções Auxiliares (Manter as mesmas lógicas de antes) ---
            function getListById(id){ return id==='main'?appState.idsToFind : appState.inventoryZoneData.get(id)||new Set(); }
            function handleFileSelect(e, zid) {
                const f = e.target.files[0]; if(!f)return;
                const r = new FileReader();
                const proc = (ids, name) => {
                    const s = new Set(ids);
                    if(zid==='main') { appState.idsToFind=s; appState.foundIds=[]; document.getElementById('file-info').textContent=`"${name}" (${s.size})`; }
                    else { appState.inventoryZoneData.set(zid, s); document.getElementById(`file-info-${zid}`).textContent=`${s.size} IDs`; document.getElementById(`file-info-${zid}`).classList.add('text-green-400'); }
                    saveToLocalStorage(); buildAnalyzerUI(); updateAllUI();
                };
                if(f.name.endsWith('.xlsx')){ r.onload=ev=>{const d=new Uint8Array(ev.target.result), w=XLSX.read(d,{type:'array'}), ws=w.Sheets[w.SheetNames[0]], j=XLSX.utils.sheet_to_json(ws,{header:1}); proc(j.map(r=>String(r[0])).filter(i=>i&&i.trim()), f.name);}; r.readAsArrayBuffer(f); }
                else { r.onload=ev=>{ proc(ev.target.result.trim().split(/[\n\r,]+/).map(i=>i.trim()).filter(i=>i), f.name); }; r.readAsText(f); }
            }
            
            // ... (Manter as funções de exportação CSV, updateDashboard, createCharts, showFeedback, initAudio, playSound, setupTabs, toggleHuntMode, etc. do código anterior, elas não mudaram drasticamente, apenas use updateAllUI onde for relevante) ...
            // [Código omitido para brevidade: Copie as funções auxiliares puras do código anterior se necessário, mas a lógica principal está acima]
            function updateFoundListUI(){document.getElementById("found-count").textContent=appState.foundIds.length;const t=document.getElementById("found-list");0===appState.foundIds.length?t.innerHTML='<li class="text-slate-500">Nenhum item encontrado ainda.</li>':t.innerHTML=appState.foundIds.map(t=>`<li class="p-2 bg-slate-700 rounded-md text-white flex justify-between"><span>${t.id}</span><span class="text-xs text-slate-400">${t.timestamp.toLocaleTimeString("pt-BR")}</span></li>`).join("")}function updateExceptionsListUI(){document.getElementById("exceptions-count").textContent=appState.notFoundIds.length;const t=document.getElementById("exceptions-list");0===appState.notFoundIds.length?t.innerHTML='<li class="text-slate-500">Nenhuma exceção registrada.</li>':t.innerHTML=appState.notFoundIds.map(t=>`<li class="p-2 bg-slate-700 rounded-md text-white flex justify-between"><span>${t.id}</span><span class="text-xs text-slate-400">${t.timestamp.toLocaleTimeString("pt-BR")}</span></li>`).join("")}function updateDashboard(){const t=appState.idsToFind.size+appState.foundIds.length,o=appState.foundIds.length,e=t>0?o/t*100:0;if(document.getElementById("progress-text").textContent=`${Math.round(e)}%`,appState.charts.progress&&(appState.charts.progress.data.datasets[0].data=[e,100-e],appState.charts.progress.update()),appState.scanHistory.length>1){const t=appState.scanHistory.slice(-10);let o=0;for(let e=1;e<t.length;e++)o+=new Date(t[e])-new Date(t[e-1]);const n=o/(t.length-1)/1e3;!isNaN(n)&&n>0?(document.getElementById("kpi-avg-time").textContent=`${n.toFixed(1)} s`,document.getElementById("kpi-bpm").textContent=n>0?Math.round(60/n):"--"):(document.getElementById("kpi-avg-time").textContent="-- s",document.getElementById("kpi-bpm").textContent="--")}else document.getElementById("kpi-avg-time").textContent="-- s",document.getElementById("kpi-bpm").textContent="--";const n=document.getElementById("zone-finds-container");if(0===appState.zoneFinds.size)n.innerHTML='<p class="text-slate-500 text-center">Nenhum item de inventário escaneado ainda.</p>';else{n.innerHTML="";appState.zoneFinds.forEach((t,o)=>{const e=appState.inventoryZones.find(t=>t.id===o),a=e?e.name:o,s=document.createElement("div");s.className="flex justify-between items-center bg-slate-800 p-2 rounded-lg",s.innerHTML=`<span class="font-medium text-slate-300">${a}</span><span class="font-bold text-white text-lg">${t}</span>`,n.appendChild(s)})}}function createCharts(){const t=document.getElementById("progressChart").getContext("2d");appState.charts.progress&&appState.charts.progress.destroy(),appState.charts.progress=new Chart(t,{type:"doughnut",data:{datasets:[{data:[0,100],backgroundColor:["#0ea5e9","#334155"],borderColor:"#1e293b",borderWidth:4,cutout:"75%"}]},options:{responsive:!0,maintainAspectRatio:!1,plugins:{tooltip:{enabled:!1}}}})}function showFeedback(t,e,n){let o=t==="success"?"ENCONTRADO":"NÃO ENCONTRADO",a=e;n&&(t==="warning_missort"||t==="hunt_success"?o=n:a=`<span class="text-sm block mt-2 text-cyan-200">${n}</span><span class="text-xs block text-slate-400 mt-1">${e}</span>`);const s=document.getElementById("feedback-overlay");let i="feedback-pulse",c="";t==="success"?c="radial-gradient(circle, rgba(34, 197, 94, 0.95) 0%, rgba(30, 41, 59, 0) 70%)":t==="warning"?c="radial-gradient(circle, rgba(245, 158, 11, 0.9) 0%, rgba(30, 41, 59, 0) 70%)":t==="warning_missort"?c="radial-gradient(circle, rgba(249, 115, 22, 0.9) 0%, rgba(30, 41, 59, 0) 70%)":t==="hunt_success"?(c="radial-gradient(circle, rgba(134, 239, 172, 0.9) 0%, rgba(30, 41, 59, 0) 70%)",i="hunt-success-pulse"):c="radial-gradient(circle, rgba(239, 68, 68, 0.9) 0%, rgba(30, 41, 59, 0) 70%)",s.style.background=c,s.innerHTML=`<div class="${i} text-center px-4"><div class="text-5xl font-black mb-2">${o}</div><div class="text-2xl font-mono p-3 bg-black/40 rounded-xl backdrop-blur-md border border-white/10 shadow-lg">${a}</div></div>`,s.style.opacity="1",playSound(t),navigator.vibrate&&(t==="success"?navigator.vibrate(200):t==="error"?navigator.vibrate([100,50,100]):t==="warning"?navigator.vibrate([80,80]):t==="warning_missort"?navigator.vibrate([120,60,120]):t==="hunt_success"&&navigator.vibrate([500,100,500]));const d=t==="hunt_success",l=appState.isFastMode&&!d?250:SCAN_DELAY,r=d?2500:l;setTimeout(()=>{s.style.opacity="0",!appState.isFastMode&&!d&&(appState.isPaused=!1)},r)}function initAudio(){appState.audioContext||(appState.audioContext=new(window.AudioContext||window.webkitAudioContext))}function playSound(t){if(!appState.audioContext)return;try{const o=appState.audioContext.createOscillator(),e=appState.audioContext.createGain();o.connect(e),e.connect(appState.audioContext.destination),e.gain.setValueAtTime(.3,appState.audioContext.currentTime),t==="success"?o.frequency.setValueAtTime(1200,o.context.currentTime):t==="error"?(o.frequency.setValueAtTime(180,o.context.currentTime),o.type="square"):t==="warning"?(o.frequency.setValueAtTime(600,o.context.currentTime),o.type="triangle"):t==="warning_missort"?(o.frequency.setValueAtTime(800,o.context.currentTime),o.frequency.setValueAtTime(400,o.context.currentTime+.07),o.type="sawtooth"):t==="hunt_success"&&(o.frequency.setValueAtTime(1e3,o.context.currentTime),o.frequency.linearRampToValueAtTime(2e3,o.context.currentTime+.3)),o.start(),o.stop(appState.audioContext.currentTime+(t==="hunt_success"?.4:.15))}catch(t){appState.audioContext=new(window.AudioContext||window.webkitAudioContext)}}function exportFoundIds(){if(0===appState.foundIds.length)return void alert("Nenhum ID foi encontrado para exportar.");let t="data:text/csv;charset=utf-8,ID_Encontrado,Data_Verificacao,Hora_Verificacao\n";appState.foundIds.forEach(e=>{t+=[e.id,e.timestamp.toLocaleDateString("pt-BR"),e.timestamp.toLocaleTimeString("pt-BR")].join(",")+"\n"});const e=encodeURI(t),n=document.createElement("a");n.setAttribute("href",e),n.setAttribute("download",`sessao_scanner_encontrados_${(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-")}.csv`),document.body.appendChild(n),n.click(),document.body.removeChild(n)}function exportExceptions(){if(0===appState.notFoundIds.length)return void alert("Nenhuma exceção foi encontrada para exportar.");let t="data:text/csv;charset=utf-8,ID_Nao_Encontrado,Data_Verificacao,Hora_Verificacao\n";appState.notFoundIds.forEach(e=>{t+=[e.id,e.timestamp.toLocaleDateString("pt-BR"),e.timestamp.toLocaleTimeString("pt-BR")].join(",")+"\n"});const e=encodeURI(t),n=document.createElement("a");n.setAttribute("href",e),n.setAttribute("download",`sessao_scanner_excecoes_${(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-")}.csv`),document.body.appendChild(n),n.click(),document.body.removeChild(n)}function exportAnalysisResults(){const{ok:t,sobra:e,faltantes:n}=appState.analysisResult;if(0===t.length&&0===e.length&&0===n.length)return void alert("Nenhum resultado de análise para exportar.");let o="data:text/csv;charset=utf-8,Itens_OK,Itens_Sobra,Itens_Faltantes\n";const a=Math.max(t.length,e.length,n.length);for(let s=0;s<a;s++){o+=`"${t[s]||""}","${e[s]||""}","${n[s]||""}"\n`}const s=encodeURI(o),i=document.createElement("a");i.setAttribute("href",s),i.setAttribute("download",`analise_listas_${(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-")}.csv`),document.body.appendChild(i),i.click(),document.body.removeChild(i)}function resumeScannerIfNeeded(){appState.huntMode.isActive||appState.html5QrCode&&2===appState.html5QrCode.getState()&&appState.html5QrCode.resume()}function startScanner(){appState.html5QrCode&&appState.html5QrCode.isScanning&&appState.html5QrCode.stop(),appState.html5QrCode=new Html5Qrcode("reader");const t={fps:15,qrbox:(t,e)=>{const n=.8*Math.min(t,e);return{width:n,height:n}}};appState.html5QrCode.start({facingMode:"environment"},t,t=>processScan(t)).catch(t=>{appState.huntMode.isActive||t&&"NotAllowedError"!==t.name&&alert("ERRO AO INICIAR A CÂMARA: "+t.message)})}

            initialize();
            if ('serviceWorker' in navigator) navigator.serviceWorker.register('./sw.js');
        });
    </script>
</body>
</html>
