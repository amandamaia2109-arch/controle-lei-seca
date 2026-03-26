<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Controle Lei Seca - ETAPA 3</title>
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
    .card {
      background: white;
      border-radius: 10px;
      padding: 15px;
      margin-bottom: 20px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    }
    .linha-filtros {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }
    .linha-filtros input,
    .linha-filtros select {
      max-width: 250px;
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
    .badge.ok {
      background: #e7f7ea;
      color: #136f2d;
    }
    .debug {
      background: #fff8e1;
      border: 1px solid #f0d98a;
      padding: 10px;
      border-radius: 8px;
      margin-top: 10px;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>

  <h1>📚 Controle de Lei Seca - ETAPA 3</h1>

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
    <button onclick="limparTudo()">Limpar tudo</button>

    <div id="debug" class="debug"></div>
  </div>

  <div class="card">
    <h2>Filtros</h2>
    <div class="linha-filtros">
      <input id="busca" placeholder="Buscar..." oninput="mostrar()">
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
    </div>
  </div>

  <div class="card">
    <h2>Artigos cadastrados</h2>
    <div id="resumo"></div>
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
        "ETAPA 3 ativa\n" +
        "Total encontrado: " + encontrados.length + "\n\n" +
        (encontrados.slice(0, 2).join("\n\n---\n\n") || "Nada encontrado.");

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

      alert("Artigos salvos: " + encontrados.length);
    }

    function alternarLido(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) a.lido = !a.lido;
        return a;
      });
      salvar();
      mostrar();
    }

    function alternarIncidencia(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) a.altaIncidencia = !a.altaIncidencia;
        return a;
      });
      salvar();
      mostrar();
    }

    function alternarCaiu(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.caiuQuestao = !a.caiuQuestao;
          if (!a.caiuQuestao) a.banca = "";
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function definirBanca(id) {
      let novaBanca = prompt("Digite a banca:", "");
      if (novaBanca === null) return;

      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.banca = novaBanca.trim();
          if (a.banca) a.caiuQuestao = true;
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
      if (!confirm("Deseja apagar tudo?")) return;
      artigos = [];
      salvar();
      mostrar();
    }

    function getFiltrados() {
      let busca = document.getElementById("busca").value.toLowerCase().trim();
      let filtroMateria = document.getElementById("filtroMateria").value;
      let filtroBanca = document.getElementById("filtroBanca").value;
      let filtroCaiu = document.getElementById("filtroCaiu").value;

      return artigos.filter(function(a) {
        let texto = (a.lei + " " + a.materia + " " + a.artigo + " " + (a.banca || "")).toLowerCase();

        if (busca && !texto.includes(busca)) return false;
        if (filtroMateria && a.materia !== filtroMateria) return false;
        if (filtroBanca && a.banca !== filtroBanca) return false;
        if (filtroCaiu === "sim" && !a.caiuQuestao) return false;
        if (filtroCaiu === "nao" && a.caiuQuestao) return false;

        return true;
      });
    }

    function mostrar() {
      let lista = document.getElementById("lista");
      let resumo = document.getElementById("resumo");
      let filtrados = getFiltrados();

      lista.innerHTML = "";

      let total = artigos.length;
      let lidos = artigos.filter(function(a) { return a.lido; }).length;
      let caiu = artigos.filter(function(a) { return a.caiuQuestao; }).length;
      let alta = artigos.filter(function(a) { return a.altaIncidencia; }).length;

      resumo.innerHTML = "Total: " + total + " | Lidos: " + lidos + " | Alta incidência: " + alta + " | Já caiu: " + caiu + " | Exibindo: " + filtrados.length;

      if (filtrados.length === 0) {
        lista.innerHTML = "<p>Nenhum artigo encontrado.</p>";
        return;
      }

      filtrados.forEach(function(a) {
        let badges = "";

        if (a.altaIncidencia) badges += '<span class="badge alerta">Alta incidência</span>';
        if (a.caiuQuestao) badges += '<span class="badge ok">Já caiu</span>';
        if (a.banca) badges += '<span class="badge">' + a.banca + '</span>';

        let div = document.createElement("div");
        div.className = "artigo" + (a.lido ? " lido" : "");

        div.innerHTML = `
          <div class="topo">${a.lei} | ${a.materia}</div>
          <div>${badges}</div>
          <div class="texto">${a.artigo}</div>
          <div class="acoes">
            <button onclick="alternarLido(${a.id})">${a.lido ? "Desmarcar lido" : "Marcar lido"}</button>
            <button onclick="alternarIncidencia(${a.id})">${a.altaIncidencia ? "Tirar incidência" : "Alta incidência"}</button>
            <button onclick="alternarCaiu(${a.id})">${a.caiuQuestao ? "Desmarcar questão" : "Marcar que caiu"}</button>
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
