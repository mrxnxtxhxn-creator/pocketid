<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Scanner de ID Rápido</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <style>
        body, html {
            margin: 0; padding: 0;
            height: 100%; width: 100%;
            overflow: hidden;
            background: #000;
        }
        #reader {
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
        }
        #reader video {
            width: 100% !important;
            height: 100% !important;
            object-fit: cover !important;
        }
        #reader__scan_region {
            border: 4px solid rgba(255, 255, 255, 0.7) !important;
            border-radius: 16px;
            background: none !important;
        }
        .scan-line {
            position: absolute;
            left: 10%;
            top: 0;
            width: 80%;
            height: 3px;
            background: #e74c3c;
            box-shadow: 0 0 10px #e74c3c, 0 0 20px #e74c3c;
            border-radius: 3px;
            animation: scan-animation 3s infinite cubic-bezier(0.5, 0, 0.5, 1);
        }
        @keyframes scan-animation {
            0% { transform: translateY(10px); }
            50% { transform: translateY(calc(100% - 20px)); }
            100% { transform: translateY(10px); }
        }
        #controls-panel {
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            z-index: 10;
            background: rgba(0, 0, 0, 0.8);
            backdrop-filter: blur(8px);
            border-top-left-radius: 24px;
            border-top-right-radius: 24px;
            padding: 1.5rem;
        }
        #feedback-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.20s ease-in-out; /* Transição mais rápida */
            display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            font-size: 1.8rem; color: white; font-weight: bold;
            text-align: center; text-shadow: 0 0 12px rgba(0,0,0,0.8);
            z-index: 100;
        }
    </style>
</head>
<body>
    <div id="reader"></div>
    <div id="feedback-overlay"></div>
    <div id="controls-panel">
        <div class="w-full max-w-lg mx-auto">
            <label for="id-list-input" class="block text-sm font-medium text-gray-200 mb-2 text-center">IDs para procurar (separados por vírgula):</label>
            <textarea id="id-list-input" rows="2" class="w-full p-3 border border-gray-600 rounded-lg bg-gray-700 text-white focus:ring-2 focus:ring-blue-500" placeholder="Ex: 8949461894921, 123456789"></textarea>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const idListInput = document.getElementById('id-list-input');
            const feedbackOverlay = document.getElementById('feedback-overlay');
            
            let validIds = new Set();
            let isPaused = false;
            // --- MODIFICAÇÃO: Tempo de feedback reduzido para 1 segundo ---
            const SCAN_DELAY = 1000;

            function updateIdList() {
                const idsText = idListInput.value.trim();
                validIds = new Set(idsText ? idsText.split(',').map(id => id.trim()).filter(id => id) : []);
            }
            idListInput.addEventListener('input', updateIdList);
            updateIdList(); 

            function showFeedback(status, scannedId) {
                isPaused = true;
                feedbackOverlay.style.backgroundColor = status === 'success' ? 'rgba(46, 204, 113, 0.9)' : 'rgba(231, 76, 60, 0.9)';
                const message = status === 'success' ? 'ENCONTRADO' : 'NÃO ENCONTRADO';
                feedbackOverlay.innerHTML = `<div style="font-size: 2.5rem;">${message}</div><div style="font-size: 1.2rem; margin-top: 8px;">${scannedId}</div>`;
                feedbackOverlay.style.opacity = '1';
                
                // --- MODIFICAÇÃO: Vibração de 200ms em caso de sucesso ---
                if (status === 'success' && navigator.vibrate) {
                    navigator.vibrate(200);
                }

                setTimeout(() => {
                    feedbackOverlay.style.opacity = '0';
                    isPaused = false;
                }, SCAN_DELAY);
            }

            const html5QrCode = new Html5Qrcode("reader");

            const onScanSuccess = (decodedText, decodedResult) => {
                if (isPaused) {
                    return;
                }
                const scannedId = decodedText;
                if (validIds.has(scannedId)) {
                    showFeedback('success', scannedId);
                } else {
                    showFeedback('error', scannedId);
                }
            };

            const onScanFailure = (error) => {
                // Não faz nada
            };

            const config = {
                fps: 10,
                qrbox: (viewfinderWidth, viewfinderHeight) => {
                    const minEdge = Math.min(viewfinderWidth, viewfinderHeight);
                    const qrboxSize = Math.floor(minEdge * 0.7);
                    return {
                        width: qrboxSize,
                        height: qrboxSize * 0.6
                    };
                },
                supportedScanTypes: [Html5QrcodeScanType.SCAN_TYPE_CAMERA],
                formatsToSupport: [ 
                    Html5QrcodeSupportedFormats.CODE_128,
                    Html5QrcodeSupportedFormats.EAN_13
                ]
            };

            html5QrCode.start({ facingMode: "environment" }, config, onScanSuccess, onScanFailure)
                .catch(err => {
                    console.error("Não foi possível iniciar o leitor de QR code.", err);
                    feedbackOverlay.innerHTML = "Erro ao iniciar a câmera. Verifique as permissões.";
                    feedbackOverlay.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
                    feedbackOverlay.style.opacity = '1';
                });
        });
    </script>
</body>
</html>
