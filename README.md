<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Controle Lei Seca - ETAPA 5</title>

<style>
body { font-family: Arial; background:#f5f7fa; margin:20px; }
.card { background:white; padding:15px; border-radius:10px; margin-bottom:20px; }
.artigo { border:1px solid #ddd; padding:10px; margin-top:10px; border-radius:8px; }
.badge { padding:4px 8px; border-radius:6px; font-size:12px; margin-right:5px; }
.inc { background:#ffe5e5; }
.caiu { background:#e7f7ea; }
</style>

</head>

<body>

<h1>📚 Controle Lei Seca - ETAPA 5</h1>

<div class="card">
<h2>Adicionar Lei</h2>

<input id="nomeLei" placeholder="Nome da lei"><br><br>

<select id="materia">
<option>Penal</option>
<option>Civil</option>
<option>Proc Penal</option>
<option>Proc Civil</option>
<option>Constitucional</option>
<option>Administrativo</option>
</select><br><br>

<textarea id="textoLei" rows="10"></textarea><br><br>

<button onclick="extrair()">Extrair</button>
</div>

<div class="card">
<h2>⚙️ Configurar Cronograma</h2>

Artigos por dia:
<input id="meta" type="number" value="20">

<button onclick="gerarCronograma()">Gerar</button>

</div>

<div class="card">
<h2>📅 Hoje estudar</h2>
<div id="hoje"></div>
</div>

<div class="card">
<h2>📅 Revisar hoje</h2>
<div id="revisar"></div>
</div>

<div class="card">
<h2>Todos os artigos</h2>
<div id="lista"></div>
</div>

<script>

let artigos = JSON.parse(localStorage.getItem("artigos")) || [];
let cronograma = JSON.parse(localStorage.getItem("cronograma")) || [];

function salvar(){
  localStorage.setItem("artigos", JSON.stringify(artigos));
  localStorage.setItem("cronograma", JSON.stringify(cronograma));
}

function hoje(){
  return new Date().toISOString().split("T")[0];
}

function extrair(){
  let nome = document.getElementById("nomeLei").value;
  let materia = document.getElementById("materia").value;
  let texto = document.getElementById("textoLei").value;

  texto = texto.replace(/Art\./g,"\nArt.");
  let partes = texto.split("\nArt.");

  partes.forEach(p=>{
    if(p.trim()){
      artigos.push({
        id:Date.now()+Math.random(),
        lei:nome,
        materia:materia,
        artigo:"Art."+p.trim(),
        revisao:hoje(),
        nivel:0
      });
    }
  });

  salvar();
  mostrar();
}

function gerarCronograma(){

  let meta = parseInt(document.getElementById("meta").value);

  let fila = [...artigos];

  cronograma = [];

  let dia = 0;

  while(fila.length > 0){
    let bloco = fila.splice(0, meta);

    cronograma.push({
      dia: dia,
      artigos: bloco.map(a=>a.id)
    });

    dia++;
  }

  salvar();
  mostrar();
}

function mostrar(){

  let hojeDiv = document.getElementById("hoje");
  let revisarDiv = document.getElementById("revisar");
  let lista = document.getElementById("lista");

  hojeDiv.innerHTML="";
  revisarDiv.innerHTML="";
  lista.innerHTML="";

  let diaAtual = cronograma[0];

  if(diaAtual){
    diaAtual.artigos.forEach(id=>{
      let a = artigos.find(x=>x.id===id);
      let div = document.createElement("div");
      div.className="artigo";
      div.innerText = a.materia + " - " + a.artigo;
      hojeDiv.appendChild(div);
    });
  }

  artigos.forEach(a=>{

    let div = document.createElement("div");
    div.className="artigo";

    div.innerHTML = `
      <b>${a.materia}</b><br>
      ${a.artigo}
    `;

    lista.appendChild(div);

    if(a.revisao <= hoje()){
      let r = document.createElement("div");
      r.className="artigo";
      r.innerText = a.artigo;
      revisarDiv.appendChild(r);
    }

  });

}

mostrar();

</script>

</body>
</html>
