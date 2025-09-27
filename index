<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner de ID</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Incluindo a biblioteca Tesseract.js para OCR (Reconhecimento Óptico de Caracteres) -->
    <script src='https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js'></script>
    <style>
        /* Estilos personalizados */
        body {
            transition: background-color 0.5s ease;
        }
        .status-found {
            background-color: #2ecc71 !important; /* Verde */
            color: white;
        }
        .status-not-found {
            background-color: #e74c3c !important; /* Vermelho */
            color: white;
        }
        #notification {
            transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-100 flex flex-col items-center justify-center min-h-screen font-sans p-4">

    <div class="w-full max-w-2xl bg-white rounded-lg shadow-xl p-6 space-y-6">
        <h1 class="text-3xl font-bold text-center text-gray-800">Scanner de ID</h1>

        <!-- Área de entrada para a lista de IDs -->
        <div class="space-y-2">
            <label for="id-list-input" class="block text-lg font-medium text-gray-700">Insira os IDs para procurar (separados por vírgula):</label>
            <textarea id="id-list-input" rows="3" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500" placeholder="Ex: 12345, 67890, 54321"></textarea>
        </div>

        <!-- Visualização da câmera -->
        <div class="bg-gray-900 rounded-lg overflow-hidden relative aspect-video">
            <video id="video" class="w-full h-full" playsinline autoplay muted></video>
            <div id="loading-message" class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50 text-white text-lg hidden">
                <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                Analisando...
            </div>
        </div>
        <canvas id="canvas" class="hidden"></canvas> <!-- Canvas oculto para processamento de imagem -->

        <!-- Botão de ação -->
        <button id="scan-button" class="w-full bg-blue-600 text-white font-bold py-4 px-6 rounded-lg text-lg hover:bg-blue-700 focus:outline-none focus:ring-4 focus:ring-blue-300 transition-transform transform hover:scale-105 disabled:bg-gray-400 disabled:cursor-not-allowed">
            Analisar Câmera
        </button>

        <!-- Lista de IDs e seus status -->
        <div>
            <h2 class="text-xl font-semibold text-gray-700 mb-2">Lista de IDs:</h2>
            <ul id="id-list" class="space-y-2">
                <li class="text-gray-500">Nenhum ID inserido ainda.</li>
            </ul>
        </div>
    </div>

    <!-- Notificação flutuante -->
    <div id="notification" class="fixed top-5 bg-gray-800 text-white py-3 px-6 rounded-lg shadow-lg opacity-0 transform -translate-y-10">
        <p id="notification-text"></p>
    </div>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const scanButton = document.getElementById('scan-button');
        const idListInput = document.getElementById('id-list-input');
        const idListElement = document.getElementById('id-list');
        const notification = document.getElementById('notification');
        const notificationText = document.getElementById('notification-text');
        const loadingMessage = document.getElementById('loading-message');
        let validIds = [];

        // Função para mostrar notificação
        function showNotification(message, isSuccess) {
            notificationText.textContent = message;
            notification.classList.remove('status-found', 'status-not-found');
            notification.classList.add(isSuccess ? 'status-found' : 'status-not-found');
            notification.classList.remove('opacity-0', '-translate-y-10');
            
            setTimeout(() => {
                notification.classList.add('opacity-0', '-translate-y-10');
            }, 3000);
        }

        // Função para atualizar a lista de IDs na tela
        function updateIdList() {
            const idsText = idListInput.value.trim();
            validIds = idsText ? idsText.split(',').map(id => id.trim()).filter(id => id) : [];
            
            idListElement.innerHTML = ''; // Limpa a lista atual
            if (validIds.length > 0) {
                validIds.forEach(id => {
                    const li = document.createElement('li');
                    li.textContent = id;
                    li.id = `id-${id}`;
                    li.className = 'p-3 bg-gray-200 rounded-lg text-gray-800 transition-colors duration-500';
                    idListElement.appendChild(li);
                });
            } else {
                idListElement.innerHTML = '<li class="text-gray-500">Nenhum ID inserido ainda.</li>';
            }
        }

        idListInput.addEventListener('input', updateIdList);

        // Iniciar a câmera
        async function startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({
                    video: {
                        facingMode: 'environment' // Usa a câmera traseira
                    }
                });
                video.srcObject = stream;
                await video.play();
            } catch (err) {
                console.error("Erro ao acessar a câmera: ", err);
                alert("Não foi possível acessar a câmera. Verifique as permissões no seu navegador.");
            }
        }

        // Função principal de escaneamento
        async function scanId() {
            if (validIds.length === 0) {
                showNotification('Por favor, insira pelo menos um ID na lista.', false);
                return;
            }

            scanButton.disabled = true;
            loadingMessage.classList.remove('hidden');
            document.body.style.backgroundColor = '';
            
            // Redefinir cores da lista
            validIds.forEach(id => {
                const li = document.getElementById(`id-${id}`);
                if (li) {
                    li.classList.remove('status-found');
                }
            });

            // Captura um quadro do vídeo para o canvas
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            // Usa Tesseract para reconhecer texto na imagem
            try {
                const { data: { text } } = await Tesseract.recognize(canvas, 'eng');
                const cleanedText = text.replace(/\s/g, ''); // Remove espaços para facilitar a correspondência

                let foundId = null;
                for (const id of validIds) {
                    if (cleanedText.includes(id)) {
                        foundId = id;
                        break;
                    }
                }

                if (foundId) {
                    document.body.style.backgroundColor = '#2ecc71'; // Verde
                    showNotification(`ID encontrado: ${foundId}`, true);
                    const li = document.getElementById(`id-${foundId}`);
                    if (li) {
                        li.classList.add('status-found');
                    }
                } else {
                    document.body.style.backgroundColor = '#e74c3c'; // Vermelho
                    showNotification('Nenhum ID correspondente encontrado.', false);
                }

            } catch (error) {
                console.error('Erro no OCR:', error);
                showNotification('Ocorreu um erro durante a análise.', false);
            } finally {
                scanButton.disabled = false;
                loadingMessage.classList.add('hidden');
                // Reseta a cor de fundo após alguns segundos
                setTimeout(() => {
                    document.body.style.backgroundColor = '';
                }, 1500);
            }
        }

        scanButton.addEventListener('click', scanId);

        // Inicializa o app
        window.addEventListener('load', () => {
            startCamera();
            updateIdList();
        });
    </script>
</body>
</html>
