<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Gestão de Clientes - Auto-Save Realtime</title>
    <script src="https://www.gstatic.com"></script>
    <script src="https://www.gstatic.com"></script>
    
    <style>
        :root { --ativo: #28a745; --pendente: #ffc107; --suspenso: #dc3545; --gray: #6c757d; --bg: #f4f6f9; --novo: #007bff; --dark: #2c3e50; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding: 20px; }
        .container { max-width: 1400px; margin: 0 auto; }
        .panel { background: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
        .card { background: white; padding: 20px; border-radius: 12px; margin-bottom: 15px; border-top: 8px solid var(--gray); display: flex; justify-content: space-between; align-items: center; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
        .card.is-new { border-top-color: var(--novo) !important; background-color: #f0f7ff; }
        .btn { border: none; padding: 8px 15px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 12px; transition: 0.2s; text-decoration: none; display: inline-block; }
        .btn-open { background: var(--dark); color: white; }
        .btn-back { background: var(--gray); color: white; margin-bottom: 20px; }
        .grid-ficha { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 20px; }
        details { background: white; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 10px; overflow: hidden; }
        summary { padding: 12px; cursor: pointer; font-weight: bold; background: #4b6584; color: white; list-style: none; display: flex; justify-content: space-between; }
        .menu-novo summary { background: #6c5ce7; }
        .month-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 5px; padding: 10px; }
        .month-item { border: 1px solid #eee; padding: 5px; display: flex; flex-direction: column; align-items: center; border-radius: 4px; font-size: 11px; }
        .dot { width: 14px; height: 14px; border-radius: 50%; border: none; cursor: pointer; margin-top: 5px; }
        .dot-pendente { background: var(--suspenso); }
        .dot-ok { background: var(--ativo); }
        .hidden { display: none; }
        .gerencia-input { width: 100%; box-sizing: border-box; margin-bottom: 8px; border: 1px solid #eee; min-height: 40px; resize: vertical; padding: 5px; font-family: inherit; }
    </style>
</head>
<body>

<div class="container">
    <div id="tela-lista">
        <div class="panel">
            <h2 style="margin-top:0">Novo Cadastro</h2>
            <div style="display:flex; gap:10px; flex-wrap:wrap">
                <input type="text" id="rs" placeholder="Razão Social" style="flex:2; padding:10px">
                <input type="text" id="cnpj" placeholder="CNPJ" style="flex:1; padding:10px">
                <input type="text" id="tel" placeholder="WhatsApp" style="flex:1; padding:10px">
                <input type="email" id="email" placeholder="E-mail" style="flex:1; padding:10px">
                <button class="btn" style="background:var(--ativo); color:white" onclick="novoCliente()">Cadastrar</button>
            </div>
        </div>
        <div id="lista-container"></div>
    </div>

    <div id="tela-ficha" class="hidden">
        <button class="btn btn-back" onclick="voltarLista()">← Voltar para Lista</button>
        <div id="ficha-container"></div>
    </div>
</div>

<script>
// CONFIGURAÇÃO FIREBASE (Suas chaves mantidas)
const firebaseConfig = {
  apiKey: "AIzaSyBUU5U5upiAqZfLb8ecuSMEcvkoboXe6rk",
  authDomain: "gestaoclientesequipe.firebaseapp.com",
  databaseURL: "https://gestaoclientesequipe-default-rtdb.firebaseio.com",
  projectId: "gestaoclientesequipe",
  storageBucket: "gestaoclientesequipe.firebasestorage.app",
  messagingSenderId: "1060376090119",
  appId: "1:1060376090119:web:3a1b38e7c57a6f0eb53ca2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();
const meses = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
const anos =;

db.ref('clientes').on('value', (snapshot) => {
    const container = document.getElementById('lista-container');
    if(!container) return;
    container.innerHTML = '';
    const data = snapshot.val();
    if (data) {
        Object.keys(data).forEach(id => {
            const c = data[id];
            const card = document.createElement('div');
            card.className = `card ${c.lido === false ? 'is-new' : ''}`;
            card.innerHTML = `
                <div><h3 style="margin:0">${c.rs}</h3><small>CNPJ: ${c.cnpj || '---'}</small></div>
                <div><button class="btn btn-open" onclick="abrirFicha('${id}')">ABRIR FICHA ↗</button></div>
            `;
            container.appendChild(card);
        });
    }
});

function novoCliente() {
    const rs = document.getElementById('rs').value;
    const cnpj = document.getElementById('cnpj').value;
    const tel = document.getElementById('tel').value;
    const email = document.getElementById('email').value;
    if(!rs) return alert("Razão Social obrigatória!");
    const id = Date.now();
    db.ref('clientes/' + id).set({ rs, cnpj, tel, email, status: 'ativo', lido: false });
    document.querySelectorAll('.panel input').forEach(i => i.value = '');
}

function abrirFicha(id) {
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

        const geraMenu = (titulo, tipo, extraClass = '') => `
            <details class="${extraClass}"><summary>${titulo}</summary>
                <div style="padding:10px">${anos.map(ano => `<details><summary style="background:#eee;color:#333;font-size:11px">Ano ${ano}</summary>${geraMeses(tipo,ano)}</details>`).join('')}</div>
            </details>`;

        document.getElementById('ficha-container').innerHTML = `
            <div class="panel">
                <h1 style="margin:0">${c.rs}</h1>
                <p><b>WhatsApp:</b> ${c.tel} | <b>E-mail:</b> ${c.email}</p>
            </div>
            <div class="grid-ficha">
                <div><h3>Base</h3>${geraMenu('📅 COLETA MENSAL','col')}${geraMenu('📺 TVpA','tv')}${geraMenu('☎️ STFC','stfc')}</div>
                <div><h3>Novas</h3>${geraMenu('📊 DICI','dici','menu-novo')}${geraMenu('📡 N_TVpA','n_tv','menu-novo')}${geraMenu('📞 N_STFC','n_stfc','menu-novo')}</div>
                <div><h3>Outros</h3>${geraMenu('💰 ECONÔMICA','eco','menu-novo')}${geraMenu('🏗️ INFRAESTRUTURA','infra','menu-novo')}
                    <div class="panel" style="margin-top:10px">
                        <b>Anotações (Auto-save):</b>
                        <textarea class="gerencia-input" placeholder="Digite aqui..." onchange="db.ref('clientes/${id}/notas').set(this.value)">${c.notas || ''}</textarea>
                    </div>
                </div>
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
