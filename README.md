
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lista Compacta de Produtos</title>
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
<body class="bg-gray-100 min-h-screen p-2">

        <div class="container mx-auto max-w-5xl">
        
                <header class="bg-white p-3 rounded-md shadow-sm mb-3">
                        <h1 class="text-sm sm:text-base font-bold text-gray-800">Lista de Produtos (Ruptura)</h1>
                        <p class="text-gray-600 mt-1 text-xs">Consulte os produtos que não podem faltar.</p>
        </header>

                        <details id="adminSection" class="bg-white p-3 rounded-md shadow-sm mb-3 cursor-pointer">
                        <summary class="font-semibold text-base text-blue-700">
                Área de Atualização (Admin)
            </summary>
            <div class="mt-2 border-t pt-2">
                                <label for="dataPasteArea" class="block text-xs font-medium text-gray-700 mb-1">
                    Cole os dados da planilha aqui:
                </label>
                                <p class="text-xs text-gray-500 mb-1">
                    Instrução: Na aba "Listas Por Setor", copie **apenas** os dados (da célula B2 até a última da coluna E) e cole na área abaixo.
                </p>
                                <textarea id="dataPasteArea" rows="5" class="w-full p-2 text-xs border border-gray-300 rounded-lg focus:ring-1 focus:ring-blue-500" placeholder="Copie do Excel (Ctrl+C) e cole aqui (Ctrl+V)..."></textarea>
                                <button id="saveButton" class="mt-2 w-full bg-blue-600 text-white font-bold py-2 px-3 rounded-lg text-sm hover:bg-blue-700 transition-colors focus:outline-none focus:ring-1 focus:ring-blue-500 focus:ring-offset-1">
                    Salvar e Atualizar Lista na Nuvem
                </button>
                                <div id="saveStatus" class="mt-1 text-center text-xs"></div>
            </div>
        </details>

                        <div class="bg-white p-3 rounded-md shadow-sm mb-3">
                        <h2 class="text-sm sm:text-base font-semibold text-gray-700 mb-3">Filtros</h2>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                <div>
                                        <label for="filterSetor" class="block text-xs font-medium text-gray-700">Filtrar por Setor</label>
                                        <select id="filterSetor" class="mt-1 w-full py-1 px-2 text-sm border border-gray-300 rounded-lg focus:ring-1 focus:ring-blue-500 bg-white">
                        <option value="">Todos os Setores</option>
                                            </select>
                </div>
                <div>
                                        <label for="searchTerm" class="block text-xs font-medium text-gray-700">Pesquisar por Item ou Endereço</label>
                                        <input type="search" id="searchTerm" placeholder="Digite o item ou endereço..." class="mt-1 w-full py-1 px-2 text-sm border border-gray-300 rounded-lg focus:ring-1 focus:ring-blue-500">
                </div>
            </div>
        </div>

                <div class="bg-white rounded-md shadow-sm">
                        <div class="p-3">
                                <h2 class="text-sm sm:text-base font-semibold text-gray-700">Produtos</h2>
                                <p id="rowCount" class="text-xs text-gray-500 mt-1">Total de itens: 0</p>
            </div>
            <div class="table-container">
                                <table class="w-full min-w-[500px] text-left">
                    <thead class="bg-gray-50 border-b border-gray-200">
                        <tr>
                                                        <th class="py-1.5 px-3 text-xs font-semibold text-gray-600 uppercase tracking-wider">Setor</th>
                            <th class="py-1.5 px-3 text-xs font-semibold text-gray-600 uppercase tracking-wider">Item</th>
                            <th class="py-1.5 px-3 text-xs font-semibold text-gray-600 uppercase tracking-wider">Descrição</th>
                            <th class="py-1.5 px-3 text-xs font-semibold text-gray-600 uppercase tracking-wider">Endereço</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-gray-200">
                                                <tr>
                                                        <td colspan="4" class="py-3 px-6 text-center text-gray-500 text-xs">
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
            onAuthStateChanged
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import {
            getFirestore,
            doc,
            setDoc,
            onSnapshot,
            setLogLevel
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Config Firebase (fornecida pelo usuário)
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

        // Variáveis globais
        let db, auth;
        let allProducts = []; 
        let authReady = false; 
        let listDocRef; 

        // Elementos da DOM
        const filterSetor = document.getElementById('filterSetor');
        const searchTerm = document.getElementById('searchTerm');
        const tableBody = document.getElementById('tableBody');
        const rowCount = document.getElementById('rowCount');
        const saveButton = document.getElementById('saveButton');
        const dataPasteArea = document.getElementById('dataPasteArea');
        const saveStatus = document.getElementById('saveStatus');
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
                            tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-red-500 text-xs">Erro de autenticação. Não foi possível carregar os dados.</td></tr>`;
                        }
                    }
                });

            } catch (error) {
                console.error("Erro ao inicializar o Firebase:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-red-500 text-xs">Erro ao conectar com o banco de dados.</td></tr>`;
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
                .filter(line => line.trim() !== "") 
                .map(line => {
                    const parts = line.split('\t'); 
                    return {
                        setor: parts[0] ? parts[0].trim() : '',
                        item: parts[1] ? parts[1].trim() : '',
                        descricao: parts[2] ? parts[2].trim() : '',
                        endereco: parts[3] ? parts[3].trim() : ''
                    };
                });
        }

        /**
         * Popula a lista suspensa de setores com valores únicos.
         */
        function populateSetorFilter() {
            // Guarda o valor selecionado antes de limpar
            const currentValue = filterSetor.value;
            
            // Extrai setores únicos e não vazios da lista de produtos
            const setores = allProducts.map(p => p.setor).filter(s => s.length > 0);
            const uniqueSetores = [...new Set(setores)];
            uniqueSetores.sort(); // Ordena alfabeticamente

            // Limpa o select e adiciona a opção padrão
            filterSetor.innerHTML = '<option value="">Todos os Setores</option>';

            // Adiciona cada setor como uma nova opção
            uniqueSetores.forEach(setor => {
                const option = document.createElement('option');
                option.value = setor;
                option.textContent = setor;
                filterSetor.appendChild(option);
            });

            // Restaura o valor selecionado, se ele ainda existir
            filterSetor.value = currentValue;
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
                    tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-gray-500 text-xs">A lista de produtos está vazia. Peça ao administrador para carregar os dados.</td></tr>`;
                }
                
                // Popula o filtro de setor *depois* que os dados são carregados
                populateSetorFilter();
                
                // Renderiza a tabela
                renderTable();
            }, (error) => {
                console.error("Erro ao carregar lista do Firestore:", error);
                tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-red-500 text-xs">Erro ao carregar a lista. Verifique sua conexão.</td></tr>`;
            });
        }

        /**
         * Renderiza a tabela com base nos filtros e na lista 'allProducts'.
         */
        function renderTable() {
            // Pega o valor exato do select
            const setorFilter = filterSetor.value;
            const searchFilter = searchTerm.value.toLowerCase();

            const filteredProducts = allProducts.filter(product => {
                const pSetor = product.setor;
                const pItem = product.item.toLowerCase();
                const pDesc = product.descricao.toLowerCase();
                const pEnd = product.endereco.toLowerCase();

                // Compara o valor exato do setor
                const matchSetor = !setorFilter || pSetor === setorFilter;
                const matchSearch = !searchFilter || pItem.includes(searchFilter) || pEnd.includes(searchFilter);
                
                return matchSetor && matchSearch;
            });

            tableBody.innerHTML = '';
            rowCount.textContent = `Total de itens: ${filteredProducts.length}`;

            if (filteredProducts.length === 0) {
                if(allProducts.length > 0) {
                    tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-gray-500 text-xs">Nenhum produto encontrado com esses filtros.</td></tr>`;
                } else if (!authReady) {
                     tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-gray-500 text-xs">Conectando...</td></tr>`;
                } else {
                    tableBody.innerHTML = `<tr><td colspan="4" class="py-1.5 px-3 text-center text-gray-500 text-xs">A lista de produtos está vazia.</td></tr>`;
                }
            } else {
                filteredProducts.forEach(product => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50';
                    
                    const cellSetor = document.createElement('td');
                    // **** REDUÇÃO: Padding e Fonte das Células para py-1.5 px-3 / text-xs ****
                    cellSetor.className = 'py-1.5 px-3 text-xs text-gray-700';
                    cellSetor.textContent = product.setor;
                    row.appendChild(cellSetor);

                    const cellItem = document.createElement('td');
                    cellItem.className = 'py-1.5 px-3 text-xs text-gray-900 font-medium';
                    cellItem.textContent = product.item;
                    row.appendChild(cellItem);

                    const cellDesc = document.createElement('td');
                    cellDesc.className = 'py-1.5 px-3 text-xs text-gray-700';
                    cellDesc.textContent = product.descricao;
                    row.appendChild(cellDesc);

                    const cellEnd = document.createElement('td');
                    cellEnd.className = 'py-1.5 px-3 text-xs text-gray-700';
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
                // **** REDUÇÃO: Fonte da mensagem de status para text-xs ****
                saveStatus.textContent = "Erro: Ainda não conectado. Tente novamente em alguns segundos.";
                saveStatus.className = "mt-1 text-center text-xs text-red-600";
                return;
            }

            const rawText = dataPasteArea.value;
            if (rawText.trim() === "") {
                saveStatus.textContent = "A área de texto está vazia. Cole os dados primeiro.";
                saveStatus.className = "mt-1 text-center text-xs text-yellow-600";
                return;
            }

            saveButton.disabled = true;
            saveStatus.textContent = "Salvando na nuvem...";
            saveStatus.className = "mt-1 text-center text-xs text-blue-600";

            try {
                await setDoc(listDocRef, { rawProductData: rawText });
                
                saveStatus.textContent = "Lista atualizada com sucesso!";
                saveStatus.className = "mt-1 text-center text-xs text-green-600";
                
                dataPasteArea.value = '';

            } catch (error) {
                console.error("Erro ao salvar no Firestore:", error);
                saveStatus.textContent = "Erro ao salvar. Tente novamente.";
                saveStatus.className = "mt-1 text-center text-xs text-red-600";
            } finally {
                saveButton.disabled = false;
                setTimeout(() => { saveStatus.textContent = ''; }, 4000);
            }
        }

        // Adiciona os event listeners para os filtros
        filterSetor.addEventListener('change', renderTable);
        searchTerm.addEventListener('input', renderTable);

        // Adiciona o event listener para o botão de salvar
        saveButton.addEventListener('click', saveListToFirebase);

        // Lógica de senha para a seção Admin
        adminSection.addEventListener('toggle', function(event) {
            if (!this.hasAttribute('open')) {
                event.preventDefault();
                const password = prompt("Por favor, digite a senha de administrador:");
                if (password === "brunofe") {
                    this.open = true;
                } else {
                    if (password !== null) { 
                        alert("Senha incorreta. Acesso negado.");
                    }
                    this.open = false;
                }
            }
        });

        // Inicia o aplicativo
        initFirebase();

    </script>
</body>
</html>
