<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Natefy Pro</title>

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
        
        /* Scanner ocupa a tela toda */
        #reader { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; object-fit: cover; }
        #reader video { object-fit: cover; width: 100% !important; height: 100% !important; }

        /* Área de Scan Visual (Mira) */
        .scan-overlay {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 80vw; height: 25vh;
            border: 2px solid rgba(255, 255, 255, 0.5);
            border-radius: 20px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.5); /* Escurece o resto */
            z-index: 10;
            pointer-events: none;
        }
        .scan-line {
            width: 100%; height: 2px; background: #06b6d4;
            box-shadow: 0 0 4px #06b6d4;
            animation: scanMove 2s infinite linear;
        }
        @keyframes scanMove { 0% { transform: translateY(0); } 50% { transform: translateY(25vh); } 100% { transform: translateY(0); } }

        /* Menu Inferior Fixo (Estilo App Nativo) */
        #bottom-nav {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 70px;
            background: rgba(15, 23, 42, 0.95);
            backdrop-filter: blur(10px);
            border-top: 1px solid #334155;
            display: flex; justify-content: space-around; align-items: center;
            z-index: 50;
            padding-bottom: env(safe-area-inset-bottom, 10px);
        }
        .nav-item {
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            color: #94a3b8; font-size: 10px; gap: 4px; width: 20%;
        }
        .nav-item i { font-size: 20px; margin-bottom: 2px; transition: transform 0.2s; }
        .nav-item.active { color: #22d3ee; }
        .nav-item.active i { transform: translateY(-2px); }

        /* Painéis de Conteúdo (Cards Flutuantes) */
        .content-panel {
            position: absolute; bottom: 80px; left: 0; width: 100%;
            max-height: 70vh; overflow-y: auto;
            background: transparent;
            z-index: 40;
            display: none; /* Escondido por padrão */
            padding: 0 16px;
        }
        .content-panel.active { display: block; animation: slideUp 0.3s ease-out; }
        .panel-card {
            background: rgba(30, 41, 59, 0.95);
            backdrop-filter: blur(12px);
            border: 1px solid #475569;
            border-radius: 16px;
            padding: 16px;
            color: white;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
        }

        @keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }

        /* Status Bar Superior */
        #top-bar {
            position: absolute; top: 0; left: 0; width: 100%; height: 60px;
            background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent);
            z-index: 30;
            display: flex; justify-content: space-between; items-center; padding: 0 20px;
            padding-top: env(safe-area-inset-top, 10px);
        }

        /* Feedback Overlay */
        #feedback-overlay { position: fixed; inset: 0; z-index: 60; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
        .scan-success { background: rgba(34, 197, 94, 0.85); }
        .scan-error { background: rgba(239, 68, 68, 0.85); }
        .scan-warning { background: rgba(234, 179, 8, 0.85); }

        /* Botão Undo Flutuante */
        #undo-btn {
            position: absolute; top: 80px; right: 20px; z-index: 40;
            background: #ef4444; color: white; width: 50px; height: 50px; border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
            opacity: 0; pointer-events: none; transition: opacity 0.3s;
        }
        #undo-btn.visible { opacity: 1; pointer-events: auto; }

        /* Utilitários */
        .hidden { display: none !important; }
        .btn-primary { background: linear-gradient(to right, #0891b2, #2563eb); padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; }
        .btn-secondary { background: #334155; padding: 12px; border-radius: 12px; font-weight: bold; width: 100%; text-align: center; border: 1px solid #475569; }
        
        /* Canvas Oculto */
        #flowchart-canvas-container { position: absolute; top: -9999px; left: -9999px; width: 1200px; background-color: #0f172a; }
    </style>
</head>
<body>

    <div id="top-bar">
        <div class="flex flex-col">
            <span id="status-operator" class="text-sm font-bold text-white shadow-black drop-shadow-md">--</span>
            <span id="status-zone" class="text-xs text-cyan-400 font-mono bg-black/40 px-2 rounded">ZONA: --</span>
        </div>
        <div class="flex gap-3">
            <div id="conn-dot" class="w-3 h-3 rounded-full bg-green-500 shadow-[0_0_10px_#22c55e]"></div>
        </div>
    </div>

    <div id="reader"></div>
    <div class="scan-overlay">
        <div class="scan-line"></div>
    </div>

    <div id="feedback-overlay">
        <div class="text-center p-6">
            <div id="fb-icon" class="text-6xl mb-4">✅</div>
            <div id="fb-msg" class="text-4xl font-black text-white drop-shadow-md mb-2">ENCONTRADO</div>
            <div id="fb-desc" class="text-lg text-white/90 font-medium bg-black/20 p-2 rounded">Descrição aqui</div>
            <div id="fb-id" class="text-sm text-white/60 font-mono mt-2">ID: 123456</div>
        </div>
    </div>

    <button id="undo-btn"><i class="fas fa-undo"></i></button>

    <div id="ocr-loading" class="fixed inset-0 z-[70] bg-black/90 flex flex-col items-center justify-center hidden">
        <i class="fas fa-sync fa-spin text-5xl text-cyan-500 mb-4"></i>
        <p class="text-xl font-bold text-white">Lendo Imagem...</p>
        <p class="text-sm text-slate-400 mt-2">Isso pode levar alguns segundos</p>
    </div>

    <div id="login-modal" class="fixed inset-0 z-[80] bg-slate-900 flex items-center justify-center p-6">
        <div class="w-full max-w-sm text-center">
            <div class="w-20 h-20 bg-cyan-500 rounded-2xl mx-auto mb-6 flex items-center justify-center text-4xl text-white shadow-[0_0_30px_rgba(6,182,212,0.4)]">
                <i class="fas fa-cube"></i>
            </div>
            <h1 class="text-3xl font-bold text-white mb-2">Olá!</h1>
            <p class="text-slate-400 mb-8">Vamos bipar?</p>
            
            <input type="text" id="operator-input" class="w-full bg-slate-800 border border-slate-600 text-white p-4 rounded-xl text-center text-lg mb-4 focus:border-cyan-500 outline-none" placeholder="Seu Nome">
            <button id="login-btn" class="btn-primary">ENTRAR</button>
        </div>
    </div>

    <div id="view-procurar" class="content-panel active">
        <div class="panel-card space-y-4">
            <div class="grid grid-cols-2 gap-3">
                <button id="load-file-btn" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                    <i class="fas fa-file-excel text-2xl text-green-500"></i>
                    <span class="text-xs font-bold">PLANILHA</span>
                </button>
                <button id="load-image-btn" class="bg-slate-800 p-4 rounded-xl border border-slate-600 flex flex-col items-center gap-2 active:bg-slate-700">
                    <i class="fas fa-camera text-2xl text-cyan-500"></i>
                    <span class="text-xs font-bold">FOTO (OCR)</span>
                </button>
            </div>
            <input type="file" id="file-input" class="hidden" accept=".xlsx,.csv">
            <input type="file" id="image-input" class="hidden" accept="image/*">
            
            <div id="file-status" class="bg-slate-900/50 p-2 rounded text-xs text-center text-slate-400">
                Nenhuma lista carregada
            </div>

            <div class="flex gap-2">
                <input type="text" id="manual-input" class="flex-1 bg-slate-900 border border-slate-600 rounded-lg px-4 py-3 text-sm text-white" placeholder="Digitar ID...">
                <button id="manual-btn" class="bg-cyan-600 px-5 rounded-lg font-bold"><i class="fas fa-check"></i></button>
            </div>

            <div class="flex items-center justify-between bg-slate-900/50 p-3 rounded-lg">
                <span class="text-sm text-slate-300">Modo Rápido</span>
                <input type="checkbox" id="fast-mode-toggle" class="w-5 h-5 accent-cyan-500">
            </div>
             
             <div class="flex gap-2 mt-2">
                 <button id="clear-btn" class="flex-1 py-2 text-xs text-red-400 border border-red-900 rounded">Limpar</button>
                 <button id="logout-btn" class="flex-1 py-2 text-xs text-slate-400 border border-slate-700 rounded">Sair</button>
             </div>
        </div>
    </div>

    <div id="view-dashboard" class="content-panel">
        <div class="panel-card space-y-4 text-center">
            <h2 class="text-lg font-bold text-white">Performance</h2>
            <div class="flex justify-center">
                <div class="relative w-32 h-32">
                    <canvas id="progressChart"></canvas>
                    <div class="absolute inset-0 flex items-center justify-center text-2xl font-bold" id="progress-text">0%</div>
                </div>
            </div>
            <div class="grid grid-cols-2 gap-4">
                <div class="bg-slate-900 p-3 rounded-lg">
                    <div class="text-2xl font-bold text-cyan-400" id="kpi-total">0</div>
                    <div class="text-[10px] text-slate-500 uppercase">Total Bipado</div>
                </div>
                <div class="bg-slate-900 p-3 rounded-lg">
                    <div class="text-2xl font-bold text-red-400" id="kpi-error">0</div>
                    <div class="text-[10px] text-slate-500 uppercase">Erros / Missort</div>
                </div>
            </div>
             <button id="download-report-btn" class="btn-primary text-sm"><i class="fas fa-file-export mr-2"></i> Baixar Relatório Excel</button>
             <button id="download-flow-btn" class="btn-secondary text-sm"><i class="fas fa-image mr-2"></i> Baixar Fluxograma</button>
        </div>
    </div>

    <div id="view-log" class="content-panel">
        <div class="panel-card">
            <h2 class="text-lg font-bold text-white mb-2">Histórico do Dia</h2>
            <div id="scan-log-list" class="space-y-2 max-h-[50vh] overflow-y-auto">
                <div class="text-center text-slate-500 text-xs py-4">Vazio</div>
            </div>
        </div>
    </div>
    
    <div id="view-zonas" class="content-panel">
         <div class="panel-card">
            <h2 class="text-lg font-bold text-white mb-2">Zonas</h2>
            <div id="zones-list" class="space-y-2 max-h-[50vh] overflow-y-auto"></div>
        </div>
    </div>

    <nav id="bottom-nav">
        <div class="nav-item active" data-target="procurar">
            <i class="fas fa-search"></i>
            <span>Scan</span>
        </div>
        <div class="nav-item" data-target="log">
            <i class="fas fa-list"></i>
            <span>Log</span>
        </div>
        <div class="nav-item" data-target="dashboard">
            <i class="fas fa-chart-pie"></i>
            <span>Dash</span>
        </div>
        <div class="nav-item" data-target="zonas">
            <i class="fas fa-map-marker-alt"></i>
            <span>Zonas</span>
        </div>
    </nav>

    <div id="flowchart-canvas-container"></div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // CONFIG
            const CONFIG = {
                STORAGE_KEY: 'natefy_pro_v4',
                WEBHOOK: 'https://mrxnxtxhxn-creator.app.n8n.cloud/webhook-test/df5b4afe-2fc4-4692-a80f-257aca92edf9' // Coloque sua URL aqui
            };

            let state = {
                operator: null,
                idsToFind: new Set(),
                idDescriptions: new Map(),
                foundIds: [], // {id, time, status, desc}
                logs: [], // Histórico completo
                activeZone: null,
                inventoryZones: [
                    {id:'buffered',name:'Buffered'},{id:'sorting',name:'Sorting'},
                    {id:'fraude',name:'Fraude'},{id:'missort',name:'Missort'},
                    {id:'returns',name:'Returns'},{id:'bulky',name:'Bulky'}
                ],
                zoneData: new Map(),
                isFastMode: false,
                isPaused: false,
                lastScanTime: 0,
                lastUndo: null
            };

            // --- INIT ---
            function init() {
                loadState();
                checkLogin();
                setupNav();
                setupActions();
                startScanner();
                updateUI();
            }

            // --- LOGIN (CORRIGIDO) ---
            function checkLogin() {
                const modal = document.getElementById('login-modal');
                // Se já tem operador salvo, esconde o modal
                if (state.operator) {
                    modal.classList.add('hidden');
                    document.getElementById('status-operator').innerText = state.operator.toUpperCase();
                } else {
                    // Se não tem, mostra o modal
                    modal.classList.remove('hidden');
                }
            }
            
            // Ação do Botão Entrar (Mais robusta)
            document.getElementById('login-btn').addEventListener('click', () => {
                const input = document.getElementById('operator-input');
                const val = input.value.trim();
                
                if(val) { 
                    // 1. Salva o estado
                    state.operator = val; 
                    saveState(); 
                    
                    // 2. Atualiza a UI
                    document.getElementById('status-operator').innerText = state.operator.toUpperCase();
                    
                    // 3. Esconde o modal explicitamente
                    document.getElementById('login-modal').classList.add('hidden');
                    
                } else {
                    alert("Por favor, digite seu nome.");
                }
            });

            // --- NAVEGAÇÃO ---
            function setupNav() {
                const navItems = document.querySelectorAll('.nav-item');
                const views = document.querySelectorAll('.content-panel');

                navItems.forEach(item => {
                    item.addEventListener('click', () => {
                        // 1. Muda visual do menu
                        navItems.forEach(n => n.classList.remove('active'));
                        item.classList.add('active');

                        // 2. Mostra/Esconde Painéis
                        const target = item.dataset.target;
                        views.forEach(v => v.classList.remove('active'));
                        
                        // Se for 'procurar', a gente esconde todos os painéis para ver a câmera
                        // Se for outro, mostra o painel correspondente
                        if (target !== 'procurar') {
                            document.getElementById(`view-${target}`).classList.add('active');
                            if(target === 'dashboard') updateDashboard();
                            if(target === 'zonas') renderZones();
                        }
                    });
                });
            }

            // --- AÇÕES E BOTÕES ---
            function setupActions() {
                // Uploads
                document.getElementById('load-file-btn').onclick = () => document.getElementById('file-input').click();
                document.getElementById('file-input').onchange = (e) => handleFile(e.target.files[0]);
                
                document.getElementById('load-image-btn').onclick = () => document.getElementById('image-input').click();
                document.getElementById('image-input').onchange = (e) => handleOCR(e.target.files[0]);

                // Manual Scan
                document.getElementById('manual-btn').onclick = () => {
                    const v = document.getElementById('manual-input').value.trim();
                    if(v) { processScan(v); document.getElementById('manual-input').value=''; }
                };

                // Fast Mode
                document.getElementById('fast-mode-toggle').onchange = (e) => state.isFastMode = e.target.checked;

                // Undo
                document.getElementById('undo-btn').onclick = doUndo;

                // System
                document.getElementById('logout-btn').onclick = () => { state.operator = null; saveState(); location.reload(); };
                document.getElementById('clear-btn').onclick = () => { if(confirm('Apagar tudo?')) { localStorage.removeItem(CONFIG.STORAGE_KEY); location.reload(); }};
                
                // Reports
                document.getElementById('download-report-btn').onclick = downloadExcel;
                document.getElementById('download-flow-btn').onclick = downloadFlow;
            }

            // --- CORE: SCANNER ---
            function startScanner() {
                const html5QrCode = new Html5Qrcode("reader");
                html5QrCode.start({ facingMode: "environment" }, { fps: 15, qrbox: 250 }, 
                    (decodedText) => processScan(decodedText)
                ).catch(err => console.log("Erro Câmera", err));
            }

            function processScan(id) {
                // Debounce (Evita bips duplos rápidos)
                const now = Date.now();
                if (now - state.lastScanTime < 1000) return;
                state.lastScanTime = now;

                if (state.isPaused) return;
                if (!state.isFastMode) state.isPaused = true;

                // --- LÓGICA DE NEGÓCIO ---
                // Limpeza básica do ID (remove espaços extras)
                id = id.trim();

                // Recupera descrição do mapa de OCR
                const desc = state.idDescriptions.get(id) || "Sem descrição";
                
                let status = "NÃO ENCONTRADO";
                let type = "error";
                let feedbackMsg = "NÃO ENCONTRADO";

                // 1. Verifica Missort (Se zona estiver ativa)
                if (state.activeZone) {
                    for (const [zId, zSet] of state.zoneData) {
                        if (zId !== state.activeZone && zSet.has(id)) {
                            status = "MISSORT";
                            type = "warning";
                            feedbackMsg = `MISSORT (${zId})`;
                            break;
                        }
                    }
                }

                // 2. Verifica Duplicidade
                if (status === "NÃO ENCONTRADO" && state.foundIds.some(x => x.id === id)) {
                    status = "DUPLICADO";
                    type = "warning";
                    feedbackMsg = "JÁ BIPADO";
                }

                // 3. Verifica Sucesso
                if (status === "NÃO ENCONTRADO" && state.idsToFind.has(id)) {
                    status = "SUCESSO";
                    type = "success";
                    feedbackMsg = "ENCONTRADO";
                    state.idsToFind.delete(id); // Remove da lista para não duplicar depois
                }

                // Registra
                const entry = {
                    id, status, desc, type,
                    time: new Date().toISOString(),
                    operator: state.operator,
                    zone: state.activeZone
                };

                state.logs.unshift(entry);
                if (status === "SUCESSO") state.foundIds.unshift(entry);
                state.lastUndo = entry;

                saveState();
                showFeedback(type, feedbackMsg, desc, id);
                updateUI();
                sendToN8n(entry);
            }

            // --- FEEDBACK VISUAL ---
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

                // Mostra overlay
                overlay.style.opacity = '1';
                
                // Mostra botão undo
                undo.classList.add('visible');
                setTimeout(() => undo.classList.remove('visible'), 5000);

                // Som e Vibração
                if(navigator.vibrate) navigator.vibrate(200);
                
                // Esconde depois de um tempo
                setTimeout(() => {
                    overlay.style.opacity = '0';
                    if (!state.isFastMode) state.isPaused = false;
                }, state.isFastMode ? 500 : 1500);
            }

            // --- OCR (LEITURA DE FOTO) ---
            async function handleOCR(file) {
                if (!file) return;
                document.getElementById('ocr-loading').classList.remove('hidden');
                
                try {
                    const worker = Tesseract.createWorker();
                    await worker.load(); await worker.loadLanguage('eng'); await worker.initialize('eng');
                    const { data: { text } } = await worker.recognize(file);
                    await worker.terminate();

                    // Processamento Inteligente
                    const lines = text.split('\n');
                    let count = 0;
                    
                    // Regex melhorada para pegar IDs grandes no início da linha
                    const idRegex = /(\d{8,14})/; 

                    const newIds = new Set();
                    
                    lines.forEach(line => {
                        // Remove caracteres estranhos comuns em OCR
                        const cleanLine = line.replace(/[^\w\s\>\-\.\(\)\/]/gi, '');
                        const match = cleanLine.match(idRegex);
                        
                        if (match) {
                            const id = match[0];
                            // A descrição é tudo que vem DEPOIS do ID
                            let desc = cleanLine.replace(id, '').replace(/^[\s\>\-\.]+/g, '').trim();
                            
                            if (desc.length < 3) desc = "Produto sem descrição"; // Fallback

                            newIds.add(id);
                            state.idDescriptions.set(id, desc);
                            count++;
                        }
                    });

                    if (count > 0) {
                        state.idsToFind = newIds;
                        state.foundIds = []; // Resetar encontrados
                        document.getElementById('file-status').innerText = `📷 Foto: ${count} itens`;
                        document.getElementById('file-status').className = "bg-green-900/50 p-2 rounded text-xs text-center text-green-400";
                        alert(`${count} itens lidos da foto!`);
                        saveState();
                    } else {
                        alert("Não encontrei IDs válidos na foto. Tente uma imagem mais nítida.");
                    }

                } catch (e) {
                    console.error(e);
                    alert("Erro ao ler imagem.");
                } finally {
                    document.getElementById('ocr-loading').classList.add('hidden');
                }
            }

            // --- EXCEL (FILE) ---
            function handleFile(file) {
                if(!file) return;
                const reader = new FileReader();
                reader.onload = (e) => {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet, {header: 1});
                    
                    // Assume ID na coluna A (index 0)
                    const ids = json.map(r => String(r[0])).filter(i => i && i.match(/\d+/));
                    
                    state.idsToFind = new Set(ids);
                    state.foundIds = [];
                    document.getElementById('file-status').innerText = `📂 Excel: ${ids.length} itens`;
                    document.getElementById('file-status').className = "bg-green-900/50 p-2 rounded text-xs text-center text-green-400";
                    saveState();
                    alert(`${ids.length} itens carregados.`);
                };
                reader.readAsArrayBuffer(file);
            }

            // --- UNDO ---
            function doUndo() {
                if(!state.lastUndo) return;
                
                // Remove do log local
                state.logs.shift();
                
                // Se era sucesso, devolve pra lista de procurar e tira de encontrados
                if (state.lastUndo.status === "SUCESSO") {
                    state.foundIds.shift();
                    state.idsToFind.add(state.lastUndo.id);
                }

                // Envia evento de cancelamento pro n8n
                sendToN8n({ ...state.lastUndo, status: "CANCELADO" });

                state.lastUndo = null;
                document.getElementById('undo-btn').classList.remove('visible');
                updateUI();
                alert("Desfeito!");
            }

            // --- N8N ---
            function sendToN8n(data) {
                if(CONFIG.WEBHOOK.includes("https")) {
                    fetch(CONFIG.WEBHOOK, {
                        method: "POST",
                        headers: {"Content-Type": "application/json"},
                        body: JSON.stringify(data)
                    }).catch(e => console.log("Offline/Erro N8N"));
                }
            }

            // --- ZONES ---
            function renderZones() {
                const c = document.getElementById('inventory-zones-container');
                c.innerHTML = `<button class="w-full bg-slate-700 py-3 rounded-lg mb-3 text-sm" onclick="window.setZone(null)">🚫 Sair da Zona</button>`;
                state.inventoryZones.forEach(z => {
                    const isActive = state.activeZone === z.id;
                    c.innerHTML += `
                        <div class="bg-slate-800 p-4 rounded-xl flex justify-between items-center mb-2 border ${isActive ? 'border-green-500' : 'border-slate-700'}">
                            <span class="font-bold text-white">${z.name}</span>
                            <button class="px-4 py-2 rounded-lg text-xs font-bold ${isActive ? 'bg-green-600 text-white' : 'bg-blue-600 text-white'}" onclick="window.setZone('${z.id}')">
                                ${isActive ? 'ATIVA' : 'ATIVAR'}
                            </button>
                        </div>`;
                });
            }
            window.setZone = (id) => { 
                state.activeZone = id; 
                document.getElementById('status-zone').innerText = `ZONA: ${id ? id.toUpperCase() : '--'}`;
                saveState();
                renderZones();
            };

            // --- UI UPDATES ---
            function updateUI() {
                // Log List
                const list = document.getElementById('scan-log-list');
                list.innerHTML = state.logs.slice(0, 50).map(l => `
                    <div class="bg-slate-800 p-3 rounded-lg border border-slate-700 flex justify-between items-center mb-2">
                        <div>
                            <div class="font-bold text-white text-sm">${l.id}</div>
                            <div class="text-[10px] text-slate-400 truncate w-40">${l.desc}</div>
                        </div>
                        <div class="text-right">
                            <div class="text-[10px] font-bold ${l.status==='SUCESSO'?'text-green-400':(l.status==='ERRO'?'text-red-400':'text-yellow-400')}">${l.status}</div>
                            <div class="text-[10px] text-slate-500">${new Date(l.time).toLocaleTimeString()}</div>
                        </div>
                    </div>
                `).join('');
            }

            function updateDashboard() {
                const total = state.idsToFind.size + state.foundIds.length;
                const pct = total > 0 ? Math.round((state.foundIds.length / total) * 100) : 0;
                
                // Update Chart
                const ctx = document.getElementById('progressChart');
                if (ctx) {
                    if(state.chart) state.chart.destroy();
                    state.chart = new Chart(ctx, {
                        type: 'doughnut',
                        data: { datasets: [{ data: [pct, 100-pct], backgroundColor: ['#0ea5e9', '#1e293b'], borderWidth: 0 }] },
                        options: { cutout: '80%', plugins: { tooltip: { enabled: false } } }
                    });
                }
                document.getElementById('progress-text').innerText = pct + "%";
                document.getElementById('kpi-total').innerText = state.logs.length;
                document.getElementById('kpi-error').innerText = state.logs.filter(l=>l.status==='ERRO' || l.status==='MISSORT').length;
            }

            // --- RELATÓRIOS ---
            function downloadExcel() {
                if(state.logs.length === 0) return alert("Sem dados");
                const ws = XLSX.utils.json_to_sheet(state.logs);
                const wb = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(wb, ws, "Logs");
                XLSX.writeFile(wb, "Relatorio_Natefy.xlsx");
            }
            
            async function downloadFlow() {
                 if(state.logs.length === 0) return alert("Sem dados");
                 const container = document.getElementById('flowchart-canvas-container');
                 // Simples visualização para print
                 container.innerHTML = `<div style="padding:50px; background:#0f172a; color:white; text-align:center">
                    <h1>Relatório Visual</h1>
                    <h2>${state.operator}</h2>
                    <div style="display:flex; justify-content:center; gap:20px; margin-top:50px">
                        <div style="border:2px solid green; padding:20px">SUCESSO: ${state.foundIds.length}</div>
                        <div style="border:2px solid red; padding:20px">ERROS: ${state.logs.filter(l=>l.status!=='SUCESSO').length}</div>
                    </div>
                 </div>`;
                 const canvas = await html2canvas(container);
                 const a = document.createElement('a'); a.href = canvas.toDataURL(); a.download = 'Fluxo.png'; a.click();
            }

            // --- SAVE/LOAD ---
            function saveState() {
                const s = { ...state, html5QrCode: null, chart: null };
                s.idsToFind = Array.from(s.idsToFind);
                s.idDescriptions = Array.from(s.idDescriptions.entries());
                s.inventoryZoneData = Array.from(s.inventoryZoneData.entries()).map(([k,v])=>[k,Array.from(v)]);
                localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(s));
            }
            function loadState() {
                const s = JSON.parse(localStorage.getItem(CONFIG.STORAGE_KEY));
                if(s) {
                    state = { ...state, ...s };
                    state.idsToFind = new Set(s.idsToFind);
                    state.idDescriptions = new Map(s.idDescriptions);
                    state.inventoryZoneData = new Map(s.inventoryZoneData.map(([k,v])=>[k,new Set(v)]));
                }
            }

            init();
        });
    </script>
</body>
</html>
