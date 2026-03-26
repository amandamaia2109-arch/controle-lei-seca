<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Controle Lei Seca</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: #f5f7fa;
    }

    h1, h2 {
      color: #1f3c88;
    }

    input, select, textarea, button {
      padding: 10px;
      margin: 5px 0;
      box-sizing: border-box;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 14px;
    }

    input, select, textarea {
      width: 100%;
      max-width: 900px;
    }

    button {
      background: #1f3c88;
      color: white;
      border: none;
      cursor: pointer;
      width: auto;
      padding: 10px 16px;
      margin-right: 8px;
    }

    button:hover {
      background: #16306d;
    }

    .card {
      background: white;
      border-radius: 10px;
      padding: 15px;
      margin-bottom: 20px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    }

    .artigo {
      border: 1px solid #ddd;
      padding: 12px;
      margin-top: 10px;
      border-radius: 8px;
      background: #fafafa;
    }

    .lido {
      background: #e8f5e9;
    }

    .topo {
      font-weight: bold;
      margin-bottom: 6px;
      color: #1f3c88;
    }

    .texto {
      white-space: pre-wrap;
      line-height: 1.4;
      margin-top: 8px;
    }

    .acoes button {
      margin-top: 8px;
      margin-right: 6px;
    }

    .resumo {
      margin-bottom: 10px;
      font-weight: bold;
    }

    .debug {
      background: #fff8e1;
      border: 1px solid #f0d98a;
      padding: 10px;
      border-radius: 8px;
      margin-top: 10px;
      white-space: pre-wrap;
    }

    .linha-filtros {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-bottom: 10px;
    }

    .linha-filtros input,
    .linha-filtros select {
      max-width: 260px;
      width: 100%;
    }

    .badge {
      display: inline-block;
      padding: 4px 8px;
      border-radius: 999px;
      font-size: 12px;
      margin-right: 6px;
      margin-top: 6px;
      background: #e3ebff;
      color: #1f3c88;
    }

    .badge.alerta {
      background: #ffe5e5;
      color: #a10000;
    }

    .badge.sucesso {
      background: #e7f7ea;
      color: #136f2d;
    }
  </style>
