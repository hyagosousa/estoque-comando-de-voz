<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Controle de Estoque por Voz - Completo</title>
<style>
body { font-family: Arial, sans-serif; background: #f5f5f5; margin: 0; padding: 0;}
.container { max-width: 1000px; margin: auto; padding: 20px; background: #fff; border-radius: 10px; margin-top: 20px; box-shadow: 0 0 10px rgba(0,0,0,0.1);}
h2 { text-align: center; margin-bottom: 10px;}
button { padding: 10px 16px; margin: 5px; font-size: 14px; border: none; border-radius: 6px; background: #007bff; color: white; cursor: pointer;}
button:hover { background: #005fcc; }
.tab { display: none; }
.tab.active { display: block; }
input, select { padding: 6px; margin: 5px; border-radius: 4px; border: 1px solid #ccc; }
table { width: 100%; border-collapse: collapse; margin-top: 10px;}
th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
th { background: #eee; }
#statusEntrada, #statusSaida { font-weight: bold; margin-top: 10px;}
</style>
</head>
<body>

<div class="container">
<h2>Controle de Estoque por Voz - Completo</h2>

<!-- Abas -->
<div>
<button onclick="abrirAba('cadastro')">Cadastro de Produtos</button>
<button onclick="abrirAba('entrada')">Entrada de Produtos</button>
<button onclick="abrirAba('saida')">Retirada de Produtos</button>
</div>

<!-- Cadastro de Produtos -->
<div id="cadastro" class="tab active">
<h3>Cadastro de Produtos</h3>
<label>Nome do Produto:</label>
<input type="text" id="nomeProduto">
<label>Tipo:</label>
<select id="tipoProduto">
<option value="Elétrico">Elétrico</option>
<option value="Hidráulico">Hidráulico</option>
</select>
<button onclick="cadastrarProduto()">Cadastrar</button>

<h4>Produtos Cadastrados</h4>
<table id="tabelaProdutos">
<thead>
<tr><th>Produto</th><th>Tipo</th></tr>
</thead>
<tbody></tbody>
</table>
</div>

<!-- Entrada de Produtos -->
<div id="entrada" class="tab">
<h3>Entrada de Produtos</h3>
<button id="btnStartEntrada">🎤 Falar</button>
<button id="btnConfirmEntrada" style="display:none;">✅ Confirmar Entrada</button>
<p id="statusEntrada">Clique em Falar e diga: "5 metros de cabo elétrico"</p>

<h4>Estoque Atual</h4>
<table id="tabelaEstoque">
<thead>
<tr><th>Produto</th><th>Tipo</th><th>Quantidade</th><th>Tamanho/Detalhe</th></tr>
</thead>
<tbody></tbody>
</table>

<h4>Histórico de Entradas</h4>
<table id="tabelaHistoricoEntrada">
<thead>
<tr><th>Produto</th><th>Tipo</th><th>Quantidade</th><th>Tamanho/Detalhe</th><th>Data/Hora</th></tr>
</thead>
<tbody></tbody>
</table>
</div>

<!-- Retirada de Produtos -->
<div id="saida" class="tab">
<h3>Retirada de Produtos</h3>
<button id="btnStartSaida">🎤 Falar</button>
<button id="btnConfirmSaida" style="display:none;">✅ Confirmar Retirada</button>
<p id="statusSaida">Clique em Falar e diga: "3 parafusos hidráulico 4 mm para Eduardo"</p>

<h4>Estoque Atual</h4>
<table id="tabelaEstoqueSaida">
<thead>
<tr><th>Produto</th><th>Tipo</th><th>Quantidade</th><th>Tamanho/Detalhe</th></tr>
</thead>
<tbody></tbody>
</table>

<h4>Histórico de Saídas</h4>
<table id="tabelaHistoricoSaida">
<thead>
<tr><th>Produto</th><th>Tipo</th><th>Quantidade</th><th>Tamanho/Detalhe</th><th>Cliente</th><th>Data/Hora</th></tr>
</thead>
<tbody></tbody>
</table>
</div>

</div>

<script>
// ================== Variáveis Globais ==================
let produtos = []; 
let estoque = {};  
let historicoEntrada = [];
let historicoSaida = [];

// ================== Funções de Aba ==================
function abrirAba(nome){
  document.querySelectorAll(".tab").forEach(tab => tab.classList.remove("active"));
  document.getElementById(nome).classList.add("active");
}

// ================== Cadastro ==================
function cadastrarProduto(){
  let nome = document.getElementById("nomeProduto").value.trim().toLowerCase();
  let tipo = document.getElementById("tipoProduto").value;
  if(nome === "") { alert("Digite o nome do produto"); return; }
  produtos.push({nome,tipo});
  atualizarTabelaProdutos();
  document.getElementById("nomeProduto").value = "";
}

function atualizarTabelaProdutos(){
  let tbody = document.querySelector("#tabelaProdutos tbody");
  tbody.innerHTML = "";
  produtos.forEach(p => {
    tbody.innerHTML += `<tr><td>${p.nome}</td><td>${p.tipo}</td></tr>`;
  });
}

// ================== Entrada ==================
const statusEntrada = document.getElementById("statusEntrada");
let ultimoEntrada = null;

function atualizarTabelasEntrada(){
  const tbodyEstoque = document.querySelector("#tabelaEstoque tbody");
  tbodyEstoque.innerHTML = "";
  for(let p in estoque){
    tbodyEstoque.innerHTML += `<tr>
      <td>${p}</td><td>${estoque[p].tipo}</td><td>${estoque[p].quantidade}</td><td>${estoque[p].detalhe}</td>
    </tr>`;
  }
  const tbodyHist = document.querySelector("#tabelaHistoricoEntrada tbody");
  tbodyHist.innerHTML = "";
  historicoEntrada.forEach(h=>{
    tbodyHist.innerHTML += `<tr>
      <td>${h.produto}</td><td>${h.tipo}</td><td>${h.quantidade}</td><td>${h.detalhe}</td><td>${h.data}</td>
    </tr>`;
  });
}

// Interpreta comando de entrada
function interpretarComandoEntrada(texto){
  texto = texto.toLowerCase();
  // Pré-processar abreviações comuns
  texto = texto.replace(/\bmm\b/g, "milímetros").replace(/\bcm\b/g,"centímetros").replace(/\bm\b/g,"metros");
  
  // Regex melhorada para entrada
  let regex = /(\d+)\s*(metros|milímetros|centímetros)?\s*(?:de\s)?([\w\s]+?)\s*(elétrico|hidráulico)?/i;
  let match = texto.match(regex);
  if(match){
    let quantidade = parseInt(match[1]);
    let detalhe = match[2]? match[1]+" "+match[2] : match[1];
    let produtoNome = match[3].trim();
    let tipo = match[4]? match[4].charAt(0).toUpperCase()+match[4].slice(1) : "Desconhecido";

    // Ajuste produto cadastrado
    let produto = produtos.find(p=>produtoNome.includes(p.nome));
    if(produto) tipo = produto.tipo;

    ultimoEntrada = {produto: produto? produto.nome : produtoNome, tipo, quantidade, detalhe};
    statusEntrada.innerText = `Você disse: "${texto}". Clique em CONFIRMAR para lançar.`;
    document.getElementById("btnConfirmEntrada").style.display="inline-block";
  } else {
    statusEntrada.innerText="Não consegui interpretar. Fale: quantidade + produto + tipo + detalhe";
    document.getElementById("btnConfirmEntrada").style.display="none";
  }
}

// Confirmar entrada
document.getElementById("btnConfirmEntrada").onclick = () => {
  if(ultimoEntrada){
    let p = ultimoEntrada.produto;
    if(!estoque[p]) estoque[p]={tipo:ultimoEntrada.tipo,quantidade:0,detalhe:ultimoEntrada.detalhe};
    estoque[p].quantidade += ultimoEntrada.quantidade;
    estoque[p].detalhe = ultimoEntrada.detalhe;
    historicoEntrada.push({...ultimoEntrada, data:new Date().toLocaleString()});
    atualizarTabelasEntrada();
    statusEntrada.innerText="✅ Entrada confirmada!";
    document.getElementById("btnConfirmEntrada").style.display="none";
    ultimoEntrada=null;
  }
}

// Reconhecimento de voz entrada
document.getElementById("btnStartEntrada").onclick = () => {
  if(!('webkitSpeechRecognition' in window)){ alert("Navegador não suporta voz."); return;}
  const recognition = new webkitSpeechRecognition();
  recognition.lang = "pt-BR"; recognition.interimResults=false; recognition.maxAlternatives=1;
  recognition.onstart = ()=>{statusEntrada.innerText="🎤 Ouvindo... fale agora!"};
  recognition.onresult = e=>{interpretarComandoEntrada(e.results[0][0].transcript);};
  recognition.onerror = e=>{statusEntrada.innerText="Erro: "+e.error; document.getElementById("btnConfirmEntrada").style.display="none";}
  recognition.start();
}

// ================== Saída ==================
const statusSaida = document.getElementById("statusSaida");
let ultimoSaida=null;

function atualizarTabelasSaida(){
  const tbodyEstoque = document.querySelector("#tabelaEstoqueSaida tbody");
  tbodyEstoque.innerHTML="";
  for(let p in estoque){
    tbodyEstoque.innerHTML += `<tr>
      <td>${p}</td><td>${estoque[p].tipo}</td><td>${estoque[p].quantidade}</td><td>${estoque[p].detalhe}</td>
    </tr>`;
  }
  const tbodyHist = document.querySelector("#tabelaHistoricoSaida tbody");
  tbodyHist.innerHTML="";
  historicoSaida.forEach(h=>{
    tbodyHist.innerHTML += `<tr>
      <td>${h.produto}</td><td>${h.tipo}</td><td>${h.quantidade}</td><td>${h.detalhe}</td><td>${h.cliente}</td><td>${h.data}</td>
    </tr>`;
  });
}

// Interpretar comando de saída
function interpretarComandoSaida(texto){
  texto = texto.toLowerCase();
  texto = texto.replace(/\bmm\b/g,"milímetros").replace(/\bcm\b/g,"centímetros").replace(/\bm\b/g,"metros");
  let regex=/(\d+)\s*(metros|milímetros|centímetros)?\s*([\w\s]+?)\s*(elétrico|hidráulico)?\s*(\d+[\w\s]*)?\s+para\s+(.+)/i;
  let match = texto.match(regex);
  if(match){
    let quantidade = parseInt(match[1]);
    let detalhe = match[2]? match[1]+" "+match[2] : match[1];
    let produtoNome = match[3].trim();
    let tipo = match[4]? match[4].charAt(0).toUpperCase()+match[4].slice(1) : "Desconhecido";
    let cliente = match[6].trim();

    let produto = produtos.find(p=>produtoNome.includes(p.nome));
    if(produto) tipo = produto.tipo;

    ultimoSaida = {produto: produto? produto.nome : produtoNome, tipo, quantidade, detalhe, cliente};
    statusSaida.innerText=`Você disse: "${texto}". Clique em CONFIRMAR para lançar.`;
    document.getElementById("btnConfirmSaida").style.display="inline-block";
  } else {
    statusSaida.innerText="Não consegui interpretar. Fale: quantidade + produto + tipo + detalhe + para + cliente";
    document.getElementById("btnConfirmSaida").style.display="none";
  }
}

// Confirmar saída
document.getElementById("btnConfirmSaida").onclick = () => {
  if(ultimoSaida){
    let p = ultimoSaida.produto;
    if(!estoque[p]) estoque[p]={tipo:ultimoSaida.tipo,quantidade:0,detalhe:ultimoSaida.detalhe};
    estoque[p].quantidade -= ultimoSaida.quantidade;
    if(estoque[p].quantidade<0) estoque[p].quantidade=0;
    historicoSaida.push({...ultimoSaida, data:new Date().toLocaleString()});
    atualizarTabelasSaida();
    statusSaida.innerText="✅ Retirada confirmada!";
    document.getElementById("btnConfirmSaida").style.display="none";
    ultimoSaida=null;
  }
}

// Reconhecimento de voz saída
document.getElementById("btnStartSaida").onclick = () => {
  if(!('webkitSpeechRecognition' in window)){ alert("Navegador não suporta voz."); return;}
  const recognition = new webkitSpeechRecognition();
  recognition.lang="pt-BR"; recognition.interimResults=false; recognition.maxAlternatives=1;
  recognition.onstart=()=>{statusSaida.innerText="🎤 Ouvindo... fale agora!"};
  recognition.onresult=e=>{interpretarComandoSaida(e.results[0][0].transcript);};
  recognition.onerror=e=>{statusSaida.innerText="Erro: "+e.error; document.getElementById("btnConfirmSaida").style.display="none";}
  recognition.start();
}
</script>

</body>
</html>


