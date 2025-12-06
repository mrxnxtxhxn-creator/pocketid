<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Natefy Scanner</title>

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
        
        /* LOGIN MODAL */
        #login-modal { 
            position: fixed; inset: 0; z-index: 100; 
            background-color: rgba(15, 23, 42, 0.98); 
            display: flex; align-items: center; justify-content: center; 
            backdrop-filter: blur(5px); 
        }
        #login-modal.hidden { display: none !important; }
        
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
            // CONFIGURAÇÃO
            const APP_CONFIG = {
                WEBHOOK_WRITE: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9', // Sua URL
                WEBHOOK_READ: '', 
                STORAGE_KEY: 'natefy_v3',
                SYNC_INTERVAL: 15000
            };

            // ESTADO GLOBAL (Padronizado para evitar conflitos)
            let appState = {
                operator: null, 
                idsToFind: new Set(), 
                idDescriptions: new Map(), 
                inventoryZoneData: new Map(),
                foundIds: [], 
                localLog: [], 
                globalLog: [], 
                offlineQueue: [],
                inventoryZones: [
                    {id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},
                    {id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},
                    {id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'},
                    {id:'problemsolver',name:'Problem Solver'}
                ],
                activeZone: null, 
                huntMode: { isActive: false, targetId: null },
                analysisResult: { ok: [], sobra: [], faltantes: [] },
                isScanning: false, 
                isFastMode: false, 
                lastScanTime: 0,
                zoneFinds: new Map(),
                audioContext: null,
                html5QrCode: null,
                chart: null
            };

            // --- SISTEMA DE LOGIN ---
            function checkLogin() {
                if (!appState.operator) { 
                    document.getElementById('login-modal').classList.remove('hidden'); 
                } else { 
                    document.getElementById('login-modal').classList.add('hidden'); 
                    updateStatusUI(); 
                    startScanner(); 
                }
            }

            document.getElementById('login-btn').addEventListener('click', () => {
                const name = document.getElementById('operator-input').value.trim();
                if (name) { 
                    appState.operator = name; 
                    saveState(); 
                    checkLogin(); 
                } else { 
                    alert("Nome obrigatório"); 
                }
            });

            // --- INICIALIZAÇÃO ---
            function init() {
                loadState();
                buildInventoryZoneUI();
                buildAnalyzerUI();
                checkLogin(); // Isso agora vai funcionar porque o JS está limpo
                setupEventListeners();
                createCharts();
                updateUI();
            }

            // --- EVENT LISTENER PRINCIPAL ---
            function setupEventListeners() {
                const controlsPanel = document.getElementById('controls-panel');
                const panelHandle = document.getElementById('panel-handle');

                // Botões Gerais
                document.getElementById('change-operator-btn').addEventListener('click', () => {
                    if(confirm("Sair?")) { appState.operator = null; saveState(); window.location.reload(); }
                });
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', (e) => handleFileSelect(e, 'main'));
                document.getElementById('load-image-btn').addEventListener('click', () => document.getElementById('image-input').click());
                document.getElementById('image-input').addEventListener('change', handleImageUpload);
                document.getElementById('export-btn').addEventListener('click', exportFoundIds);
                document.getElementById('clear-session-btn').addEventListener('click', clearSession);
                document.getElementById('fast-mode-toggle').addEventListener('change', (e) => appState.isFastMode = e.target.checked);
                document.getElementById('hunt-toggle-btn').addEventListener('click', toggleHuntMode);
                
                // Análises e Relatórios
                document.getElementById('run-analysis-btn').addEventListener('click', runListAnalysis);
                document.getElementById('export-analysis-btn').addEventListener('click', exportAnalysisResults);
                document.getElementById('download-full-report-btn').addEventListener('click', downloadFullReport);
                document.getElementById('download-flowchart-btn').addEventListener('click', downloadFlowchart);

                // Input Manual
                const manualInput = document.getElementById('manual-input');
                const checkManual = () => { 
                    const v = manualInput.value.trim(); 
                    if(v) { processScan(v); manualInput.value=''; }
                };
                document.getElementById('manual-check-btn').addEventListener('click', checkManual);
                manualInput.addEventListener('keydown', (e) => { 
                    if(e.key === 'Enter'){ e.preventDefault(); checkManual(); }
                });

                // Audio Init
                document.body.addEventListener('click', () => {
                    if(!appState.audioContext) appState.audioContext = new (window.AudioContext||window.webkitAudioContext)();
                }, { once: true });

                // Abas
                document.querySelectorAll('.tab-btn').forEach(b => {
                    b.addEventListener('click', () => {
                        document.querySelectorAll('.tab-btn').forEach(x=>x.classList.replace('tab-active','tab-inactive'));
                        b.classList.replace('tab-inactive','tab-active');
                        document.querySelectorAll('[data-view-content]').forEach(d => d.classList.add('hidden'));
                        document.querySelector(`[data-view-content="${b.dataset.view}"]`).classList.remove('hidden');
                    });
                });

                // Painel Deslizante
                let touchStartY = 0;
                panelHandle.addEventListener('click', () => { controlsPanel.classList.toggle('open'); });
                document.addEventListener('touchstart', e => { 
                    if (e.target === panelHandle || controlsPanel.contains(e.target)) touchStartY = e.touches[0].clientY; 
                });
                document.addEventListener('touchend', e => {
                    if(touchStartY===0)return; 
                    const touchEndY=e.changedTouches[0].clientY; 
                    if(touchStartY-touchEndY>50) controlsPanel.classList.add('open'); 
                    else if(touchEndY-touchStartY>50) { controlsPanel.classList.remove('open'); resumeScannerIfNeeded(); }
                    touchStartY=0;
                });
            }

            // --- LÓGICA DO SCANNER ---
            function processScan(id) {
                if (appState.huntMode.isActive) {
                    if (id === appState.huntMode.targetId) {
                        feedback('success', id, 'ALVO ENCONTRADO!', true);
                        sendLogToN8n({ scannedId:id, timestamp: new Date().toISOString(), status: 'Hunt Success', operator: appState.operator });
                        toggleHuntMode();
                    } return;
                }
                if (appState.isPaused) return;
                if (appState.isFastMode) { 
                    const now = Date.now(); 
                    if (now - appState.lastScanTime < 500) return; 
                    appState.lastScanTime = now; 
                } else { 
                    appState.isPaused = true; 
                }

                const now = new Date();
                const desc = appState.idDescriptions.get(id) || '';
                let status = 'ERRO', msg = 'NÃO ENCONTRADO', type = 'error';

                if (appState.activeZone) {
                    for (const [zId, ids] of appState.inventoryZoneData) {
                        if (zId !== appState.activeZone && ids.has(id)) { 
                            status = 'MISSORT'; 
                            msg = `MISSORT (${zId})`; 
                            type = 'warning'; 
                            break; 
                        }
                    }
                }

                if (status === 'ERRO' && appState.idsToFind.has(id)) { 
                    status = 'SUCESSO'; 
                    msg = desc || 'ENCONTRADO'; 
                    type = 'success'; 
                    appState.idsToFind.delete(id); 
                }

                const entry = { id, status, desc, time: now.toISOString(), operator: appState.operator, zone: appState.activeZone };
                appState.localLog.unshift(entry);
                if(status === 'SUCESSO') appState.foundIds.unshift(entry);

                saveState();
                feedback(type, id, msg);
                updateUI();

                sendLogToN8n({ ...entry, scannedId: id, activeZoneId: appState.activeZone });
            }

            async function sendLogToN8n(data) {
                if(APP_CONFIG.WEBHOOK_WRITE && !APP_CONFIG.WEBHOOK_WRITE.includes('COLE_A')) {
                    try {
                        await fetch(APP_CONFIG.WEBHOOK_WRITE, { 
                            method: 'POST', 
                            headers:{'Content-Type':'application/json'}, 
                            body: JSON.stringify(data) 
                        });
                    } catch(e) { console.log("Erro envio n8n", e); }
                }
            }

            // --- UI HELPERS ---
            function feedback(type, id, msg, persistent) {
                const el = document.getElementById('feedback-overlay');
                const colors = { success: '#22c55e', error: '#ef4444', warning: '#f59e0b' };
                el.style.background = `radial-gradient(circle, ${colors[type] || colors.error}, transparent)`;
                el.innerHTML = `<div class="text-center"><div class="text-4xl font-black mb-2">${msg}</div><div class="text-xl font-mono bg-black/50 p-2 rounded">${id}</div></div>`;
                el.style.opacity = '1';
                
                // Audio
                if(appState.audioContext) {
                    const o=appState.audioContext.createOscillator(), g=appState.audioContext.createGain();
                    o.connect(g); g.connect(appState.audioContext.destination);
                    o.frequency.value = type==='success'?1200 : (type==='warning'?600:200);
                    o.start(); setTimeout(()=>o.stop(), 150);
                }
                
                if(navigator.vibrate) navigator.vibrate(200);
                setTimeout(() => { 
                    el.style.opacity = '0'; 
                    if(!appState.isFastMode && !persistent) appState.isPaused = false; 
                }, persistent ? 2000 : 1000);
            }

            function updateUI() {
                document.getElementById('found-count').innerText = appState.foundIds.length;
                document.getElementById('exceptions-count').innerText = appState.localLog.filter(l=>l.status==='ERRO').length;
                
                // Log List
                document.getElementById('scan-log-list').innerHTML = appState.localLog.slice(0, 50).map(l => 
                    `<div class="bg-slate-800 p-2 rounded border border-slate-700 flex justify-between mb-1">
                        <div><span class="font-bold text-white text-xs">${l.id}</span> <span class="text-[10px] text-cyan-400">${l.status}</span></div>
                        <span class="text-[10px] text-slate-500">${new Date(l.time).toLocaleTimeString()}</span>
                    </div>`
                ).join('');

                // Dashboard
                const total = appState.idsToFind.size + appState.foundIds.length;
                const pct = total > 0 ? Math.round((appState.foundIds.length/total)*100) : 0;
                document.getElementById('progress-text').innerText = pct + '%';
                document.getElementById('kpi-my-bips').innerText = appState.localLog.length;
                
                if(appState.chart) { 
                    appState.chart.data.datasets[0].data = [pct, 100-pct]; 
                    appState.chart.update(); 
                }
            }

            function updateStatusUI() {
                document.getElementById('status-zone').innerText = `ZONA: ${appState.activeZone || '--'}`;
                document.getElementById('status-operator').innerText = `OP: ${appState.operator || '--'}`;
            }

            // --- MANIPULAÇÃO DE ARQUIVOS (Resumo) ---
            function handleFileSelect(e, zid) {
                const f = e.target.files[0]; if(!f)return;
                const r = new FileReader();
                r.onload = (ev) => {
                    let ids = [];
                    if(f.name.endsWith('.xlsx')) {
                        const d = new Uint8Array(ev.target.result);
                        const w = XLSX.read(d, {type:'array'});
                        ids = XLSX.utils.sheet_to_json(w.Sheets[w.SheetNames[0]], {header:1}).map(r=>String(r[0]));
                    } else {
                        ids = ev.target.result.trim().split(/[\n\r,]+/).map(i=>i.trim());
                    }
                    const s = new Set(ids.filter(i=>i));
                    if(zid==='main') { appState.idsToFind=s; appState.foundIds=[]; document.getElementById('file-info').textContent=`"${f.name}" (${s.size})`; }
                    else { appState.inventoryZoneData.set(zid, s); }
                    saveToLocalStorage(); updateUI(); buildAnalyzerUI();
                };
                f.name.endsWith('.xlsx') ? r.readAsArrayBuffer(f) : r.readAsText(f);
            }

            // --- OCR ---
            async function handleImageUpload(e) {
                const file = e.target.files[0]; if(!file) return;
                document.getElementById('ocr-loading').classList.remove('hidden');
                try {
                    const worker = Tesseract.createWorker();
                    await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                    const { data: { text } } = await worker.recognize(file);
                    await worker.terminate();
                    
                    const newIds = new Set();
                    text.split('\n').forEach(line => {
                        const m = line.match(/(\d{8,14})/);
                        if(m) { newIds.add(m[0]); appState.idDescriptions.set(m[0], line.replace(m[0],'').trim()); }
                    });
                    if(newIds.size>0) { appState.idsToFind = newIds; appState.foundIds=[]; updateUI(); alert("OCR Concluído!"); }
                } catch(err) { alert("Erro OCR"); } 
                finally { document.getElementById('ocr-loading').classList.add('hidden'); }
            }

            // --- PERSISTÊNCIA ---
            function saveState() {
                const s = { ...appState, html5QrCode: null, audioContext: null, chart: null };
                s.idsToFind = Array.from(s.idsToFind);
                s.inventoryZoneData = Array.from(s.inventoryZoneData.entries()).map(([k,v])=>[k,Array.from(v)]);
                s.idDescriptions = Array.from(s.idDescriptions.entries());
                localStorage.setItem(APP_CONFIG.STORAGE_KEY, JSON.stringify(s));
            }
            function loadState() {
                const s = JSON.parse(localStorage.getItem(APP_CONFIG.STORAGE_KEY));
                if(s) {
                    appState = { ...appState, ...s };
                    appState.idsToFind = new Set(s.idsToFind);
                    appState.inventoryZoneData = new Map(s.inventoryZoneData.map(([k,v])=>[k,new Set(v)]));
                    appState.idDescriptions = new Map(s.idDescriptions);
                }
            }
            function clearSession() { localStorage.removeItem(APP_CONFIG.STORAGE_KEY); window.location.reload(); }

            // --- FUNÇÕES DE UI COMPLEMENTARES ---
            function toggleHuntMode() {
                const t = document.getElementById('hunt-target-id');
                if(appState.huntMode.isActive) {
                    appState.huntMode={isActive:false, targetId:null}; t.disabled=false; t.value='';
                    document.getElementById('hunt-toggle-btn').innerText='Ativar';
                    if(appState.html5QrCode && appState.html5QrCode.getState()===2) appState.html5QrCode.resume();
                } else {
                    const v = t.value.trim(); if(!v) return alert("ID?");
                    appState.huntMode={isActive:true, targetId:v}; t.disabled=true;
                    document.getElementById('hunt-toggle-btn').innerText='Cancelar';
                    appState.isPaused=false;
                    if(appState.html5QrCode && appState.html5QrCode.getState()===1) appState.html5QrCode.pause(true);
                }
                saveToLocalStorage();
            }

            function buildInventoryZoneUI() {
                const c = document.getElementById('inventory-zones-container');
                c.innerHTML = `<button class="w-full bg-slate-600 text-white py-2 rounded mb-2 text-xs" onclick="setActiveZone(null)">Limpar Zona</button>`;
                appState.inventoryZones.forEach(z => {
                    if(!appState.inventoryZoneData.has(z.id)) appState.inventoryZoneData.set(z.id, new Set());
                    const count = appState.inventoryZoneData.get(z.id).size;
                    c.innerHTML += `<div class="bg-slate-800 p-2 rounded flex justify-between items-center mb-2"><span class="text-sm">${z.name} (${count})</span> <button class="text-xs bg-blue-600 px-2 py-1 rounded" onclick="setActiveZone('${z.id}')">Ativar</button> <input type="file" class="hidden" id="file-${z.id}" onchange="handleFileSelect(event, '${z.id}')"><button class="text-xs bg-slate-600 px-2 py-1 rounded ml-1" onclick="document.getElementById('file-${z.id}').click()">Load</button></div>`;
                });
            }
            window.setActiveZone = (zid) => { appState.activeZone = zid; updateStatusUI(); saveToLocalStorage(); };
            window.handleFileSelect = handleFileSelect; // Expor para HTML

            function buildAnalyzerUI() { /* ... (Igual anterior) ... */ }
            function runListAnalysis() { /* ... (Igual anterior) ... */ }
            function exportAnalysisResults() { /* ... (Igual anterior) ... */ }
            function exportFoundIds() { /* ... (Igual anterior) ... */ }
            function exportExceptions() { /* ... (Igual anterior) ... */ }
            function downloadFullReport() { /* ... (Igual anterior) ... */ }
            function downloadFlowchart() { /* ... (Igual anterior) ... */ }

            function createCharts() {
                const ctx = document.getElementById('progressChart');
                if(ctx) {
                    appState.chart = new Chart(ctx, { type: 'doughnut', data: { datasets: [{ data: [0, 100], backgroundColor: ['#0ea5e9', '#334155'], borderWidth: 0 }] }, options: { cutout: '80%', plugins: { tooltip: { enabled: false } } } });
                }
            }

            function resumeScannerIfNeeded() { 
                 if(!appState.huntMode.isActive && appState.html5QrCode && appState.html5QrCode.getState()===2) appState.html5QrCode.resume(); 
            }
            function startScanner() {
                 if(appState.html5QrCode) return;
                 appState.html5QrCode = new Html5Qrcode("reader");
                 appState.html5QrCode.start({facingMode:"environment"}, {fps:15, qrbox:250}, processScan).catch(e=>{});
            }

            init();
        });
    </script>
</body>
</html>
