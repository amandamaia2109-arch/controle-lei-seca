<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Controle Lei Seca PRO</title>

<style>
body {
  font-family: Arial;
  background:#f4f7fb;
  margin:20px;
}

h1 { color:#1f3c88; }

.card {
  background:white;
  padding:15px;
  border-radius:10px;
  margin-bottom:20px;
  box-shadow:0 2px 8px rgba(0,0,0,0.08);
}

input, select, textarea, button {
  width:100%;
  padding:10px;
  margin-top:5px;
  border-radius:8px;
  border:1px solid #ccc;
}

button {
  background:#1f3c88;
  color:white;
  cursor:pointer;
}

.artigo {
  border:1px solid #ddd;
  padding:10px;
  margin-top:10px;
  border-radius:8px;
}

.badge {
  padding:4px 8px;
  border-radius:6px;
  font-size:12px;
  margin-right:5px;
}

.inc { background:#ffe5e5; }
.caiu { background:#e7f7ea; }
.rev { background:#fff3cd; }

</style>
</head>

<body>

<h1>📚 Controle Lei Seca</h1>

<div class="card">
<h2>Adicionar Lei</h2>

<input id="nomeLei" placeholder="Nome da lei">

<select id="materia">
<option>Direito Penal</option>
<option>Direito Civil</option>
<option>Processual Penal</option>
<option>Processual Civil</option>
<option>Constitucional</option>
<option>Administrativo</option>
<option>Processo Coletivo</option>
</select>

<textarea id="textoLei" rows="8" placeholder="Cole a lei aqui"></textarea>

<button onclick="extrair()">Extrair</button>
</div>

<div class="card">
<h2>📅 Hoje</h2>
<div id="hoje"></div>
<button onclick="concluirDia()">Concluir dia</button>
</div>

<div class="card">
<h2>🔁 Revisão</h2>
<div id="revisao"></div>
</div>
<div class="card">
<h2>Filtros</h2>

<input id="busca" placeholder="Buscar por texto, lei ou artigo..." oninput="mostrar()"><br><br>

<select id="filtroMateria" onchange="mostrar()">
  <option value="">Todas as matérias</option>
  <option>Direito Penal</option>
  <option>Direito Civil</option>
  <option>Processual Penal</option>
  <option>Processual Civil</option>
  <option>Constitucional</option>
  <option>Administrativo</option>
  <option>Processo Coletivo</option>
</select><br><br>

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
</select><br><br>

<select id="filtroCaiu" onchange="mostrar()">
  <option value="">Caiu ou não</option>
  <option value="sim">Já caiu</option>
  <option value="nao">Não caiu</option>
</select><br><br>

<select id="filtroIncidencia" onchange="mostrar()">
  <option value="">Alta incidência?</option>
  <option value="sim">Sim</option>
  <option value="nao">Não</option>
</select>
</div>
<div class="card">
<h2>📚 Artigos</h2>
<div id="lista"></div>
</div>
<script>
let artigos = JSON.parse(localStorage.getItem("artigosLeiSecaPro")) || [];
let progresso = JSON.parse(localStorage.getItem("progressoLeiSecaPro")) || 0;
let metaDia = 20;

function salvar() {
  localStorage.setItem("artigosLeiSecaPro", JSON.stringify(artigos));
  localStorage.setItem("progressoLeiSecaPro", JSON.stringify(progresso));
}

function hoje() {
  return new Date().toISOString().split("T")[0];
}

function somarDias(data, dias) {
  let d = new Date(data + "T00:00:00");
  d.setDate(d.getDate() + dias);
  return d.toISOString().split("T")[0];
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
function mostrar() {
  let lista = document.getElementById("lista");
  let hojeDiv = document.getElementById("hoje");
  let revisaoDiv = document.getElementById("revisao");
let busca = document.getElementById("busca") ? document.getElementById("busca").value.toLowerCase().trim() : "";
let filtroMateria = document.getElementById("filtroMateria") ? document.getElementById("filtroMateria").value : "";
let filtroBanca = document.getElementById("filtroBanca") ? document.getElementById("filtroBanca").value : "";
let filtroCaiu = document.getElementById("filtroCaiu") ? document.getElementById("filtroCaiu").value : "";
let filtroIncidencia = document.getElementById("filtroIncidencia") ? document.getElementById("filtroIncidencia").value : "";
  lista.innerHTML = "";
  hojeDiv.innerHTML = "";
  revisaoDiv.innerHTML = "";

  let artigosHoje = artigos.slice(progresso, progresso + metaDia);

  if (artigosHoje.length === 0) {
    hojeDiv.innerHTML = "<p>Nenhum artigo programado para hoje.</p>";
  } else {
    artigosHoje.forEach(function(a) {
      let div = document.createElement("div");
      div.className = "artigo";
      div.innerHTML = "<b>" + a.materia + "</b><br>" + a.artigo.substring(0, 180) + "...";
      hojeDiv.appendChild(div);
    });
  }
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

    if (a.dataRevisao <= hoje()) {
      badges += '<span class="badge rev">Revisar</span>';
    }
if (a.precisaRevisao) {
  badges += '<span class="badge rev">Revisão extra</span>';
}

if (a.banca) {
  badges += '<span class="badge caiu">' + a.banca + '</span>';
}

if (a.lido) {
  badges += '<span class="badge">Lido</span>';
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

    if (a.dataRevisao <= hoje()) {
      let rev = document.createElement("div");
      rev.className = "artigo";
      rev.innerHTML = `
        <b>${a.materia}</b><br>
        ${a.artigo.substring(0, 180)}...
      `;
      revisaoDiv.appendChild(rev);
    }
  });
}

function concluirDia() {
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
  progresso += metaDia;
  if (progresso > artigos.length) {
    progresso = artigos.length;
  }
  salvar();
  mostrar();
}

mostrar();
</script>
</body>
</html>
