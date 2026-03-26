<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Controle de Lei Seca</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: Arial, sans-serif;
    }

    body {
      margin: 0;
      background: #f4f6f8;
      color: #222;
    }

    header {
      background: #1f3c88;
      color: white;
      padding: 20px;
      text-align: center;
    }

    .container {
      max-width: 1000px;
      margin: 20px auto;
      padding: 15px;
    }

    .card {
      background: white;
      border-radius: 12px;
      padding: 18px;
      margin-bottom: 20px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    }

    h1, h2 {
      margin-top: 0;
    }

    .form-grid {
      display: grid;
      grid-template-columns: 1fr 1fr 2fr auto;
      gap: 10px;
    }

    input, select, button {
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 14px;
    }

    button {
      background: #1f3c88;
      color: white;
      border: none;
      cursor: pointer;
    }

    button:hover {
      background: #16306d;
    }

    .stats {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 12px;
      text-align: center;
    }

    .stat-box {
      background: #eef3ff;
      padding: 15px;
      border-radius: 10px;
      font-weight: bold;
    }

    .progress-bar {
      width: 100%;
      height: 20px;
      background: #ddd;
      border-radius: 10px;
      overflow: hidden;
      margin-top: 10px;
    }

    .progress-fill {
      height: 100%;
      background: #28a745;
      width: 0%;
      transition: 0.3s;
    }

    .toolbar {
      display: grid;
      grid-template-columns: 2fr 1fr 1fr;
      gap: 10px;
    }

    .item {
      border: 1px solid #ddd;
      border-radius: 10px;
      padding: 12px;
      margin-top: 10px;
      background: #fafafa;
    }

    .item-top {
      display: flex;
      justify-content: space-between;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
    }

    .item-title {
      font-weight: bold;
      font-size: 16px;
    }

    .badge {
      display: inline-block;
      background: #dbe7ff;
      color: #1f3c88;
      padding: 4px 8px;
      border-radius: 999px;
      font-size: 12px;
      margin-top: 6px;
    }

    .actions {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      margin-top: 10px;
    }

    .done {
      text-decoration: line-through;
      opacity: 0.65;
    }

    .small-btn {
      padding: 8px 10px;
      font-size: 13px;
    }

    @media (max-width: 700px) {
      .form-grid,
      .toolbar,
      .stats {
        grid-template-columns: 1fr;
      }
    }
  </style>