</head>
<body>

  <h1>📚 Controle de Lei Seca</h1>

  <div class="card">
    <h2>Adicionar Lei</h2>

    <input id="nomeLei" placeholder="Nome da lei">

    <select id="materia">
      <option>Direito Penal</option>
      <option>Direito Civil</option>
      <option>Direito Processual Penal</option>
      <option>Direito Processual Civil</option>
      <option>Constitucional</option>
      <option>Administrativo</option>
      <option>Processo Coletivo</option>
    </select>

    <textarea id="textoLei" placeholder="Cole a lei aqui" rows="12"></textarea><br>

    <button onclick="extrair()">Extrair e salvar artigos</button>
    <button onclick="limparTudo()">Limpar todos os artigos</button>

    <div id="debug" class="debug"></div>
  </div>

  <div class="card">
    <h2>Filtros</h2>

    <div class="linha-filtros">
      <input id="busca" placeholder="Buscar por texto, lei ou artigo..." oninput="mostrar()">

      <select id="filtroMateria" onchange="mostrar()">
        <option value="">Todas as matérias</option>
        <option>Direito Penal</option>
        <option>Direito Civil</option>
        <option>Direito Processual Penal</option>
        <option>Direito Processual Civil</option>
        <option>Constitucional</option>
        <option>Administrativo</option>
        <option>Processo Coletivo</option>
      </select>

      <select id="filtroBanca" onchange="mostrar()">
        <option value="">Todas as bancas</option>
        <option>FGV</option>
        <option>Cebraspe</option>
        <option>FCC</option>
        <option>Vunesp</option>
        <option>IBFC</option>
        <option>Consulplan</option>
        <option>AOCP</option>
        <option>Outra</option>
      </select>

      <select id="filtroCaiu" onchange="mostrar()">
        <option value="">Caiu ou não</option>
        <option value="sim">Já caiu</option>
        <option value="nao">Não caiu</option>
      </select>

      <select id="filtroIncidencia" onchange="mostrar()">
        <option value="">Alta incidência?</option>
        <option value="sim">Sim</option>
        <option value="nao">Não</option>
      </select>
    </div>
  </div>

  <div class="card">
    <h2>Artigos cadastrados</h2>
    <div class="resumo" id="resumo"></div>
    <div id="lista"></div>
  </div>

  <script>
    let artigos = JSON.parse(localStorage.getItem("artigosLeiSeca")) || [];

    function salvar() {
      localStorage.setItem("artigosLeiSeca", JSON.stringify(artigos));
    }

    function extrair() {
      let nomeLei = document.getElementById("nomeLei").value.trim();
      let materia = document.getElementById("materia").value;
      let texto = document.getElementById("textoLei").value.trim();
      let debug = document.getElementById("debug");

      if (!nomeLei || !texto) {
        alert("Preencha o nome da lei e cole o texto.");
        return;
      }

      texto = texto.replace(/\r/g, "\n");
      texto = texto.replace(/\n+/g, "\n");
      texto = texto.replace(/Art\.\s*/g, "\nArt. ");
      texto = texto.replace(/Art\s+(\d+)/g, "\nArt. $1");

      let partes = texto.split("\nArt. ");
      let encontrados = [];

      for (let i = 0; i < partes.length; i++) {
        let parte = partes[i].trim();
        if (!parte) continue;

        if (!parte.startsWith("Art. ")) {
          parte = "Art. " + parte;
        }

        if (/^Art\.\s*\d+/i.test(parte)) {
          encontrados.push(parte);
        }
      }

      debug.innerHTML =
        "Prévia de extração:\n" +
        "Total encontrado: " + encontrados.length + "\n\n" +
        (encontrados.slice(0, 3).join("\n\n---\n\n") || "Nada encontrado.");

      if (encontrados.length === 0) {
        alert("Nenhum artigo encontrado.");
        return;
      }

      let baseId = Date.now();

      for (let i = 0; i < encontrados.length; i++) {
        artigos.push({
          id: baseId + i,
          lei: nomeLei,
          materia: materia,
          artigo: encontrados[i],
          lido: false,
          altaIncidencia: false,
          caiuQuestao: false,
          banca: ""
        });
      }

      salvar();
      mostrar();

      document.getElementById("nomeLei").value = "";
      document.getElementById("textoLei").value = "";

      alert("Artigos salvos com sucesso: " + encontrados.length);
    }

    function alternarLido(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.lido = !a.lido;
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function alternarIncidencia(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.altaIncidencia = !a.altaIncidencia;
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function alternarCaiu(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.caiuQuestao = !a.caiuQuestao;
          if (!a.caiuQuestao) {
            a.banca = "";
          }
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function definirBanca(id) {
      let artigo = artigos.find(function(a) { return a.id === id; });
      if (!artigo) return;

      let bancaAtual = artigo.banca || "";
      let novaBanca = prompt("Digite a banca (FGV, Cebraspe, FCC, Vunesp, IBFC, Consulplan, AOCP ou Outra):", bancaAtual);

      if (novaBanca === null) return;

      novaBanca = novaBanca.trim();

      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.banca = novaBanca;
          if (novaBanca !== "") {
            a.caiuQuestao = true;
          }
        }
        return a;
      });

      salvar();
      mostrar();
    }

    function excluirArtigo(id) {
      artigos = artigos.filter(function(a) {
        return a.id !== id;
      });
      salvar();
      mostrar();
    }

    function limparTudo() {
      let confirmar = confirm("Deseja apagar todos os artigos cadastrados?");
      if (!confirmar) return;
      artigos = [];
      salvar();
      mostrar();
    }

    function getArtigosFiltrados() {
      let busca = document.getElementById("busca").value.toLowerCase().trim();
      let filtroMateria = document.getElementById("filtroMateria").value;
      let filtroBanca = document.getElementById("filtroBanca").value;
      let filtroCaiu = document.getElementById("filtroCaiu").value;
      let filtroIncidencia = document.getElementById("filtroIncidencia").value;

      return artigos.filter(function(a) {
        let textoCompleto = (a.lei + " " + a.materia + " " + a.artigo + " " + (a.banca || "")).toLowerCase();

        if (busca && !textoCompleto.includes(busca)) {
          return false;
        }

        if (filtroMateria && a.materia !== filtroMateria) {
          return false;
        }

        if (filtroBanca && a.banca !== filtroBanca) {
          return false;
        }

        if (filtroCaiu === "sim" && !a.caiuQuestao) {
          return false;
        }

        if (filtroCaiu === "nao" && a.caiuQuestao) {
          return false;
        }

        if (filtroIncidencia === "sim" && !a.altaIncidencia) {
          return false;
        }

        if (filtroIncidencia === "nao" && a.altaIncidencia) {
          return false;
        }

        return true;
      });
    }

    function mostrar() {
      let lista = document.getElementById("lista");
      let resumo = document.getElementById("resumo");

      lista.innerHTML = "";

      let filtrados = getArtigosFiltrados();

      if (artigos.length === 0) {
        resumo.innerHTML = "Total: 0 artigos";
        lista.innerHTML = "<p>Nenhum artigo cadastrado.</p>";
        return;
      }

      let total = artigos.length;
      let lidos = artigos.filter(function(a) { return a.lido; }).length;
      let pendentes = total - lidos;
      let alta = artigos.filter(function(a) { return a.altaIncidencia; }).length;
      let caiu = artigos.filter(function(a) { return a.caiuQuestao; }).length;

      resumo.innerHTML =
        "Total: " + total +
        " | Lidos: " + lidos +
        " | Pendentes: " + pendentes +
        " | Alta incidência: " + alta +
        " | Já caiu: " + caiu +
        " | Exibindo agora: " + filtrados.length;

      if (filtrados.length === 0) {
        lista.innerHTML = "<p>Nenhum artigo encontrado com esses filtros.</p>";
        return;
      }

      filtrados.forEach(function(a) {
        let div = document.createElement("div");
        div.className = "artigo" + (a.lido ? " lido" : "");

        let badges = "";
        if (a.altaIncidencia) {
          badges += '<span class="badge alerta">Alta incidência</span>';
        }
        if (a.caiuQuestao) {
          badges += '<span class="badge sucesso">Já caiu em questão</span>';
        }
        if (a.banca) {
          badges += '<span class="badge">' + a.banca + '</span>';
        }

        div.innerHTML = `
          <div class="topo">${a.lei} | ${a.materia}</div>
          <div>${badges}</div>
          <div class="texto">${a.artigo}</div>
          <div class="acoes">
            <button onclick="alternarLido(${a.id})">
              ${a.lido ? "Desmarcar lido" : "Marcar como lido"}
            </button>
            <button onclick="alternarIncidencia(${a.id})">
              ${a.altaIncidencia ? "Tirar alta incidência" : "Marcar alta incidência"}
            </button>
            <button onclick="alternarCaiu(${a.id})">
              ${a.caiuQuestao ? "Desmarcar questão" : "Marcar que caiu"}
            </button>
            <button onclick="definirBanca(${a.id})">Definir banca</button>
            <button onclick="excluirArtigo(${a.id})">Excluir</button>
          </div>
        `;

        lista.appendChild(div);
      });
    }

    mostrar();
  </script>

</body>
</html>
