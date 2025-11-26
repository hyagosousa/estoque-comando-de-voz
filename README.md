<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Controle de Estoque por Voz - Qualquer Material</title>
<style>
body { font-family: Arial, sans-serif; background: #f5f5f5; margin: 0; padding: 0; }
.container { max-width: 900px; margin: auto; padding: 20px; background: #fff; border-radius: 10px; margin-top: 20px; box-shadow: 0 0 10px rgba(0,0,0,0.1);}
h2 { text-align: center; margin-bottom: 10px;}
button { padding: 12px 20px; margin-top: 10px; font-size: 16px; border: none; border-radius: 6px; background: #007bff; color: white; cursor: pointer;}
button:hover { background: #005fcc; }
#btnConfirm { background: #28a745; margin-left: 10px;}
#btnConfirm:hover { background: #218838;}
table { width: 100%; border-collapse: collapse; margin-top: 20px;}
th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }
th { background: #eee; }
#status { margin-top: 10px; font-weight: bold; }
</style>
</head>
<body>

<div class="container">
<h2>Controle de Estoque por Voz - Qualquer Material</h2>
<button id="btnStart">🎤 Falar</button>
<button id="btnConfirm" style="display:none;">✅ Confirmar lançamento</button>
<p id="status">Clique em "Falar" e diga algo como: "3 parafusos 4 mm para van do Eduardo"</p>

<h3>Estoque Atual</h3>
<table id="tabelaEstoque">
<thead>
<tr>
<th>Produto</th>
<th>Quantidade</th>
</tr>
</thead>
<tbody></tbody>
</table>

<h3>Histórico</h3>
<table id="tabelaHistorico">
<thead>
<tr>
<th>Produto</th>
<th>Quantidade</th>
<th>Tipo</th>
<th>Carro</th>
<th>Data/Hora</th>
</tr>
</thead>
<tbody></tbody>
</table>
</div>

<script>
let estoque = {};
let historico = [];
let ultimoComando = null;

const tabelaEstoque = document.querySelector("#tabelaEstoque tbody");
const tabelaHistorico = document.querySelector("#tabelaHistorico tbody");
const status = document.getElementById("status");
const btnConfirm = document.getElementById("btnConfirm");

function atualizarTabelas(){
    // Estoque
    tabelaEstoque.innerHTML = "";
    for(let produto in estoque){
        tabelaEstoque.innerHTML += `<tr><td>${produto}</td><td>${estoque[produto]}</td></tr>`;
    }
    // Histórico
    tabelaHistorico.innerHTML = "";
    historico.forEach(h => {
        tabelaHistorico.innerHTML += `<tr>
            <td>${h.produto}</td>
            <td>${h.quantidade}</td>
            <td>${h.tipo}</td>
            <td>${h.carro}</td>
            <td>${h.data}</td>
        </tr>`;
    });
}

// Interpreta o comando de voz e guarda temporariamente
function interpretarComando(texto){
    texto = texto.toLowerCase();
    // Regex genérica para qualquer material com detalhes
    let regex = /(\d+)\s+(.+)\s+para\s+(.+)/i;
    let match = texto.match(regex);
    if(match){
        ultimoComando = {
            quantidade: parseInt(match[1]),
            produto: match[2].trim(),
            carro: match[3].trim(),
            tipo: "Saída" // padrão Saída, pode alterar se quiser Entrada
        };
        status.innerText = `Você disse: "${texto}". Clique em CONFIRMAR se estiver correto.`;
        btnConfirm.style.display = "inline-block";
    } else {
        ultimoComando = null;
        status.innerText = "Não consegui interpretar. Fale: quantidade + produto + para + carro";
        btnConfirm.style.display = "none";
    }
}

// Função principal para iniciar reconhecimento de voz
document.getElementById("btnStart").onclick = () => {
    if(!('webkitSpeechRecognition' in window)){
        alert("Seu navegador não suporta reconhecimento de voz! Use o Chrome ou Edge.");
        return;
    }

    const recognition = new webkitSpeechRecognition();
    recognition.lang = "pt-BR";
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    recognition.onstart = () => { status.innerText = "🎤 Ouvindo... fale agora!"; };

    recognition.onresult = (event) => {
        let texto = event.results[0][0].transcript;
        interpretarComando(texto);
    };

    recognition.onerror = (event) => {
        status.innerText = "❌ Erro no reconhecimento: " + event.error;
        btnConfirm.style.display = "none";
    };

    recognition.onend = () => {
        if(!ultimoComando) status.innerText += " | Clique em 'Falar' para tentar novamente.";
    };

    recognition.start();
};

// Botão de confirmação
btnConfirm.onclick = () => {
    if(ultimoComando){
        if(!estoque[ultimoComando.produto]) estoque[ultimoComando.produto] = 0;
        estoque[ultimoComando.produto] += ultimoComando.quantidade;

        historico.push({
            ...ultimoComando,
            data: new Date().toLocaleString()
        });

        atualizarTabelas();
        status.innerText = "✅ Lançamento confirmado!";
        btnConfirm.style.display = "none";
        ultimoComando = null;
    }
};
</script>

</body>
</html>


