<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
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
      max-width: 700px;
      box-sizing: border-box;
    }

    button {
      background: #1f3c88;
      color: white;
      border: none;
      cursor: pointer;
      width: auto;
      padding: 10px 16px;
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
    }

    .acoes button {
      margin-right: 8px;
      margin-top: 8px;
    }

    .texto {
      white-space: pre-wrap;
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

  <textarea id="textoLei" placeholder="Cole a lei aqui" rows="10"></textarea><br>

  <button onclick="extrair()">Extrair e salvar artigos</button>
</div>

<div class="card">
  <h2>Artigos cadastrados</h2>
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
  let texto = document.getElementById("textoLei").value;

  if (!nomeLei || !texto) {
    alert("Preencha o nome da lei e cole o texto.");
    return;
  }

  let regex = /Art\.\s*\d+[^\n]*[\s\S]*?(?=Art\.\s*\d+|$)/g;
  let encontrados = texto.match(regex);

  if (!encontrados) {
    alert("Nenhum artigo encontrado.");
    return;
  }

  encontrados.forEach((art, i) => {
    artigos.push({
      id: Date.now() + i,
      lei: nomeLei,
      materia: materia,
      artigo: art.trim(),
      lido: false
    });
  });

  salvar();
  mostrar();

  document.getElementById("nomeLei").value = "";
  document.getElementById("textoLei").value = "";

  alert("Artigos salvos com sucesso.");
}

function alternarLido(id) {
  artigos = artigos.map(a => {
    if (a.id === id) {
      a.lido = !a.lido;
    }
    return a;
  });

  salvar();
  mostrar();
}

function excluirArtigo(id) {
  artigos = artigos.filter(a => a.id !== id);
  salvar();
  mostrar();
}

function mostrar() {
  let lista = document.getElementById("lista");
  lista.innerHTML = "";

  if (artigos.length === 0) {
    lista.innerHTML = "<p>Nenhum artigo cadastrado.</p>";
    return;
  }

  artigos.forEach(a => {
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
