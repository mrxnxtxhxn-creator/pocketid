<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Scanner de ID com Módulo de Inventário</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; }
        #reader__scan_region { border: 4px solid rgba(255, 255, 255, 0.5) !important; border-radius: 1.5rem; background: none !important; box-shadow: 0 0 20px rgba(0, 255, 255, 0.3); }
        .scan-line { position: absolute; left: 5%; top: 10px; width: 90%; height: 4px; background: linear-gradient(to right, transparent, #06b6d4, transparent); box-shadow: 0 0 15px #06b6d4, 0 0 5px #fff; border-radius: 4px; animation: scan-animation 2.5s infinite ease-in-out; }
        @keyframes scan-animation { 0% { transform: translateY(0); } 50% { transform: translateY(calc(100% - 20px)); } 100% { transform: translateY(0); } }
        #controls-panel { 
            background: rgba(30, 41, 59, 0.8); 
            backdrop-filter: blur(16px); 
            border-top: 1px solid rgba(71, 85, 105, 0.5);
            transform: translateY(calc(100% - 70px));
            transition: transform 0.3s ease-in-out;
        }
        #controls-panel.open { transform: translateY(0); }
        .feedback-pulse { animation: pulse-feedback 0.8s ease-out; }
        @keyframes pulse-feedback { from { transform: scale(0.9); opacity: 0.7; } to { transform: scale(1); opacity: 1; } }
        .tab-btn { border-bottom: 3px solid transparent; transition: all 0.2s; }
        .tab-active { border-color: #06b6d4; color: white; }
        .tab-inactive { color: #94a3b8; }
    </style>
</head>
<body class="text-slate-200">
    <div id="reader" class="fixed top-0 left-0 w-full h-full z-1"><div class="scan-line"></div></div>
    <div id="feedback-overlay" class="fixed inset-0 z-50 flex items-center justify-center p-4 text-white font-black opacity-0 pointer-events-none transition-opacity duration-200"></div>

    <div id="controls-panel" class="fixed bottom-0 left-0 right-0 z-10 rounded-t-2xl">
        <div id="panel-handle" class="w-full h-10 flex justify-center items-center cursor-pointer">
             <div class="w-10 h-1.5 bg-slate-500 rounded-full"></div>
        </div>
        <div class="w-full max-w-lg mx-auto px-4 pb-4">
            <div class="flex justify-center mb-4 space-x-2 sm:space-x-4">
                <button data-view="procurar" class="tab-btn tab-active py-2 px-4 font-semibold text-sm sm:text-base">Procurar</button>
                <button data-view="encontrados" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Encontrados (<span id="found-count">0</span>)</button>
                <button data-view="dashboard" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Dashboard</button>
                <button data-view="inventario" class="tab-btn tab-inactive py-2 px-4 font-semibold text-sm sm:text-base">Inventário</button>
            </div>
            
            <div data-view-content="procurar" class="text-center">
                <button id="load-file-btn" class="w-full bg-cyan-600 hover:bg-cyan-700 text-white font-bold py-3 px-5 rounded-lg shadow-lg text-lg">Carregar Ficheiro Principal</button>
                <input type="file" id="file-input" class="hidden" accept=".txt,.csv,.xlsx">
                <p id="file-info" class="text-xs text-green-400 mt-2 h-4"></p>
                 <div class="mt-4 text-left border-t border-slate-700 pt-4">
                    <label for="manual-input" class="text-xs text-slate-400">Ou digite o ID manualmente:</label>
                    <div class="flex gap-2 mt-1">
                        <input type="text" id="manual-input" class="w-full bg-slate-700 p-2 rounded-lg font-mono text-white" placeholder="ID do pacote...">
                        <button id="manual-check-btn" class="bg-blue-600 hover:bg-blue-700 font-bold px-4 rounded-lg">Verificar</button>
                    </div>
                </div>
            </div>

            <div data-view-content="encontrados" class="hidden">
                <button id="export-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-5 rounded-lg mb-4">Exportar Encontrados (.csv)</button>
                <div class="max-h-32 overflow-y-auto pr-2"><ul id="found-list" class="space-y-2 text-center font-mono text-sm"></ul></div>
            </div>

            <div data-view-content="dashboard" class="hidden">
                <h3 class="text-lg font-bold text-center text-white mb-4">Performance da Sessão</h3>
                <div class="grid grid-cols-3 gap-4">
                    <div class="flex flex-col items-center justify-center p-4 bg-slate-800 rounded-lg">
                         <h4 class="text-sm font-semibold text-slate-400 mb-2">Progresso</h4>
                         <div class="relative w-24 h-24"><canvas id="progressChart"></canvas><div id="progress-text" class="absolute inset-0 flex items-center justify-center text-2xl font-bold">0%</div></div>
                    </div>
                    <div class="p-4 bg-slate-800 rounded-lg text-center"><h4 class="text-sm font-semibold text-slate-400">Tempo Médio / Bip</h4><p id="kpi-avg-time" class="text-4xl font-black text-white mt-2">-- s</p></div>
                    <div class="p-4 bg-slate-800 rounded-lg text-center"><h4 class="text-sm font-semibold text-slate-400">Bips por Minuto</h4><p id="kpi-bpm" class="text-4xl font-black text-white mt-2">--</p></div>
                </div>
            </div>

            <div data-view-content="inventario" class="hidden">
                <h3 class="text-lg font-bold text-center text-white mb-4">Carregar Listas de Inventário por Zona</h3>
                <div id="inventory-zones-container" class="space-y-4 max-h-64 overflow-y-auto pr-2">
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            let appState = {
                currentView: 'procurar',
                idsToFind: new Set(),
                inventoryZones: [
                    { id: 'buffered', name: 'Buffered' }, { id: 'sorting', name: 'Sorting' },
                    { id: 'fraude', name: 'Fraude' }, { id: 'missort', name: 'Missort' },
                    { id: 'returns', name: 'Returns' }, { id: 'bulky', name: 'Bulky' }
                ],
                inventoryZoneData: new Map(),
                foundIds: [], 
                isPaused: false,
                audioContext: null,
                html5QrCode: null,
                scanHistory: [], 
                charts: {}
            };
            
            const controlsPanel = document.getElementById('controls-panel');
            const panelHandle = document.getElementById('panel-handle');
            const togglePanel = () => controlsPanel.classList.toggle('open');
            panelHandle.addEventListener('click', togglePanel);
            
            let touchStartY = 0;
            document.addEventListener('touchstart', e => { if (e.target === panelHandle || controlsPanel.contains(e.target)) touchStartY = e.touches[0].clientY; });
            document.addEventListener('touchend', e => {
                if (touchStartY === 0) return;
                const touchEndY = e.changedTouches[0].clientY;
                if (touchStartY - touchEndY > 50) controlsPanel.classList.add('open');
                else if (touchEndY - touchStartY > 50) controlsPanel.classList.remove('open');
                touchStartY = 0;
            });

            function initialize() {
                buildInventoryZoneUI();
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', (e) => handleFileSelect(e, 'main'));
                document.getElementById('export-btn').addEventListener('click', exportFoundIds);
                document.body.addEventListener('click', initAudio, { once: true });
                setupTabs();
                startScanner();
                const manualInput = document.getElementById('manual-input');
                const manualCheckBtn = document.getElementById('manual-check-btn');
                manualCheckBtn.addEventListener('click', () => {
                    const manualId = manualInput.value.trim();
                    if (manualId) { processScan(manualId); manualInput.value = ''; }
                });
                manualInput.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter') {
                        e.preventDefault();
                        const manualId = manualInput.value.trim();
                        if (manualId) { processScan(manualId); manualInput.value = ''; }
                    }
                });
                createCharts();
            }

            function buildInventoryZoneUI() {
                const container = document.getElementById('inventory-zones-container');
                container.innerHTML = '';
                appState.inventoryZones.forEach(zone => {
                    appState.inventoryZoneData.set(zone.id, new Set());
                    const div = document.createElement('div');
                    div.className = "p-3 bg-slate-800 rounded-lg flex items-center justify-between";
                    div.innerHTML = `
                        <div>
                            <p class="font-semibold text-white">${zone.name}</p>
                            <p id="file-info-${zone.id}" class="text-xs text-slate-400">Nenhum ficheiro carregado.</p>
                        </div>
                        <button data-zone-id="${zone.id}" class="load-zone-file-btn bg-cyan-700 hover:bg-cyan-600 text-white font-bold py-2 px-3 rounded-lg text-sm">Carregar</button>
                        <input type="file" id="file-input-${zone.id}" class="hidden" accept=".txt,.csv,.xlsx">
                    `;
                    container.appendChild(div);
                });

                document.querySelectorAll('.load-zone-file-btn').forEach(btn => {
                    btn.addEventListener('click', (e) => {
                        document.getElementById(`file-input-${e.currentTarget.dataset.zoneId}`).click();
                    });
                });
                 document.querySelectorAll('input[type="file"]').forEach(input => {
                    if (input.id.startsWith('file-input-')) {
                        input.addEventListener('change', (e) => handleFileSelect(e, e.target.id.replace('file-input-', '')));
                    }
                });
            }

            function handleFileSelect(event, zoneId) {
                const file = event.target.files[0]; if (!file) return;
                const isMainSearch = zoneId === 'main';
                
                const reader = new FileReader();
                const processIds = (ids, fileName) => {
                    const idSet = new Set(ids);
                    if (isMainSearch) {
                        appState.idsToFind = idSet;
                        appState.foundIds = [];
                        appState.scanHistory = [];
                        document.getElementById('file-info').textContent = `"${fileName}" (${ids.length} IDs)`;
                        updateFoundListUI();
                        updateDashboard();
                    } else {
                        appState.inventoryZoneData.set(zoneId, idSet);
                         document.getElementById(`file-info-${zoneId}`).textContent = `${ids.length} IDs carregados.`;
                         document.getElementById(`file-info-${zoneId}`).classList.add('text-green-400');
                    }
                };

                if (file.name.endsWith('.xlsx')) {
                    reader.onload = (e) => {
                        const data = new Uint8Array(e.target.result);
                        const workbook = XLSX.read(data, {type: 'array'});
                        const firstSheetName = workbook.SheetNames[0];
                        const worksheet = workbook.Sheets[firstSheetName];
                        const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
                        const ids = json.map(row => String(row[0])).filter(id => id && id !== 'undefined');
                        processIds(ids, file.name);
                    };
                    reader.readAsArrayBuffer(file);
                } else {
                    reader.onload = (e) => {
                        const text = e.target.result;
                        const ids = text.trim().split(/[\n\r,]+/).map(id => id.trim()).filter(id => id);
                        processIds(ids, file.name);
                    };
                    reader.readAsText(file);
                }
            }
            
            function processScan(scannedId) {
                if (appState.isPaused) return;
                
                if (appState.idsToFind.has(scannedId)) {
                    showFeedback('success', scannedId);
                    appState.idsToFind.delete(scannedId);
                    appState.foundIds.unshift({ id: scannedId, timestamp: new Date() });
                    appState.scanHistory.push(new Date());
                    updateFoundListUI();
                    updateDashboard();
                    return;
                }
                
                if (appState.foundIds.some(item => item.id === scannedId)) {
                    showFeedback('warning', scannedId, 'JÁ ENCONTRADO');
                    return;
                }

                for (const [zoneId, idSet] of appState.inventoryZoneData.entries()) {
                    if (idSet.has(scannedId)) {
                        const zone = appState.inventoryZones.find(z => z.id === zoneId);
                        showFeedback('success', scannedId, `ENCONTRADO (EM ${zone.name.toUpperCase()})`);
                        return;
                    }
                }
                
                showFeedback('error', scannedId);
            }
            
            function setupTabs() {
                const tabButtons = document.querySelectorAll('.tab-btn');
                tabButtons.forEach(button => {
                    button.addEventListener('click', () => {
                        const viewId = button.dataset.view;
                        appState.currentView = viewId;
                        
                        tabButtons.forEach(btn => btn.classList.replace('tab-active', 'tab-inactive'));
                        button.classList.replace('tab-inactive', 'tab-active');
                        
                        document.querySelectorAll('[data-view-content]').forEach(view => {
                            view.classList.toggle('hidden', view.dataset.viewContent !== viewId);
                        });
                    });
                });
            }

            function exportFoundIds() {
                if (appState.foundIds.length === 0) { alert("Nenhum ID foi encontrado para exportar."); return; }
                let csvContent = "data:text/csv;charset=utf-8,ID_Encontrado,Data_Verificacao,Hora_Verificacao\n";
                appState.foundIds.forEach(item => {
                    const row = [item.id, item.timestamp.toLocaleDateString('pt-BR'), item.timestamp.toLocaleTimeString('pt-BR')].join(',');
                    csvContent += row + "\n";
                });
                const encodedUri = encodeURI(csvContent);
                const link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                const date = new Date().toLocaleDateString('pt-BR').replace(/\//g, '-');
                link.setAttribute("download", `sessao_scanner_${date}.csv`);
                document.body.appendChild(link); link.click(); document.body.removeChild(link);
            }

            function startScanner() {
                appState.html5QrCode = new Html5Qrcode("reader");
                const config = { fps: 15, qrbox: (w, h) => { const s = Math.min(w, h) * 0.8; return { width: s, height: s }; } };
                appState.html5QrCode.start({ facingMode: "environment" }, config, (decodedText) => processScan(decodedText))
                    .catch(err => alert("ERRO AO INICIAR A CÂMARA: Por favor, verifique se deu permissão de acesso à câmara."));
            }
            
            function updateFoundListUI() {
                document.getElementById('found-count').textContent = appState.foundIds.length;
                const foundList = document.getElementById('found-list');
                if (appState.foundIds.length === 0) {
                    foundList.innerHTML = '<li class="text-slate-500">Nenhum item encontrado ainda.</li>';
                } else {
                    foundList.innerHTML = appState.foundIds.map(item => `<li class="p-2 bg-slate-700 rounded-md text-white flex justify-between"><span>${item.id}</span><span class="text-xs text-slate-400">${item.timestamp.toLocaleTimeString('pt-BR')}</span></li>`).join('');
                }
            }

            function updateDashboard() {
                const totalItems = appState.idsToFind.size + appState.foundIds.length;
                const foundCount = appState.foundIds.length;
                const progress = totalItems > 0 ? (foundCount / totalItems) * 100 : 0;
                document.getElementById('progress-text').textContent = `${Math.round(progress)}%`;
                appState.charts.progress.data.datasets[0].data = [progress, 100 - progress];
                appState.charts.progress.update();

                if (appState.scanHistory.length > 1) {
                    const lastScans = appState.scanHistory.slice(-10);
                    let totalDiff = 0;
                    for (let i = 1; i < lastScans.length; i++) totalDiff += (lastScans[i] - lastScans[i-1]);
                    const avgTime = (totalDiff / (lastScans.length - 1)) / 1000;
                    if (!isNaN(avgTime)) {
                        document.getElementById('kpi-avg-time').textContent = `${avgTime.toFixed(1)} s`;
                        const bpm = avgTime > 0 ? Math.round(60 / avgTime) : '--';
                        document.getElementById('kpi-bpm').textContent = bpm;
                    }
                }
            }

            function createCharts() {
                const progressCtx = document.getElementById('progressChart').getContext('2d');
                appState.charts.progress = new Chart(progressCtx, {
                    type: 'doughnut',
                    data: { datasets: [{ data: [0, 100], backgroundColor: ['#0ea5e9', '#334155'], borderColor: '#1e293b', borderWidth: 4, cutout: '75%' }] },
                    options: { responsive: true, maintainAspectRatio: false, plugins: { tooltip: { enabled: false } } }
                });
            }

            function showFeedback(status, scannedId, messageOverride) {
                appState.isPaused = true;
                const message = messageOverride || (status === 'success' ? 'ENCONTRADO' : 'NÃO ENCONTRADO');
                const feedbackOverlay = document.getElementById('feedback-overlay');
                feedbackOverlay.style.background = status === 'success' ? 'radial-gradient(circle, rgba(34, 197, 94, 0.8) 0%, rgba(30, 41, 59, 0) 70%)' : (status === 'warning' ? 'radial-gradient(circle, rgba(245, 158, 11, 0.8) 0%, rgba(30, 41, 59, 0) 70%)' : 'radial-gradient(circle, rgba(239, 68, 68, 0.8) 0%, rgba(30, 41, 59, 0) 70%)');
                feedbackOverlay.innerHTML = `<div class="feedback-pulse text-center"><div class="text-6xl">${message}</div><div class="text-2xl mt-4 font-mono p-2 bg-black/30 rounded-lg">${scannedId}</div></div>`;
                feedbackOverlay.style.opacity = '1';
                playSound(status);
                if (navigator.vibrate) {
                    if (status === 'success') navigator.vibrate(200);
                    else if (status === 'error') navigator.vibrate([100, 50, 100]);
                }
                setTimeout(() => {
                    feedbackOverlay.style.opacity = '0';
                    appState.isPaused = false;
                }, SCAN_DELAY);
            }
            
            function initAudio() { if (!appState.audioContext) appState.audioContext = new (window.AudioContext || window.webkitAudioContext)(); }
            function playSound(type) {
                if (!appState.audioContext) return;
                const osc = appState.audioContext.createOscillator();
                const gain = appState.audioContext.createGain();
                osc.connect(gain); gain.connect(appState.audioContext.destination);
                gain.gain.setValueAtTime(0.3, appState.audioContext.currentTime);
                if (type === 'success') { osc.frequency.setValueAtTime(1200, osc.context.currentTime); } 
                else if (type === 'error') { osc.frequency.setValueAtTime(180, osc.context.currentTime); osc.type = 'square'; } 
                else { osc.frequency.setValueAtTime(600, osc.context.currentTime); osc.type = 'triangle'; }
                osc.start(); osc.stop(osc.context.currentTime + 0.12);
            }
            
            initialize();
        });
    </script>
</body>
</html>
