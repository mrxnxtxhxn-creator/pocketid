<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Scanner de ID</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Usando a versão mais recente da biblioteca de scanner -->
    <script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@zxing/browser@latest/umd/zxing-browser.min.js"></script>
    <style>
        :root {
            --scan-window-size: 80vw;
            --max-scan-window-size: 400px;
            --corner-size: 30px;
            --corner-thickness: 4px;
            --corner-color: #ffffff;
        }

        body, html {
            overscroll-behavior: contain;
            margin: 0;
            padding: 0;
            height: 100%;
            width: 100%;
            overflow: hidden;
            background: #000;
        }

        #scanner-container {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
        }

        #video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        /* Camada de sobreposição que cria a área escura em volta da janela de scanner */
        #scanner-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        /* A janela de scanner */
        #scan-window {
            position: relative;
            width: var(--scan-window-size);
            height: calc(var(--scan-window-size) * 0.6); /* Formato retangular */
            max-width: var(--max-scan-window-size);
            max-height: calc(var(--max-scan-window-size) * 0.6);
            /* Sombra que cria o efeito de "buraco" */
            box-shadow: 0 0 0 4000px rgba(0, 0, 0, 0.5);
            overflow: hidden; /* Para a animação da linha não sair da janela */
        }
        
        /* Cantos da janela de scanner */
        .corner {
            position: absolute;
            width: var(--corner-size);
            height: var(--corner-size);
            border: var(--corner-thickness) solid var(--corner-color);
        }
        .corner.top-left { top: 0; left: 0; border-right: none; border-bottom: none; }
        .corner.top-right { top: 0; right: 0; border-left: none; border-bottom: none; }
        .corner.bottom-left { bottom: 0; left: 0; border-right: none; border-top: none; }
        .corner.bottom-right { bottom: 0; right: 0; border-left: none; border-top: none; }

        /* Linha de scanner animada */
        .scan-line {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 2px;
            background: linear-gradient(90deg, transparent, #33ff00, transparent);
            box-shadow: 0 0 10px #33ff00;
            animation: scan-animation 2.5s infinite linear;
        }

        @keyframes scan-animation {
            0% { top: 0; }
            50% { top: 100%; }
            100% { top: 0; }
        }

        /* Texto de instrução */
        .scan-instruction {
            margin-top: 20px;
            color: #fff;
            font-size: 1rem;
            text-shadow: 0 1px 2px rgba(0,0,0,0.5);
        }

        /* Painel de controle na parte inferior */
        #controls-panel {
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            background: rgba(0, 0, 0, 0.75);
            backdrop-filter: blur(5px);
            border-top-left-radius: 20px;
            border-top-right-radius: 20px;
            padding: 1.5rem;
        }
        
        /* Notificação de status que aparece no centro */
        #status-notification {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 1rem 2rem;
            border-radius: 1rem;
            font-size: 1.25rem;
            font-weight: bold;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
            text-align: center;
            z-index: 100;
        }
        #status-notification.visible {
            opacity: 1;
        }

        /* Camada para o feedback visual de sucesso/erro */
        #feedback-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            transition: background-color 0.3s ease;
            opacity: 0;
            pointer-events: none;
        }
        #feedback-overlay.active {
            opacity: 0.6;
        }
    </style>
</head>
<body>

    <div id="scanner-container">
        <video id="video" playsinline></video>
        
        <div id="scanner-overlay">
            <div id="scan-window">
                <div class="corner top-left"></div>
                <div class="corner top-right"></div>
                <div class="corner bottom-left"></div>
                <div class="corner bottom-right"></div>
                <div class="scan-line"></div>
            </div>
            <p class="scan-instruction">Posicione o código de barras na janela</p>
        </div>

        <div id="feedback-overlay"></div>
        <div id="status-notification"></div>

        <div id="controls-panel">
            <div class="w-full max-w-lg mx-auto">
                <label for="id-list-input" class="block text-sm font-medium text-gray-200 mb-2 text-center">Insira os IDs para procurar (separados por vírgula):</label>
                <textarea id="id-list-input" rows="2" class="w-full p-3 border border-gray-600 rounded-lg bg-gray-800 text-white focus:ring-2 focus:ring-blue-500" placeholder="Ex: 987654, 123456"></textarea>
            </div>
        </div>
    </div>

    <script>
        const video = document.getElementById('video');
        const idListInput = document.getElementById('id-list-input');
        const feedbackOverlay = document.getElementById('feedback-overlay');
        const statusNotification = document.getElementById('status-notification');
        let validIds = [];
        let lastScanTime = 0;
        const scanDelay = 3000; // 3 segundos de espera

        function updateIdList() {
            const idsText = idListInput.value.trim();
            validIds = idsText ? idsText.split(',').map(id => id.trim()).filter(id => id) : [];
        }

        idListInput.addEventListener('input', updateIdList);

        function showFeedback(status, message) {
            feedbackOverlay.style.backgroundColor = status === 'success' ? '#2ecc71' : '#e74c3c';
            feedbackOverlay.classList.add('active');

            statusNotification.textContent = message;
            statusNotification.classList.add('visible');
            
            if (status === 'success' && navigator.vibrate) {
                navigator.vibrate(200);
            }

            setTimeout(() => {
                feedbackOverlay.classList.remove('active');
                statusNotification.classList.remove('visible');
            }, scanDelay - 500);
        }

        async function startScanner() {
            const hints = new Map();
            const formats = [
                ZXingBrowser.BarcodeFormat.CODE_128,
                ZXingBrowser.BarcodeFormat.EAN_13,
                ZXingBrowser.BarcodeFormat.CODE_39,
                ZXingBrowser.BarcodeFormat.QR_CODE
            ];
            hints.set(2, formats); // Enum for POSSIBLE_FORMATS
            const codeReader = new ZXingBrowser.BrowserMultiFormatReader(hints);

            try {
                codeReader.decodeFromVideoDevice(undefined, 'video', (result, err) => {
                    if (result) {
                        const currentTime = Date.now();
                        if (currentTime - lastScanTime < scanDelay) {
                           return;
                        }
                        lastScanTime = currentTime;

                        const scannedId = result.getText();
                        console.log('Código lido:', scannedId);

                        if (validIds.includes(scannedId)) {
                            showFeedback('success', `ID ENCONTRADO!\n${scannedId}`);
                        } else {
                            showFeedback('error', `NÃO ENCONTRADO\n${scannedId}`);
                        }
                    }

                    if (err && err.name !== 'NotFoundException') {
                        console.error("Erro de leitura:", err);
                    }
                });

            } catch (err) {
                console.error("Erro ao iniciar a câmera: ", err);
                statusNotification.textContent = "Erro ao iniciar a câmera.";
                statusNotification.classList.add('visible');
            }
        }

        window.addEventListener('load', () => {
            updateIdList();
            startScanner();
        });
    </script>
</body>
</html>


