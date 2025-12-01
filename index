<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Natefy Scanner</title>

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
    
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        html { height: 100%; }
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; position: relative; min-height: 100%; overflow: hidden; }
        #reader__scan_region { border: 4px solid rgba(255, 255, 255, 0.5) !important; border-radius: 1.5rem; background: none !important; box-shadow: 0 0 20px rgba(0, 255, 255, 0.3); }
        .scan-line { position: absolute; left: 5%; top: 10px; width: 90%; height: 4px; background: linear-gradient(to right, transparent, #06b6d4, transparent); box-shadow: 0 0 15px #06b6d4, 0 0 5px #fff; border-radius: 4px; animation: scan-animation 2.5s infinite ease-in-out; }
        @keyframes scan-animation { 0% { transform: translateY(0); } 50% { transform: translateY(calc(100% - 20px)); } 100% { transform: translateY(0); } }
        #controls-panel { background: rgba(30, 41, 59, 0.8); backdrop-filter: blur(16px); border-top: 1px solid rgba(71, 85, 105, 0.5); transform: translateY(calc(100% - 70px)); transition: transform 0.3s ease-in-out; padding-bottom: env(safe-area-inset-bottom, 0); }
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
        
        #login-modal { position: fixed; inset: 0; z-index: 100; background-color: rgba(15, 23, 42, 0.95); display: flex; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        #login-modal.hidden { display: none; }
        
        /* Loading Overlay para OCR */
        #ocr-loading { position: fixed; inset: 0; z-index: 90; background-color: rgba(0,0,0,0.8); display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
        #ocr-loading.hidden { display: none; }
    </style>
</head>
<body class="text-slate-200">
    <div id="global-status-bar" class="fixed top-0 left-0 right-0 z-20 p-2 text-center text-xs font-bold tracking-wider transition-all duration-300 flex justify-between px-4 bg-slate-900/50 backdrop-blur-sm">
        <span id="status-zone">ZONA: --</span>
        <span id="status-operator" class="text-cyan-400">OP: --</span>
    </div>

    <div id="ocr-loading" class="hidden">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-cyan-500 mb-4"></div>
        <p class="text-lg font-bold">Lendo imagem...</p>
        <p class="text-sm text-slate-400" id="ocr-status-text">Inicializando...</p>
    </div>

    <div id="login-modal">
        <div class="bg-slate-800 p-6 rounded-xl shadow-2xl w-11/12 max-w-md text-center border border-slate-600">
            <h2 class="text-2xl font-bold text-white mb-4">Identificação</h2>
            <p class="text-slate-400 mb-6 text-sm">Por favor, informe seu nome ou matrícula para iniciar a operação.</p>
            <input type="text" id="operator-input" class="w-full bg-slate-700 text-white p-3 rounded-lg mb-4 border border-slate-600 focus:border-cyan-500 outline-none text-center text-lg" placeholder="Seu Nome">
            <button id="login-btn" class="w-full bg-cyan-600 hover:bg-cyan-700 text-white font-bold py-3 px-4 rounded-lg transition-colors">INICIAR</button>
        </div>
    </div>

    <div id="reader" class="fixed top-0 left-0 w-full h-full z-1"><div class="scan-line"></div></div>
    <div id="feedback-overlay" class="fixed inset-0 z-50 flex items-center justify-center p-4 text-white font-black opacity-0 pointer-events-none transition-opacity duration-200"></div>

    <div id="controls-panel" class="fixed bottom-0 left-0 right-0 z-10 rounded-t-2xl">
        <div id="panel-handle" class="w-full h-10 flex justify-center items-center cursor-pointer">
              <div class="w-10 h-1.5 bg-slate-500 rounded-full"></div>
        </div>
        <div class="w-full max-w-lg mx-auto px-4 pb-4">
             <div class="w-full overflow-x-auto pb-2">
                <div class="flex justify-start mb-4 space-x-2 sm:space-x-4">
                    <button data-view="procurar" class="tab-btn tab-active py-2 px-4 font-semibold text-sm sm:text-base">Procurar</button>
                    <button data-view="encontrados" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Encontrados (<span id="found-count">0</span>)</button>
                    <button data-view="dashboard" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Dashboard</button>
                    <button data-view="analisador" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Analisador</button>
                    <button data-view="inventario" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Inventário</button>
                    <button data-view="excecoes" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Exceções (<span id="exceptions-count">0</span>)</button>
                    <button data-view="log" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Log</button>
                </div>
            </div>

            <div data-view-content="procurar" class="text-center">
                 <div class="grid grid-cols-2 gap-2 mb-4">
                     <button id="load-file-btn" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-3 px-2 rounded-lg text-sm border border-slate-600">
                         📂 Excel/CSV
                     </button>
                     <button id="load-image-btn" class="bg-cyan-700 hover:bg-cyan-600 text-white font-bold py-3 px-2 rounded-lg text-sm border border-cyan-500 shadow-lg shadow-cyan-500/20">
                         📷 Ler Foto da Lista
                     </button>
                 </div>
                 
                <input type="file" id="file-input" class="hidden" accept=".txt,.csv,.xlsx">
                <input type="file" id="image-input" class="hidden" accept="image/*"> <p id="file-info" class="text-xs text-green-400 mt-2 min-h-[1rem]"></p>
                
                 <div class="mt-4 text-left border-t border-slate-700 pt-4 space-y-4">
                    <div class="p-3 bg-slate-800 rounded-lg">
                        <label class="text-sm text-slate-300 font-medium">Modo Caça ao Tesouro</label>
                        <p id="hunt-status" class="text-xs text-cyan-400 h-4 mb-2"></p>
                        <div class="flex gap-2">
                            <input type="text" id="hunt-target-id" class="w-full bg-slate-700 p-2 rounded-lg font-mono text-white" placeholder="ID para caçar...">
                            <button id="hunt-toggle-btn" class="bg-blue-600 hover:bg-blue-700 font-bold px-4 rounded-lg whitespace-nowrap">Caçar</button>
                        </div>
                    </div>
                    <div class="flex justify-between items-center">
                        <label for="fast-mode-toggle" class="text-sm text-slate-300 font-medium">Modo Rápido (Sem pausa)</label>
                        <div class="relative inline-block w-10 align-middle select-none transition duration-200 ease-in">
                            <input type="checkbox" name="fast-mode-toggle" id="fast-mode-toggle" class="toggle-checkbox absolute block w-5 h-5 rounded-full bg-white border-4 appearance-none cursor-pointer"/>
                            <label for="fast-mode-toggle" class="toggle-bg block overflow-hidden h-6 w-11 rounded-full bg-slate-600 cursor-pointer"></label>
                        </div>
                    </div>
                    <button id="clear-session-btn" class="w-full bg-red-800 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg text-sm">Limpar Sessão (Apagar Dados)</button>
                    <div class="mt-2 text-center">
                        <button id="change-operator-btn" class="text-xs text-slate-500 underline">Trocar Operador</button>
                    </div>
                    <div>
                        <label for="manual-input" class="text-xs text-slate-400">Ou digite o ID manualmente:</label>
                        <div class="flex gap-2 mt-1">
                            <input type="text" id="manual-input" class="w-full bg-slate-700 p-2 rounded-lg font-mono text-white" placeholder="ID do pacote...">
                            <button id="manual-check-btn" class="bg-blue-600 hover:bg-blue-700 font-bold px-4 rounded-lg">Verificar</button>
                        </div>
                    </div>
                </div>
            </div>

            <div data-view-content="encontrados" class="hidden">
                 <button id="export-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-5 rounded-lg mb-4">Exportar Encontrados (.csv)</button>
                <div class="max-h-48 overflow-y-auto pr-2"><ul id="found-list" class="space-y-2 text-center font-mono text-sm"></ul></div>
            </div>
            <div data-view-content="dashboard" class="hidden">
                 <h3 class="text-lg font-bold text-center text-white mb-4">Performance da Sessão</h3>
                <div class="grid grid-cols-3 gap-4 mb-4">
                    <div class="flex flex-col items-center justify-center p-4 bg-slate-800 rounded-lg">
                          <h4 class="text-sm font-semibold text-slate-400 mb-2">Progresso</h4>
                          <div class="relative w-24 h-24"><canvas id="progressChart"></canvas><div id="progress-text" class="absolute inset-0 flex items-center justify-center text-2xl font-bold">0%</div></div>
                    </div>
                    <div class="p-4 bg-slate-800 rounded-lg text-center"><h4 class="text-sm font-semibold text-slate-400">Tempo Médio / Bip</h4><p id="kpi-avg-time" class="text-4xl font-black text-white mt-2">-- s</p></div>
                    <div class="p-4 bg-slate-800 rounded-lg text-center"><h4 class="text-sm font-semibold text-slate-400">Bips por Minuto</h4><p id="kpi-bpm" class="text-4xl font-black text-white mt-2">--</p></div>
                </div>
                <div class="border-t border-slate-700 pt-4">
                     <h3 class="text-lg font-bold text-center text-white mb-2">Contagem por Zona</h3>
                     <div id="zone-finds-container" class="space-y-2 max-h-32 overflow-y-auto pr-2">
                         <p class="text-slate-500 text-center">Nenhum item de inventário escaneado ainda.</p>
                     </div>
                </div>
            </div>
             <div data-view-content="analisador" class="hidden space-y-4">
                <h3 class="text-lg font-bold text-center text-white">Analisador de Listas</h3>
                <div class="space-y-2">
                    <label class="text-sm font-medium text-slate-300">Lista A</label>
                    <select id="analysis-list-a" class="w-full bg-slate-700 p-2 rounded-lg text-white"><option value="">Selecione...</option></select>
                </div>
                <div class="space-y-2">
                    <label class="text-sm font-medium text-slate-300">Lista B</label>
                    <select id="analysis-list-b" class="w-full bg-slate-700 p-2 rounded-lg text-white"><option value="">Selecione...</option></select>
                </div>
                <button id="run-analysis-btn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-5 rounded-lg">Analisar</button>
                <div id="analysis-results-container" class="hidden pt-4 border-t border-slate-700 space-y-3">
                    <div class="grid grid-cols-3 gap-2 text-center">
                        <div class="bg-green-800 p-3 rounded-lg"><span class="block text-2xl font-bold" id="result-ok">--</span><span class="text-xs">OK</span></div>
                        <div class="bg-orange-800 p-3 rounded-lg"><span class="block text-2xl font-bold" id="result-sobra">--</span><span class="text-xs">Sobra</span></div>
                        <div class="bg-red-800 p-3 rounded-lg"><span class="block text-2xl font-bold" id="result-faltantes">--</span><span class="text-xs">Faltantes</span></div>
                    </div>
                    <button id="export-analysis-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg text-sm">Exportar (.csv)</button>
                </div>
            </div>
            <div data-view-content="inventario" class="hidden">
                 <h3 class="text-lg font-bold text-center text-white mb-4">Carregar Listas por Zona</h3>
                <div id="inventory-zones-container" class="space-y-4 max-h-64 overflow-y-auto pr-2"></div>
            </div>
            <div data-view-content="excecoes" class="hidden">
                 <button id="export-exceptions-btn" class="w-full bg-amber-600 hover:bg-amber-700 text-white font-bold py-3 px-5 rounded-lg mb-4">Exportar Exceções (.csv)</button>
                <div class="max-h-48 overflow-y-auto pr-2"><ul id="exceptions-list" class="space-y-2 text-center font-mono text-sm"></ul></div>
            </div>
            <div data-view-content="log" class="hidden">
                  <h3 class="text-lg font-bold text-center text-white mb-4">Log de Atividade</h3>
                <div class="max-h-48 overflow-y-auto pr-2"><ul id="scan-log-list" class="space-y-2 text-center font-mono text-xs"></ul></div>
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
                idDescriptions: new Map(), // NOVO: Mapa de ID -> Descrição
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

            // --- Lógica de Login ---
            function checkLogin() {
                if (!appState.operatorName) { document.getElementById('login-modal').classList.remove('hidden'); } 
                else { document.getElementById('login-modal').classList.add('hidden'); updateStatusUI(); }
            }
            document.getElementById('login-btn').addEventListener('click', () => {
                const name = document.getElementById('operator-input').value.trim();
                if (name) { appState.operatorName = name; saveToLocalStorage(); checkLogin(); startScanner(); } 
                else { alert("Por favor, digite seu nome."); }
            });
            document.getElementById('change-operator-btn').addEventListener('click', () => {
                if(confirm("Deseja trocar de operador?")) { appState.operatorName = null; document.getElementById('operator-input').value = ''; checkLogin(); }
            });

            // --- Lógica de OCR (Ler Imagem) ---
            document.getElementById('load-image-btn').addEventListener('click', () => document.getElementById('image-input').click());
            document.getElementById('image-input').addEventListener('change', handleImageUpload);

            async function handleImageUpload(e) {
                const file = e.target.files[0];
                if (!file) return;

                const ocrOverlay = document.getElementById('ocr-loading');
                const ocrText = document.getElementById('ocr-status-text');
                ocrOverlay.classList.remove('hidden');
                ocrText.textContent = "Iniciando motor OCR...";

                try {
                    const worker = Tesseract.createWorker({
                        logger: m => {
                            if(m.status === 'recognizing text') {
                                ocrText.textContent = `Lendo texto: ${Math.round(m.progress * 100)}%`;
                            } else {
                                ocrText.textContent = m.status;
                            }
                        }
                    });

                    await worker.load();
                    await worker.loadLanguage('eng'); // Inglês geralmente lê melhor números e texto misto
                    await worker.initialize('eng');
                    
                    ocrText.textContent = "Processando imagem...";
                    const { data: { text } } = await worker.recognize(file);
                    
                    ocrText.textContent = "Extraindo dados...";
                    await worker.terminate();

                    // Processar o texto extraído
                    const lines = text.split('\n');
                    let count = 0;
                    
                    // Regex para encontrar IDs que parecem com os da sua foto (459...)
                    // Procura por sequências de 10 a 12 dígitos
                    const idRegex = /(\d{10,12})/; 

                    const newIds = new Set();

                    lines.forEach(line => {
                        const match = line.match(idRegex);
                        if (match) {
                            const id = match[0];
                            // Tenta limpar o ID da linha para pegar o resto como descrição
                            // Remove caracteres estranhos do início da descrição
                            let description = line.replace(id, '').trim();
                            // Remove caracteres comuns de lixo do OCR no inicio da descrição (ex: . , - >)
                            description = description.replace(/^[\.\,\-\>\s]+/, '');
                            
                            if (id.length >= 9) { // Validação básica
                                newIds.add(id);
                                appState.idDescriptions.set(id, description || "Sem descrição");
                                count++;
                            }
                        }
                    });

                    if (count > 0) {
                        appState.idsToFind = newIds;
                        appState.foundIds = []; // Resetar encontrados ao carregar nova lista
                        updateFoundListUI();
                        updateDashboard();
                        document.getElementById('file-info').textContent = `Foto carregada: ${count} itens identificados.`;
                        alert(`Sucesso! ${count} itens lidos da imagem.`);
                    } else {
                        alert("Não foi possível identificar IDs na imagem. Tente uma foto mais clara e focada na tabela.");
                    }

                } catch (err) {
                    console.error(err);
                    alert("Erro ao processar imagem: " + err.message);
                } finally {
                    ocrOverlay.classList.add('hidden');
                    saveToLocalStorage();
                }
            }

            // --- Funções Padrão ---
            function resumeScannerIfNeeded() { if (!appState.huntMode.isActive && appState.html5QrCode && appState.html5QrCode.getState() === 2) { try{appState.html5QrCode.resume()}catch(e){} } }
            const togglePanel = () => { controlsPanel.classList.toggle('open'); if (!controlsPanel.classList.contains('open')) resumeScannerIfNeeded(); };
            panelHandle.addEventListener('click', togglePanel);
            
            // Toques no painel
            let touchStartY = 0;
            document.addEventListener('touchstart', e => { if (e.target === panelHandle || controlsPanel.contains(e.target)) touchStartY = e.touches[0].clientY; });
            document.addEventListener('touchend', e => { if(touchStartY===0)return; const touchEndY=e.changedTouches[0].clientY; if(touchStartY-touchEndY>50)controlsPanel.classList.add('open'); else if(touchEndY-touchStartY>50){controlsPanel.classList.remove('open'); resumeScannerIfNeeded();} touchStartY=0; });

            // Persistência
            function saveToLocalStorage() {
                 try {
                    const dataToSave = {
                        idsToFind: Array.from(appState.idsToFind),
                        idDescriptions: Array.from(appState.idDescriptions.entries()), // Salva as descrições
                        inventoryZoneData: Array.from(appState.inventoryZoneData.entries()).map(([k, v]) => [k, Array.from(v)]),
                        foundIds: appState.foundIds,
                        notFoundIds: appState.notFoundIds,
                        zoneFinds: Array.from(appState.zoneFinds.entries()),
                        scanHistory: appState.scanHistory,
                        scanLog: appState.scanLog,
                        activeZoneId: appState.activeZoneId,
                        huntMode: appState.huntMode,
                        operatorName: appState.operatorName
                    };
                    localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave));
                } catch (e) { console.error("Erro save:", e); }
            }
            function loadFromLocalStorage() {
                 const savedData = localStorage.getItem(STORAGE_KEY);
                if (!savedData) return;
                try {
                    const data = JSON.parse(savedData);
                    appState.idsToFind = new Set(data.idsToFind || []);
                    appState.idDescriptions = new Map(data.idDescriptions || []); // Carrega descrições
                    appState.inventoryZoneData = new Map((data.inventoryZoneData || []).map(([k, v]) => [k, new Set(v)]));
                    appState.foundIds = (data.foundIds || []).map(i => ({...i, timestamp: new Date(i.timestamp)}));
                    appState.notFoundIds = data.notFoundIds || [];
                    appState.zoneFinds = new Map(data.zoneFinds || []);
                    appState.scanHistory = (data.scanHistory || []).map(ts => new Date(ts));
                    appState.scanLog = (data.scanLog || []).map(i => ({...i, time: new Date(i.time)}));
                    appState.activeZoneId = data.activeZoneId || null;
                    appState.huntMode = data.huntMode || { isActive: false, targetId: null };
                    appState.operatorName = data.operatorName || null;
                    const total = appState.idsToFind.size + appState.foundIds.length;
                    if (total > 0) { document.getElementById('file-info').textContent = `Sessão carregada (${total} IDs)`; }
                } catch (e) { console.error("Erro load:", e); localStorage.removeItem(STORAGE_KEY); }
            }
            function clearSession() { if (confirm("Limpar dados?")) { localStorage.removeItem(STORAGE_KEY); window.location.reload(); } }
            function toggleFastMode() { appState.isFastMode = document.getElementById('fast-mode-toggle').checked; }

            async function sendLogToN8n(logData) {
                if (!N8N_REALTIME_WEBHOOK_URL) return;
                logData.operator = appState.operatorName || "Desconhecido";
                try {
                    const controller = new AbortController();
                    const timeoutId = setTimeout(() => controller.abort(), 5000); 
                    const response = await fetch(N8N_REALTIME_WEBHOOK_URL, {
                        method: 'POST', headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(logData), keepalive: true, signal: controller.signal
                    });
                    clearTimeout(timeoutId); 
                } catch (error) { console.error('Erro log n8n:', error); }
            }

            function initialize() {
                loadFromLocalStorage();
                buildInventoryZoneUI();
                buildAnalyzerUI();
                checkLogin();
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', (e) => handleFileSelect(e, 'main'));
                document.getElementById('export-btn').addEventListener('click', exportFoundIds);
                document.getElementById('export-exceptions-btn').addEventListener('click', exportExceptions);
                document.getElementById('clear-session-btn').addEventListener('click', clearSession);
                document.getElementById('fast-mode-toggle').addEventListener('change', toggleFastMode);
                document.getElementById('hunt-toggle-btn').addEventListener('click', toggleHuntMode);
                document.getElementById('run-analysis-btn').addEventListener('click', runListAnalysis);
                document.getElementById('export-analysis-btn').addEventListener('click', exportAnalysisResults);
                document.body.addEventListener('click', initAudio, { once: true });
                setupTabs();
                // startScanner chamado no login
                const mIn = document.getElementById('manual-input');
                document.getElementById('manual-check-btn').addEventListener('click', () => { if(mIn.value.trim()){ processScan(mIn.value.trim()); mIn.value=''; } });
                mIn.addEventListener('keydown', (e) => { if(e.key==='Enter'){ e.preventDefault(); if(mIn.value.trim()){ processScan(mIn.value.trim()); mIn.value=''; } } });
                createCharts();
                updateFoundListUI(); updateExceptionsListUI(); updateScanLogUI(); updateDashboard(); updateActiveZoneUI(appState.activeZoneId); updateHuntModeUI();
            }
             
            // ... (Funções de UI: buildInventoryZoneUI, setActiveZone, updateActiveZoneUI, updateStatusUI, toggleHuntMode, updateHuntModeUI, handleFileSelect - iguais) ...
            function buildInventoryZoneUI() {
                const container = document.getElementById('inventory-zones-container');
                container.innerHTML = `<button id="clear-active-zone-btn" class="w-full bg-slate-600 hover:bg-slate-500 text-white font-bold py-2 px-4 rounded-lg text-sm ${appState.activeZoneId ? '' : 'hidden'}">Limpar Zona Ativa</button>`;
                appState.inventoryZones.forEach(zone => {
                    if (!appState.inventoryZoneData.has(zone.id)) appState.inventoryZoneData.set(zone.id, new Set());
                    const div = document.createElement('div');
                    div.className = "p-3 bg-slate-800 rounded-lg";
                    div.innerHTML = `
                        <div class="flex items-center justify-between mb-3">
                            <div><p class="font-semibold text-white">${zone.name}</p><p id="file-info-${zone.id}" class="text-xs text-slate-400">Nenhum ficheiro carregado.</p></div>
                            <button data-zone-id="${zone.id}" class="set-active-zone-btn bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-3 rounded-lg text-sm whitespace-nowrap">Ativar</button>
                        </div>
                        <div class="flex gap-2"><button data-zone-id="${zone.id}" class="load-zone-file-btn bg-cyan-700 hover:bg-cyan-600 text-white font-bold py-2 px-3 rounded-lg text-sm w-full">Carregar Lista</button><input type="file" id="file-input-${zone.id}" class="hidden" accept=".txt,.csv,.xlsx"></div>`;
                    const existingData = appState.inventoryZoneData.get(zone.id);
                    if (existingData && existingData.size > 0) {
                        const infoP = div.querySelector(`#file-info-${zone.id}`);
                        infoP.textContent = `${existingData.size} IDs carregados.`;
                        infoP.classList.add('text-green-400');
                    }
                    container.appendChild(div);
                });
                document.getElementById('clear-active-zone-btn').addEventListener('click', () => setActiveZone(null));
                document.querySelectorAll('.load-zone-file-btn').forEach(btn => btn.addEventListener('click', (e) => document.getElementById(`file-input-${e.currentTarget.dataset.zoneId}`).click()));
                document.querySelectorAll('.set-active-zone-btn').forEach(btn => btn.addEventListener('click', (e) => { const zid = e.currentTarget.dataset.zoneId; setActiveZone(appState.activeZoneId === zid ? null : zid); }));
                document.querySelectorAll('input[type="file"]').forEach(input => { if (input.id.startsWith('file-input-')) input.addEventListener('change', (e) => handleFileSelect(e, e.target.id.replace('file-input-', ''))); });
                updateActiveZoneUI(appState.activeZoneId);
            }
            function setActiveZone(zid) { appState.activeZoneId = zid; updateActiveZoneUI(zid); saveToLocalStorage(); }
            function updateActiveZoneUI(aid) {
                updateStatusUI();
                document.querySelectorAll('.set-active-zone-btn').forEach(btn => {
                    if (btn.dataset.zoneId === aid) { btn.textContent = 'ATIVA'; btn.classList.replace('bg-blue-600', 'bg-green-600'); btn.classList.replace('hover:bg-blue-700', 'hover:bg-green-700'); } 
                    else { btn.textContent = 'Ativar'; btn.classList.replace('bg-green-600', 'bg-blue-600'); btn.classList.replace('hover:bg-green-700', 'hover:bg-blue-700'); }
                });
                const clr = document.getElementById('clear-active-zone-btn'); if (clr) clr.classList.toggle('hidden', !aid);
            }
            function updateStatusUI() {
                const z = document.getElementById('status-zone'), o = document.getElementById('status-operator');
                if (appState.activeZoneId) { const zn = appState.inventoryZones.find(i => i.id === appState.activeZoneId); z.textContent = `ZONA: ${zn ? zn.name.toUpperCase() : appState.activeZoneId}`; z.className = 'text-green-400'; } else { z.textContent = 'ZONA: --'; z.className = 'text-slate-400'; }
                o.textContent = appState.operatorName ? `OP: ${appState.operatorName.toUpperCase()}` : 'OP: --';
            }
            function toggleHuntMode() {
                const inp = document.getElementById('hunt-target-id');
                if (appState.huntMode.isActive) { appState.huntMode = { isActive: false, targetId: null }; try{if(appState.html5QrCode&&appState.html5QrCode.getState()===2)appState.html5QrCode.resume()}catch(e){} }
                else { const t = inp.value.trim(); if (!t) { alert("Digite um ID"); return; } appState.huntMode = { isActive: true, targetId: t }; appState.isPaused = false; try{if(appState.html5QrCode&&appState.html5QrCode.getState()===1)appState.html5QrCode.pause(true)}catch(e){} }
                updateHuntModeUI(); saveToLocalStorage();
            }
            function updateHuntModeUI() {
                const inp = document.getElementById('hunt-target-id'), btn = document.getElementById('hunt-toggle-btn'), st = document.getElementById('hunt-status');
                if (appState.huntMode.isActive) { inp.value = appState.huntMode.targetId; inp.disabled = true; btn.textContent = 'Cancelar'; btn.classList.replace('bg-blue-600', 'bg-red-600'); st.textContent = `CAÇANDO: ${appState.huntMode.targetId}`; }
                else { inp.value = ''; inp.disabled = false; btn.textContent = 'Caçar'; btn.classList.replace('bg-red-600', 'bg-blue-600'); st.textContent = ''; }
            }
            function handleFileSelect(e, zid) {
                const f = e.target.files[0]; if (!f) return;
                const r = new FileReader();
                const proc = (ids, fn) => {
                    const s = new Set(ids);
                    if (zid === 'main') { appState.idsToFind = s; appState.foundIds = []; appState.scanHistory = []; appState.scanLog = []; appState.notFoundIds = []; document.getElementById('file-info').textContent = `"${fn}" (${ids.length} IDs)`; updateFoundListUI(); updateExceptionsListUI(); updateScanLogUI(); updateDashboard(); }
                    else { appState.inventoryZoneData.set(zid, s); document.getElementById(`file-info-${zid}`).textContent = `${ids.length} IDs carregados.`; document.getElementById(`file-info-${zid}`).classList.add('text-green-400'); }
                    saveToLocalStorage(); buildAnalyzerUI();
                };
                if (f.name.endsWith('.xlsx')) { r.onload = (ev) => { const d = new Uint8Array(ev.target.result), w = XLSX.read(d, {type: 'array'}), sn = w.SheetNames[0], ws = w.Sheets[sn], j = XLSX.utils.sheet_to_json(ws, { header: 1 }); proc(j.map(r => String(r[0])).filter(i => i && i.trim() !== '' && i !== 'undefined'), f.name); }; r.readAsArrayBuffer(f); }
                else { r.onload = (ev) => { proc(ev.target.result.trim().split(/[\n\r,]+/).map(i => i.trim()).filter(i => i), f.name); }; r.readAsText(f); }
            }

            // --- FUNÇÃO PROCESSSCAN (ATUALIZADA COM DESCRIÇÃO) ---
            function processScan(scannedId) {
                if (appState.huntMode.isActive) {
                    if (scannedId === appState.huntMode.targetId) {
                        showFeedback('hunt_success', scannedId, "ITEM-ALVO ENCONTRADO!");
                        sendLogToN8n({ scannedId, timestamp: new Date().toISOString(), status: 'Hunt Success', activeZoneId: appState.activeZoneId });
                        toggleHuntMode();
                    }
                    return; 
                }
                if (appState.isPaused) return;
                if (appState.isFastMode) { const now = Date.now(); if (now - appState.lastScanTime < 350) return; appState.lastScanTime = now; } 
                else { appState.isPaused = true; }

                const logEntry = { id: scannedId, time: new Date() };
                let n8nLogData = { scannedId, timestamp: logEntry.time.toISOString(), status: '', activeZoneId: appState.activeZoneId };

                // Verifica Missort
                if (appState.activeZoneId) {
                    for (const [zoneId, idSet] of appState.inventoryZoneData.entries()) {
                        if (zoneId !== appState.activeZoneId && idSet.has(scannedId)) {
                            const zn = appState.inventoryZones.find(z => z.id === zoneId)?.name.toUpperCase() || 'OUTRA ZONA';
                            logEntry.status = `Missort (${zn})`; n8nLogData.status = logEntry.status;
                            appState.scanLog.unshift(logEntry); updateScanLogUI();
                            showFeedback('warning_missort', scannedId, `ALERTA: ITEM DE ${zn}`);
                            sendLogToN8n(n8nLogData); saveToLocalStorage(); return;
                        }
                    }
                }
                // Duplicado
                if (appState.foundIds.some(i => i.id === scannedId)) {
                    logEntry.status = 'Duplicado'; n8nLogData.status = logEntry.status;
                    appState.scanLog.unshift(logEntry); updateScanLogUI();
                    showFeedback('warning', scannedId, 'JÁ ENCONTRADO');
                    sendLogToN8n(n8nLogData); return;
                }
                // Encontrado (Principal)
                if (appState.idsToFind.has(scannedId)) {
                    logEntry.status = 'Encontrado'; n8nLogData.status = logEntry.status;
                    appState.scanLog.unshift(logEntry); updateScanLogUI();
                    
                    // --- MUDANÇA AQUI: Busca Descrição ---
                    const description = appState.idDescriptions.get(scannedId);
                    const msg = description ? description.substring(0, 30) + '...' : null; // Corta se for muito longo
                    
                    showFeedback('success', scannedId, msg); // Passa a descrição para o feedback
                    
                    appState.idsToFind.delete(scannedId); appState.foundIds.unshift({ id: scannedId, timestamp: logEntry.time });
                    appState.scanHistory.push(logEntry.time); updateFoundListUI(); updateDashboard();
                    sendLogToN8n(n8nLogData); saveToLocalStorage(); return;
                }
                // Zonas
                for (const [zoneId, idSet] of appState.inventoryZoneData.entries()) {
                    if (idSet.has(scannedId)) {
                        const zn = appState.inventoryZones.find(z => z.id === zoneId)?.name.toUpperCase() || 'ZONA';
                        logEntry.status = `Encontrado (${zn})`; n8nLogData.status = logEntry.status; n8nLogData.foundInZoneId = zoneId;
                        appState.scanLog.unshift(logEntry); updateScanLogUI();
                        showFeedback('success', scannedId, `ENCONTRADO (EM ${zn})`);
                        appState.zoneFinds.set(zoneId, (appState.zoneFinds.get(zoneId) || 0) + 1);
                        updateDashboard(); sendLogToN8n(n8nLogData); saveToLocalStorage(); return;
                    }
                }
                // Erro
                logEntry.status = 'Não Encontrado'; n8nLogData.status = logEntry.status;
                appState.scanLog.unshift(logEntry); updateScanLogUI();
                appState.notFoundIds.unshift({ id: scannedId, timestamp: logEntry.time });
                updateExceptionsListUI(); showFeedback('error', scannedId);
                sendLogToN8n(n8nLogData); saveToLocalStorage();
            }
            
            // ... (Restante das funções auxiliares setupTabs, buildAnalyzerUI, etc. iguais) ...
            function setupTabs(){const t=document.querySelectorAll(".tab-btn");t.forEach(o=>{o.addEventListener("click",()=>{const e=o.dataset.view;appState.currentView=e,t.forEach(n=>n.classList.replace("tab-active","tab-inactive")),o.classList.replace("tab-inactive","tab-active"),document.querySelectorAll("[data-view-content]").forEach(n=>{n.classList.toggle("hidden",n.dataset.viewContent!==e)}),resumeScannerIfNeeded()})})}function buildAnalyzerUI(){const t=document.getElementById("analysis-list-a"),o=document.getElementById("analysis-list-b"),e='<option value="">Selecione uma lista...</option>';if(t.innerHTML=e,o.innerHTML=e,appState.idsToFind.size>0){const n=`<option value="main">Ficheiro Principal (${appState.idsToFind.size} IDs)</option>`;t.innerHTML+=n,o.innerHTML+=n}appState.inventoryZoneData.forEach((n,a)=>{if(n.size>0){const s=appState.inventoryZones.find(l=>l.id===a),i=s?s.name:a,c=`<option value="${a}">${i} (${n.size} IDs)</option>`;t.innerHTML+=c,o.innerHTML+=c}})}function getListSetById(t){return t==="main"?appState.idsToFind:appState.inventoryZoneData.get(t)||new Set}function runListAnalysis(){const t=document.getElementById("analysis-list-a").value,o=document.getElementById("analysis-list-b").value;if(!t||!o)return void alert("Por favor, selecione ambas as listas para analisar.");const e=getListSetById(t),n=getListSetById(o),a=[],s=[],i=Array.from(e);n.forEach(c=>{if(e.has(c)){a.push(c);const d=i.indexOf(c);d>-1&&i.splice(d,1)}else s.push(c)}),appState.analysisResult={ok:a,sobra:s,faltantes:i},document.getElementById("result-ok").textContent=a.length,document.getElementById("result-sobra").textContent=s.length,document.getElementById("result-faltantes").textContent=i.length,document.getElementById("analysis-results-container").classList.remove("hidden")}function exportAnalysisResults(){const{ok:t,sobra:o,faltantes:e}=appState.analysisResult;if(t.length===0&&o.length===0&&e.length===0)return void alert("Nenhum resultado de análise para exportar.");let n="data:text/csv;charset=utf-8,Itens_OK,Itens_Sobra,Itens_Faltantes\n";const a=Math.max(t.length,o.length,e.length);for(let s=0;s<a;s++){const i=t[s]||"",c=o[s]||"",d=e[s]||"";n+=`"${i}","${c}","${d}"\n`}const l=encodeURI(n),r=document.createElement("a");r.setAttribute("href",l);const u=(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-");r.setAttribute("download",`analise_listas_${u}.csv`),document.body.appendChild(r),r.click(),document.body.removeChild(r)}function exportFoundIds(){if(appState.foundIds.length===0)return void alert("Nenhum ID foi encontrado para exportar.");let t="data:text/csv;charset=utf-8,ID_Encontrado,Data_Verificacao,Hora_Verificacao\n";appState.foundIds.forEach(o=>{const e=[o.id,o.timestamp.toLocaleDateString("pt-BR"),o.timestamp.toLocaleTimeString("pt-BR")].join(",");t+=e+"\n"});const o=encodeURI(t),e=document.createElement("a");e.setAttribute("href",o);const n=(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-");e.setAttribute("download",`sessao_scanner_encontrados_${n}.csv`),document.body.appendChild(e),e.click(),document.body.removeChild(e)}function exportExceptions(){if(appState.notFoundIds.length===0)return void alert("Nenhuma exceção foi encontrada para exportar.");let t="data:text/csv;charset=utf-8,ID_Nao_Encontrado,Data_Verificacao,Hora_Verificacao\n";appState.notFoundIds.forEach(o=>{const e=[o.id,o.timestamp.toLocaleDateString("pt-BR"),o.timestamp.toLocaleTimeString("pt-BR")].join(",");t+=e+"\n"});const o=encodeURI(t),e=document.createElement("a");e.setAttribute("href",o);const n=(new Date).toLocaleDateString("pt-BR").replace(/\//g,"-");e.setAttribute("download",`sessao_scanner_excecoes_${n}.csv`),document.body.appendChild(e),e.click(),document.body.removeChild(e)}function startScanner(){if(appState.html5QrCode&&appState.html5QrCode.isScanning)try{appState.html5QrCode.stop()}catch(t){console.warn("Erro stop",t),appState.html5QrCode=null,document.getElementById("reader").innerHTML=""}appState.html5QrCode=new Html5Qrcode("reader");const t={fps:15,qrbox:(t,o)=>{const e=.8*Math.min(t,o);return{width:e,height:e}}};appState.html5QrCode.start({facingMode:"environment"},t,t=>processScan(t)).catch(t=>{appState.huntMode.isActive?console.warn("Caça ativa"):t&&"NotAllowedError"!==t.name&&alert("Erro Câm: "+t.message)})}function updateFoundListUI(){document.getElementById("found-count").textContent=appState.foundIds.length;const t=document.getElementById("found-list");0===appState.foundIds.length?t.innerHTML='<li class="text-slate-500">Nenhum item encontrado ainda.</li>':t.innerHTML=appState.foundIds.map(t=>`<li class="p-2 bg-slate-700 rounded-md text-white flex justify-between"><span>${t.id}</span><span class="text-xs text-slate-400">${t.timestamp.toLocaleTimeString("pt-BR")}</span></li>`).join("")}function updateExceptionsListUI(){document.getElementById("exceptions-count").textContent=appState.notFoundIds.length;const t=document.getElementById("exceptions-list");0===appState.notFoundIds.length?t.innerHTML='<li class="text-slate-500">Nenhuma exceção registrada.</li>':t.innerHTML=appState.notFoundIds.map(t=>`<li class="p-2 bg-slate-700 rounded-md text-white flex justify-between"><span>${t.id}</span><span class="text-xs text-slate-400">${t.timestamp.toLocaleTimeString("pt-BR")}</span></li>`).join("")}function updateScanLogUI(){const t=document.getElementById("scan-log-list");if(0===appState.scanLog.length)t.innerHTML='<li class="text-slate-500">Nenhuma atividade registrada.</li>';else{t.innerHTML=appState.scanLog.slice(0,50).map(t=>{let o="text-white";return"Duplicado"===t.status?o="text-amber-400":"Não Encontrado"===t.status?o="text-red-400":t.status.includes("Encontrado")?o="text-green-400":t.status.includes("Missort")&&(o="text-orange-400 font-bold"),`<li class="p-2 bg-slate-800 rounded-md flex justify-between items-center"><div><span class="font-bold text-white">${t.id}</span><span class="block ${o} text-xs">${t.status}</span></div><span class="text-xs text-slate-400">${t.time.toLocaleTimeString("pt-BR")}</span></li>`}).join("")}}function updateDashboard(){const t=appState.idsToFind.size+appState.foundIds.length,o=appState.foundIds.length,e=t>0?o/t*100:0;if(document.getElementById("progress-text").textContent=`${Math.round(e)}%`,appState.charts.progress&&(appState.charts.progress.data.datasets[0].data=[e,100-e],appState.charts.progress.update()),appState.scanHistory.length>1){const t=appState.scanHistory.slice(-10);let o=0;for(let e=1;e<t.length;e++)o+=t[e]-t[e-1];const n=o/(t.length-1)/1e3;!isNaN(n)&&n>0?(document.getElementById("kpi-avg-time").textContent=`${n.toFixed(1)} s`,document.getElementById("kpi-bpm").textContent=n>0?Math.round(60/n):"--"):(document.getElementById("kpi-avg-time").textContent="-- s",document.getElementById("kpi-bpm").textContent="--")}else document.getElementById("kpi-avg-time").textContent="-- s",document.getElementById("kpi-bpm").textContent="--";const n=document.getElementById("zone-finds-container");if(0===appState.zoneFinds.size)n.innerHTML='<p class="text-slate-500 text-center">Nenhum item de inventário escaneado ainda.</p>';else{n.innerHTML="";appState.zoneFinds.forEach((t,o)=>{const e=appState.inventoryZones.find(t=>t.id===o),a=e?e.name:o,s=document.createElement("div");s.className="flex justify-between items-center bg-slate-800 p-2 rounded-lg",s.innerHTML=`<span class="font-medium text-slate-300">${a}</span><span class="font-bold text-white text-lg">${t}</span>`,n.appendChild(s)})}}function createCharts(){const t=document.getElementById("progressChart").getContext("2d");appState.charts.progress&&appState.charts.progress.destroy(),appState.charts.progress=new Chart(t,{type:"doughnut",data:{datasets:[{data:[0,100],backgroundColor:["#0ea5e9","#334155"],borderColor:"#1e293b",borderWidth:4,cutout:"75%"}]},options:{responsive:!0,maintainAspectRatio:!1,plugins:{tooltip:{enabled:!1}}}})}

            function showFeedback(status, scannedId, messageOverride) {
                // Atualizado para exibir descrição se disponível
                let message = status === 'success' ? 'ENCONTRADO' : 'NÃO ENCONTRADO';
                let subMessage = scannedId;
                
                // Se messageOverride foi passado (a descrição), use-o
                if (messageOverride) {
                    // Se for mensagem de Missort/Hunt, mantenha o texto original
                    if(status === 'warning_missort' || status === 'hunt_success') {
                        message = messageOverride;
                    } else {
                        // Se for encontrado normal, messageOverride é a descrição do produto
                        subMessage = `<span class="text-sm block mt-2 text-cyan-200">${messageOverride}</span><span class="text-xs block text-slate-400 mt-1">${scannedId}</span>`;
                    }
                }

                const feedbackOverlay = document.getElementById('feedback-overlay');
                let pulseClass = "feedback-pulse";
                let bgColor = '';
                
                if (status === 'success') bgColor = 'radial-gradient(circle, rgba(34, 197, 94, 0.95) 0%, rgba(30, 41, 59, 0) 70%)';
                else if (status === 'warning') bgColor = 'radial-gradient(circle, rgba(245, 158, 11, 0.9) 0%, rgba(30, 41, 59, 0) 70%)';
                else if (status === 'warning_missort') bgColor = 'radial-gradient(circle, rgba(249, 115, 22, 0.9) 0%, rgba(30, 41, 59, 0) 70%)'; 
                else if (status === 'hunt_success') { bgColor = 'radial-gradient(circle, rgba(134, 239, 172, 0.9) 0%, rgba(30, 41, 59, 0) 70%)'; pulseClass = "hunt-success-pulse"; } 
                else bgColor = 'radial-gradient(circle, rgba(239, 68, 68, 0.9) 0%, rgba(30, 41, 59, 0) 70%)';
                
                feedbackOverlay.style.background = bgColor;
                // Ajuste para permitir HTML no subMessage (descrição)
                feedbackOverlay.innerHTML = `<div class="${pulseClass} text-center px-4"><div class="text-5xl font-black mb-2">${message}</div><div class="text-2xl font-mono p-3 bg-black/40 rounded-xl backdrop-blur-md border border-white/10 shadow-lg">${subMessage}</div></div>`;
                feedbackOverlay.style.opacity = '1';
                
                playSound(status);
                if (navigator.vibrate) {
                    if (status === 'success') navigator.vibrate(200);
                    else if (status === 'error') navigator.vibrate([100, 50, 100]);
                    else if (status === 'warning') navigator.vibrate([80, 80]);
                    else if (status === 'warning_missort') navigator.vibrate([120, 60, 120]); 
                    else if (status === 'hunt_success') navigator.vibrate([500, 100, 500]); 
                }
                
                const isHunt = (status === 'hunt_success');
                const currentDelay = (appState.isFastMode && !isHunt) ? 250 : SCAN_DELAY;
                const finalDelay = isHunt ? 2500 : currentDelay;
                
                setTimeout(() => {
                    feedbackOverlay.style.opacity = '0';
                    if (!appState.isFastMode && !isHunt) { appState.isPaused = false; }
                }, finalDelay);
            }
            
            function initAudio() { if (!appState.audioContext) appState.audioContext = new (window.AudioContext || window.webkitAudioContext)(); }
            function playSound(type) {
                if (!appState.audioContext) return;
                try {
                    const osc = appState.audioContext.createOscillator(); const gain = appState.audioContext.createGain();
                    osc.connect(gain); gain.connect(appState.audioContext.destination);
                    gain.gain.setValueAtTime(0.3, appState.audioContext.currentTime);
                    if (type === 'success') { osc.frequency.setValueAtTime(1200, osc.context.currentTime); } 
                    else if (type === 'error') { osc.frequency.setValueAtTime(180, osc.context.currentTime); osc.type = 'square'; } 
                    else if (type === 'warning') { osc.frequency.setValueAtTime(600, osc.context.currentTime); osc.type = 'triangle'; } 
                    else if (type === 'warning_missort') { osc.frequency.setValueAtTime(800, osc.context.currentTime); osc.frequency.setValueAtTime(400, osc.context.currentTime + 0.07); osc.type = 'sawtooth'; } 
                    else if (type === 'hunt_success') { osc.frequency.setValueAtTime(1000, osc.context.currentTime); osc.frequency.linearRampToValueAtTime(2000, osc.context.currentTime + 0.3); }
                    osc.start(); osc.stop(appState.audioContext.currentTime + (type === 'hunt_success' ? 0.4 : 0.15));
                } catch (e) { appState.audioContext = new (window.AudioContext || window.webkitAudioContext)(); }
            }
            
            initialize();
            if ('serviceWorker' in navigator) { window.addEventListener('load', () => { navigator.serviceWorker.register('./sw.js'); }); }
        });
    </script>
</body>
</html>
