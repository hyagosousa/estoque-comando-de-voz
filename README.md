# estoque-comando-de-voz
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Controle de Estoque por Voz</title>
<style>
    body { font-family: Arial, sans-serif; background: #f5f5f5; margin: 0; padding: 0; }
    .container { max-width: 800px; margin: auto; padding: 20px; background: #fff; border-radius: 10px; margin-top: 20px; box-shadow: 0 0 10px rgba(0,0,0,0.1);}
    h2 { text-align: center; }
    button { padding: 12px 20px; margin-top: 10px; font-size: 16px; border: none; border-radius: 6px; background: #007bff; color: white; cursor: pointer;}
    button:hover { background: #005fcc; }
    table { width: 100%; border-collapse: collapse; margin-top: 20px;}
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }
    th { background: #eee; }
</style>
</head>
<body>

<div class="container">
    <h2>Controle de Estoque por Voz</h2>
    <button id="btnStart">🎤 Iniciar Gravação de Voz</button>
    <p id="status">Clique no botão e fale: "Saída 3 parafuso para Van01"</p>

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

const tabelaEstoque = document.querySelector("#tabelaEstoque tbody");
const tabelaHistorico = document.querySelector("#tabelaHistorico tbody");

function atualizarTabelas(){
    // Atualiza estoque
    tabelaEstoque.innerHTML = "";
    for(let produto in estoque){
        tabelaEstoque.innerHTML += `<tr>
            <td>${produto}</td>
            <td>${estoque[produto]}</td>
        </tr>`;
    }

    // Atualiza histórico
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

// Função simples para interpretar o comando de voz
function interpretarComando(texto){
    texto = texto.toLowerCase();
    // Exemplo de frase: "saída 3 parafuso para van01"
    let tipo = texto.includes("saída") ? "Saída" : "Entrada";
    let regex = /(\d+)\s+([a-z0-9\s]+)\s+para\s+([a-z0-9]+)/i;
    let match = texto.match(regex);
    if(match){
        let quantidade = parseInt(match[1]);
        let produto = match[2].trim();
        let carro = match[3].trim();

        // Atualiza estoque
        if(!estoque[produto]) estoque[produto] = 0;
        if(tipo === "Entrada"){
            estoque[produto] += quantidade;
        } else {
            estoque[produto] -= quantidade;
            if(estoque[produto] < 0) estoque[produto] = 0;
        }

        // Adiciona ao histórico
        historico.push({
            produto,
            quantidade,
            tipo,
            carro,
            data: new Date().toLocaleString()
        });

        atualizarTabelas();
    } else {
        alert("Não entendi o comando. Fale: Entrada/Saída + quantidade + produto + para + carro");
    }
}

// Inicializa Web Speech API
const btnStart = document.getElementById("btnStart");
btnStart.onclick = () => {
    if(!('webkitSpeechRecognition' in window)){
        alert("Seu navegador não suporta reconhecimento de voz!");
        return;
    }
    const recognition = new webkitSpeechRecognition();
    recognition.lang = "pt-BR";
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    recognition.onstart = () => {
        document.getElementById("status").innerText = "🎤 Ouvindo... fale agora!";
    };

    recognition.onresult = (event) => {
        let texto = event.results[0][0].transcript;
        document.getElementById("status").innerText = "Você disse: " + texto;
        interpretarComando(texto);
    };

    recognition.onerror = (event) => {
        alert("Erro: " + event.error);
    };

    recognition.onend = () => {
        document.getElementById("status").innerText += " | Clique novamente para novo comando.";
    };

    recognition.start();
};
</script>

</body>
</html>
