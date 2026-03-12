<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Clientes - Equipe</title>
    <!-- Firebase SDKs (Versão estável por CDN) -->
    <script src="https://www.gstatic.com"></script>
    <script src="https://www.gstatic.com"></script>
    
    <style>
        :root { --ativo: #28a745; --pendente: #ffc107; --suspenso: #dc3545; --gray: #6c757d; --bg: #f4f6f9; --novo: #007bff; --dark: #2c3e50; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--bg); margin: 0; padding: 20px; color: #333; }
        .container { max-width: 1200px; margin: 0 auto; }
        
        /* Layout */
        .panel { background: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
        .card { background: white; padding: 20px; border-radius: 12px; margin-bottom: 15px; border-top: 8px solid var(--gray); display: flex; justify-content: space-between; align-items: center; box-shadow: 0 4px 10px rgba(0,0,0,0.05); cursor: pointer; }
        .card.is-new { border-top-color: var(--novo) !important; background-color: #f0f7ff; border-left: 5px solid var(--novo); }

        /* Botões */
        .btn { border: none; padding: 10px 18px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 13px; transition: 0.2s; }
        .btn-open { background: var(--dark); color: white; }
        .btn-back { background: var(--gray); color: white; margin-bottom: 20px; }

        /* Ficha Individual */
        .grid-ficha { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
        details { background: white; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 10px; }
        summary { padding: 15px; cursor: pointer; font-weight: bold; background: #4b6584; color: white; border-radius: 6px; display: flex; justify-content: space-between; list-style: none; }
        .menu-especial summary { background: #6c5ce7; }
        
        .month-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 15px; }
        .month-item { display: flex; flex-direction: column; align-items: center; font-size: 12px; font-weight: bold; }
        .dot { width: 20px; height: 20px; border-radius: 50%; border: none; cursor: pointer; margin-top: 5px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .dot-pendente { background: var(--suspenso); }
        .dot-ok { background: var(--ativo); }

        .hidden { display: none; }
        input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; font-size: 14px; }
    </style>
</head>
<body>

<div class="container">
    <!-- TELA 1: LISTA -->
    <div id="tela-lista">
        <div class="panel">
            <h2 style="margin-top:0">Cadastrar Cliente</h2>
            <div style="display:flex; gap:10px; flex-wrap:wrap">
                <input type="text" id="rs" placeholder="Razão Social" style="flex:2">
                <input type="text" id="cnpj" placeholder="CNPJ" style="flex:1">
                <button class="btn" style="background:var(--ativo); color:white" onclick="novoCliente()">Adicionar Cliente</button>
            </div>
        </div>
        <div id="lista-container">Aguardando conexão com banco de dados...</div>
    </div>

    <!-- TELA 2: FICHA -->
    <div id="tela-ficha" class="hidden">
        <button class="btn btn-back" onclick="voltarLista()">← Voltar para Lista</button>
        <div id="ficha-container"></div>
    </div>
</div>

<script>
// CONFIGURAÇÃO FIREBASE (Sincronizado com seu projeto)
const firebaseConfig = {
    apiKey: "AIzaSyBUU5U5upiAqZfLb8ecuSMEcvkoboXe6rk",
    authDomain: "gestaoclientesequipe.firebaseapp.com",
    databaseURL: "https://gestaoclientesequipe-default-rtdb.firebaseio.com",
    projectId: "gestaoclientesequipe",
    storageBucket: "gestaoclientesequipe.firebasestorage.app",
    messagingSenderId: "1060376090119",
    appId: "1:1060376090119:web:3a1b38e7c57a6f0eb53ca2"
};

// Inicialização segura
try {
    firebase.initializeApp(firebaseConfig);
    var db = firebase.database();
} catch (e) {
    console.error("Erro Firebase:", e);
    document.getElementById('lista-container').innerHTML = "Erro ao conectar ao Firebase. Verifique as configurações.";
}

const meses = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
const anos =;

// Escuta em tempo real
db.ref('clientes').on('value', (snapshot) => {
    const container = document.getElementById('lista-container');
    container.innerHTML = '';
    const data = snapshot.val();
    
    if (!data) {
        container.innerHTML = "Nenhum cliente cadastrado ainda.";
        return;
    }

    Object.keys(data).forEach(id => {
        const c = data[id];
        const card = document.createElement('div');
        card.className = `card ${c.lido === false ? 'is-new' : ''}`;
        card.innerHTML = `
            <div>
                <h3 style="margin:0">${c.rs}</h3>
                <small>CNPJ: ${c.cnpj || '---'}</small>
            </div>
            <button class="btn btn-open" onclick="abrirFicha('${id}')">ABRIR FICHA</button>
        `;
        container.appendChild(card);
    });
});

function novoCliente() {
    const rs = document.getElementById('rs').value;
    const cnpj = document.getElementById('cnpj').value;
    if(!rs) return alert("Digite o nome do cliente!");
    
    const id = Date.now();
    db.ref('clientes/' + id).set({
        rs: rs,
        cnpj: cnpj,
        lido: false,
        status: 'ativo'
    });
    document.getElementById('rs').value = '';
    document.getElementById('cnpj').value = '';
}

function abrirFicha(id) {
    db.ref('clientes/' + id).update({ lido: true });
    document.getElementById('tela-lista').classList.add('hidden');
    document.getElementById('tela-ficha').classList.remove('hidden');

    db.ref('clientes/' + id).on('value', (snapshot) => {
        const c = snapshot.val();
        if(!c) return;

        const geraMeses = (tipo, ano) => {
            return `<div class="month-grid">` + meses.map(m => {
                const status = (c.dots && c.dots[tipo] && c.dots[tipo][ano] && c.dots[tipo][ano][m]) || 'pendente';
                return `<div class="month-item">${m}<button class="dot dot-${status}" onclick="toggleDot('${id}','${tipo}','${ano}','${m}','${status}')"></button></div>`;
            }).join('') + `</div>`;
        };

        const geraMenu = (tit, tip, cl = '') => `
            <details class="${cl}"><summary>${tit} <span>▼</span></summary>
                <div style="padding:10px">${anos.map(a => `<details style="border:none;border-bottom:1px solid #eee"><summary style="background:transparent;color:#333;font-size:12px">${a}</summary>${geraMeses(tip,a)}</details>`).join('')}</div>
            </details>`;

        document.getElementById('ficha-container').innerHTML = `
            <div class="panel"><h1>${c.rs}</h1><p>CNPJ: ${c.cnpj}</p></div>
            <div class="grid-ficha">
                <div><h3>Básico</h3>${geraMenu('📅 COLETA MENSAL','col')}${geraMenu('📺 TVpA','tv')}${geraMenu('☎️ STFC','stfc')}</div>
                <div><h3>Especiais</h3>${geraMenu('📊 DICI','dici','menu-especial')}${geraMenu('📡 NOVA TVpA','n_tv','menu-especial')}${geraMenu('📞 NOVA STFC','n_stfc','menu-especial')}</div>
                <div><h3>Extra</h3>${geraMenu('💰 ECONÔMICA','eco','menu-especial')}${geraMenu('🏗️ INFRAESTRUTURA','infra','menu-especial')}</div>
            </div>`;
    });
}

function toggleDot(id, tipo, ano, mes, atual) {
    const novo = atual === 'pendente' ? 'ok' : 'pendente';
    db.ref(`clientes/${id}/dots/${tipo}/${ano}/${mes}`).set(novo);
}

function voltarLista() {
    document.getElementById('tela-lista').classList.remove('hidden');
    document.getElementById('tela-ficha').classList.add('hidden');
}
</script>
</body>
</html>
