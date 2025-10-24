<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lista de Produtos - Ruptura</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        .table-container {
            overflow-x: auto;
        }
        input[type="search"]::-webkit-search-decoration,
        input[type="search"]::-webkit-search-cancel-button,
        input[type="search"]::-webkit-search-results-button,
        input[type="search"]::-webkit-search-results-decoration {
            -webkit-appearance: none;
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">

    <div class="container mx-auto max-w-6xl px-3 py-4">
        
        <!-- Área de Atualização (Admin) -->
        <details id="adminSection" class="bg-white p-4 rounded-lg shadow mb-4 cursor-pointer">
            <summary class="font-semibold text-base text-blue-700">
                Área de Atualização (Admin)
            </summary>
            <div class="mt-3 border-t pt-3">
                <label for="dataPasteArea" class="block text-sm font-medium text-gray-700 mb-2">
                    Cole os dados da planilha aqui:
                </label>
                <p class="text-xs text-gray-500 mb-2">
                    Instrução: Na aba "Listas Por Setor", copie **apenas** os dados (da célula B2 até a última da coluna E) e cole na área abaixo.
                </p>
                <textarea id="dataPasteArea" rows="8" class="w-full p-3 text-sm border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500" placeholder="Copie do Excel (Ctrl+C) e cole aqui (Ctrl+V)..."></textarea>
                <button id="saveButton" class="mt-3 w-full bg-blue-600 text-white font-semibold py-3 px-4 text-sm rounded-lg hover:bg-blue-700 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500">
                    Salvar e Atualizar Lista
                </button>
                <div id="saveStatus" class="mt-2 text-center text-sm"></div>
            </div>
        </details>

        <!-- Seção de Pesquisa -->
        <div class="bg-white p-4 rounded-lg shadow mb-4">
            <label for="searchTerm" class="block text-sm font-semibold text-gray-700 mb-2">Pesquisar</label>
            <input type="search" id="searchTerm" placeholder="Digite o item ou endereço..." class="w-full p-3 text-base border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
        </div>

        <!-- Tabela de Produtos -->
        <div class="bg-white rounded-lg shadow">
            <div class="p-4 border-b border-gray-200">
                <h2 class="text-lg font-semibold text-gray-800">
                    <span id="lastUpdateDate" class="text-blue-600"></span>Lista-Ruptura
                </h2>
                <p id="rowCount" class="text-sm text-gray-500 mt-0.5">Total de itens: 0</p>
            </div>
            <div class="table-container">
                <table class="w-full text-left text-sm">
                    <thead class="bg-gray-50 border-b border-gray-200">
                        <tr>
                            <th class="p-3 text-xs font-semibold text-gray-600 uppercase">Setor</th>
                            <th class="p-3 text-xs font-semibold text-gray-600 uppercase">Item</th>
                            <th class="p-3 text-xs font-semibold text-gray-600 uppercase">Descrição</th>
                            <th class="p-3 text-xs font-semibold text-gray-600 uppercase">Endereço</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-gray-200">
                        <tr>
                            <td colspan="4" class="p-4 text-center text-gray-500 text-sm">
                                Carregando lista de produtos...
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </div>

    <!-- Firebase -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import {
            getAuth,
            signInAnonymously,
            onAuthStateChanged
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import {
            getFirestore,
            doc,
            setDoc,
            onSnapshot,
            setLogLevel
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyBo8G3ZcWk4EepN0cHdVBtXc7tGOfcw-yg",
            authDomain: "inscricaosinuca.firebaseapp.com",
            projectId: "inscricaosinuca",
            storageBucket: "inscricaosinuca.firebasestorage.app",
            messagingSenderId: "338241576305",
            appId: "1:338241576305:web:288b6124384c6be4f76ad0",
            measurementId: "G-PEDG30FS2R"
        };
        
        const appId = firebaseConfig.projectId || 'default-app-id';

        let db, auth;
        let allProducts = [];
        let authReady = false;
        let listDocRef; 

        const searchTerm = document.getElementById('searchTerm');
        const tableBody = document.getElementById('tableBody');
        const rowCount = document.getElementById('rowCount');
        const saveButton = document.getElementById('saveButton');
        const dataPasteArea = document.getElementById('dataPasteArea');
        const saveStatus = document.getElementById('saveStatus');
        const adminSection = document.getElementById('adminSection');
        const lastUpdateDate = document.getElementById('lastUpdateDate');

        async function initFirebase() {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                listDocRef = doc(db, "artifacts", appId, "public/data", "productList", "mainList");

                setLogLevel('debug');

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        console.log("Usuário autenticado:", user.uid);
                        authReady = true;
                        loadProductList();
                    } else {
                        console.log("Nenhum usuário. Tentando login anônimo...");
                        authReady = false;
                        try {
                            await signInAnonymously(auth);
                        } catch (error) {
                            console.error("Erro ao autenticar anonimamente:", error);
                            tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-red-500 text-sm">Erro de autenticação. Não foi possível carregar os dados.</td></tr>`;
                        }
                    }
                });

            } catch (error) {
                console.error("Erro ao inicializar o Firebase:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-red-500 text-sm">Erro ao conectar com o banco de dados.</td></tr>`;
            }
        }

        function parsePastedData(text) {
            if (!text || text.trim() === "") {
                return [];
            }
            
            const lines = text.trim().split('\n');
            return lines
                .filter(line => line.trim() !== "")
                .map(line => {
                    const parts = line.split('\t');
                    return {
                        setor: parts[0] || '',
                        item: parts[1] || '',
                        descricao: parts[2] || '',
                        endereco: parts[3] || ''
                    };
                });
        }

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
                    
                    // Atualiza a data de atualização
                    if (data.lastUpdate) {
                        const date = new Date(data.lastUpdate);
                        const formattedDate = date.toLocaleDateString('pt-BR');
                        lastUpdateDate.textContent = formattedDate + ' - ';
                    } else {
                        lastUpdateDate.textContent = '';
                    }
                } else {
                    console.log("Nenhum documento encontrado. A lista está vazia.");
                    allProducts = [];
                    lastUpdateDate.textContent = '';
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-500 text-sm">A lista de produtos está vazia. Peça ao administrador para carregar os dados.</td></tr>`;
                }
                renderTable();
            }, (error) => {
                console.error("Erro ao carregar lista do Firestore:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-red-500 text-sm">Erro ao carregar a lista. Verifique sua conexão.</td></tr>`;
            });
        }

        function renderTable() {
            const searchFilter = searchTerm.value.toLowerCase();

            const filteredProducts = allProducts.filter(product => {
                const pItem = product.item.toLowerCase();
                const pEnd = product.endereco.toLowerCase();
                const matchSearch = !searchFilter || pItem.includes(searchFilter) || pEnd.includes(searchFilter);
                return matchSearch;
            });

            tableBody.innerHTML = '';
            rowCount.textContent = `Total de itens: ${filteredProducts.length}`;

            if (filteredProducts.length === 0) {
                if(allProducts.length > 0) {
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-500 text-sm">Nenhum produto encontrado com esses filtros.</td></tr>`;
                } else if (!authReady) {
                     tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-500 text-sm">Conectando...</td></tr>`;
                } else {
                    tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-500 text-sm">A lista de produtos está vazia.</td></tr>`;
                }
            } else {
                filteredProducts.forEach(product => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50';
                    
                    const cellSetor = document.createElement('td');
                    cellSetor.className = 'p-3 text-sm text-gray-700';
                    cellSetor.textContent = product.setor;
                    row.appendChild(cellSetor);

                    const cellItem = document.createElement('td');
                    cellItem.className = 'p-3 text-sm text-gray-900 font-medium';
                    cellItem.textContent = product.item;
                    row.appendChild(cellItem);

                    const cellDesc = document.createElement('td');
                    cellDesc.className = 'p-3 text-sm text-gray-700';
                    cellDesc.textContent = product.descricao;
                    row.appendChild(cellDesc);

                    const cellEnd = document.createElement('td');
                    cellEnd.className = 'p-3 text-sm text-gray-700';
                    cellEnd.textContent = product.endereco;
                    row.appendChild(cellEnd);

                    tableBody.appendChild(row);
                });
            }
        }

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
                await setDoc(listDocRef, { 
                    rawProductData: rawText,
                    lastUpdate: new Date().toISOString()
                });
                
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

        searchTerm.addEventListener('input', renderTable);
        saveButton.addEventListener('click', saveListToFirebase);

        adminSection.addEventListener('toggle', function(event) {
            if (this.open) {
                event.preventDefault();
                const password = prompt("Por favor, digite a senha de administrador:");
                if (password === "brunofe") {
                    this.setAttribute('data-authenticated', 'true');
                    this.open = true;
                } else {
                    if (password !== null) {
                        alert("Senha incorreta. Acesso negado.");
                    }
                    this.open = false;
                }
            }
        });

        initFirebase();

    </script>
</body>
</html>
