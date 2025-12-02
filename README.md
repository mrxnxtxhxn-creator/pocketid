<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Natefy Enterprise</title>

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
        
        /* Feedback Animations */
        @keyframes pulse-green { 0% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7); } 70% { box-shadow: 0 0 0 20px rgba(34, 197, 94, 0); } 100% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); } }
        .scan-success { animation: pulse-green 0.5s ease-out; }

        /* Botão Desfazer Flutuante */
        #undo-btn {
            position: fixed; bottom: 140px; right: 20px; z-index: 60;
            background-color: #ef4444; color: white;
            width: 56px; height: 56px; border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            transition: transform 0.2s, opacity 0.2s;
            opacity: 0; pointer-events: none; transform: scale(0.8);
        }
        #undo-btn.visible { opacity: 1; pointer-events: auto; transform: scale(1); }

        /* Status de Conexão */
        .conn-indicator { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 5px; }
        .conn-online { background-color: #22c55e; box-shadow: 0 0 5px #22c55e; }
        .conn-offline { background-color: #ef4444; box-shadow: 0 0 5px #ef4444; }
        .conn-syncing { background-color: #eab308; animation: blink 1s infinite; }
        @keyframes blink { 50% { opacity: 0.5; } }

        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background-color: #0f172a; }
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

    <div id="ocr-loading" class="hidden fixed inset-0 z-50 bg-black/90 flex flex-col items-center justify-center">
        <i class="fas fa-circle-notch fa-spin text-4xl text-cyan-500 mb-4"></i>
        <p class="text-white font-bold">Lendo Imagem...</p>
    </div>

    <div id="login-modal" class="fixed inset-0 z-50 bg-slate-950 flex items-center justify-center p-4">
        <div class="bg-slate-900 p-8 rounded-2xl border border-slate-700 w-full max-w-sm text-center shadow-2xl">
            <div class="mb-6 text-cyan-500 text-5xl"><i class="fas fa-box-open"></i></div>
            <h2 class="text-2xl font-bold text-white mb-2">Natefy Enterprise</h2>
            <p class="text-slate-400 text-sm mb-6">Sistema de Gestão de Inventário</p>
            <input type="text" id="operator-input" class="w-full bg-slate-800 text-white p-4 rounded-xl mb-4 border border-slate-700 focus:border-cyan-500 outline-none text-center text-lg" placeholder="Nome do Operador">
            <button id="login-btn" class="w-full bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-4 rounded-xl transition-all">ENTRAR</button>
        </div>
    </div>

    <div id="controls-panel" class="fixed bottom-0 w-full shadow-[0_-10px_40px_rgba(0,0,0,0.5)]">
        <div id="panel-handle" class="w-full h-8 flex justify-center items-center cursor-pointer"><div class="w-16 h-1 bg-slate-600 rounded-full"></div></div>
        
        <div class="w-full max-w-2xl mx-auto px-4">
            <div class="flex overflow-x-auto pb-4 mb-2 gap-4 no-scrollbar">
                <button data-view="procurar" class="tab-btn tab-active"><i class="fas fa-search mr-1"></i> Procurar</button>
                <button data-view="encontrados" class="tab-btn tab-inactive"><i class="fas fa-list-check mr-1"></i> Global</button>
                <button data-view="dashboard" class="tab-btn tab-inactive"><i class="fas fa-chart-pie mr-1"></i> Dash</button>
                <button data-view="analisador" class="tab-btn tab-inactive"><i class="fas fa-balance-scale mr-1"></i> Comp.</button>
                <button data-view="zonas" class="tab-btn tab-inactive"><i class="fas fa-map-marker-alt mr-1"></i> Zonas</button>
            </div>

            <div data-view-content="procurar" class="block">
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <button id="load-file-btn" class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-xs flex items-center justify-center gap-2">
                        <i class="fas fa-file-excel text-green-500"></i> Carregar Lista
                    </button>
                    <button id="load-image-btn" class="bg-slate-800 p-3 rounded-xl border border-slate-700 text-xs flex items-center justify-center gap-2">
                        <i class="fas fa-camera text-cyan-500"></i> Ler Foto
                    </button>
                </div>
                <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv"><input type="file" id="image-input" class="hidden" accept="image/*">
                <p id="file-info" class="text-xs text-center text-slate-400 mb-2 h-4"></p>

                <div class="bg-slate-800/50 p-3 rounded-xl border border-slate-700 space-y-3">
                    <div class="flex gap-2">
                        <input type="text" id="manual-input" class="flex-1 bg-slate-900 border border-slate-600 rounded-lg px-3 text-sm font-mono" placeholder="ID Manual...">
                        <button id="manual-check-btn" class="bg-cyan-600 px-4 rounded-lg text-white font-bold"><i class="fas fa-check"></i></button>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-xs text-slate-400">Modo Rápido</span>
                        <input type="checkbox" id="fast-mode-toggle" class="w-4 h-4 accent-cyan-500">
                    </div>
                     <div class="flex items-center justify-between">
                         <span class="text-xs text-slate-400">Modo Caça</span>
                         <div class="flex gap-2">
                             <input type="text" id="hunt-id" class="w-24 bg-slate-900 border border-slate-600 rounded px-2 text-xs font-mono" placeholder="ID Alvo">
                             <button id="hunt-btn" class="text-xs bg-slate-700 px-2 rounded text-white">Ativar</button>
                         </div>
                    </div>
                </div>
            </div>

            <div data-view-content="encontrados" class="hidden">
                <div class="flex justify-between items-center mb-2">
                    <h3 class="font-bold text-white">Status Global</h3>
                    <button id="export-btn" class="text-xs bg-green-700 px-3 py-1 rounded text-white">Baixar Excel</button>
                </div>
                <div id="global-list" class="space-y-2 max-h-[50vh] overflow-y-auto pb-20">
                    <p class="text-center text-slate-500 text-sm mt-4">Aguardando sincronização...</p>
                </div>
            </div>

            <div data-view-content="dashboard" class="hidden text-center">
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-slate-800 p-4 rounded-xl border border-slate-700 relative">
                        <canvas id="progressChart"></canvas>
                        <div class="absolute inset-0 flex items-center justify-center font-bold text-xl" id="progress-text">0%</div>
                    </div>
                    <div class="flex flex-col gap-2">
                        <div class="bg-slate-800 p-3 rounded-xl border border-slate-700">
                            <span class="text-xs text-slate-400">SEUS BIPS</span>
                            <div id="kpi-my-bips" class="text-2xl font-bold text-cyan-400">0</div>
                        </div>
                         <div class="bg-slate-800 p-3 rounded-xl border border-slate-700">
                            <span class="text-xs text-slate-400">GLOBAL</span>
                            <div id="kpi-global-bips" class="text-2xl font-bold text-white">0</div>
                        </div>
                    </div>
                </div>
                <button id="download-flowchart-btn" class="w-full bg-purple-700 py-3 rounded-xl text-white font-bold mb-2"><i class="fas fa-project-diagram"></i> Gerar Fluxograma & Relatório</button>
            </div>

            <div data-view-content="analisador" class="hidden text-center space-y-4">
                 <div class="space-y-2"><label class="text-xs">Lista A</label><select id="list-a" class="w-full bg-slate-800 p-2 rounded"></select></div>
                 <div class="space-y-2"><label class="text-xs">Lista B</label><select id="list-b" class="w-full bg-slate-800 p-2 rounded"></select></div>
                 <button id="compare-btn" class="w-full bg-blue-600 py-2 rounded text-white">Comparar</button>
                 <div id="compare-results" class="hidden grid grid-cols-3 gap-2 text-xs"></div>
            </div>

            <div data-view-content="zonas" class="hidden">
                <div id="zones-list" class="space-y-2 max-h-[50vh] overflow-y-auto"></div>
            </div>

            <div data-view-content="excecoes" class="hidden">
                <div id="exceptions-list" class="space-y-2 text-xs font-mono"></div>
            </div>
            <div data-view-content="log" class="hidden">
                 <div id="local-log-list" class="space-y-2 text-xs"></div>
            </div>
        </div>
    </div>

    <div id="flowchart-canvas-container"></div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- CONFIGURAÇÃO ---
            const APP_CONFIG = {
                // URL para ENVIAR dados (Write - Método POST)
                WEBHOOK_WRITE: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9',
                // URL para LER dados globais (Read - Método GET - CRIE ESSE WORKFLOW NO N8N!)
                WEBHOOK_READ: 'COLE_SUA_URL_DE_LEITURA_AQUI_SE_TIVER', 
                STORAGE_KEY: 'natefy_v2',
                SYNC_INTERVAL: 15000 // Sincroniza global a cada 15s
            };

            let state = {
                operator: null,
                idsToFind: new Set(),
                idDescriptions: new Map(),
                zoneData: new Map(), // Map<ZoneID, Set<ID>>
                activeZone: null,
                
                // Listas de Dados
                localLog: [], // Histórico local completo
                globalLog: [], // Dados vindos do servidor
                
                // Filas e Controle
                offlineQueue: [], // Store & Forward
                lastUndoable: null, // Para desfazer
                
                // Estado UI
                view: 'procurar',
                isScanning: false,
                isFastMode: false,
                huntTarget: null
            };

            // --- INICIALIZAÇÃO ---
            function init() {
                loadState();
                checkLogin();
                setupUI();
                
                // Tenta enviar fila offline periodicamente
                setInterval(processOfflineQueue, 5000);
                
                // Tenta buscar dados globais periodicamente
                if(APP_CONFIG.WEBHOOK_READ && !APP_CONFIG.WEBHOOK_READ.includes('COLE_SUA')) {
                    setInterval(fetchGlobalData, APP_CONFIG.SYNC_INTERVAL);
                }

                // Monitor de Conexão
                window.addEventListener('online', () => updateConnStatus(true));
                window.addEventListener('offline', () => updateConnStatus(false));
            }

            // --- CORE: PROCESSAMENTO DO BIP ---
            function processScan(id) {
                if (state.huntTarget) {
                    if (id === state.huntTarget) {
                        feedback('success', id, 'ALVO ENCONTRADO!', true);
                        logAction(id, 'HUNT_SUCCESS');
                        state.huntTarget = null; updateHuntUI();
                    } return;
                }

                const now = new Date();
                const desc = state.idDescriptions.get(id) || '';
                let status = 'ERRO';
                let msg = 'NÃO ENCONTRADO';
                let type = 'error';

                // 1. Verifica Missort
                if (state.activeZone) {
                    for (const [zId, ids] of state.zoneData) {
                        if (zId !== state.activeZone && ids.has(id)) {
                            status = 'MISSORT';
                            msg = `MISSORT (${zId})`;
                            type = 'warning';
                            executeLog(id, status, msg, type, desc);
                            return;
                        }
                    }
                }

                // 2. Verifica Duplicidade (Local e Global se possível)
                // Nota: Verifica localmente primeiro
                if (state.localLog.some(l => l.id === id && l.status.includes('SUCESSO'))) {
                    status = 'DUPLICADO';
                    msg = 'JÁ BIPADO';
                    type = 'warning';
                    executeLog(id, status, msg, type, desc);
                    return;
                }

                // 3. Verifica Lista Principal
                if (state.idsToFind.has(id)) {
                    status = 'SUCESSO';
                    msg = desc || 'ENCONTRADO';
                    type = 'success';
                    state.idsToFind.delete(id); // Remove da lista de busca
                } else {
                    // 4. Verifica outras zonas (sem ser missort)
                    let foundInZone = false;
                    for (const [zId, ids] of state.zoneData) {
                        if (ids.has(id)) {
                            status = 'SUCESSO';
                            msg = `ZONA: ${zId}`;
                            type = 'success';
                            foundInZone = true;
                            break;
                        }
                    }
                    if (!foundInZone) {
                        // Realmente não encontrado
                        status = 'ERRO';
                        msg = 'NÃO ENCONTRADO';
                        type = 'error';
                    }
                }

                executeLog(id, status, msg, type, desc);
            }

            function executeLog(id, status, msg, type, desc) {
                const entry = {
                    id, status, desc, 
                    time: new Date().toISOString(),
                    operator: state.operator,
                    zone: state.activeZone
                };

                // Salva Local
                state.localLog.unshift(entry);
                state.lastUndoable = entry; // Permite desfazer
                
                // Salva e Atualiza
                saveState();
                feedback(type, id, msg);
                updateUI();
                showUndoButton();

                // Envia para n8n (Com Fila Offline)
                queueForSending(entry);
            }

            // --- STORE & FORWARD (FILA OFFLINE) ---
            function queueForSending(data) {
                state.offlineQueue.push(data);
                saveState();
                updateQueueUI();
                processOfflineQueue(); // Tenta enviar imediatamente
            }

            async function processOfflineQueue() {
                if (state.offlineQueue.length === 0 || !navigator.onLine) return;

                const item = state.offlineQueue[0]; // Pega o primeiro
                updateConnStatus(true, true); // Amarelo (Sincronizando)

                try {
                    // Se tiver URL de escrita configurada
                    if(APP_CONFIG.WEBHOOK_WRITE && !APP_CONFIG.WEBHOOK_WRITE.includes('COLE_A')) {
                         await fetch(APP_CONFIG.WEBHOOK_WRITE, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(item)
                        });
                    }
                    
                    // Sucesso: Remove da fila
                    state.offlineQueue.shift();
                    saveState();
                    updateQueueUI();
                    updateConnStatus(true); // Verde
                    
                    // Se ainda tem itens, processa o próximo (recursivo com delay pequeno)
                    if (state.offlineQueue.length > 0) setTimeout(processOfflineQueue, 100);

                } catch (e) {
                    console.error("Falha no envio, item mantido na fila", e);
                    updateConnStatus(true); // Volta para verde (conectado, mas falhou envio específico)
                }
            }

            // --- FETCH GLOBAL DATA (SINCRONIZAÇÃO MULTI-USUÁRIO) ---
            async function fetchGlobalData() {
                if (!APP_CONFIG.WEBHOOK_READ || APP_CONFIG.WEBHOOK_READ.includes('COLE_SUA')) return;

                try {
                    const res = await fetch(APP_CONFIG.WEBHOOK_READ);
                    if (res.ok) {
                        const data = await res.json();
                        // Assume que o n8n retorna um array de logs
                        // [{id: '123', operator: 'JOAO', status: 'SUCESSO', desc: '...', time: '...'}]
                        if (Array.isArray(data)) {
                            state.globalLog = data;
                            updateGlobalListUI();
                        }
                    }
                } catch (e) {
                    console.warn("Erro ao buscar dados globais", e);
                }
            }

            // --- UI & INTERFACE ---
            function feedback(type, id, msg, persistent = false) {
                const el = document.getElementById('feedback-overlay');
                let color = type === 'success' ? 'rgba(34, 197, 94, 0.9)' : 
                            type === 'warning' ? 'rgba(234, 179, 8, 0.9)' : 
                            'rgba(239, 68, 68, 0.9)';
                
                el.style.background = `radial-gradient(circle, ${color}, transparent)`;
                el.innerHTML = `
                    <div class="transform scale-110 transition-transform duration-200">
                        <div class="text-4xl font-black mb-2 drop-shadow-lg">${msg}</div>
                        <div class="text-xl font-mono bg-black/50 px-4 py-2 rounded-lg inline-block border border-white/20">${id}</div>
                    </div>`;
                el.style.opacity = '1';
                
                // Som e Vibração
                playSound(type);
                if(navigator.vibrate) navigator.vibrate(type==='error'?[100,50,100]:200);

                if (!state.isFastMode || persistent) {
                    setTimeout(() => el.style.opacity = '0', persistent ? 3000 : 1500);
                } else {
                    setTimeout(() => el.style.opacity = '0', 500);
                }
            }

            function updateUI() {
                // Atualiza contadores e listas
                document.getElementById('found-count').innerText = state.localLog.filter(l => l.status === 'SUCESSO').length;
                document.getElementById('exceptions-count').innerText = state.localLog.filter(l => l.status === 'ERRO').length;
                
                // Atualiza Dashboard Local
                const total = state.idsToFind.size + state.localLog.filter(l => l.status === 'SUCESSO').length; // Aproximado
                const done = state.localLog.filter(l => l.status === 'SUCESSO').length;
                const pct = total > 0 ? Math.round((done/total)*100) : 0;
                document.getElementById('progress-text').innerText = `${pct}%`;
                document.getElementById('kpi-my-bips').innerText = state.localLog.length;
                
                // Atualiza gráfico (se existir)
                if(state.chart) {
                    state.chart.data.datasets[0].data = [pct, 100-pct];
                    state.chart.update();
                }

                renderLogList();
            }

            function updateGlobalListUI() {
                // Mescla Local + Global para exibição na aba "Encontrados"
                const container = document.getElementById('global-list');
                // Combina logs, preferindo o global se disponível, ou local
                // Aqui simplificamos: mostramos o globalLog que veio do n8n
                
                if(state.globalLog.length === 0 && state.localLog.length === 0) {
                     container.innerHTML = '<div class="text-center text-slate-500 text-sm p-4">Nenhum dado ainda.</div>';
                     return;
                }

                // Para demonstração, vamos renderizar o localLog se o global estiver vazio, ou misturar
                const displayData = state.globalLog.length > 0 ? state.globalLog : state.localLog;

                container.innerHTML = displayData.slice(0, 100).map(l => `
                    <div class="bg-slate-800 p-3 rounded-lg border border-slate-700 flex justify-between items-start">
                        <div>
                            <div class="font-bold text-white flex items-center gap-2">
                                ${l.id} 
                                <span class="text-[10px] bg-slate-700 px-1 rounded text-cyan-400">${l.operator || 'EU'}</span>
                            </div>
                            <div class="text-xs text-slate-400 truncate w-48">${l.desc || ''}</div>
                        </div>
                        <div class="text-right">
                            <div class="text-[10px] font-bold ${getStatusColor(l.status)}">${l.status}</div>
                            <div class="text-[10px] text-slate-500">${new Date(l.time).toLocaleTimeString('pt-BR', {hour:'2-digit', minute:'2-digit'})}</div>
                        </div>
                    </div>
                `).join('');
                
                document.getElementById('kpi-global-bips').innerText = displayData.length;
            }
            
            function getStatusColor(s) {
                if(s.includes('SUCESSO')) return 'text-green-400';
                if(s.includes('ERRO')) return 'text-red-400';
                if(s.includes('MISSORT')) return 'text-orange-400';
                return 'text-yellow-400';
            }

            function renderLogList() {
                const list = document.getElementById('scan-log-list');
                list.innerHTML = state.localLog.slice(0, 50).map(l => `
                     <div class="flex justify-between border-b border-slate-700 py-2">
                        <span>${l.id} <span class="text-slate-500 text-[10px]">${l.desc ? '('+l.desc.substring(0,10)+'..)' : ''}</span></span>
                        <span class="${getStatusColor(l.status)} text-xs">${l.status}</span>
                     </div>
                `).join('');
            }

            // --- FUNÇÕES AUXILIARES (Login, Undo, OCR, etc) ---
            
            // UNDO (Desfazer)
            function showUndoButton() {
                const btn = document.getElementById('undo-btn');
                btn.classList.add('visible');
                setTimeout(() => btn.classList.remove('visible'), 5000); // Some após 5s
            }
            
            document.getElementById('undo-btn').addEventListener('click', () => {
                if(!state.lastUndoable) return;
                // Remove do local
                if(state.lastUndoable.status === 'SUCESSO') {
                    state.idsToFind.add(state.lastUndoable.id); // Devolve para a lista
                }
                state.localLog.shift(); // Remove do log
                
                // Envia evento de cancelamento
                queueForSending({
                    ...state.lastUndoable,
                    status: 'CANCELADO',
                    time: new Date().toISOString()
                });
                
                state.lastUndoable = null;
                document.getElementById('undo-btn').classList.remove('visible');
                alert("Última ação desfeita.");
                updateUI();
            });

            // OCR E IMPORTAÇÃO DE ARQUIVO (Simplificado do anterior)
            document.getElementById('load-image-btn').addEventListener('click', () => document.getElementById('image-input').click());
            document.getElementById('image-input').addEventListener('change', async (e) => {
                const file = e.target.files[0]; if(!file) return;
                document.getElementById('ocr-loading').classList.remove('hidden');
                try {
                    const worker = Tesseract.createWorker();
                    await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                    const { data: { text } } = await worker.recognize(file);
                    await worker.terminate();
                    const ids = new Set();
                    text.split('\n').forEach(line => {
                        const m = line.match(/(\d{8,14})/);
                        if(m) {
                            ids.add(m[0]);
                            let desc = line.replace(m[0], '').trim();
                            if(desc) state.idDescriptions.set(m[0], desc);
                        }
                    });
                    if(ids.size > 0) {
                        state.idsToFind = ids;
                        alert(`${ids.size} itens carregados da foto!`);
                        updateUI();
                    }
                } catch(err) { alert("Erro OCR"); }
                finally { document.getElementById('ocr-loading').classList.add('hidden'); }
            });

            // LOGIN
            function checkLogin() {
                if(!state.operator) document.getElementById('login-modal').classList.remove('hidden');
                else {
                    document.getElementById('login-modal').classList.add('hidden');
                    document.getElementById('status-operator').innerText = state.operator.toUpperCase();
                    startScanner();
                }
            }
            document.getElementById('login-btn').addEventListener('click', () => {
                const name = document.getElementById('operator-input').value;
                if(name) { state.operator = name; saveState(); checkLogin(); }
            });

            // RELATÓRIOS (Excel Completo)
            document.getElementById('download-full-report-btn').addEventListener('click', () => {
                // Combina local e global para o relatório
                const allData = [...state.globalLog, ...state.localLog]; 
                // Remove duplicatas de exibição se necessário, aqui apenas dumpamos tudo
                if(allData.length === 0) return alert("Sem dados");
                
                const ws = XLSX.utils.json_to_sheet(allData);
                const wb = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(wb, ws, "Relatório Geral");
                XLSX.writeFile(wb, `Relatorio_Natefy_${new Date().toISOString().slice(0,10)}.xlsx`);
            });

            // FLUXOGRAMA
            document.getElementById('download-flowchart-btn').addEventListener('click', async () => {
                 const container = document.getElementById('flowchart-canvas-container');
                 // Lógica visual do fluxograma (similar ao anterior, mas com dados atualizados)
                 // ... (Inserir lógica de renderização HTML aqui)
                 // Simplificado para brevidade:
                 container.innerHTML = `<div style="background:#0f172a;padding:50px;color:white;text-align:center"><h1>RELATÓRIO VISUAL</h1><p>Operador: ${state.operator}</p><p>Total Bips: ${state.localLog.length}</p></div>`;
                 
                 const canvas = await html2canvas(container);
                 const a = document.createElement('a');
                 a.href = canvas.toDataURL();
                 a.download = 'Fluxo.png';
                 a.click();
            });

            // SYSTEM
            function saveState() { localStorage.setItem(STORAGE_KEY, JSON.stringify({ ...state, scanHistory: [], charts: null })); } // Não salvamos charts/history pesado
            function loadState() {
                const s = JSON.parse(localStorage.getItem(STORAGE_KEY));
                if(s) { 
                    state = { ...state, ...s }; 
                    // Recupera Sets e Maps
                    state.idsToFind = new Set(s.idsToFind);
                    state.idDescriptions = new Map(s.idDescriptions);
                    state.inventoryZoneData = new Map(s.inventoryZoneData.map(i => [i[0], new Set(i[1])]));
                }
            }
            function updateQueueUI() {
                const qc = document.getElementById('queue-count');
                const qContainer = document.getElementById('queue-counter');
                qc.innerText = state.offlineQueue.length;
                if(state.offlineQueue.length > 0) qContainer.classList.remove('hidden');
                else qContainer.classList.add('hidden');
            }
            function updateConnStatus(online, syncing=false) {
                const el = document.getElementById('connection-status');
                el.className = 'conn-indicator ' + (syncing ? 'conn-syncing' : (online ? 'conn-online' : 'conn-offline'));
            }
            function startScanner() {
                if(!state.html5QrCode) {
                    state.html5QrCode = new Html5Qrcode("reader");
                    state.html5QrCode.start({facingMode:"environment"}, {fps:15, qrbox:250}, processScan);
                }
            }
            function playSound(type) {
                // Implementação de som simples
                if(!appState.audioContext) appState.audioContext = new (window.AudioContext||window.webkitAudioContext)();
                const o = appState.audioContext.createOscillator();
                const g = appState.audioContext.createGain();
                o.connect(g); g.connect(appState.audioContext.destination);
                if(type==='success') o.frequency.value=1200;
                else if(type==='error') o.frequency.value=200;
                else o.frequency.value=600;
                o.start(); setTimeout(()=>o.stop(), 150);
            }
            
            // Tabs Logic
            document.querySelectorAll('.tab-btn').forEach(b => {
                b.addEventListener('click', () => {
                    document.querySelectorAll('.tab-btn').forEach(x=>x.classList.replace('tab-active','tab-inactive'));
                    b.classList.replace('tab-inactive','tab-active');
                    document.querySelectorAll('[data-view-content]').forEach(d => d.classList.add('hidden'));
                    document.querySelector(`[data-view-content="${b.dataset.view}"]`).classList.remove('hidden');
                    if(b.dataset.view === 'encontrados') fetchGlobalData(); // Atualiza ao abrir
                });
            });

            init();
        });
    </script>
</body>
</html>
