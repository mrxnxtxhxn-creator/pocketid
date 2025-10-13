<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Scanner de ID Profissional</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
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
            transform: translateY(calc(100% - 70px)); /* Começa recolhido */
            transition: transform 0.3s ease-in-out;
        }
        #controls-panel.open {
            transform: translateY(0);
        }
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
            <div class="flex justify-center mb-4 space-x-4">
                <button id="tab-procurar" class="tab-btn tab-active py-2 px-4 font-semibold">Procurar</button>
                <button id="tab-encontrados" class="tab-btn tab-inactive py-2 px-4 font-semibold">Encontrados (<span id="found-count">0</span>)</button>
            </div>
            <div id="view-procurar" class="text-center">
                <p class="text-sm text-slate-400 mb-4">Carregue um ficheiro (.txt, .csv, ou .xlsx) com a sua lista de IDs.</p>
                <button id="load-file-btn" class="w-full bg-cyan-600 hover:bg-cyan-700 text-white font-bold py-3 px-5 rounded-lg shadow-lg text-lg">
                    Carregar Ficheiro de IDs
                </button>
                <input type="file" id="file-input" class="hidden" accept=".txt,.csv,.xlsx,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel">
                <p id="file-info" class="text-xs text-green-400 mt-2 h-4"></p>
            </div>
            <div id="view-encontrados" class="hidden">
                <button id="export-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-5 rounded-lg mb-4">
                    Exportar para Excel (.csv)
                </button>
                <div class="max-h-32 overflow-y-auto pr-2">
                    <ul id="found-list" class="space-y-2 text-center font-mono text-sm"></ul>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            let appState = {
                idsToFind: new Set(),
                foundIds: [], 
                isPaused: false,
                audioContext: null,
                html5QrCode: null
            };
            const SCAN_DELAY = 800;

            const controlsPanel = document.getElementById('controls-panel');
            const panelHandle = document.getElementById('panel-handle');

            const togglePanel = () => controlsPanel.classList.toggle('open');
            panelHandle.addEventListener('click', togglePanel);
            
            // Lógica de swipe (simplificada)
            let touchStartY = 0;
            document.addEventListener('touchstart', e => {
                if (e.target === panelHandle || controlsPanel.contains(e.target)) {
                    touchStartY = e.touches[0].clientY;
                }
            });
            document.addEventListener('touchend', e => {
                if (touchStartY === 0) return;
                const touchEndY = e.changedTouches[0].clientY;
                if (touchStartY - touchEndY > 50) { // Swipe para cima
                    controlsPanel.classList.add('open');
                } else if (touchEndY - touchStartY > 50) { // Swipe para baixo
                    controlsPanel.classList.remove('open');
                }
                touchStartY = 0;
            });


            function initialize() {
                document.getElementById('load-file-btn').addEventListener('click', () => document.getElementById('file-input').click());
                document.getElementById('file-input').addEventListener('change', handleFileSelect);
                document.getElementById('export-btn').addEventListener('click', exportFoundIds);
                document.body.addEventListener('click', initAudio, { once: true });
                setupTabs();
                startScanner();
            }

            function handleFileSelect(event) {
                const file = event.target.files[0];
                if (!file) return;
                const reader = new FileReader();
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
            
            function processIds(ids, fileName) {
                 appState.idsToFind = new Set(ids);
                 appState.foundIds = [];
                 document.getElementById('file-info').textContent = `Ficheiro "${fileName}" carregado com ${ids.length} IDs.`;
                 updateFoundListUI();
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
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            }

            function startScanner() {
                appState.html5QrCode = new Html5Qrcode("reader");
                const config = {
                    fps: 15,
                    qrbox: (viewfinderWidth, viewfinderHeight) => {
                        const size = Math.floor(Math.min(viewfinderWidth, viewfinderHeight) * 0.8);
                        return { width: size, height: size };
                    }
                };
                appState.html5QrCode.start({ facingMode: "environment" }, config, (decodedText) => handleScan(decodedText))
                    .catch(err => alert("ERRO AO INICIAR A CÂMARA: Por favor, verifique se deu permissão de acesso à câmara."));
            }
            
            function handleScan(scannedId) {
                if (appState.isPaused) return;
                
                if (appState.idsToFind.has(scannedId)) {
                    showFeedback('success', scannedId);
                    appState.idsToFind.delete(scannedId);
                    appState.foundIds.unshift({ id: scannedId, timestamp: new Date() });
                    updateFoundListUI();
                } else if (appState.foundIds.some(item => item.id === scannedId)) {
                    showFeedback('warning', scannedId, 'JÁ ENCONTRADO');
                } else {
                    showFeedback('error', scannedId);
                }
            }
            
            function updateFoundListUI() {
                const foundCountSpan = document.getElementById('found-count');
                const foundList = document.getElementById('found-list');
                foundCountSpan.textContent = appState.foundIds.length;
                if (appState.foundIds.length === 0) {
                    foundList.innerHTML = '<li class="text-slate-500">Nenhum item encontrado ainda.</li>';
                } else {
                    foundList.innerHTML = appState.foundIds.map(item => `<li class="p-2 bg-slate-700 rounded-md text-white">${item.id}</li>`).join('');
                }
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
            
            function setupTabs() {
                const tabs = ['procurar', 'encontrados'];
                tabs.forEach(tabId => {
                    document.getElementById(`tab-${tabId}`).addEventListener('click', () => {
                        tabs.forEach(t => {
                            document.getElementById(`tab-${t}`).classList.replace('tab-active', 'tab-inactive');
                            document.getElementById(`view-${t}`).classList.add('hidden');
                        });
                        document.getElementById(`tab-${tabId}`).classList.replace('tab-inactive', 'tab-active');
                        document.getElementById(`view-${tabId}`).classList.remove('hidden');
                    });
                });
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
