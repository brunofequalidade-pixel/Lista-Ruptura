<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lista de Produtos - Ruptura</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Fonte Inter para um visual mais limpo */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Ajuste para a tabela não estourar em telas pequenas */
        .table-container {
            overflow-x: auto;
        }
        /* Esconde as setas do input de pesquisa */
        input[type="search"]::-webkit-search-decoration,
        input[type="search"]::-webkit-search-cancel-button,
        input[type="search"]::-webkit-search-results-button,
        input[type="search"]::-webkit-search-results-decoration {
            -webkit-appearance: none;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen p-4 md:p-8">

    <div class="container mx-auto max-w-6xl">
        
        <header class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h1 class="text-3xl font-bold text-gray-800">Lista de Produtos (Ruptura)</h1>
            <p class="text-gray-600 mt-1">Consulte os produtos que não podem faltar.</p>
        </header>

        <details id="adminSection" class="bg-white p-6 rounded-lg shadow-lg mb-6 cursor-pointer">
            <summary class="font-semibold text-lg text-blue-700">
                Área de Atualização (Admin)
            </summary>
            <div class="mt-4 border-t pt-4">
                <label for="dataPasteArea" class="block text-sm font-medium text-gray-700 mb-2">
                    Cole os dados da planilha aqui:
                </label>
                <p class="text-xs text-gray-500 mb-2">
                    Instrução: Na aba "Listas Por Setor", copie **apenas** os dados (da célula B2 até a última da coluna E) e cole na área abaixo.
                </p>
                <textarea id="dataPasteArea" rows="10" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500" placeholder="Copie do Excel (Ctrl+C) e cole aqui (Ctrl+V)..."></textarea>
                <button id="saveButton" class="mt-4 w-full bg-blue-600 text-white font-bold py-3 px-5 rounded-lg hover:bg-blue-700 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
                    Salvar e Atualizar Lista na Nuvem
                </button>
                <div id="saveStatus" class="mt-2 text-center text-sm"></div>
            </div>
        </details>

        <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h2 class="text-xl font-semibold text-gray-700 mb-4">Filtros</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <label for="filterSetor" class="block text-sm font-medium text-gray-700">Filtrar por Setor</label>
                    <input type="text" id="filterSetor" placeholder="Digite o nome do setor..." class="mt-1 w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                </div>
                <div>
                    <label for="searchTerm" class="block text-sm font-medium text-gray-700">Pesquisar por Item ou Endereço</label>
                    <input type="search" id="searchTerm" placeholder="Digite o item ou endereço..." class="mt-1 w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                </div>
            </div>
        </div>

        <div class="bg-white rounded-lg shadow-lg">
            <div class="p-6">
                <h2 class="text-xl font-semibold text-gray-700">Produtos</h2>
                <p id="rowCount" class="text-sm text-gray-500 mt-1">Total de itens: 0</p>
            </div>
            <div class="table-container">
                <table class="w-full min-w-[600px] text-left">
                    <thead class="bg-gray-50 border-b border-gray-200">
                        <tr>
                            <th class="p-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">Setor</th>
                            <th class="p-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">Item</th>
                            <th class="p-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">Descrição</th>
                            <th class="p-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">Endereço</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-gray-200">
                        <tr>
                            <td colspan="4" class="p-6 text-center text-gray-500">
                                Carregando lista de produtos...
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </div>

    <script type="module">
        // Importações do Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import {
            getAuth,
            signInAnonymously,
            signInWithCustomToken,
            onAuthStateChanged
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import {
            getFirestore,
            doc,
            setDoc,
            onSnapshot,
            setLogLevel
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Configuração do Firebase (será injetada pelo ambiente)
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Variáveis globais
        let db, auth;
        let allProducts = []; // Armazena a lista completa vinda do Firebase
        let authReady = false; // Controle para saber se a autenticação foi concluída
        let listDocRef; 

        // Elementos da DOM
        const filterSetor = document.getElementById('filterSetor');
        const searchTerm = document.getElementById('searchTerm');
        const tableBody = document.getElementById('tableBody');
        const rowCount = document.getElementById('rowCount');
        const saveButton = document.getElementById('saveButton');
        const dataPasteArea = document.getElementById('dataPasteArea');
        const saveStatus = document.getElementById('saveStatus');
        // **** NOVA REFERÊNCIA DE ELEMENTO ****
        const adminSection = document.getElementById('adminSection');

        /**
         * Inicializa o Firebase e configura a autenticação
         */
        async function initFirebase() {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                listDocRef = doc(db, "artifacts", appId, "public/data", "productList", "mainList");

                // Habilita logs para depuração
                setLogLevel('debug');

                // Escuta mudanças no estado de autenticação
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        console.log("Usuário autenticado:", user.uid);
                        authReady = true;
                        loadProductList();
                    } else {
                        console.log("Nenhum usuário. Tentando login...");
                        authReady = false;
                        try {
                            if (initialAuthToken) {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } else {
                                await signInAnonymously(auth);
                            }
                        } catch (error) {
                            console.error("Erro ao autenticar:", error);
                            tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-red-500">Erro de autenticação. Não foi possível carregar os dados.</td></tr>`;
                        }
                    }
                });

            } catch (error) {
                console.error("Erro ao inicializar o Firebase:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-red-500">Erro ao conectar com o banco de dados.</td></tr>`;
            }
        }

        /**
         * Converte o texto colado (separado por tabulação) em um array de objetos.
         */
        function parsePastedData(text) {
            if (!text || text.trim() === "") {
                return [];
            }
            
            const lines = text.trim().split('\n');
            return lines
                .filter(line => line.trim() !== "") // Ignora linhas em branco
                .map(line => {
                    const parts = line.split('\t'); // Excel cola com TABS
                    return {
                        setor: parts[0] || '',
                        item: parts[1] || '',
                        descricao: parts[2] || '',
                        endereco: parts[3] || ''
                    };
                });
        }

        /**
         * Carrega a lista de produtos do Firebase em tempo real (onSnapshot).
         */
        function loadProductList() {
            if (!authReady) {
                console.warn("Aguardando autenticação para carregar a lista...");
                return;
            }

            console.log("Tentando carregar lista do Firestore...");
            
            onSnapshot(listDocRef, (docSnap) => {
                if (docSnap.exists()) {
                    console.log("Dados recebidos do Firestore.");
                    const data = docSnap.data();
                    allProducts = parsePastedData(data.rawProductData || "");
                    console.log(`Lista carregada com ${allProducts.length} produtos.`);
                } else {
                    console.log("Nenhum documento encontrado. A lista está vazia.");
                    allProducts = [];
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-gray-500">A lista de produtos está vazia. Peça ao administrador para carregar os dados.</td></tr>`;
                }
                renderTable();
            }, (error) => {
                console.error("Erro ao carregar lista do Firestore:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-red-500">Erro ao carregar a lista. Verifique sua conexão.</td></tr>`;
            });
        }

        /**
         * Renderiza a tabela com base nos filtros e na lista 'allProducts'.
         */
        function renderTable() {
            const setorFilter = filterSetor.value.toLowerCase();
            const searchFilter = searchTerm.value.toLowerCase();

            const filteredProducts = allProducts.filter(product => {
                const pSetor = product.setor.toLowerCase();
                const pItem = product.item.toLowerCase();
                const pDesc = product.descricao.toLowerCase();
                const pEnd = product.endereco.toLowerCase();
                const matchSetor = !setorFilter || pSetor.includes(setorFilter);
                const matchSearch = !searchFilter || pItem.includes(searchFilter) || pEnd.includes(searchFilter);
                return matchSetor && matchSearch;
            });

            tableBody.innerHTML = '';
            rowCount.textContent = `Total de itens: ${filteredProducts.length}`;

            if (filteredProducts.length === 0) {
                if(allProducts.length > 0) {
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-gray-500">Nenhum produto encontrado com esses filtros.</td></tr>`;
                } else if (!authReady) {
                     tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-gray-500">Conectando...</td></tr>`;
                } else {
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-gray-500">A lista de produtos está vazia.</td></tr>`;
                }
            } else {
                filteredProducts.forEach(product => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50';
                    
                    const cellSetor = document.createElement('td');
                    cellSetor.className = 'p-4 text-sm text-gray-700';
                    cellSetor.textContent = product.setor;
                    row.appendChild(cellSetor);

                    const cellItem = document.createElement('td');
                    cellItem.className = 'p-4 text-sm text-gray-900 font-medium';
                    cellItem.textContent = product.item;
                    row.appendChild(cellItem);

                    const cellDesc = document.createElement('td');
                    cellDesc.className = 'p-4 text-sm text-gray-700';
                    cellDesc.textContent = product.descricao;
                    row.appendChild(cellDesc);

                    const cellEnd = document.createElement('td');
                    cellEnd.className = 'p-4 text-sm text-gray-700';
                    cellEnd.textContent = product.endereco;
                    row.appendChild(cellEnd);

                    tableBody.appendChild(row);
                });
            }
        }

        /**
         * Salva o texto bruto da área de texto no Firestore.
         */
        async function saveListToFirebase() {
            if (!authReady) {
                saveStatus.textContent = "Erro: Ainda não conectado. Tente novamente em alguns segundos.";
                saveStatus.className = "mt-2 text-center text-sm text-red-600";
                return;
            }

            const rawText = dataPasteArea.value;
            if (rawText.trim() === "") {
                saveStatus.textContent = "A área de texto está vazia. Cole os dados primeiro.";
                saveStatus.className = "mt-2 text-center text-sm text-yellow-600";
                return;
            }

            saveButton.disabled = true;
            saveStatus.textContent = "Salvando na nuvem...";
            saveStatus.className = "mt-2 text-center text-sm text-blue-600";

            try {
                await setDoc(listDocRef, { rawProductData: rawText });
                
                saveStatus.textContent = "Lista atualizada com sucesso!";
                saveStatus.className = "mt-2 text-center text-sm text-green-600";
                
                dataPasteArea.value = '';

            } catch (error) {
                console.error("Erro ao salvar no Firestore:", error);
                saveStatus.textContent = "Erro ao salvar. Tente novamente.";
                saveStatus.className = "mt-2 text-center text-sm text-red-600";
            } finally {
                saveButton.disabled = false;
                setTimeout(() => { saveStatus.textContent = ''; }, 4000);
            }
        }

        // Adiciona os event listeners para os filtros
        filterSetor.addEventListener('input', renderTable);
        searchTerm.addEventListener('input', renderTable);

        // Adiciona o event listener para o botão de salvar
        saveButton.addEventListener('click', saveListToFirebase);

        // **** CÓDIGO NOVO ADICIONADO PARA A SENHA ****
        adminSection.addEventListener('toggle', function(event) {
            // Se a intenção é ABRIR a seção (o 'open' ainda não foi totalmente setado)
            if (!this.hasAttribute('open')) {
                // Previne a abertura padrão para podermos validar a senha
                event.preventDefault();

                // Pede a senha ao usuário
                const password = prompt("Por favor, digite a senha de administrador:");

                // Verifica se a senha está correta
                if (password === "brunofe") {
                    // Se estiver correta, abre a seção programaticamente
                    this.open = true;
                } else {
                     // Se a senha estiver errada (e o usuário não clicou em 'cancelar')
                    if (password !== null) {
                        alert("Senha incorreta. Acesso negado.");
                    }
                    // Garante que a seção permaneça fechada
                    this.open = false;
                }
            }
            // Se a intenção é FECHAR, não fazemos nada e deixamos o comportamento padrão ocorrer
        });

        // Inicia o aplicativo
        initFirebase();

    </script>
</body>
</html>
