<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Controle Lei Seca PRO</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f7fb;
      margin: 20px;
      color: #1f2937;
    }

    h1, h2 {
      color: #1f3c88;
    }

    .tabs {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-bottom: 20px;
    }

    .tab-btn {
      background: white;
      border: 1px solid #cfd8ea;
      padding: 10px 16px;
      border-radius: 999px;
      cursor: pointer;
      font-weight: bold;
      color: #1f3c88;
    }

    .tab-btn.ativa {
      background: #1f3c88;
      color: white;
    }

    .aba {
      display: none;
    }

    .aba.ativa {
      display: block;
    }

    .card {
      background: white;
      padding: 15px;
      border-radius: 10px;
      margin-bottom: 20px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    }

    input, select, textarea, button {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      border-radius: 8px;
      border: 1px solid #ccc;
      box-sizing: border-box;
      font-size: 14px;
    }

    button {
      background: #1f3c88;
      color: white;
      cursor: pointer;
      border: none;
    }

    button:hover {
      background: #16306d;
    }

    .linha-botoes {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      margin-top: 10px;
    }

    .linha-botoes button {
      width: auto;
    }

    .artigo {
      border: 1px solid #ddd;
      padding: 10px;
      margin-top: 10px;
      border-radius: 8px;
      background: #fafafa;
    }

    .badge {
      display: inline-block;
      padding: 4px 8px;
      border-radius: 999px;
      font-size: 12px;
      margin-right: 5px;
      margin-top: 5px;
      background: #e3ebff;
      color: #1f3c88;
    }

    .inc { background: #ffe5e5; color: #a10000; }
    .caiu { background: #e7f7ea; color: #136f2d; }
    .rev { background: #fff3cd; color: #8a6500; }
    .lidoBadge { background: #e8f5e9; color: #136f2d; }

    .filtros {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
      gap: 10px;
    }

    .subtitulo {
      font-size: 13px;
      color: #666;
      margin-top: -8px;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>

  <h1>📚 Controle Lei Seca PRO</h1>

  <div class="tabs">
    <button class="tab-btn ativa" onclick="abrirAba('abaLeis', this)">Leis</button>
    <button class="tab-btn" onclick="abrirAba('abaCronograma', this)">Cronograma</button>
    <button class="tab-btn" onclick="abrirAba('abaRevisao', this)">Revisão</button>
    <button class="tab-btn" onclick="abrirAba('abaArtigos', this)">Artigos</button>
  </div>

  <div id="abaLeis" class="aba ativa">
    <div class="card">
      <h2>Adicionar Lei</h2>

      <label>Nome da lei</label>
      <input id="nomeLei" placeholder="Ex.: Código Penal">

      <label>Matéria</label>
      <select id="materia">
      <option>Direito Penal</option>
      <option>Direito Civil</option>
      <option>Processual Penal</option>
      <option>Processual Civil</option>
      <option>Constitucional</option>
      <option>Administrativo</option>
      <option>Processo Coletivo</option>
      <option>Tributário</option>
      <option>Improbidade</option>
      <option>Empresarial</option>
      <option>Consumidor</option>
    </select>

    <label>Texto da lei</label>
    <textarea id="textoLei" rows="8" placeholder="Cole a lei aqui"></textarea>

    <div class="linha-botoes">
      <button onclick="extrair()">Extrair artigos</button>
      <button onclick="limparTudo()">Limpar tudo</button>
    </div>
  </div>

  <div class="card">
    <h2>Configuração do cronograma</h2>
    <div class="subtitulo">Defina a data inicial e quantos artigos quer por dia.</div>

    <label>Data inicial do cronograma</label>
    <input id="dataInicial" type="date" onchange="salvarDataInicial()">

    <label>Meta de artigos por dia</label>
    <input id="metaDia" type="number" value="20" onchange="salvarMetaDia()">
  </div>

  <div class="grid-2">
    <div class="card">
      <h2>📅 Hoje</h2>
      <div id="hoje"></div>
      <div class="linha-botoes">
        <button onclick="concluirDia()">Concluir dia</button>
      </div>
    </div>

    <div class="card">
      <h2>🗓 Próximos dias</h2>
      <div id="proximosDias"></div>
    </div>
  </div>

  <div class="card">
    <h2>🔁 Revisão</h2>
    <div id="revisao"></div>
  </div>

  <div class="card">
    <h2>Filtros</h2>
    <div class="filtros">
      <input id="busca" placeholder="Buscar por texto, lei ou artigo..." oninput="mostrar()">

      <select id="filtroMateria" onchange="mostrar()">
        <option value="">Todas as matérias</option>
        <option>Direito Penal</option>
        <option>Direito Civil</option>
        <option>Processual Penal</option>
        <option>Processual Civil</option>
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
    <h2>📚 Artigos</h2>
    <div id="lista"></div>
  </div>

  <script>
    let artigos = JSON.parse(localStorage.getItem("artigosLeiSecaPro")) || [];
    let progresso = JSON.parse(localStorage.getItem("progressoLeiSecaPro")) || 0;
    let metaDia = JSON.parse(localStorage.getItem("metaDiaLeiSecaPro")) || 20;
    let dataInicial = localStorage.getItem("dataInicialLeiSecaPro") || "";

    function salvar() {
      localStorage.setItem("artigosLeiSecaPro", JSON.stringify(artigos));
      localStorage.setItem("progressoLeiSecaPro", JSON.stringify(progresso));
      localStorage.setItem("metaDiaLeiSecaPro", JSON.stringify(metaDia));
      localStorage.setItem("dataInicialLeiSecaPro", dataInicial);
    }

    function hoje() {
      return new Date().toISOString().split("T")[0];
    }

    function somarDias(data, dias) {
      let d = new Date(data + "T00:00:00");
      d.setDate(d.getDate() + dias);
      return d.toISOString().split("T")[0];
    }

    function diferencaDias(data1, data2) {
      let d1 = new Date(data1 + "T00:00:00");
      let d2 = new Date(data2 + "T00:00:00");
      let ms = d2 - d1;
      return Math.floor(ms / (1000 * 60 * 60 * 24));
    }

    function salvarDataInicial() {
      dataInicial = document.getElementById("dataInicial").value;
      salvar();
      mostrar();
    }

    function salvarMetaDia() {
      metaDia = parseInt(document.getElementById("metaDia").value) || 20;
      salvar();
      mostrar();
    }

    function extrair() {
      let nomeLei = document.getElementById("nomeLei").value.trim();
      let materia = document.getElementById("materia").value;
      let texto = document.getElementById("textoLei").value.trim();

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
          banca: "",
          precisaRevisao: false,
          dataRevisao: hoje(),
          nivelRevisao: 0
        });
      }

      salvar();
      mostrar();

      document.getElementById("nomeLei").value = "";
      document.getElementById("textoLei").value = "";

      alert("Artigos extraídos: " + encontrados.length);
    }

    function montarBlocoDia(indiceBase, titulo) {
      let bloco = document.createElement("div");
      bloco.className = "artigo";

      let artigosDia = artigos.slice(indiceBase, indiceBase + metaDia);

      if (artigosDia.length === 0) {
        bloco.innerHTML = "<b>" + titulo + "</b><br>Nenhum artigo programado.";
        return bloco;
      }

      let texto = "<b>" + titulo + "</b><br><br>";

      artigosDia.forEach(function(a) {
        texto += "<b>" + a.materia + "</b> - " + a.artigo.substring(0, 120) + "...<br><br>";
      });

      bloco.innerHTML = texto;
      return bloco;
    }

    function marcarLido(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.lido = !a.lido;
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function toggleIncidencia(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.altaIncidencia = !a.altaIncidencia;
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function toggleCaiu(id) {
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
      let atual = artigos.find(function(a) { return a.id === id; });
      if (!atual) return;

      let banca = prompt("Digite a banca:", atual.banca || "");
      if (banca === null) return;

      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.banca = banca.trim();
          if (a.banca !== "") {
            a.caiuQuestao = true;
          }
        }
        return a;
      });

      salvar();
      mostrar();
    }

    function togglePrecisaRevisao(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.precisaRevisao = !a.precisaRevisao;
        }
        return a;
      });
      salvar();
      mostrar();
    }

    function marcarRevisado(id) {
      artigos = artigos.map(function(a) {
        if (a.id === id) {
          a.nivelRevisao = (a.nivelRevisao || 0) + 1;

          let intervalos = [1, 3, 7, 15, 30];
          let dias = intervalos[a.nivelRevisao] || 30;

          if (a.altaIncidencia || a.caiuQuestao || a.precisaRevisao) {
            dias = Math.max(1, Math.floor(dias / 2));
          }

          a.dataRevisao = somarDias(hoje(), dias);
        }
        return a;
      });

      salvar();
      mostrar();
    }

    function concluirDia() {
      progresso += metaDia;
      if (progresso > artigos.length) {
        progresso = artigos.length;
      }
      salvar();
      mostrar();
    }

    function limparTudo() {
      if (!confirm("Deseja apagar tudo?")) return;
      artigos = [];
      progresso = 0;
      salvar();
      mostrar();
    }

    function mostrar() {
      let lista = document.getElementById("lista");
      let hojeDiv = document.getElementById("hoje");
      let revisaoDiv = document.getElementById("revisao");
      let proximosDiasDiv = document.getElementById("proximosDias");

      let campoDataInicial = document.getElementById("dataInicial");
      let campoMetaDia = document.getElementById("metaDia");

      if (campoDataInicial) campoDataInicial.value = dataInicial;
      if (campoMetaDia) campoMetaDia.value = metaDia;

      let busca = document.getElementById("busca") ? document.getElementById("busca").value.toLowerCase().trim() : "";
      let filtroMateria = document.getElementById("filtroMateria") ? document.getElementById("filtroMateria").value : "";
      let filtroBanca = document.getElementById("filtroBanca") ? document.getElementById("filtroBanca").value : "";
      let filtroCaiu = document.getElementById("filtroCaiu") ? document.getElementById("filtroCaiu").value : "";
      let filtroIncidencia = document.getElementById("filtroIncidencia") ? document.getElementById("filtroIncidencia").value : "";

      lista.innerHTML = "";
      hojeDiv.innerHTML = "";
      revisaoDiv.innerHTML = "";
      proximosDiasDiv.innerHTML = "";

      let indiceInicial = progresso;

      if (dataInicial) {
        let diasPassados = diferencaDias(dataInicial, hoje());
        if (diasPassados < 0) diasPassados = 0;
        indiceInicial = diasPassados * metaDia;
      }

      hojeDiv.innerHTML = "<p><b>Data de hoje:</b> " + hoje() + "</p>";
      hojeDiv.appendChild(montarBlocoDia(indiceInicial, "Hoje"));

      proximosDiasDiv.appendChild(montarBlocoDia(indiceInicial + metaDia, "Amanhã"));
      proximosDiasDiv.appendChild(montarBlocoDia(indiceInicial + (metaDia * 2), "Depois de amanhã"));
      proximosDiasDiv.appendChild(montarBlocoDia(indiceInicial + (metaDia * 3), "Próximo bloco"));

      artigos.forEach(function(a) {
        if (a.dataRevisao <= hoje()) {
          let rev = document.createElement("div");
          rev.className = "artigo";
          rev.innerHTML = `
            <b>${a.materia}</b><br>
            ${a.artigo.substring(0, 180)}...<br><br>
            <button onclick="marcarRevisado(${a.id})">Revisado hoje</button>
          `;
          revisaoDiv.appendChild(rev);
        }
      });

      let artigosFiltrados = artigos.filter(function(a) {
        let textoCompleto = (a.lei + " " + a.materia + " " + a.artigo + " " + (a.banca || "")).toLowerCase();

        if (busca && !textoCompleto.includes(busca)) return false;
        if (filtroMateria && a.materia !== filtroMateria) return false;
        if (filtroBanca && a.banca !== filtroBanca) return false;
        if (filtroCaiu === "sim" && !a.caiuQuestao) return false;
        if (filtroCaiu === "nao" && a.caiuQuestao) return false;
        if (filtroIncidencia === "sim" && !a.altaIncidencia) return false;
        if (filtroIncidencia === "nao" && a.altaIncidencia) return false;

        return true;
      });

      artigosFiltrados.forEach(function(a) {
        let badges = "";

        if (a.altaIncidencia) {
          badges += '<span class="badge inc">Alta incidência</span>';
        }

        if (a.caiuQuestao) {
          badges += '<span class="badge caiu">Já caiu</span>';
        }

        if (a.precisaRevisao) {
          badges += '<span class="badge rev">Revisão extra</span>';
        }

        if (a.banca) {
          badges += '<span class="badge caiu">' + a.banca + '</span>';
        }

        if (a.lido) {
          badges += '<span class="badge lidoBadge">Lido</span>';
        }

        if (a.dataRevisao <= hoje()) {
          badges += '<span class="badge rev">Revisar</span>';
        }

        let div = document.createElement("div");
        div.className = "artigo";

        div.innerHTML = `
          <b>${a.lei} | ${a.materia}</b><br>
          ${badges}<br><br>
          ${a.artigo}
          <br><br>
          <button onclick="marcarLido(${a.id})">${a.lido ? "Desmarcar lido" : "Marcar lido"}</button>
          <button onclick="toggleIncidencia(${a.id})">${a.altaIncidencia ? "Tirar incidência" : "Alta incidência"}</button>
          <button onclick="toggleCaiu(${a.id})">${a.caiuQuestao ? "Desmarcar caiu" : "Marcar que caiu"}</button>
          <button onclick="definirBanca(${a.id})">Banca</button>
          <button onclick="togglePrecisaRevisao(${a.id})">${a.precisaRevisao ? "Tirar revisão extra" : "Precisa revisão"}</button>
          <button onclick="marcarRevisado(${a.id})">Revisado hoje</button>
        `;

        lista.appendChild(div);
      });

      if (artigosFiltrados.length === 0) {
        lista.innerHTML = "<p>Nenhum artigo encontrado com esses filtros.</p>";
      }

      if (revisaoDiv.innerHTML === "") {
        revisaoDiv.innerHTML = "<p>Nada para revisar hoje.</p>";
      }
    }

    mostrar();
  </script>

</body>
</html>
