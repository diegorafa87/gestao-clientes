<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Clientes Oficial</title>
    <!-- Firebase SDK v8 (Versão mais estável para navegadores) -->
    <script src="https://www.gstatic.com"></script>
    <script src="https://www.gstatic.com"></script>
    
    <style>
        :root { --ativo: #28a745; --pendente: #ffc107; --suspenso: #dc3545; --gray: #6c757d; --bg: #f4f6f9; --novo: #007bff; --dark: #2c3e50; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--bg); margin: 0; padding: 20px; }
        .container { max-width: 1300px; margin: 0 auto; }
        .panel { background: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
        .card { background: white; padding: 20px; border-radius: 12px; margin-bottom: 15px; border-top: 8px solid var(--gray); display: flex; justify-content: space-between; align-items: center; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
        
        /* Card Novo (Azul) */
        .card.is-new { border-top-color: var(--novo) !important; background-color: #f0f7ff; border-left: 5px solid var(--novo); }
        
        .btn { border: none; padding: 10px 18px; border-radius: 6px; cursor: pointer; font-weight: bold; color: white; transition: 0.2s; }
        .btn-open { background: var(--dark); }
        .btn-back { background: var(--gray); margin-bottom: 15px; }

        .grid-ficha { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
        details { background: white; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 8px; overflow: hidden; }
        summary { padding: 12px; cursor: pointer; font-weight: bold; background: #4b6584; color: white; display: flex; justify-content: space-between; border-radius: 6px; list-style: none; }
        
        /* Menus Especiais (Roxo) */
        .menu-especial summary { background: #6c5ce7; }
        .ano-completo summary { background: var(--ativo) !important; }
        .ano-completo summary::after { content: ' ✅'; }

        .month-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 5px; padding: 10px; }
        .month-item { display: flex; flex-direction: column; align-items: center; font-size: 12px; font-weight: bold; }
        .dot { width: 18px; height: 18px; border-radius: 50%; border: none; cursor: pointer; margin-top: 4px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .dot-pendente { background: var(--suspenso); }
        .dot-ok { background: var(--ativo); }
        
        .hidden { display: none; }
        input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; margin: 5px; font-size: 14px; }
        #status-conexao { font-size: 12px; font-weight: bold; margin-bottom: 10px; }
    </style>
</head>
<body>

<div class="container">
    <div id="tela-lista">
        <div class="panel">
            <h2 style="margin-top:0">Novo Cadastro</h2>
            <div style="display:flex; flex-wrap:wrap; gap:5px">
                <input type="text" id="rs" placeholder="Razão Social" style="flex:2">
                <input type="text" id="cnpj" placeholder="CNPJ" style="flex:1">
                <input type="text" id="tel" placeholder="WhatsApp" style="flex:1">
                <input type="email" id="email" placeholder="E-mail" style="flex:1">
                <button class="btn" style="background:var(--ativo)" onclick="novoCliente()">Cadastrar Cliente</button>
            </div>
        </div>
        <div id="status-conexao" style="color: orange;">Iniciando conexão...</div>
        <div id="lista-container">Aguardando dados do Firebase...</div>
    </div>

    <div id="tela-ficha" class="hidden">
        <button class="btn btn-back" onclick="window.location.reload()">← Voltar para Lista</button>
        <div id="ficha-container"></div>
    </div>
</div>

<script>
// CONFIGURAÇÃO BASEADA NO SEU PROJETO (gestaooficial-f1527)
const firebaseConfig = {
    apiKey: "AIzaSyBUU5U5upiAqZfLb8ecuSMEcvkoboXe6rk", // Chave padrão que você usou
    authDomain: "gestaooficial-f1527.firebaseapp.com",
    databaseURL: "https://gestaooficial-f1527-default-rtdb.firebaseio.com",
    projectId: "gestaooficial-f1527",
    storageBucket: "gestaooficial-f1527.appspot.com",
    messagingSenderId: "1060376090119",
    appId: "1:1060376090119:web:3a1b38e7c57a6f0eb53ca2"
};

// Inicialização
if (!firebase.apps.length) {
    firebase.initializeApp(firebaseConfig);
}
const db = firebase.database();

const meses = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
const anos =;

// Monitor de Conexão
db.ref(".info/connected").on("value", (snap) => {
    const status = document.getElementById('status-conexao');
    if (snap.val() === true) {
        status.innerText = "● SISTEMA ONLINE (Firebase Conectado)";
        status.style.color = "green";
    } else {
        status.innerText = "○ SISTEMA OFFLINE (Verifique as Regras do Firebase)";
        status.style.color = "red";
    }
});

// Sincronizar Lista de Clientes
db.ref('clientes').on('value', (snap) => {
    const cont = document.getElementById('lista-container');
    cont.innerHTML = '';
    const data = snap.val();
    
    if(!data) {
        cont.innerHTML = "Nenhum cliente cadastrado no banco de dados.";
        return;
    }
    
    Object.keys(data).forEach(id => {
        const c = data[id];
        const card = document.createElement('div');
        // Regra do Card Azul para novos
        const classeEstilo = (c.lido === false) ? 'is-new' : '';
        card.className = `card ${classeEstilo}`;
        
        card.innerHTML = `
            <div>
                <h3 style="margin:0">${c.rs}</h3>
                <small>CNPJ: ${c.cnpj || '---'}</small>
            </div>
            <button class="btn btn-open" onclick="abrirFicha('${id}')">ABRIR FICHA</button>
        `;
        cont.appendChild(card);
    });
});

function novoCliente() {
    const rs = document.getElementById('rs').value;
    if(!rs) return alert("Razão Social é obrigatória!");
    
    const id = Date.now();
    db.ref('clientes/' + id).set({
        rs: rs,
        cnpj: document.getElementById('cnpj').value,
        tel: document.getElementById('tel').value,
        email: document.getElementById('email').value,
        lido: false, // Novo cliente sempre nasce azul
        status: 'ativo'
    }).then(() => {
        document.querySelectorAll('input').forEach(i => i.value = '');
    });
}

function abrirFicha(id) {
    // Marca como lido (remove o azul) ao abrir
    db.ref('clientes/' + id).update({ lido: true });
    
    document.getElementById('tela-lista').classList.add('hidden');
    document.getElementById('tela-ficha').classList.remove('hidden');

    db.ref('clientes/' + id).on('value', (snap) => {
        const c = snap.val();
        if(!c) return;

        const geraMeses = (tip, ano) => meses.map(m => {
            const st = (c.dots && c.dots[tip] && c.dots[tip][ano] && c.dots[tip][ano][m]) || 'pendente';
            return `<div class="month-item">${m}<button class="dot dot-${st}" onclick="toggleDot('${id}','${tip}','${ano}','${m}','${st}')"></button></div>`;
        }).join('');

        const geraMenu = (tit, tip, cl='') => {
            const anosHtml = anos.map(a => {
                const completo = meses.every(m => c.dots && c.dots[tip] && c.dots[tip][a] && c.dots[tip][a][m] === 'ok');
                return `<details class="${completo ? 'ano-completo' : ''}">
                            <summary style="color:#333;background:#eee">${a} ${completo ? '✅' : ''}</summary>
                            <div class="month-grid">${geraMeses(tip,a)}</div>
                        </details>`;
            }).join('');
            return `<details class="${cl}"><summary>${tit} <span>▼</span></summary><div style="padding:10px">${anosHtml}</div></details>`;
        };

        document.getElementById('ficha-container').innerHTML = `
            <div class="panel">
                <h1 style="margin:0">${c.rs}</h1>
                <p><b>WhatsApp:</b> ${c.tel || '---'} | <b>E-mail:</b> ${c.email || '---'}</p>
            </div>
            <div class="grid-ficha">
                <div>
                    <h3>Menus Base</h3>
                    ${geraMenu('📅 COLETA MENSAL','col')}
                    ${geraMenu('📺 TVpA','tv')}
                    ${geraMenu('☎️ STFC','stfc')}
                </div>
                <div>
                    <h3>Novas Coletas</h3>
                    ${geraMenu('📊 COLETA DICI','dici','menu-especial')}
                    ${geraMenu('📡 NOVA TVpA','n_tv','menu-especial')}
                    ${geraMenu('📞 NOVA STFC','n_stfc','menu-especial')}
                </div>
                <div>
                    <h3>Outros</h3>
                    ${geraMenu('💰 ECONÔMICA','eco','menu-especial')}
                    ${geraMenu('🏗️ INFRAESTRUTURA','infra','menu-especial')}
                </div>
            </div>`;
    });
}

function toggleDot(id, tip, ano, m, st) {
    const novoStatus = (st === 'pendente') ? 'ok' : 'pendente';
    db.ref(`clientes/${id}/dots/${tip}/${ano}/${m}`).set(novoStatus);
}
</script>
</body>
</html>
