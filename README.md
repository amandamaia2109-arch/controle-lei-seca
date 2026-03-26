<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Controle Lei Seca</title>
</head>

<body>

<h1>📚 Controle de Lei Seca</h1>

<h2>Adicionar Lei</h2>

<input id="nomeLei" placeholder="Nome da lei"><br><br>

<select id="materia">
  <option>Direito Penal</option>
  <option>Direito Civil</option>
  <option>Processo Penal</option>
  <option>Processo Civil</option>
  <option>Constitucional</option>
  <option>Administrativo</option>
</select><br><br>

<textarea id="textoLei" placeholder="Cole a lei aqui" rows="10" cols="50"></textarea><br><br>

<button onclick="extrair()">Extrair artigos</button>

<h2>Artigos</h2>
<ul id="lista"></ul>

<script>
let artigos = [];

function extrair() {
  let texto = document.getElementById("textoLei").value;

  let regex = /Art\\.\\s*\\d+[^]*?(?=Art\\.\\s*\\d+|$)/g;
  let encontrados = texto.match(regex);

  if (!encontrados) {
    alert("Nenhum artigo encontrado");
    return;
  }

  artigos = encontrados;
  mostrar();
}

function mostrar() {
  let lista = document.getElementById("lista");
  lista.innerHTML = "";

  artigos.forEach((art, i) => {
    let li = document.createElement("li");
    li.textContent = art.substring(0, 100) + "...";
    lista.appendChild(li);
  });
}
</script>

</body>
</html>
