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
      width: 100%;
      max-width: 900px;
      box-sizing: border-box;
      border-radius: 8px;
      border: 1px solid #ccc;
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

    .acoes button {
      margin-top: 8px;
    }

    .texto {
      white-space: pre-wrap;
      line-height: 1.4;
    }

    .resumo {
      margin-bottom: 10px;
      font-weight: bold;
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

      if (!nomeLei || !texto) {
        alert("Preencha o nome da lei e cole o texto.");
        return;
      }

      texto = texto.replace(/\r/g, "");

      // quebra o texto toda vez que aparece um novo artigo
      let partes = texto.split(/\n\s*(?=Art\.?\s*\d+[A-Za-zº°\-]*)/i);

      let encontrados = partes
        .map(function(p) { return p.trim(); })
        .filter(function(p) {
          return /^Art\.?\s*\d+[A-Za-zº°\-]*/i.test(p);
        });

      // plano B, caso a primeira quebra não funcione por causa da formatação
      if (encontrados.length === 0) {
        let regex = /Art\.?\s*\d+[A-Za-zº°\-]*[\s\S]*?(?=(\n\s*Art\.?\s*\d+[A-Za-zº°\-]*)|$)/gi;
        encontrados = texto.match(regex) || [];
        encontrados = encontrados.map(function(p) { return p.trim(); });
      }

      if (encontrados.length === 0) {
        alert("Nenhum artigo encontrado. O texto colado veio fora do padrão.");
        return;
      }

      let baseId = Date.now();

      encontrados.forEach(function(art, i) {
        artigos.push({
          id: baseId + i,
          lei: nomeLei,
          materia: materia,
          artigo: art,
          lido: false
        });
      });

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

    function mostrar() {
      let lista = document.getElementById("lista");
      let resumo = document.getElementById("resumo");

      lista.innerHTML = "";

      if (artigos.length === 0) {
        resumo.innerHTML = "Total: 0 artigos";
        lista.innerHTML = "<p>Nenhum artigo cadastrado.</p>";
        return;
      }

      let total = artigos.length;
      let lidos = artigos.filter(function(a) { return a.lido; }).length;
      let pendentes = total - lidos;

      resumo.innerHTML = "Total: " + total + " | Lidos: " + lidos + " | Pendentes: " + pendentes;

      artigos.forEach(function(a) {
        let div = document.createElement("div");
        div.className = "artigo" + (a.lido ? " lido" : "");

        div.innerHTML = `
          <div class="topo">${a.lei} | ${a.materia}</div>
          <div class="texto">${a.artigo}</div>
          <div class="acoes">
            <button onclick="alternarLido(${a.id})">
              ${a.lido ? "Desmarcar lido" : "Marcar como lido"}
            </button>
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
    }

    mostrar();
  </script>

</body>
</html>