</head>
<body>
  <header>
    <h1>📚 Controle de Lei Seca</h1>
    <p>Organize seus artigos, marque o que já leu e acompanhe seu progresso</p>
  </header>

  <div class="container">
    <div class="card">
      <h2>Adicionar artigo</h2>
      <div class="form-grid">
        <select id="disciplina">
          <option value="">Disciplina</option>
          <option>Constitucional</option>
          <option>Penal</option>
          <option>Processo Penal</option>
          <option>Administrativo</option>
          <option>Civil</option>
          <option>Processo Civil</option>
          <option>Tributário</option>
          <option>Legislação Extravagante</option>
        </select>

        <input type="text" id="artigo" placeholder="Ex.: Art. 5º" />
        <input type="text" id="descricao" placeholder="Ex.: Direitos e garantias fundamentais" />
        <button onclick="adicionarItem()">Adicionar</button>
      </div>
    </div>

    <div class="card">
      <h2>Seu progresso</h2>
      <div class="stats">
        <div class="stat-box">Total: <span id="totalItens">0</span></div>
        <div class="stat-box">Lidos: <span id="totalLidos">0</span></div>
        <div class="stat-box">Faltam: <span id="totalFaltam">0</span></div>
      </div>
      <div class="progress-bar">
        <div class="progress-fill" id="barraProgresso"></div>
      </div>
      <p><strong>Progresso:</strong> <span id="percentual">0%</span></p>
    </div>

    <div class="card">
      <h2>Filtrar e buscar</h2>
      <div class="toolbar">
        <input type="text" id="busca" placeholder="Pesquisar artigo ou descrição..." oninput="renderizar()" />
        <select id="filtroDisciplina" onchange="renderizar()">
          <option value="">Todas as disciplinas</option>
          <option>Constitucional</option>
          <option>Penal</option>
          <option>Processo Penal</option>
          <option>Administrativo</option>
          <option>Civil</option>
          <option>Processo Civil</option>
          <option>Tributário</option>
          <option>Legislação Extravagante</option>
        </select>
        <select id="filtroStatus" onchange="renderizar()">
          <option value="">Todos</option>
          <option value="lido">Somente lidos</option>
          <option value="nao-lido">Somente não lidos</option>
        </select>
      </div>
    </div>

    <div class="card">
      <h2>Lista de artigos</h2>
      <div id="lista"></div>
    </div>
  </div>

  <script>
    let itens = JSON.parse(localStorage.getItem("controleLeiSeca")) || [];

    function salvar() {
      localStorage.setItem("controleLeiSeca", JSON.stringify(itens));
    }

    function adicionarItem() {
      const disciplina = document.getElementById("disciplina").value;
      const artigo = document.getElementById("artigo").value.trim();
      const descricao = document.getElementById("descricao").value.trim();

      if (!disciplina || !artigo || !descricao) {
        alert("Preencha disciplina, artigo e descrição.");
        return;
      }

      itens.push({
        id: Date.now(),
        disciplina,
        artigo,
        descricao,
        lido: false
      });

      document.getElementById("disciplina").value = "";
      document.getElementById("artigo").value = "";
      document.getElementById("descricao").value = "";

      salvar();
      renderizar();
    }

    function alternarLido(id) {
      itens = itens.map(item =>
        item.id === id ? { ...item, lido: !item.lido } : item
      );
      salvar();
      renderizar();
    }

    function excluirItem(id) {
      if (!confirm("Deseja excluir este item?")) return;
      itens = itens.filter(item => item.id !== id);
      salvar();
      renderizar();
    }

    function renderizar() {
      const busca = document.getElementById("busca").value.toLowerCase();
      const filtroDisciplina = document.getElementById("filtroDisciplina").value;
      const filtroStatus = document.getElementById("filtroStatus").value;
      const lista = document.getElementById("lista");

      let filtrados = itens.filter(item => {
        const correspondeBusca =
          item.artigo.toLowerCase().includes(busca) ||
          item.descricao.toLowerCase().includes(busca) ||
          item.disciplina.toLowerCase().includes(busca);

        const correspondeDisciplina =
          !filtroDisciplina || item.disciplina === filtroDisciplina;

        const correspondeStatus =
          !filtroStatus ||
          (filtroStatus === "lido" && item.lido) ||
          (filtroStatus === "nao-lido" && !item.lido);

        return correspondeBusca && correspondeDisciplina && correspondeStatus;
      });

      lista.innerHTML = "";

      if (filtrados.length === 0) {
        lista.innerHTML = "<p>Nenhum artigo encontrado.</p>";
      }

      filtrados.forEach(item => {
        const div = document.createElement("div");
        div.className = "item";

        div.innerHTML = `
          <div class="item-top">
            <div>
              <div class="item-title ${item.lido ? "done" : ""}">${item.artigo} - ${item.descricao}</div>
              <div class="badge">${item.disciplina}</div>
            </div>
          </div>
          <div class="actions">
            <button class="small-btn" onclick="alternarLido(${item.id})">
              ${item.lido ? "Desmarcar leitura" : "Marcar como lido"}
            </button>
            <button class="small-btn" onclick="excluirItem(${item.id})">Excluir</button>
          </div>
        `;

        lista.appendChild(div);
      });

      atualizarResumo();
    }

    function atualizarResumo() {
      const total = itens.length;
      const lidos = itens.filter(i => i.lido).length;
      const faltam = total - lidos;
      const percentual = total === 0 ? 0 : Math.round((lidos / total) * 100);

      document.getElementById("totalItens").textContent = total;
      document.getElementById("totalLidos").textContent = lidos;
      document.getElementById("totalFaltam").textContent = faltam;
      document.getElementById("percentual").textContent = percentual + "%";
      document.getElementById("barraProgresso").style.width = percentual + "%";
    }

    renderizar();
  </script>
</body>
</html>
