<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Controle Lei Seca - ETAPA 4</title>

<style>
body { font-family: Arial; background:#f5f7fa; margin:20px; }
h1 { color:#1f3c88; }

.card {
  background:white;
  padding:15px;
  border-radius:10px;
  margin-bottom:20px;
  box-shadow:0 2px 8px rgba(0,0,0,0.08);
}

.artigo {
  border:1px solid #ddd;
  padding:10px;
  margin-top:10px;
  border-radius:8px;
}

.lido { background:#e8f5e9; }

.badge {
  padding:4px 8px;
  border-radius:999px;
  font-size:12px;
  margin-right:5px;
}

.inc { background:#ffe5e5; }
.caiu { background:#e7f7ea; }
.revisar { background:#fff3cd; }

button {
  margin-top:5px;
  padding:6px 10px;
  border:none;
  border-radius:6px;
  cursor:pointer;
}

</style>
</head>

<body>

<h1>📚 Controle Lei Seca - ETAPA 4</h1>

<div class="card">
<h2>Adicionar Lei</h2>

<input id="nomeLei" placeholder="Nome da lei"><br><br>

<select id="materia">
<option>Direito Penal</option>
<option>Direito Civil</option>
<option>Processual Penal</option>
<option>Processual Civil</option>
<option>Constitucional</option>
<option>Administrativo</option>
</select><br><br>

<textarea id="textoLei" rows="10" placeholder="Cole a lei"></textarea><br><br>

<button onclick="extrair()">Extrair</button>

</div>

<div class="card">
<h2>📅 Revisar hoje</h2>
<div id="revisarHoje"></div>
</div>

<div class="card">
<h2>🔥 Alta incidência</h2>
<div id="alta"></div>
</div>

<div class="card">
<h2>🎯 Já caiu</h2>
<div id="caiu"></div>
</div>

<div class="card">
<h2>Todos os artigos</h2>
<div id="lista"></div>
</div>

<script>

let artigos = JSON.parse(localStorage.getItem("artigos")) || [];

function salvar(){
  localStorage.setItem("artigos", JSON.stringify(artigos));
}

function hoje(){
  return new Date().toISOString().split("T")[0];
}

function somarDias(data, dias){
  let d = new Date(data);
  d.setDate(d.getDate()+dias);
  return d.toISOString().split("T")[0];
}

function extrair(){
  let nome = document.getElementById("nomeLei").value;
  let materia = document.getElementById("materia").value;
  let texto = document.getElementById("textoLei").value;

  texto = texto.replace(/Art\./g, "\nArt.");
  let partes = texto.split("\nArt.");

  partes.forEach(p=>{
    if(p.trim()){
      artigos.push({
        id: Date.now()+Math.random(),
        lei:nome,
        materia:materia,
        artigo:"Art."+p.trim(),
        lido:false,
        incidencia:false,
        caiu:false,
        revisao: hoje(),
        nivel:0
      });
    }
  });

  salvar();
  mostrar();
}

function marcarRevisado(id){
  artigos = artigos.map(a=>{
    if(a.id===id){
      a.nivel++;

      let dias = [1,3,7,15];
      let intervalo = dias[a.nivel] || 30;

      a.revisao = somarDias(hoje(), intervalo);
    }
    return a;
  });

  salvar();
  mostrar();
}

function toggleInc(id){
  artigos = artigos.map(a=>{
    if(a.id===id) a.incidencia=!a.incidencia;
    return a;
  });
  salvar();
  mostrar();
}

function toggleCaiu(id){
  artigos = artigos.map(a=>{
    if(a.id===id) a.caiu=!a.caiu;
    return a;
  });
  salvar();
  mostrar();
}

function mostrar(){

  let lista = document.getElementById("lista");
  let revisarHoje = document.getElementById("revisarHoje");
  let alta = document.getElementById("alta");
  let caiu = document.getElementById("caiu");

  lista.innerHTML="";
  revisarHoje.innerHTML="";
  alta.innerHTML="";
  caiu.innerHTML="";

  artigos.forEach(a=>{

    let div = document.createElement("div");
    div.className="artigo";

    div.innerHTML=`
      <b>${a.lei} - ${a.materia}</b><br>
      ${a.artigo}<br>

      ${a.incidencia ? '<span class="badge inc">Alta incidência</span>' : ''}
      ${a.caiu ? '<span class="badge caiu">Já caiu</span>' : ''}
      ${a.revisao<=hoje() ? '<span class="badge revisar">Revisar hoje</span>' : ''}

      <br>
      <button onclick="marcarRevisado(${a.id})">✔ Revisado</button>
      <button onclick="toggleInc(${a.id})">🔥 Incidência</button>
      <button onclick="toggleCaiu(${a.id})">🎯 Caiu</button>
    `;

    lista.appendChild(div);

    if(a.revisao<=hoje()){
      revisarHoje.appendChild(div.cloneNode(true));
    }

    if(a.incidencia){
      alta.appendChild(div.cloneNode(true));
    }

    if(a.caiu){
      caiu.appendChild(div.cloneNode(true));
    }

  });

}

mostrar();

</script>

</body>
</html>
