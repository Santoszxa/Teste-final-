<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema PCP — Produção Industrial</title>
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<style>
:root{--n950:#020b16;--n900:#061020;--n850:#09182e;--n800:#0c1f3c;--n750:#0f274a;--n700:#132f58;--n600:#1a3d6e;}
*{box-sizing:border-box;margin:0;padding:0;}
body{background:var(--n950);font-family:'ui-sans-serif',system-ui,sans-serif;color:#e2e8f0;min-height:100vh;}
.card{background:var(--n900);border:1px solid var(--n700);border-radius:12px;}
.card2{background:var(--n850);border:1px solid var(--n700);border-radius:10px;}
.inp{width:100%;background:var(--n950);border:1px solid var(--n700);border-radius:8px;padding:7px 11px;color:#e2e8f0;font-size:12px;outline:none;transition:.2s;}
.inp:focus{border-color:#3b82f6;}
.inp-sm{padding:4px 8px;font-size:11px;border-radius:6px;}
.btn{padding:8px 18px;border-radius:9px;border:none;cursor:pointer;font-weight:700;font-size:12px;transition:.2s;}
.btn-blue{background:#3b82f6;color:#fff;}.btn-blue:hover{background:#2563eb;}
.btn-gray{background:var(--n700);color:#93c5fd;}.btn-gray:hover{background:var(--n600);}
.btn-green{background:#059669;color:#fff;}.btn-green:hover{background:#047857;}
.btn-sm{padding:5px 12px;font-size:11px;border-radius:7px;}
.tab-btn{padding:9px 16px;border-radius:8px;font-weight:700;font-size:12px;color:#64748b;cursor:pointer;border:none;background:transparent;transition:.2s;white-space:nowrap;}
.tab-btn:hover{background:rgba(59,130,246,.08);color:#93c5fd;}
.tab-btn.active{background:rgba(59,130,246,.15);color:#3b82f6;border-bottom:2px solid #3b82f6;}
.modal-ov{position:fixed;inset:0;background:rgba(0,0,0,.88);backdrop-filter:blur(8px);z-index:300;display:flex;align-items:center;justify-content:center;padding:12px;}
.modal-box{background:var(--n850);border:1px solid var(--n600);border-radius:16px;width:100%;max-height:94vh;overflow-y:auto;}
.kpi{background:linear-gradient(135deg,var(--n800),var(--n900));border:1px solid var(--n700);border-radius:12px;padding:16px;text-align:center;}
.plano-A{border-left:3px solid #ef4444;background:rgba(239,68,68,.07);border-radius:8px;padding:12px;}
.plano-M{border-left:3px solid #f59e0b;background:rgba(245,158,11,.07);border-radius:8px;padding:12px;}
.plano-B{border-left:3px solid #10b981;background:rgba(16,185,129,.07);border-radius:8px;padding:12px;}
.robo-card{background:var(--n850);border:1px solid var(--n700);border-radius:10px;padding:10px;transition:.2s;}
.robo-card:hover{border-color:#3b82f6;}
.efic-badge{display:inline-block;padding:2px 8px;border-radius:12px;font-size:10px;font-weight:700;}
.upload-zone{border:2px dashed var(--n600);border-radius:12px;padding:24px;text-align:center;cursor:pointer;transition:.2s;}
.upload-zone:hover{border-color:#3b82f6;background:rgba(59,130,246,.04);}
.ai-spin{animation:spin 1s linear infinite;}
@keyframes spin{to{transform:rotate(360deg)}}
::-webkit-scrollbar{width:5px;height:5px;}
::-webkit-scrollbar-track{background:var(--n950);}
::-webkit-scrollbar-thumb{background:var(--n700);border-radius:3px;}
.sec-title{font-size:12px;font-weight:800;text-transform:uppercase;letter-spacing:.05em;margin-bottom:10px;color:#93c5fd;}
.turn-chip{padding:5px 12px;border-radius:20px;font-size:11px;font-weight:700;cursor:pointer;border:1px solid var(--n700);background:var(--n850);color:#64748b;transition:.2s;}
.turn-chip:hover{border-color:#3b82f6;color:#93c5fd;}
.turn-chip.act{background:rgba(59,130,246,.2);border-color:#3b82f6;color:#60a5fa;}
</style>
</head>
<body>

<!-- MODAIS -->
<div id="m-login" class="modal-ov hidden">
<div class="modal-box p-6 space-y-4" style="max-width:340px">
  <h2 class="text-base font-extrabold text-white text-center">🔐 Acesso PCP</h2>
  <div class="space-y-3">
    <div><label class="text-xs font-bold text-blue-300 uppercase">Usuário</label><input id="lg-user" class="inp mt-1" placeholder="Usuário" onkeydown="if(event.key==='Enter')login()"></div>
    <div><label class="text-xs font-bold text-blue-300 uppercase">Senha</label><input id="lg-pass" type="password" class="inp mt-1" placeholder="Senha" onkeydown="if(event.key==='Enter')login()"></div>
    <p id="lg-err" class="text-xs text-red-400 hidden">❌ Credenciais incorretas.</p>
  </div>
  <div class="flex gap-2"><button onclick="login()" class="btn btn-blue flex-1">Entrar</button><button onclick="fM('m-login')" class="btn btn-gray">Cancelar</button></div>
</div></div>

<div id="m-turno" class="modal-ov hidden">
<div class="modal-box p-6 space-y-4" style="max-width:560px">
  <div class="flex justify-between items-center"><h2 class="text-base font-extrabold text-white">🔄 Novo Turno</h2><button onclick="fM('m-turno')" class="text-blue-400 text-lg font-bold">✕</button></div>
  <div class="grid grid-cols-2 gap-3">
    <div class="col-span-2"><label class="text-xs font-bold text-blue-300 uppercase">Líder</label><input id="nt-lider" class="inp mt-1" placeholder="Nome do líder"></div>
    <div><label class="text-xs font-bold text-blue-300 uppercase">Data</label><input id="nt-data" type="date" class="inp mt-1"></div>
    <div><label class="text-xs font-bold text-blue-300 uppercase">Turno</label>
      <select id="nt-turno" class="inp mt-1"><option>1º Turno (06:00–14:00)</option><option>2º Turno (14:00–22:00)</option><option>3º Turno (22:00–06:00)</option></select>
    </div>
    <div><label class="text-xs font-bold text-amber-400 uppercase">Meta Estufas</label><input id="nt-me" type="number" class="inp mt-1" value="6725"></div>
    <div><label class="text-xs font-bold text-cyan-400 uppercase">Meta Glicerina</label><input id="nt-mg" type="number" class="inp mt-1" value="7852"></div>
    <div class="col-span-2"><label class="text-xs font-bold text-red-400 uppercase">Ausências</label><input id="nt-aus" class="inp mt-1" placeholder="Carlos (falta), Yuri (atestado)..."></div>
  </div>
  <div class="flex gap-2"><button onclick="criarTurno()" class="btn btn-blue flex-1">✅ Criar Novo Turno</button><button onclick="fM('m-turno')" class="btn btn-gray">Cancelar</button></div>
</div></div>

<div id="m-import" class="modal-ov hidden">
<div class="modal-box p-6 space-y-4" style="max-width:560px">
  <div class="flex justify-between items-center"><h2 class="text-base font-extrabold text-white">📥 Importar Arquivo</h2><button onclick="fM('m-import')" class="text-blue-400 text-lg">✕</button></div>
  <p class="text-xs text-blue-400/60">IA analisa e preenche cada campo no lugar correto. Ocorrências filtradas por grau de importância.</p>
  <div class="upload-zone" onclick="document.getElementById('fi').click()" ondragover="event.preventDefault();this.style.borderColor='#3b82f6'" ondragleave="this.style.borderColor=''" ondrop="dropFile(event)">
    <div class="text-3xl mb-2">📂</div>
    <p class="text-blue-300 font-bold text-sm">Clique ou arraste</p>
    <p class="text-blue-400/50 text-xs mt-1">JPG · PNG · PDF · XLSX · CSV</p>
    <input type="file" id="fi" class="hidden" accept="image/*,.pdf,.xlsx,.xls,.csv" onchange="processFile(this.files[0])">
  </div>
  <div id="im-loading" class="hidden text-center py-4">
    <div class="text-3xl ai-spin inline-block mb-2">⚙️</div>
    <p class="text-blue-300 font-bold text-sm">IA analisando...</p>
    <p id="im-step" class="text-xs text-blue-400/50 mt-1">Lendo arquivo...</p>
  </div>
  <div id="im-result" class="hidden space-y-3">
    <p class="text-xs font-bold text-emerald-400">✅ Dados extraídos:</p>
    <div id="im-campos" class="space-y-1 max-h-56 overflow-y-auto"></div>
    <button onclick="aplicarImport()" class="btn btn-blue w-full">✅ Aplicar ao Relatório</button>
  </div>
</div></div>

<div id="m-plano" class="modal-ov hidden">
<div class="modal-box p-6 space-y-4" style="max-width:680px">
  <div class="flex justify-between items-center"><h2 class="text-base font-extrabold text-white">🎯 Plano de Ação — IA</h2><button onclick="fM('m-plano')" class="text-blue-400 text-lg">✕</button></div>
  <div id="pl-loading" class="hidden text-center py-6"><div class="text-4xl ai-spin inline-block mb-2">🤖</div><p class="text-blue-300 font-bold">Gerando planos...</p></div>
  <div id="pl-lista" class="space-y-3 max-h-[58vh] overflow-y-auto pr-1"></div>
  <div class="flex gap-2">
    <button onclick="gerarPlanos()" class="btn btn-blue flex-1">🔄 Regenerar</button>
    <button onclick="incluirPlanos()" class="btn btn-green">📋 Incluir</button>
    <button onclick="fM('m-plano')" class="btn btn-gray">Fechar</button>
  </div>
</div></div>

<!-- PRINCIPAL -->
<div class="max-w-screen-2xl mx-auto p-3 space-y-3">

<!-- HEADER -->
<div class="card p-4 border-l-4 border-blue-500 flex flex-col md:flex-row justify-between items-start md:items-center gap-3">
  <div>
    <h1 class="text-xl font-extrabold text-white tracking-tight">SISTEMA PCP — PRODUÇÃO INDUSTRIAL</h1>
    <div class="flex flex-wrap gap-4 mt-1.5 text-xs text-blue-300">
      <span>Líder: <strong id="h-lider" class="text-cyan-400">—</strong></span>
      <span>Data: <strong id="h-data" class="text-cyan-400 font-mono">—</strong></span>
      <span>Turno: <strong id="h-turno" class="text-cyan-400">—</strong></span>
      <span>Eficiência: <strong id="h-efic" class="text-green-400">0%</strong></span>
    </div>
  </div>
  <div class="flex items-center gap-2">
    <span id="sync-dot" class="flex items-center gap-1.5 text-[10px] text-emerald-400 font-semibold bg-emerald-950/40 px-3 py-1.5 rounded-full border border-emerald-800/50">
      <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse"></span>Salvo
    </span>
    <div class="relative">
      <button id="btn3pts" onclick="toggle3pts()" class="text-slate-700 hover:text-slate-500 font-black text-xl px-2 py-0.5 rounded select-none">···</button>
      <div id="menu3pts" class="hidden absolute right-0 top-8 card p-2 space-y-0.5 z-50 min-w-[180px] shadow-2xl">
        <button onclick="abrirLogin()" class="w-full text-left text-xs text-blue-300 hover:text-white px-3 py-2 rounded hover:bg-blue-900/30" id="btn-lm">🔐 Admin PCP</button>
        <button onclick="logout()" id="btn-logout" class="hidden w-full text-left text-xs text-red-400 px-3 py-2 rounded hover:bg-red-900/20">🚪 Sair do Admin</button>
      </div>
    </div>
  </div>
</div>

<!-- HISTÓRICO -->
<div class="flex items-center gap-2 flex-wrap">
  <span class="text-xs text-blue-400/50 font-bold uppercase">Turnos:</span>
  <div id="hist-turnos" class="flex gap-2 flex-wrap"></div>
  <button id="btn-novo-turno" onclick="oM('m-turno')" class="btn btn-sm btn-blue hidden">+ Novo Turno</button>
</div>

<!-- ABAS -->
<div class="card p-1.5 flex gap-1 overflow-x-auto">
  <button onclick="aba('consolidado')" id="tab-consolidado" class="tab-btn active">📊 Consolidado</button>
  <button onclick="aba('glicerina')"   id="tab-glicerina"   class="tab-btn">🧪 Glicerina</button>
  <button onclick="aba('estufas')"     id="tab-estufas"     class="tab-btn">🔥 Estufas</button>
  <button onclick="aba('planos')"      id="tab-planos"      class="tab-btn">🎯 Plano de Ação</button>
  <button onclick="aba('comparacao')"  id="tab-comparacao"  class="tab-btn">📈 Comparativo</button>
  <button onclick="aba('admin')"       id="tab-admin"       class="tab-btn hidden">⚙️ Admin PCP</button>
  <div class="flex-1"></div>
  <button onclick="oM('m-import')" class="btn btn-sm btn-gray">📥 Importar</button>
  <button onclick="gerarPDF()" class="btn btn-sm btn-blue">📄 PDF</button>
</div>

<!-- ABA CONSOLIDADO -->
<div id="aba-consolidado" class="space-y-4">
  <div class="grid grid-cols-2 md:grid-cols-5 gap-3">
    <div class="kpi"><p class="text-xs font-bold text-blue-300 uppercase">Meta Total</p><p id="k-meta" class="text-3xl font-extrabold text-amber-400 mt-1">0</p><p class="text-[10px] text-blue-400/40 mt-1">peças</p></div>
    <div class="kpi"><p class="text-xs font-bold text-blue-300 uppercase">Realizado</p><p id="k-real" class="text-3xl font-extrabold text-blue-400 mt-1">0</p><p class="text-[10px] text-blue-400/40 mt-1">peças</p></div>
    <div class="kpi"><p class="text-xs font-bold text-blue-300 uppercase">Eficiência</p><p id="k-efic" class="text-3xl font-extrabold text-green-400 mt-1">0%</p><div class="w-full h-1.5 rounded-full mt-2" style="background:var(--n700)"><div id="k-efic-bar" class="h-1.5 rounded-full bg-green-500 transition-all" style="width:0%"></div></div></div>
    <div class="kpi"><p class="text-xs font-bold text-blue-300 uppercase">Refugo Total</p><p id="k-refugo" class="text-3xl font-extrabold text-red-400 mt-1">0</p><p class="text-[10px] text-blue-400/40 mt-1">peças</p></div>
    <div class="kpi"><p class="text-xs font-bold text-blue-300 uppercase">Gap Meta</p><p id="k-gap" class="text-3xl font-extrabold text-orange-400 mt-1">0</p><p class="text-[10px] text-blue-400/40 mt-1">a atingir</p></div>
  </div>
  <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
    <div class="card p-4"><p class="sec-title">Meta vs Realizado</p><div style="height:220px"><canvas id="cg-meta-setor"></canvas></div></div>
    <div class="card p-4">
      <div class="flex justify-between items-center mb-2"><p class="sec-title mb-0">Ocorrências Críticas</p><span class="text-[9px] text-blue-400/30">por grau</span></div>
      <div class="grid grid-cols-2 gap-1 mb-2" id="oc-legend"></div>
      <div style="height:140px"><canvas id="cg-ocorrencias"></canvas></div>
      <div class="grid grid-cols-4 gap-1 mt-2" id="oc-inputs"></div>
    </div>
    <div class="card p-4"><p class="sec-title">Distribuição do Refugo</p><div style="height:200px"><canvas id="cg-refugo"></canvas></div></div>
  </div>
  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
    <div class="card p-4" style="border-color:rgba(239,68,68,.3);background:rgba(239,68,68,.04);">
      <div class="flex justify-between items-center mb-3"><p class="sec-title mb-0 text-red-400">🚨 Principais Alertas</p><span class="text-[9px] text-red-400/50">por grau de importância</span></div>
      <div id="d-alertas" class="text-xs leading-relaxed space-y-1.5 min-h-[80px]"><p class="text-blue-400/40 italic">Alertas aparecerão após importação...</p></div>
    </div>
    <div class="card p-4" style="border-color:rgba(16,185,129,.3);background:rgba(16,185,129,.04);">
      <div class="flex justify-between items-center mb-3"><p class="sec-title mb-0 text-emerald-400">✅ Recomendações Gestão</p><span class="text-[9px] text-emerald-400/40 animate-pulse">🤖 IA</span></div>
      <div id="d-recomendacoes" class="text-xs leading-relaxed space-y-1.5 min-h-[80px]"><p class="text-blue-400/40 italic">Recomendações geradas automaticamente...</p></div>
    </div>
  </div>
</div>

<!-- ABA GLICERINA -->
<div id="aba-glicerina" class="space-y-4 hidden">
  <div class="card p-4 border-l-4 border-cyan-500">
    <div class="flex flex-wrap justify-between items-center mb-3 gap-3">
      <h2 class="text-lg font-bold text-cyan-400">🧪 Setor Glicerina — 13 Robôs</h2>
      <div class="flex gap-4 text-xs">
        <span class="text-blue-300">Meta: <strong id="gm-disp" class="text-amber-400">0</strong></span>
        <span class="text-blue-300">Real: <strong id="gr-disp" class="text-cyan-400">0</strong></span>
        <span class="text-blue-300">Efic: <strong id="ge-disp" class="text-green-400">0%</strong></span>
        <span class="text-blue-300">Refugo: <strong id="grf-disp" class="text-red-400">0</strong></span>
      </div>
    </div>
    <div class="w-full h-2 rounded-full mb-4" style="background:var(--n700)"><div id="glic-efic-bar" class="h-2 rounded-full bg-green-500 transition-all" style="width:0%"></div></div>
    <div id="glic-robos-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-7 gap-3 mb-5"></div>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mt-2">
      <div class="card2 p-4 md:col-span-2"><p class="sec-title text-cyan-300">Produção por Robô</p><div style="height:220px"><canvas id="cg-glic-robos"></canvas></div></div>
      <div class="card2 p-4"><p class="sec-title text-cyan-300">Eficiência por Robô (%)</p><div style="height:220px"><canvas id="cg-glic-efic"></canvas></div></div>
      <div class="card2 p-4"><p class="sec-title text-cyan-300">Meta vs Realizado</p><div style="height:220px"><canvas id="cg-glic-meta"></canvas></div></div>
    </div>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
      <div class="card2 p-4">
        <p class="sec-title text-cyan-300">Ocorrências Glicerina</p>
        <div id="d-glic-ocorr" class="text-xs leading-relaxed space-y-1.5 min-h-[70px]"><p class="text-blue-400/40 italic">Sem ocorrências.</p></div>
      </div>
      <div class="card2 p-4"><p class="sec-title text-cyan-300">Produto vs Produção</p><div style="height:160px"><canvas id="cg-glic-produto"></canvas></div></div>
    </div>
  </div>
</div>

<!-- ABA ESTUFAS -->
<div id="aba-estufas" class="space-y-4 hidden">
  <div class="card p-4 border-l-4 border-amber-500">
    <div class="flex flex-wrap justify-between items-center mb-3 gap-3">
      <h2 class="text-lg font-bold text-amber-400">🔥 Setor Estufas — 23 posições</h2>
      <div class="flex gap-4 text-xs">
        <span class="text-blue-300">Meta: <strong id="em-disp" class="text-amber-400">0</strong></span>
        <span class="text-blue-300">Real: <strong id="er-disp" class="text-amber-400">0</strong></span>
        <span class="text-blue-300">Efic: <strong id="ee-disp" class="text-green-400">0%</strong></span>
        <span class="text-blue-300">Refugo: <strong id="erf-disp" class="text-red-400">0</strong></span>
      </div>
    </div>
    <div class="w-full h-2 rounded-full mb-4" style="background:var(--n700)"><div id="est-efic-bar" class="h-2 rounded-full bg-amber-500 transition-all" style="width:0%"></div></div>

    <h3 class="text-sm font-bold text-amber-300 mb-2">Estufa 02 — <span class="text-xs text-blue-400/60">13 posições · 6 operadores</span></h3>
    <div id="est02-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 lg:grid-cols-7 gap-2 mb-3"></div>
    <div class="card2 p-3 mb-5">
      <p class="text-xs font-bold text-amber-300 uppercase mb-2">Operadores Estufa 02 (6)</p>
      <div id="est02-ops" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-2"></div>
    </div>

    <h3 class="text-sm font-bold text-teal-300 mb-2">Estufa 03 — <span class="text-xs text-blue-400/60">10 posições · 4 operadores</span></h3>
    <div id="est03-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 lg:grid-cols-6 gap-2 mb-3"></div>
    <div class="card2 p-3 mb-5">
      <p class="text-xs font-bold text-teal-300 uppercase mb-2">Operadores Estufa 03 (4)</p>
      <div id="est03-ops" class="grid grid-cols-2 md:grid-cols-4 gap-2"></div>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      <div class="card2 p-4 md:col-span-2"><p class="sec-title text-amber-300">Estufa 02 vs Estufa 03</p><div style="height:220px"><canvas id="cg-est-split"></canvas></div></div>
      <div class="card2 p-4"><p class="sec-title text-amber-300">Eficiência por Posição</p><div style="height:220px"><canvas id="cg-est-efic"></canvas></div></div>
      <div class="card2 p-4"><p class="sec-title text-amber-300">Pareto Ocorrências</p><div style="height:220px"><canvas id="cg-est-pareto"></canvas></div></div>
    </div>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
      <div class="card2 p-4"><p class="sec-title text-amber-300">Ocorrências Estufas</p><div id="d-est-ocorr" class="text-xs leading-relaxed space-y-1.5 min-h-[70px]"><p class="text-blue-400/40 italic">Sem ocorrências.</p></div></div>
      <div class="card2 p-4"><p class="sec-title text-amber-300">Produto vs Produção</p><div style="height:160px"><canvas id="cg-est-produto"></canvas></div></div>
      <div class="card2 p-4"><p class="sec-title text-amber-300">Meta vs Realizado Estufas</p><div style="height:160px"><canvas id="cg-est-meta"></canvas></div></div>
    </div>
  </div>
</div>

<!-- ABA PLANOS -->
<div id="aba-planos" class="space-y-4 hidden">
  <div class="card p-4">
    <div class="flex flex-wrap justify-between items-center mb-4 gap-3">
      <h2 class="text-lg font-bold text-white">🎯 Plano de Ação</h2>
      <button onclick="oM('m-plano');gerarPlanos()" class="btn btn-blue btn-sm">🤖 Gerar com IA</button>
    </div>
    <div id="pl-aba" class="space-y-3 mb-6"><p class="text-xs text-blue-400/40 italic text-center py-8">Clique em "Gerar com IA" para criar planos baseados nas ocorrências.</p></div>
    <div id="pl-charts" class="hidden space-y-4">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="card2 p-4"><p class="sec-title">Prioridades</p><div style="height:200px"><canvas id="cg-pl-prior"></canvas></div></div>
        <div class="card2 p-4"><p class="sec-title">Por Responsável</p><div style="height:200px"><canvas id="cg-pl-resp"></canvas></div></div>
        <div class="card2 p-4"><p class="sec-title">Status das Ações</p><div style="height:200px"><canvas id="cg-pl-status"></canvas></div></div>
      </div>
      <div class="card2 p-4"><p class="sec-title">Tendência de Ocorrências (Linha)</p><div style="height:220px"><canvas id="cg-pl-tendencia"></canvas></div></div>
    </div>
  </div>
</div>

<!-- ABA COMPARATIVO -->
<div id="aba-comparacao" class="space-y-4 hidden">
  <div class="card p-4">
    <h2 class="text-lg font-bold text-white mb-4">📈 Comparativo de Eficiência entre Turnos</h2>
    <div id="comp-vazio" class="text-center py-12"><p class="text-4xl mb-3">📊</p><p class="text-blue-300 font-bold">Crie pelo menos 2 turnos para comparar</p></div>
    <div id="comp-dados" class="hidden space-y-4">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="card2 p-4"><p class="sec-title">Eficiência Geral por Turno</p><div style="height:240px"><canvas id="cg-comp-efic"></canvas></div></div>
        <div class="card2 p-4"><p class="sec-title">Realizado vs Meta</p><div style="height:240px"><canvas id="cg-comp-meta"></canvas></div></div>
      </div>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="card2 p-4"><p class="sec-title">Refugo por Turno</p><div style="height:220px"><canvas id="cg-comp-refugo"></canvas></div></div>
        <div class="card2 p-4"><p class="sec-title">Glicerina vs Estufas (%)</p><div style="height:220px"><canvas id="cg-comp-setores"></canvas></div></div>
      </div>
      <div class="card2 p-4"><p class="sec-title">Tendência de Eficiência (Linha)</p><div style="height:220px"><canvas id="cg-comp-tendencia"></canvas></div></div>
      <div class="card2 p-4 overflow-x-auto">
        <p class="sec-title">Tabela Comparativa</p>
        <table class="w-full text-xs">
          <thead><tr class="border-b text-blue-400 uppercase text-left" style="border-color:var(--n700)">
            <th class="pb-2 pr-4">Líder / Data</th><th class="pb-2 pr-4">Turno</th><th class="pb-2 pr-4">Meta</th><th class="pb-2 pr-4">Realizado</th><th class="pb-2 pr-4">Efic.</th><th class="pb-2 pr-4">Refugo</th><th class="pb-2">Glic%</th>
          </tr></thead>
          <tbody id="comp-tbody"></tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<!-- ABA ADMIN -->
<div id="aba-admin" class="space-y-4 hidden">
<div class="card p-5 border-l-4 border-yellow-500">
  <h2 class="text-base font-bold text-yellow-400 mb-5">⚙️ Administrador PCP — Controle Total</h2>
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Dados do Turno</h3>
      <div><label class="text-[10px] text-blue-400 font-bold uppercase">Líder</label><input id="a-lider" class="inp inp-sm mt-1" oninput="aSync('h-lider','a-lider');curT().lider=this.value;salvar()"></div>
      <div><label class="text-[10px] text-blue-400 font-bold uppercase">Data</label><input id="a-data" class="inp inp-sm mt-1" oninput="aSync('h-data','a-data');curT().data=this.value;salvar()"></div>
      <div><label class="text-[10px] text-blue-400 font-bold uppercase">Turno</label><input id="a-turno" class="inp inp-sm mt-1" oninput="aSync('h-turno','a-turno');curT().turno=this.value;salvar()"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Produção</h3>
      <div class="grid grid-cols-2 gap-2">
        <div><label class="text-[10px] text-amber-400 font-bold uppercase">Meta Est.</label><input id="a-me" type="number" class="inp inp-sm mt-1" oninput="curT().metaEst=+this.value;recalc()"></div>
        <div><label class="text-[10px] text-amber-400 font-bold uppercase">Real Est.</label><input id="a-re" type="number" class="inp inp-sm mt-1" oninput="curT().realEst=+this.value;recalc()"></div>
        <div><label class="text-[10px] text-cyan-400 font-bold uppercase">Meta Glic.</label><input id="a-mg" type="number" class="inp inp-sm mt-1" oninput="curT().metaGlic=+this.value;recalc()"></div>
        <div><label class="text-[10px] text-cyan-400 font-bold uppercase">Real Glic.</label><input id="a-rg" type="number" class="inp inp-sm mt-1" oninput="curT().realGlic=+this.value;recalc()"></div>
        <div><label class="text-[10px] text-red-400 font-bold uppercase">Refugo Est.</label><input id="a-rfest" type="number" class="inp inp-sm mt-1" oninput="curT().refugoEst=+this.value;recalc()"></div>
        <div><label class="text-[10px] text-red-400 font-bold uppercase">Refugo Glic.</label><input id="a-rfglic" type="number" class="inp inp-sm mt-1" oninput="curT().refugoGlic=+this.value;recalc()"></div>
      </div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Ocorrências (labels + valores)</h3>
      <div id="a-oc-div" class="space-y-2"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Pareto Estufas</h3>
      <div id="a-ep-div" class="space-y-2"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Fontes do Refugo (pizza)</h3>
      <div id="a-rf-div" class="space-y-2"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Alertas Principais</h3>
      <div id="a-alertas" contenteditable="true" class="inp text-xs min-h-[80px]" oninput="syncHTML('d-alertas','a-alertas');curT().alertas=this.innerHTML;gerarRecomendacoes()"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Ocorrências Glicerina</h3>
      <div id="a-goc" contenteditable="true" class="inp text-xs min-h-[80px]" oninput="syncHTML('d-glic-ocorr','a-goc');curT().ocorrGlic=this.innerHTML"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Ocorrências Estufas</h3>
      <div id="a-eoc" contenteditable="true" class="inp text-xs min-h-[80px]" oninput="syncHTML('d-est-ocorr','a-eoc');curT().ocorrEst=this.innerHTML"></div>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Recomendações</h3>
      <div id="a-recom" contenteditable="true" class="inp text-xs min-h-[80px]" oninput="syncHTML('d-recomendacoes','a-recom');curT().recomendacoes=this.innerHTML"></div>
      <button onclick="gerarRecomendacoes()" class="btn btn-sm btn-blue w-full">🤖 Regerar IA</button>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Planos de Ação</h3>
      <div id="a-planos-div" class="space-y-2 max-h-44 overflow-y-auto"></div>
      <button onclick="oM('m-plano');gerarPlanos()" class="btn btn-sm btn-blue w-full">🤖 Gerar Planos IA</button>
    </div>
    <div class="space-y-3 md:col-span-2">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">🔗 Link de Compartilhamento (somente leitura)</h3>
      <p class="text-[10px] text-blue-400/50">Outros usuários só visualizam — não editam. Somente Admin pode compartilhar.</p>
      <div class="flex gap-2">
        <input id="a-link" class="inp inp-sm flex-1 font-mono text-cyan-400" readonly placeholder="Clique em Gerar...">
        <button onclick="gerarLink()" class="btn btn-sm btn-blue">🔗 Gerar</button>
        <button onclick="copiarLink()" class="btn btn-sm btn-gray">📋 Copiar</button>
      </div>
      <p id="link-ok" class="text-[10px] text-emerald-400 hidden">✅ Link copiado!</p>
    </div>
    <div class="space-y-3">
      <h3 class="text-xs font-bold text-blue-300 uppercase border-b pb-2" style="border-color:var(--n700)">Backup</h3>
      <div class="flex gap-2">
        <button onclick="exportJSON()" class="btn btn-sm btn-gray flex-1">📤 Exportar</button>
        <button onclick="document.getElementById('fi-json').click()" class="btn btn-sm btn-gray flex-1">📥 Importar</button>
        <input type="file" id="fi-json" class="hidden" accept=".json" onchange="importJSON(this.files[0])">
      </div>
      <button onclick="salvar();notif('Salvo!','✅')" class="btn btn-green w-full">💾 Salvar Tudo</button>
    </div>
  </div>
</div>
</div>

<!-- RODAPÉ -->
<div class="card p-2.5 flex justify-between text-[10px] text-blue-400/30">
  <span>Sistema PCP v4 — Produção Industrial</span>
  <span id="r-ts" class="font-mono"></span>
</div>

</div>

<!-- NOTIF -->
<div id="notif-box" class="fixed bottom-6 right-6 bg-gradient-to-r from-emerald-700 to-teal-700 text-white px-5 py-3 rounded-xl shadow-2xl hidden z-[400] border border-emerald-500">
  <div class="flex items-center gap-2 text-sm font-bold"><span id="n-icon">💾</span><span id="n-msg">Salvo!</span></div>
</div>

<script>
const ADMIN_U='PCP',ADMIN_P='PCP8803',SK='pcp_v4';
Chart.defaults.color='#64748b';
Chart.defaults.font.family='ui-sans-serif,system-ui,sans-serif';
Chart.register(ChartDataLabels);
const GR='rgba(19,47,88,0.6)';
const SC={beginAtZero:true,grid:{color:GR},ticks:{color:'#64748b',font:{size:10}}};

let isAdmin=false,abaAtual='consolidado';
let turnos={},turnoId=null;
let planosGerados=[],dadosImport={},charts={};

// Turno vazio
const mkTurno=()=>({
  id:'',lider:'',data:'',turno:'',
  metaEst:6725,realEst:0,refugoEst:0,metaGlic:7852,realGlic:0,refugoGlic:0,
  eficGeral:0,eficGlic:0,eficEst:0,
  ocLabels:['Quebra Maquinário','Faltas Pessoal','Alto Refugo','Bloqueio Produto'],
  ocVals:[0,0,0,0],
  epLabels:['Manutenção','Gabarito Eng.','Revezamento','Falta Tubos'],
  epVals:[0,0,0,0],
  rfLabels:['Robô 5 Hiago','BA0300 Bloq.','BA0300 Bloq.2','Outros'],
  rfVals:[0,0,0,0],
  alertas:'',recomendacoes:'',ocorrGlic:'',ocorrEst:'',
  glicRobos:mkRobosGlic(),est02:mkEst02(),est03:mkEst03(),
  ops02:Array(6).fill(''),ops03:Array(4).fill(''),planos:[]
});

function mkRobosGlic(){
  return ['R-01','R-02A','R-02B','R-02C','R-03','R-04','R-05','R-06','R-07','R-08','R-09','R-10','R-11']
    .map(n=>({nome:n,operador:'',produto:'',meta:0,real:0}));
}
function mkEst02(){return Array.from({length:13},(_,i)=>({nome:`Est02-${String(i+1).padStart(2,'0')}`,operador:'',produto:'',meta:0,real:0}));}
function mkEst03(){return Array.from({length:10},(_,i)=>({nome:`Est03-${String(i+1).padStart(2,'0')}`,operador:'',produto:'',meta:0,real:0}));}
const curT=()=>turnoId&&turnos[turnoId]?turnos[turnoId]:{};

// ADMIN
function toggle3pts(){document.getElementById('menu3pts').classList.toggle('hidden');}
function abrirLogin(){document.getElementById('menu3pts').classList.add('hidden');if(isAdmin){logout();return;}document.getElementById('lg-user').value='';document.getElementById('lg-pass').value='';document.getElementById('lg-err').classList.add('hidden');oM('m-login');}
function login(){
  if(document.getElementById('lg-user').value.trim()===ADMIN_U&&document.getElementById('lg-pass').value.trim()===ADMIN_P){
    isAdmin=true;
    document.getElementById('tab-admin').classList.remove('hidden');
    document.getElementById('btn-novo-turno').classList.remove('hidden');
    document.getElementById('btn-logout').classList.remove('hidden');
    document.getElementById('btn-lm').innerText='🔐 Admin (ativo)';
    fM('m-login');sincAdmin();notif('Admin ativo','🔐');aba('admin');
  }else document.getElementById('lg-err').classList.remove('hidden');
}
function logout(){
  isAdmin=false;
  ['tab-admin','btn-novo-turno','btn-logout'].forEach(id=>document.getElementById(id).classList.add('hidden'));
  document.getElementById('btn-lm').innerText='🔐 Admin PCP';
  document.getElementById('menu3pts').classList.add('hidden');
  if(abaAtual==='admin')aba('consolidado');
  notif('Sessão encerrada','🔒');
}

// ABAS
function aba(nome){
  ['consolidado','glicerina','estufas','planos','comparacao','admin'].forEach(a=>{
    document.getElementById('aba-'+a)?.classList.add('hidden');
    document.getElementById('tab-'+a)?.classList.remove('active');
  });
  document.getElementById('aba-'+nome)?.classList.remove('hidden');
  document.getElementById('tab-'+nome)?.classList.add('active');
  abaAtual=nome;
  if(nome==='comparacao')renderComp();
  setTimeout(()=>Object.values(charts).forEach(c=>{try{c.resize();c.update();}catch(e){}}),80);
}

// TURNOS
function criarTurno(){
  const l=document.getElementById('nt-lider').value.trim()||'Líder';
  const dv=document.getElementById('nt-data').value;
  const dt=dv?new Date(dv+'T12:00').toLocaleDateString('pt-BR'):new Date().toLocaleDateString('pt-BR');
  salvarAtual();
  const id='t'+Date.now();
  const t=mkTurno();
  t.id=id;t.lider=l;t.data=dt;t.turno=document.getElementById('nt-turno').value;
  t.metaEst=+document.getElementById('nt-me').value||6725;
  t.metaGlic=+document.getElementById('nt-mg').value||7852;
  const aus=document.getElementById('nt-aus').value.trim();
  if(aus)t.alertas=`<p class="text-orange-300">🟠 [ALTO] Ausências: ${aus}</p>`;
  turnos[id]=t;carregarTurno(id);renderHist();salvar();fM('m-turno');notif(`Turno ${l} criado!`,'✅');
}

function carregarTurno(id){
  const t=turnos[id];if(!t)return;
  turnoId=id;
  setText('h-lider',t.lider);setText('h-data',t.data);setText('h-turno',t.turno);
  setHTML('d-alertas',t.alertas||'<p class="text-blue-400/40 italic">Sem alertas.</p>');
  setHTML('d-recomendacoes',t.recomendacoes||'<p class="text-blue-400/40 italic">Sem recomendações.</p>');
  setHTML('d-glic-ocorr',t.ocorrGlic||'<p class="text-blue-400/40 italic">Sem ocorrências.</p>');
  setHTML('d-est-ocorr',t.ocorrEst||'<p class="text-blue-400/40 italic">Sem ocorrências.</p>');
  renderOcInputs();
  renderRobosGlic(t);renderEst02(t);renderEst03(t);renderOps02(t);renderOps03(t);
  recalc();atuCharts();
  planosGerados=t.planos||[];
  if(planosGerados.length){renderPlanosAba();renderGrafPlanos();}
  sincAdmin();renderHist();
}

function salvarAtual(){
  const t=turnos[turnoId];if(!t)return;
  t.lider=getText('h-lider');t.data=getText('h-data');t.turno=getText('h-turno');
  t.alertas=document.getElementById('d-alertas')?.innerHTML||'';
  t.recomendacoes=document.getElementById('d-recomendacoes')?.innerHTML||'';
  t.ocorrGlic=document.getElementById('d-glic-ocorr')?.innerHTML||'';
  t.ocorrEst=document.getElementById('d-est-ocorr')?.innerHTML||'';
  t.ocLabels=t.ocLabels.map((_,i)=>document.getElementById('oc-lbl-'+i)?.innerText||t.ocLabels[i]);
  t.ocVals=t.ocVals.map((_,i)=>gN('oc-inp-'+i));
  t.epLabels=t.epLabels.map((_,i)=>document.getElementById('ep-lbl-'+i)?.innerText||t.epLabels[i]);
  t.epVals=[0,1,2,3].map((_,i)=>parseInt(document.querySelector(`#a-ep-div input[type=number]:nth-child(${i+1})`)?.value)||t.epVals[i]||0);
  t.rfLabels=[0,1,2,3].map(i=>document.getElementById('rf-lbl-'+i)?.value||t.rfLabels[i]);
  t.rfVals=[0,1,2,3].map(i=>gN('rf-val-'+i));
  t.glicRobos?.forEach((_,i)=>{t.glicRobos[i]={...t.glicRobos[i],operador:document.getElementById(`gr-op-${i}`)?.value||'',produto:document.getElementById(`gr-prod-${i}`)?.value||'',meta:gN(`gr-meta-${i}`),real:gN(`gr-real-${i}`)};});
  t.est02?.forEach((_,i)=>{t.est02[i]={...t.est02[i],operador:document.getElementById(`e2-op-${i}`)?.value||'',produto:document.getElementById(`e2-prod-${i}`)?.value||'',meta:gN(`e2-meta-${i}`),real:gN(`e2-real-${i}`)};});
  t.est03?.forEach((_,i)=>{t.est03[i]={...t.est03[i],operador:document.getElementById(`e3-op-${i}`)?.value||'',produto:document.getElementById(`e3-prod-${i}`)?.value||'',meta:gN(`e3-meta-${i}`),real:gN(`e3-real-${i}`)};});
  t.ops02=[0,1,2,3,4,5].map(i=>document.getElementById(`ops2-${i}`)?.value||'');
  t.ops03=[0,1,2,3].map(i=>document.getElementById(`ops3-${i}`)?.value||'');
  t.planos=planosGerados;
}

function renderHist(){
  const c=document.getElementById('hist-turnos');c.innerHTML='';
  Object.values(turnos).forEach(t=>{
    const b=document.createElement('button');
    b.className='turn-chip'+(t.id===turnoId?' act':'');
    b.innerText=`${t.lider} — ${t.data}`;
    b.onclick=()=>{salvarAtual();carregarTurno(t.id);};
    c.appendChild(b);
  });
}

// CARDS
function renderRobosGlic(t){
  const g=document.getElementById('glic-robos-grid');if(!g)return;
  g.innerHTML='';
  (t.glicRobos||[]).forEach((r,i)=>{
    const ef=r.meta>0?Math.round((r.real/r.meta)*100):0;
    const ec=ef>=80?'#10b981':ef>=60?'#f59e0b':'#ef4444';
    const ro=isAdmin?'':'readonly';
    g.innerHTML+=`<div class="robo-card space-y-2">
      <div class="flex justify-between items-center border-b pb-1" style="border-color:var(--n700)">
        <span class="text-xs font-extrabold text-cyan-300">${r.nome}</span>
        <span class="efic-badge text-white text-[10px]" style="background:${ec}">${ef}%</span>
      </div>
      <input id="gr-op-${i}" class="inp inp-sm" placeholder="Operador" value="${r.operador||''}" ${ro} onchange="saveGR(${i})">
      <input id="gr-prod-${i}" class="inp inp-sm text-blue-300" placeholder="Produto" value="${r.produto||''}" ${ro} onchange="saveGR(${i});atuCharts()">
      <div class="grid grid-cols-2 gap-1">
        <input id="gr-meta-${i}" type="number" class="inp inp-sm text-amber-400 text-center" placeholder="Meta" value="${r.meta||0}" ${ro} onchange="saveGR(${i});recalc()">
        <input id="gr-real-${i}" type="number" class="inp inp-sm text-cyan-400 text-center" placeholder="Real" value="${r.real||0}" ${ro} onchange="saveGR(${i});recalc()">
      </div>
      <div class="text-xs font-bold text-center" style="color:${ec}">Efic: ${ef}%</div>
      <div class="w-full h-1 rounded-full" style="background:var(--n700)"><div class="h-1 rounded-full transition-all" style="width:${Math.min(ef,100)}%;background:${ec}"></div></div>
    </div>`;
  });
}
function saveGR(i){const t=curT();if(!t.glicRobos)return;t.glicRobos[i]={...t.glicRobos[i],operador:document.getElementById(`gr-op-${i}`)?.value||'',produto:document.getElementById(`gr-prod-${i}`)?.value||'',meta:gN(`gr-meta-${i}`),real:gN(`gr-real-${i}`)};salvar();}

function renderEst02(t){
  const g=document.getElementById('est02-grid');if(!g)return;g.innerHTML='';
  (t.est02||[]).forEach((r,i)=>{
    const ef=r.meta>0?Math.round((r.real/r.meta)*100):0;
    const ec=ef>=80?'#10b981':ef>=60?'#f59e0b':'#ef4444';
    const ro=isAdmin?'':'readonly';
    g.innerHTML+=`<div class="robo-card space-y-1.5">
      <div class="flex justify-between items-center border-b pb-1" style="border-color:var(--n700)">
        <span class="text-[10px] font-extrabold text-amber-300">${r.nome}</span>
        <span class="efic-badge text-white text-[9px]" style="background:${ec}">${ef}%</span>
      </div>
      <input id="e2-op-${i}" class="inp inp-sm text-[10px]" placeholder="Operador" value="${r.operador||''}" ${ro} onchange="saveE2(${i})">
      <input id="e2-prod-${i}" class="inp inp-sm text-blue-300 text-[10px]" placeholder="Produto" value="${r.produto||''}" ${ro} onchange="saveE2(${i});atuCharts()">
      <div class="grid grid-cols-2 gap-1">
        <input id="e2-meta-${i}" type="number" class="inp inp-sm text-amber-400 text-center text-[10px]" value="${r.meta||0}" ${ro} onchange="saveE2(${i});recalc()">
        <input id="e2-real-${i}" type="number" class="inp inp-sm text-green-400 text-center text-[10px]" value="${r.real||0}" ${ro} onchange="saveE2(${i});recalc()">
      </div>
    </div>`;
  });
}
function saveE2(i){const t=curT();if(!t.est02)return;t.est02[i]={...t.est02[i],operador:document.getElementById(`e2-op-${i}`)?.value||'',produto:document.getElementById(`e2-prod-${i}`)?.value||'',meta:gN(`e2-meta-${i}`),real:gN(`e2-real-${i}`)};salvar();}

function renderEst03(t){
  const g=document.getElementById('est03-grid');if(!g)return;g.innerHTML='';
  (t.est03||[]).forEach((r,i)=>{
    const ef=r.meta>0?Math.round((r.real/r.meta)*100):0;
    const ec=ef>=80?'#10b981':ef>=60?'#f59e0b':'#ef4444';
    const ro=isAdmin?'':'readonly';
    g.innerHTML+=`<div class="robo-card space-y-1.5">
      <div class="flex justify-between items-center border-b pb-1" style="border-color:var(--n700)">
        <span class="text-[10px] font-extrabold text-teal-300">${r.nome}</span>
        <span class="efic-badge text-white text-[9px]" style="background:${ec}">${ef}%</span>
      </div>
      <input id="e3-op-${i}" class="inp inp-sm text-[10px]" placeholder="Operador" value="${r.operador||''}" ${ro} onchange="saveE3(${i})">
      <input id="e3-prod-${i}" class="inp inp-sm text-blue-300 text-[10px]" placeholder="Produto" value="${r.produto||''}" ${ro} onchange="saveE3(${i});atuCharts()">
      <div class="grid grid-cols-2 gap-1">
        <input id="e3-meta-${i}" type="number" class="inp inp-sm text-amber-400 text-center text-[10px]" value="${r.meta||0}" ${ro} onchange="saveE3(${i});recalc()">
        <input id="e3-real-${i}" type="number" class="inp inp-sm text-green-400 text-center text-[10px]" value="${r.real||0}" ${ro} onchange="saveE3(${i});recalc()">
      </div>
    </div>`;
  });
}
function saveE3(i){const t=curT();if(!t.est03)return;t.est03[i]={...t.est03[i],operador:document.getElementById(`e3-op-${i}`)?.value||'',produto:document.getElementById(`e3-prod-${i}`)?.value||'',meta:gN(`e3-meta-${i}`),real:gN(`e3-real-${i}`)};salvar();}

function renderOps02(t){
  const g=document.getElementById('est02-ops');if(!g)return;g.innerHTML='';
  (t.ops02||Array(6).fill('')).forEach((op,i)=>{
    g.innerHTML+=`<div><label class="text-[9px] text-amber-400 font-bold uppercase">Op. ${i+1}</label><input id="ops2-${i}" class="inp inp-sm mt-0.5" placeholder="Nome" value="${op||''}" ${isAdmin?'':'readonly'} onchange="curT().ops02[${i}]=this.value;salvar()"></div>`;
  });
}
function renderOps03(t){
  const g=document.getElementById('est03-ops');if(!g)return;g.innerHTML='';
  (t.ops03||Array(4).fill('')).forEach((op,i)=>{
    g.innerHTML+=`<div><label class="text-[9px] text-teal-400 font-bold uppercase">Op. ${i+1}</label><input id="ops3-${i}" class="inp inp-sm mt-0.5" placeholder="Nome" value="${op||''}" ${isAdmin?'':'readonly'} onchange="curT().ops03[${i}]=this.value;salvar()"></div>`;
  });
}

function renderOcInputs(){
  const t=curT();if(!t.id)return;const lg=document.getElementById('oc-legend');const ig=document.getElementById('oc-inputs');if(!lg||!ig)return;
  const cors=['#ef4444','#f97316','#f59e0b','#3b82f6'];
  lg.innerHTML='';ig.innerHTML='';
  (t.ocLabels||[]).forEach((l,i)=>{
    lg.innerHTML+=`<div class="flex items-center gap-1"><span class="w-2 h-2 rounded-full shrink-0" style="background:${cors[i]}"></span><span id="oc-lbl-${i}" class="text-[9px] font-bold uppercase" style="color:${cors[i]}">${l}</span></div>`;
    ig.innerHTML+=`<input type="number" id="oc-inp-${i}" value="${(t.ocVals||[])[i]||0}" class="text-center text-xs bg-transparent rounded font-bold py-0.5 focus:outline-none" style="border:1px solid ${cors[i]}40;color:${cors[i]}" ${isAdmin?'':'readonly'} oninput="curT().ocVals[${i}]=+this.value;atuCharts();gerarRecomendacoes()">`;
  });
}

// RECALCULAR
function recalc(){
  const t=curT();if(!t.id)return;
  const rGCard=(t.glicRobos||[]).reduce((s,r)=>s+(r.real||0),0);
  const rECard=(t.est02||[]).reduce((s,r)=>s+(r.real||0),0)+(t.est03||[]).reduce((s,r)=>s+(r.real||0),0);
  const rG=rGCard>0?rGCard:(t.realGlic||0);
  const rE=rECard>0?rECard:(t.realEst||0);
  const mG=Math.max(1,t.metaGlic||gN('a-mg')||1);
  const mE=Math.max(1,t.metaEst||gN('a-me')||1);
  const tM=mG+mE,tR=rG+rE,tRef=(t.refugoEst||0)+(t.refugoGlic||0);
  const efG=((tR/tM)*100).toFixed(1);
  const efE=((rE/mE)*100).toFixed(1);
  const efGl=((rG/mG)*100).toFixed(1);
  t.eficGeral=+efG;t.eficEst=+efE;t.eficGlic=+efGl;
  setText('k-meta',tM.toLocaleString('pt-BR'));setText('k-real',tR.toLocaleString('pt-BR'));
  setText('k-efic',`${efG}%`);setText('k-refugo',tRef.toLocaleString('pt-BR'));setText('k-gap',Math.max(0,tM-tR).toLocaleString('pt-BR'));
  setText('h-efic',`${efG}%`);
  bar('k-efic-bar',efG,+efG);
  setText('gm-disp',mG.toLocaleString('pt-BR'));setText('gr-disp',rG.toLocaleString('pt-BR'));setText('ge-disp',`${efGl}%`);setText('grf-disp',t.refugoGlic||0);
  setText('em-disp',mE.toLocaleString('pt-BR'));setText('er-disp',rE.toLocaleString('pt-BR'));setText('ee-disp',`${efE}%`);setText('erf-disp',t.refugoEst||0);
  bar('glic-efic-bar',efGl,+efGl);bar('est-efic-bar',efE,+efE);
  atuCharts();
}

function bar(id,pct,val){
  const el=document.getElementById(id);if(!el)return;
  el.style.width=`${Math.min(pct,100)}%`;
  el.style.background=val>=80?'#10b981':val>=60?'#f59e0b':'#ef4444';
}

// GRÁFICOS
function initCharts(){
  const mk=(id,cfg)=>{try{if(charts[id])charts[id].destroy();charts[id]=new Chart(document.getElementById(id),cfg);return charts[id];}catch(e){return null;}};

  mk('cg-meta-setor',{type:'bar',data:{labels:['Estufas','Glicerina'],datasets:[{label:'Meta',data:[0,0],backgroundColor:'rgba(59,130,246,.75)',borderRadius:6},{label:'Realizado',data:[0,0],backgroundColor:'rgba(16,185,129,.75)',borderRadius:6}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:11,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v.toLocaleString('pt-BR'):''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:10}}}}}});

  mk('cg-ocorrencias',{type:'bar',data:{labels:['L1','L2','L3','L4'],datasets:[{data:[0,0,0,0],backgroundColor:['#ef4444','#f97316','#f59e0b','#3b82f6'],borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,indexAxis:'y',plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:11,weight:'bold'},anchor:'end',align:'right',formatter:v=>v>0?v:''}},scales:{x:SC,y:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});

  mk('cg-refugo',{type:'doughnut',data:{labels:['Item1','Item2','Item3','Item4'],datasets:[{data:[1,1,1,1],backgroundColor:['#3b82f6','#10b981','#f59e0b','#6b7280'],borderWidth:2,borderColor:'#061020'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{position:'right',labels:{color:'#64748b',font:{size:9},boxWidth:10}},datalabels:{color:'#fff',font:{size:10,weight:'bold'},formatter:(v,c)=>v>0?`${c.chart.data.labels[c.dataIndex]}\n${v}`:''}}}});

  mk('cg-glic-robos',{type:'bar',data:{labels:[],datasets:[{label:'Real',data:[],backgroundColor:'rgba(6,182,212,.8)',borderRadius:4},{label:'Meta',data:[],backgroundColor:'rgba(245,158,11,.35)',borderRadius:4}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{display:false}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:8},maxRotation:45}}}}});

  mk('cg-glic-efic',{type:'bar',data:{labels:[],datasets:[{label:'Efic %',data:[],backgroundColor:ctx=>{const v=ctx.raw||0;return v>=80?'rgba(16,185,129,.85)':v>=60?'rgba(245,158,11,.85)':'rgba(239,68,68,.85)';},borderRadius:4}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:9,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?`${v}%`:''}},scales:{y:{...SC,max:120},x:{grid:{display:false},ticks:{color:'#64748b',font:{size:8},maxRotation:45}}}}});

  mk('cg-glic-meta',{type:'doughnut',data:{labels:['Realizado','Falta'],datasets:[{data:[0,100],backgroundColor:['#06b6d4','#0f274a'],borderWidth:2,borderColor:'#061020'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{position:'bottom',labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:11,weight:'bold'},formatter:(v,c)=>v>0?`${c.chart.data.labels[c.dataIndex]}\n${v.toLocaleString('pt-BR')}`:''}}}}); 

  mk('cg-glic-produto',{type:'bar',data:{labels:[],datasets:[{data:[],backgroundColor:'rgba(6,182,212,.7)',borderRadius:4}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:9,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v:''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:8},maxRotation:60}}}}});

  mk('cg-est-split',{type:'bar',data:{labels:['Estufa 02','Estufa 03'],datasets:[{label:'Realizado',data:[0,0],backgroundColor:['rgba(245,158,11,.8)','rgba(20,184,166,.8)'],borderRadius:6},{label:'Meta',data:[0,0],backgroundColor:['rgba(245,158,11,.3)','rgba(20,184,166,.3)'],borderRadius:6}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:10,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v.toLocaleString('pt-BR'):''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:10}}}}}});

  mk('cg-est-efic',{type:'bar',data:{labels:[],datasets:[{data:[],backgroundColor:ctx=>{const v=ctx.raw||0;return v>=80?'rgba(16,185,129,.85)':v>=60?'rgba(245,158,11,.85)':'rgba(239,68,68,.85)';},borderRadius:3}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{display:false}},scales:{y:{...SC,max:120},x:{grid:{display:false},ticks:{color:'#64748b',font:{size:7},maxRotation:45}}}}});

  mk('cg-est-pareto',{type:'bar',data:{labels:['L1','L2','L3','L4'],datasets:[{data:[0,0,0,0],backgroundColor:['#ef4444','#f97316','#f59e0b','#3b82f6'],borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:10,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v:''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});

  mk('cg-est-meta',{type:'doughnut',data:{labels:['Realizado','Falta'],datasets:[{data:[0,100],backgroundColor:['#f59e0b','#0f274a'],borderWidth:2,borderColor:'#061020'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{position:'bottom',labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:11,weight:'bold'},formatter:(v,c)=>v>0?`${c.chart.data.labels[c.dataIndex]}\n${v.toLocaleString('pt-BR')}`:''}}}}); 

  mk('cg-est-produto',{type:'bar',data:{labels:[],datasets:[{data:[],backgroundColor:'rgba(245,158,11,.7)',borderRadius:4}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:9,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v:''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:8},maxRotation:60}}}}});
}

function atuCharts(){
  const t=curT();if(!t.id)return;
  const mE=t.metaEst||0,rE=t.realEst||0,mG=t.metaGlic||0,rG=t.realGlic||0;
  const rE02=(t.est02||[]).reduce((s,r)=>s+(r.real||0),0);
  const rE03=(t.est03||[]).reduce((s,r)=>s+(r.real||0),0);
  const mE02=(t.est02||[]).reduce((s,r)=>s+(r.meta||0),0);
  const mE03=(t.est03||[]).reduce((s,r)=>s+(r.meta||0),0);
  const rGfinal=rG||((t.glicRobos||[]).reduce((s,r)=>s+(r.real||0),0));
  const rEfinal=rE||(rE02+rE03);
  const fG=Math.max(0,mG-rGfinal),fE=Math.max(0,mE-rEfinal);

  upd('cg-meta-setor',d=>{d.datasets[0].data=[mE||mE02+mE03,mG];d.datasets[1].data=[rEfinal,rGfinal];});
  upd('cg-ocorrencias',d=>{d.labels=[...(t.ocLabels||[])];d.datasets[0].data=[...(t.ocVals||[0,0,0,0])];});
  upd('cg-refugo',d=>{d.labels=[...(t.rfLabels||[])];d.datasets[0].data=[...(t.rfVals||[0,0,0,0])];});
  if(t.glicRobos){
    const labs=t.glicRobos.map(r=>r.nome);
    const reais=t.glicRobos.map(r=>r.real||0);
    const metas=t.glicRobos.map(r=>r.meta||0);
    const efics=t.glicRobos.map(r=>r.meta>0?+(((r.real/r.meta)*100).toFixed(1)):0);
    const prods=t.glicRobos.map(r=>r.produto||r.nome);
    upd('cg-glic-robos',d=>{d.labels=[...labs];d.datasets[0].data=[...reais];d.datasets[1].data=[...metas];});
    upd('cg-glic-efic',d=>{d.labels=[...labs];d.datasets[0].data=[...efics];});
    upd('cg-glic-meta',d=>{d.datasets[0].data=[rGfinal,fG];d.labels=['Realizado '+rGfinal.toLocaleString('pt-BR'),'Falta '+fG.toLocaleString('pt-BR')];});
    upd('cg-glic-produto',d=>{d.labels=[...prods];d.datasets[0].data=[...reais];});
  }
  if(t.est02&&t.est03){
    upd('cg-est-split',d=>{d.datasets[0].data=[rE02,rE03];d.datasets[1].data=[mE02,mE03];});
    const all=[...t.est02,...t.est03];
    const labsE=all.map(r=>r.nome);
    const efE=all.map(r=>r.meta>0?+(((r.real/r.meta)*100).toFixed(1)):0);
    const prodsE=all.map(r=>r.produto||r.nome);
    const reaisE=all.map(r=>r.real||0);
    upd('cg-est-efic',d=>{d.labels=[...labsE];d.datasets[0].data=[...efE];});
    const faltaE2=Math.max(0,(mE02+mE03)-(rE02+rE03));
    upd('cg-est-meta',d=>{d.datasets[0].data=[rE02+rE03,faltaE2];});
    upd('cg-est-produto',d=>{d.labels=[...prodsE];d.datasets[0].data=[...reaisE];});
    upd('cg-est-pareto',d=>{d.labels=[...(t.epLabels||[])];d.datasets[0].data=[...(t.epVals||[0,0,0,0])];});
  }
  setTimeout(()=>Object.values(charts).forEach(c=>{try{c.update();}catch(e){}}),10);
}

// COMPARATIVO
function renderComp(){
  const lista=Object.values(turnos);
  if(lista.length<2){document.getElementById('comp-vazio').classList.remove('hidden');document.getElementById('comp-dados').classList.add('hidden');return;}
  document.getElementById('comp-vazio').classList.add('hidden');document.getElementById('comp-dados').classList.remove('hidden');
  const labs=lista.map(t=>`${t.lider}\n${t.data}`);
  const efics=lista.map(t=>t.eficGeral||0);
  const reais=lista.map(t=>(t.realEst||0)+(t.realGlic||0));
  const metas=lista.map(t=>(t.metaEst||0)+(t.metaGlic||0));
  const refs=lista.map(t=>(t.refugoEst||0)+(t.refugoGlic||0));
  const efGl=lista.map(t=>t.eficGlic||0);
  const efEst=lista.map(t=>t.eficEst||0);
  const mkC=(id,cfg)=>{try{if(charts[id])charts[id].destroy();charts[id]=new Chart(document.getElementById(id),cfg);}catch(e){}};
  mkC('cg-comp-efic',{type:'bar',data:{labels:labs,datasets:[{label:'Efic %',data:efics,backgroundColor:efics.map(v=>v>=80?'rgba(16,185,129,.8)':v>=60?'rgba(245,158,11,.8)':'rgba(239,68,68,.8)'),borderRadius:6}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:10,weight:'bold'},anchor:'end',align:'top',formatter:v=>`${v}%`}},scales:{y:{...SC,max:120},x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});
  mkC('cg-comp-meta',{type:'bar',data:{labels:labs,datasets:[{label:'Meta',data:metas,backgroundColor:'rgba(245,158,11,.6)',borderRadius:5},{label:'Realizado',data:reais,backgroundColor:'rgba(59,130,246,.8)',borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:9,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v.toLocaleString('pt-BR'):''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});
  mkC('cg-comp-refugo',{type:'bar',data:{labels:labs,datasets:[{label:'Refugo',data:refs,backgroundColor:'rgba(239,68,68,.7)',borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:10,weight:'bold'},anchor:'end',align:'top',formatter:v=>v>0?v:''}},scales:{y:SC,x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});
  mkC('cg-comp-setores',{type:'bar',data:{labels:labs,datasets:[{label:'Glicerina %',data:efGl,backgroundColor:'rgba(6,182,212,.8)',borderRadius:5},{label:'Estufas %',data:efEst,backgroundColor:'rgba(245,158,11,.8)',borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{display:false}},scales:{y:{...SC,max:120},x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});
  mkC('cg-comp-tendencia',{type:'line',data:{labels:labs,datasets:[{label:'Efic. Geral %',data:efics,borderColor:'#3b82f6',backgroundColor:'rgba(59,130,246,.1)',tension:.4,fill:true,pointRadius:6,pointBackgroundColor:'#3b82f6'},{label:'Glicerina %',data:efGl,borderColor:'#06b6d4',tension:.4,fill:false,pointRadius:5,pointBackgroundColor:'#06b6d4'},{label:'Estufas %',data:efEst,borderColor:'#f59e0b',tension:.4,fill:false,pointRadius:5,pointBackgroundColor:'#f59e0b'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{display:false}},scales:{y:{...SC,max:120},x:{grid:{color:GR},ticks:{color:'#64748b',font:{size:9}}}}}});
  const tb=document.getElementById('comp-tbody');tb.innerHTML='';
  lista.forEach(t=>{
    const ef=t.eficGeral||0;
    const cor=ef>=80?'#10b981':ef>=60?'#f59e0b':'#ef4444';
    tb.innerHTML+=`<tr class="border-b text-blue-300" style="border-color:var(--n750)">
      <td class="py-2 pr-4 text-cyan-400 font-bold">${t.lider} / ${t.data}</td>
      <td class="pr-4">${t.turno}</td>
      <td class="pr-4 text-amber-400">${((t.metaEst||0)+(t.metaGlic||0)).toLocaleString('pt-BR')}</td>
      <td class="pr-4 text-blue-400">${((t.realEst||0)+(t.realGlic||0)).toLocaleString('pt-BR')}</td>
      <td class="pr-4 font-bold" style="color:${cor}">${ef}%</td>
      <td class="pr-4 text-red-400">${(t.refugoEst||0)+(t.refugoGlic||0)}</td>
      <td class="font-bold" style="color:${(t.eficGlic||0)>=80?'#10b981':(t.eficGlic||0)>=60?'#f59e0b':'#ef4444'}">${t.eficGlic||0}%</td>
    </tr>`;
  });
}

// PLANOS

async function gerarPlanosSilencioso(){
  try{
    const t=curT();
    const alertas=document.getElementById('d-alertas')?.innerText||'';
    const ocorrGlic=document.getElementById('d-glic-ocorr')?.innerText||'';
    const ocorrEst=document.getElementById('d-est-ocorr')?.innerText||'';
    const oc=(t.ocLabels||[]).map((l,i)=>`${l}: ${(t.ocVals||[])[i]||0}`).join(', ');
    const res=await callAI([{type:'text',text:`Especialista em gestão industrial. Analise os problemas ESPECÍFICOS e gere 6 planos de ação diretos e práticos com solução concreta.
ALERTAS: ${alertas}
GLICERINA: ${ocorrGlic}
ESTUFAS: ${ocorrEst}
PARETO: ${oc}
Eficiência: ${t.eficGeral||0}%
Responda SOMENTE JSON sem markdown:
[{"titulo":"","problema":"","solucao":"","responsavel":"","prazo":"","prioridade":"Alta|Média|Baixa","setor":"GLICERINA|ESTUFAS|GERAL","status":"Pendente"}]`}],2000);
    planosGerados=JSON.parse(res.replace(/```json|```/g,'').trim());
    renderPlanosAba();renderGrafPlanos();
    if(turnoId&&turnos[turnoId])turnos[turnoId].planos=planosGerados;
  }catch(e){console.warn('Planos silenciosos erro:',e);}
}

async function gerarPlanos(){
  document.getElementById('pl-loading').classList.remove('hidden');
  document.getElementById('pl-lista').innerHTML='';
  const t=curT();
  const alertas=document.getElementById('d-alertas')?.innerText||'';
  const ocorrGlic=document.getElementById('d-glic-ocorr')?.innerText||'';
  const ocorrEst=document.getElementById('d-est-ocorr')?.innerText||'';
  const oc=(t.ocLabels||[]).map((l,i)=>`${l}: ${(t.ocVals||[])[i]||0}`).join(', ');
  try{
    const res=await callAI([{type:'text',text:`Especialista em gestão industrial. Analise os problemas ESPECÍFICOS abaixo e gere 6 planos de ação diretos e práticos. Cada plano deve ter solução concreta para o problema identificado.

ALERTAS PRINCIPAIS: ${alertas}
OCORRÊNCIAS GLICERINA: ${ocorrGlic}
OCORRÊNCIAS ESTUFAS: ${ocorrEst}
PARETO: ${oc}
Eficiência Geral: ${t.eficGeral||0}%

Responda SOMENTE JSON sem markdown:
[{"titulo":"","problema":"","solucao":"solução específica e prática","responsavel":"","prazo":"","prioridade":"Alta|Média|Baixa","setor":"GLICERINA|ESTUFAS|GERAL","status":"Pendente"}]`}]);
    planosGerados=JSON.parse(res.replace(/```json|```/g,'').trim());
  }catch(e){
    const l=t.lider||'Líder';
    planosGerados=[
      {titulo:'Restaurar Equipamento Parado',problema:'Máquina quebrada reduz capacidade produtiva',solucao:'Acionar manutenção corretiva imediatamente. Executar diagnóstico elétrico e mecânico. Registrar causa raiz no sistema.',responsavel:'Líder Manutenção',prazo:'Mesmo turno',prioridade:'Alta',setor:'GLICERINA',status:'Pendente'},
      {titulo:'Cobertura de Postos Vagos',problema:'Ausência de operadores reduz produção',solucao:'Redistribuir operadores presentes priorizando postos com maior meta. Acionar banco de reservas para cobertura.',responsavel:l,prazo:'Imediato',prioridade:'Alta',setor:'GERAL',status:'Pendente'},
      {titulo:'Controle de Refugo Elevado',problema:'Alto índice de peças refugadas aumenta custo',solucao:'Bloquear lote com refugo e acionar qualidade. Ajustar parâmetros da máquina causadora e registrar causa raiz.',responsavel:'Eng. Qualidade',prazo:'2 horas',prioridade:'Alta',setor:'GERAL',status:'Pendente'},
      {titulo:'Manutenção Preventiva Estufas',problema:'Desgaste em barramentos e gabaritos',solucao:'Agendar parada programada para substituição de componentes. Verificar calibração dos gabaritos de engenharia.',responsavel:'Manutenção Preventiva',prazo:'Próximo turno',prioridade:'Média',setor:'ESTUFAS',status:'Pendente'},
      {titulo:'Reposição de Insumos',problema:'Falta de materiais interrompe linha',solucao:'Emitir requisição urgente ao almoxarifado com justificativa de impacto produtivo. Mapear consumo futuro.',responsavel:'Logística',prazo:'1 hora',prioridade:'Média',setor:'ESTUFAS',status:'Pendente'},
      {titulo:'Acelerar Curva de Treinamento',problema:'Operadores em treinamento reduzem velocidade',solucao:'Designar operador sênior como mentor. Definir meta progressiva por hora com acompanhamento do líder.',responsavel:l,prazo:'Este turno',prioridade:'Baixa',setor:'GLICERINA',status:'Pendente'}
    ];
  }
  document.getElementById('pl-loading').classList.add('hidden');
  renderPlanosModal();renderPlanosAba();renderGrafPlanos();
  if(turnoId&&turnos[turnoId])turnos[turnoId].planos=planosGerados;
  salvar();
}

function renderPlanosModal(){
  const l=document.getElementById('pl-lista');l.innerHTML='';
  const ad=document.getElementById('a-planos-div');if(ad)ad.innerHTML='';
  planosGerados.forEach((p,i)=>{
    const cls=p.prioridade==='Alta'?'plano-A':p.prioridade==='Média'?'plano-M':'plano-B';
    const bg=p.prioridade==='Alta'?'#ef4444':p.prioridade==='Média'?'#f59e0b':'#10b981';
    const sc=p.setor==='GLICERINA'?'rgba(6,182,212,.15)':p.setor==='ESTUFAS'?'rgba(245,158,11,.15)':'rgba(100,116,139,.15)';
    const stc=p.setor==='GLICERINA'?'#06b6d4':p.setor==='ESTUFAS'?'#f59e0b':'#64748b';
    l.innerHTML+=`<div class="${cls} mb-2">
      <div class="flex justify-between items-start mb-2 flex-wrap gap-1">
        <span class="text-sm font-bold text-white">${p.titulo}</span>
        <div class="flex gap-1">
          <span class="efic-badge text-white text-[10px]" style="background:${bg}">${p.prioridade}</span>
          <span class="efic-badge text-[10px]" style="background:${sc};color:${stc}">${p.setor}</span>
        </div>
      </div>
      <p class="text-[11px] mb-1" style="color:#fca5a5"><strong>Problema:</strong> ${p.problema}</p>
      <p class="text-xs text-blue-200 mb-2"><strong>Solução:</strong> ${p.solucao}</p>
      <div class="flex flex-wrap gap-3 text-[10px] text-blue-400/60 items-center">
        <span>👤 ${p.responsavel}</span><span>⏱️ ${p.prazo}</span>
        <select onchange="planosGerados[${i}].status=this.value;renderPlanosAba();renderGrafPlanos()" class="inp inp-sm text-[9px] w-auto py-0 px-1 h-5">
          <option ${p.status==='Pendente'?'selected':''}>Pendente</option>
          <option ${p.status==='Em andamento'?'selected':''}>Em andamento</option>
          <option ${p.status==='Concluído'?'selected':''}>Concluído</option>
        </select>
      </div>
    </div>`;
    if(ad)ad.innerHTML+=`<div class="card2 p-2 text-xs"><div class="flex justify-between"><span class="font-bold text-white">${p.titulo}</span><span style="color:${bg}">${p.prioridade}</span></div><p class="text-blue-400/50 mt-0.5">${p.responsavel} · ${p.prazo}</p></div>`;
  });
}

function renderPlanosAba(){
  const d=document.getElementById('pl-aba');if(!d)return;d.innerHTML='';
  planosGerados.forEach(p=>{
    const cls=p.prioridade==='Alta'?'plano-A':p.prioridade==='Média'?'plano-M':'plano-B';
    const bg=p.prioridade==='Alta'?'#ef4444':p.prioridade==='Média'?'#f59e0b':'#10b981';
    const stc=p.status==='Concluído'?'#10b981':p.status==='Em andamento'?'#f59e0b':'#64748b';
    d.innerHTML+=`<div class="${cls} mb-2">
      <div class="flex justify-between mb-1 flex-wrap gap-1"><span class="text-sm font-bold text-white">${p.titulo}</span><span class="efic-badge text-white text-[10px]" style="background:${bg}">${p.prioridade}</span></div>
      <p class="text-[11px] mb-1" style="color:#fca5a5"><strong>Problema:</strong> ${p.problema}</p>
      <p class="text-xs text-blue-200 mb-1.5"><strong>Solução:</strong> ${p.solucao}</p>
      <div class="flex gap-3 text-[10px] text-blue-400/60"><span>👤 ${p.responsavel}</span><span>⏱️ ${p.prazo}</span><span class="font-bold" style="color:${stc}">${p.status||'Pendente'}</span></div>
    </div>`;
  });
}

function renderGrafPlanos(){
  document.getElementById('pl-charts').classList.remove('hidden');
  const A=planosGerados.filter(p=>p.prioridade==='Alta').length;
  const M=planosGerados.filter(p=>p.prioridade==='Média').length;
  const B=planosGerados.filter(p=>p.prioridade==='Baixa').length;
  const resp={};planosGerados.forEach(p=>resp[p.responsavel]=(resp[p.responsavel]||0)+1);
  const st={Pendente:0,'Em andamento':0,'Concluído':0};planosGerados.forEach(p=>st[p.status||'Pendente']++);
  const mkC=(id,cfg)=>{try{if(charts[id])charts[id].destroy();charts[id]=new Chart(document.getElementById(id),cfg);}catch(e){}};
  mkC('cg-pl-prior',{type:'doughnut',data:{labels:['Alta','Média','Baixa'],datasets:[{data:[A,M,B],backgroundColor:['#ef4444','#f59e0b','#10b981'],borderWidth:2,borderColor:'#061020'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{position:'bottom',labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:12,weight:'bold'},formatter:(v,c)=>v>0?`${c.chart.data.labels[c.dataIndex]}\n${v}`:''}}}});
  mkC('cg-pl-resp',{type:'bar',data:{labels:Object.keys(resp),datasets:[{data:Object.values(resp),backgroundColor:'rgba(59,130,246,.8)',borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},datalabels:{color:'#fff',font:{size:10,weight:'bold'},anchor:'end',align:'top'}},scales:{y:{...SC,ticks:{...SC.ticks,stepSize:1}},x:{grid:{display:false},ticks:{color:'#64748b',font:{size:9}}}}}});
  mkC('cg-pl-status',{type:'doughnut',data:{labels:['Pendente','Em andamento','Concluído'],datasets:[{data:[st.Pendente,st['Em andamento'],st.Concluído],backgroundColor:['#64748b','#f59e0b','#10b981'],borderWidth:2,borderColor:'#061020'}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{position:'bottom',labels:{color:'#64748b',font:{size:10}}},datalabels:{color:'#fff',font:{size:11,weight:'bold'},formatter:(v,c)=>v>0?`${c.chart.data.labels[c.dataIndex]}\n${v}`:''}}}});
  const tList=Object.values(turnos);
  const tLabs=tList.map(t=>`${t.lider}\n${t.data}`);
  const mkLine=(i,cor)=>({label:(curT().ocLabels||[])[i]||`Oc${i+1}`,data:tList.map(t=>(t.ocVals||[])[i]||0),borderColor:cor,backgroundColor:cor+'18',tension:.4,fill:false,pointRadius:5,pointBackgroundColor:cor});
  mkC('cg-pl-tendencia',{type:'line',data:{labels:tLabs.length?tLabs:['Turno atual'],datasets:[mkLine(0,'#ef4444'),mkLine(1,'#f97316'),mkLine(2,'#f59e0b'),mkLine(3,'#3b82f6')]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:'#64748b',font:{size:10}}},datalabels:{display:false}},scales:{y:SC,x:{grid:{color:GR},ticks:{color:'#64748b',font:{size:9}}}}}});
}

function incluirPlanos(){renderPlanosAba();renderGrafPlanos();fM('m-plano');notif('Planos incluídos!','🎯');}

// RECOMENDAÇÕES
async function gerarRecomendacoes(){
  const alertas=document.getElementById('d-alertas')?.innerText||'';
  if(!alertas||alertas.includes('Alertas aparecerão'))return;
  const t=curT();
  try{
    const res=await callAI([{type:'text',text:`Gestão industrial. Com base nos alertas e ocorrências, gere 4 recomendações objetivas. Cada linha começa com "—" e tem máximo 15 palavras. Responda SOMENTE as recomendações.
Alertas: ${alertas}
Eficiência: ${t.eficGeral||0}%, Glicerina: ${t.eficGlic||0}%, Estufas: ${t.eficEst||0}%`}]);
    const html=res.trim().split('\n').filter(l=>l.trim()).map(l=>`<p class="text-emerald-300">${l.trim()}</p>`).join('');
    setHTML('d-recomendacoes',html);setHTML('a-recom',html);
    if(turnoId&&turnos[turnoId])turnos[turnoId].recomendacoes=html;
    salvar();
  }catch(e){}
}

// IMPORTAR
function dropFile(e){e.preventDefault();const f=e.dataTransfer.files[0];if(f)processFile(f);}

// ═══ PROMPT INTELIGENTE ═══
function buildPrompt(){
  return `Você é especialista em leitura de relatórios de produção industrial brasileiros. Analise este documento com MÁXIMA PRECISÃO.
REGRAS OBRIGATÓRIAS:
1. IGNORE totalmente qualquer aba/seção chamada "Montagem" ou "montagem" - não extraia nada dela
2. Procure especificamente por: "1° Turno Estufa","1° Turno Glicerina","2° Turno Estufa","2° Turno Glicerina","3° Turno Estufa","3° Turno Glicerina"
3. Se encontrar dados de múltiplos turnos (1, 2 e/ou 3), extraia TODOS separadamente em objetos distintos
4. Para cada robô Glicerina extraia: nome(R-01,R-02A,R-02B,R-02C,R-03 a R-11), operador, produto(código BA...), meta e real produzido
5. Para Estufas extraia: posição(Est02-01 a Est02-13, Est03-01 a Est03-10), operador, produto, meta e real
6. Classifique ocorrências: CRÍTICO=paradas/quebras/bloqueios, ALTO=faltas/refugo elevado, MÉDIO=manutenção/atrasos, BAIXO=informações gerais
7. Extraia SOMENTE campos que existem claramente - não invente dados. null para campos ausentes.
8. lider = nome do responsável/supervisor. data = data de operação no formato DD/MM/AAAA.

Responda SOMENTE JSON válido sem markdown:
{"turnos":[{"numero":1,"lider":null,"data":null,"turno_label":"1º Turno (06:00-14:00)","meta_estufas":null,"real_estufas":null,"refugo_estufas":null,"meta_glicerina":null,"real_glicerina":null,"refugo_glicerina":null,"ocorrencias":[{"descricao":"","grau":"CRÍTICO|ALTO|MÉDIO|BAIXO","setor":"ESTUFAS|GLICERINA|GERAL"}],"robos_glic":[{"nome":"R-01","operador":"","produto":"","meta":0,"real":0}],"est02":[{"pos":"Est02-01","operador":"","produto":"","meta":0,"real":0}],"est03":[{"pos":"Est03-01","operador":"","produto":"","meta":0,"real":0}],"ops02":["","","","","",""],"ops03":["","","",""]}]}`;
}

async function processFile(file){
  if(!file)return;
  document.getElementById('im-result').classList.add('hidden');
  document.getElementById('im-loading').classList.remove('hidden');
  dadosImport={};
  const ext=file.name.split('.').pop().toLowerCase();
  const step=t=>document.getElementById('im-step').innerText=t;
  try{
    let raw='';
    if(['jpg','jpeg','png','webp','bmp','tiff'].includes(ext)){
      step('🔍 IA lendo imagem com alta precisão...');
      const b64=await toB64(file);
      // Use vision com prompt de alta precisão
      raw=await callAI([
        {type:'image',source:{type:'base64',media_type:file.type||'image/jpeg',data:b64}},
        {type:'text',text:buildPrompt()}
      ],3000);
    }else if(['xlsx','xls'].includes(ext)){
      step('📊 Lendo planilha — todas as abas...');
      const buf=await file.arrayBuffer();
      const wb=XLSX.read(buf,{type:'array'});
      // Lê TODAS as abas, ignora "Montagem"
      let allText='';
      wb.SheetNames.forEach(name=>{
        if(/montagem/i.test(name))return; // ignora Montagem
        const ws=wb.Sheets[name];
        const csv=XLSX.utils.sheet_to_csv(ws);
        allText+=`\n\n=== ABA: ${name} ===\n${csv}`;
      });
      step('🤖 IA analisando planilha...');
      raw=await callAI([{type:'text',text:buildPrompt()+'\n\nCONTEÚDO DA PLANILHA:\n'+allText.substring(0,6000)}],3000);
    }else if(ext==='csv'){
      step('📄 Lendo CSV...');
      const txt=await file.text();
      step('🤖 IA analisando dados...');
      raw=await callAI([{type:'text',text:buildPrompt()+'\n\nDADOS CSV:\n'+txt.substring(0,5000)}],3000);
    }else if(ext==='pdf'){
      step('🔍 IA analisando PDF...');
      const b64=await toB64(file);
      raw=await callAI([
        {type:'document',source:{type:'base64',media_type:'application/pdf',data:b64}},
        {type:'text',text:buildPrompt()}
      ],3000);
    }else throw new Error('Formato não suportado. Use JPG, PNG, PDF, XLSX ou CSV.');

    const parsed=JSON.parse(raw.replace(/```json|```/g,'').trim());
    dadosImport=parsed;
    mostrarResultImport();
  }catch(e){
    step('❌ Erro: '+e.message);
    console.error(e);
  }
  document.getElementById('im-loading').classList.add('hidden');
}

function mostrarResultImport(){
  const c=document.getElementById('im-campos');c.innerHTML='';
  const d=dadosImport;
  const turnos=d.turnos||[];
  if(!turnos.length){c.innerHTML='<p class="text-red-400 text-xs">Nenhum dado encontrado no arquivo.</p>';document.getElementById('im-result').classList.remove('hidden');return;}
  const grCor={CRÍTICO:'#f87171',ALTO:'#fb923c',MÉDIO:'#fbbf24',BAIXO:'#60a5fa'};
  const grEmoji={CRÍTICO:'🔴',ALTO:'🟠',MÉDIO:'🟡',BAIXO:'🔵'};
  turnos.forEach(t=>{
    c.innerHTML+=`<div class="text-xs font-bold text-cyan-400 mt-3 mb-1 border-b pb-1" style="border-color:var(--n700)">Turno ${t.numero} — ${t.lider||'?'} — ${t.data||'?'}</div>`;
    const add=(l,v,cor)=>{if(v!=null&&v!==''&&v!==0&&v!==null)c.innerHTML+=`<div class="flex justify-between text-xs py-1 border-b" style="border-color:var(--n750)"><span class="text-blue-400">${l}</span><span style="color:${cor||'#e2e8f0'}" class="font-mono">${v}</span></div>`;};
    add('Meta Estufas',t.meta_estufas,'#fbbf24');
    add('Real Estufas',t.real_estufas,'#60a5fa');
    add('Meta Glicerina',t.meta_glicerina,'#fbbf24');
    add('Real Glicerina',t.real_glicerina,'#60a5fa');
    add('Refugo Estufas',t.refugo_estufas,'#f87171');
    add('Refugo Glicerina',t.refugo_glicerina,'#f87171');
    (t.robos_glic||[]).filter(r=>r.real>0||r.produto).forEach(r=>add(`Robô ${r.nome} — ${r.operador||'?'}`,`${r.produto||''} Meta:${r.meta} Real:${r.real}`,'#06b6d4'));
    (t.est02||[]).filter(r=>r.real>0||r.produto).forEach(r=>add(`${r.pos} — ${r.operador||'?'}`,`${r.produto||''} Meta:${r.meta} Real:${r.real}`,'#f59e0b'));
    (t.est03||[]).filter(r=>r.real>0||r.produto).forEach(r=>add(`${r.pos} — ${r.operador||'?'}`,`${r.produto||''} Meta:${r.meta} Real:${r.real}`,'#14b8a6'));
    (t.ocorrencias||[]).forEach(oc=>add(`[${oc.grau}/${oc.setor}]`,oc.descricao,grCor[oc.grau]||'#e2e8f0'));
  });
  document.getElementById('im-result').classList.remove('hidden');
}

async function aplicarImport(){
  const d=dadosImport;
  const importTurnos=d.turnos||[];
  if(!importTurnos.length){notif('Nenhum dado para aplicar','❌');return;}
  const grEmoji={CRÍTICO:'🔴',ALTO:'🟠',MÉDIO:'🟡',BAIXO:'🔵'};
  const grCor={CRÍTICO:'#f87171',ALTO:'#fb923c',MÉDIO:'#fbbf24',BAIXO:'#60a5fa'};
  const ord={CRÍTICO:0,ALTO:1,MÉDIO:2,BAIXO:3};
  const turnoLabels={1:'1º Turno (06:00–14:00)',2:'2º Turno (14:00–22:00)',3:'3º Turno (22:00–06:00)'};
  let firstId=null;

  for(const dt of importTurnos){
    // Cria ou encontra turno correspondente
    let tid=Object.keys(turnos).find(k=>turnos[k]._importNum===dt.numero);
    if(!tid){
      tid='t_imp_'+dt.numero+'_'+Date.now();
      const nt=mkTurno();
      nt.id=tid;nt._importNum=dt.numero;
      turnos[tid]=nt;
    }
    const t=turnos[tid];
    if(!firstId)firstId=tid;

    // Cabeçalho
    if(dt.lider)t.lider=dt.lider;
    if(dt.data)t.data=dt.data;
    t.turno=dt.turno_label||turnoLabels[dt.numero]||`${dt.numero}º Turno`;
    if(dt.meta_estufas!=null)t.metaEst=+dt.meta_estufas;
    if(dt.real_estufas!=null)t.realEst=+dt.real_estufas;
    if(dt.refugo_estufas!=null)t.refugoEst=+dt.refugo_estufas;
    if(dt.meta_glicerina!=null)t.metaGlic=+dt.meta_glicerina;
    if(dt.real_glicerina!=null)t.realGlic=+dt.real_glicerina;
    if(dt.refugo_glicerina!=null)t.refugoGlic=+dt.refugo_glicerina;

    // Robôs Glicerina
    if(dt.robos_glic?.length){
      dt.robos_glic.forEach(rImport=>{
        const idx=t.glicRobos.findIndex(r=>r.nome===rImport.nome);
        if(idx>=0){
          if(rImport.operador)t.glicRobos[idx].operador=rImport.operador;
          if(rImport.produto)t.glicRobos[idx].produto=rImport.produto;
          if(rImport.meta>0)t.glicRobos[idx].meta=rImport.meta;
          if(rImport.real>0)t.glicRobos[idx].real=rImport.real;
        }
      });
    }

    // Estufa 02
    if(dt.est02?.length){
      dt.est02.forEach(eImport=>{
        const idx=t.est02.findIndex(e=>e.nome===eImport.pos);
        if(idx>=0){
          if(eImport.operador)t.est02[idx].operador=eImport.operador;
          if(eImport.produto)t.est02[idx].produto=eImport.produto;
          if(eImport.meta>0)t.est02[idx].meta=eImport.meta;
          if(eImport.real>0)t.est02[idx].real=eImport.real;
        }
      });
    }

    // Estufa 03
    if(dt.est03?.length){
      dt.est03.forEach(eImport=>{
        const idx=t.est03.findIndex(e=>e.nome===eImport.pos);
        if(idx>=0){
          if(eImport.operador)t.est03[idx].operador=eImport.operador;
          if(eImport.produto)t.est03[idx].produto=eImport.produto;
          if(eImport.meta>0)t.est03[idx].meta=eImport.meta;
          if(eImport.real>0)t.est03[idx].real=eImport.real;
        }
      });
    }

    // Operadores fixos
    if(dt.ops02?.length)t.ops02=dt.ops02.slice(0,6);
    if(dt.ops03?.length)t.ops03=dt.ops03.slice(0,4);

    // Ocorrências — ordenadas por grau
    if(dt.ocorrencias?.length){
      const sorted=[...dt.ocorrencias].filter(o=>!/montagem/i.test(o.descricao)).sort((a,b)=>(ord[a.grau]||9)-(ord[b.grau]||9));
      const alHTML=sorted.map(oc=>`<p style="color:${grCor[oc.grau]||'#e2e8f0'}">${grEmoji[oc.grau]||'⚪'} [${oc.grau}] ${oc.descricao}</p>`).join('');
      t.alertas=alHTML;
      const ocG=sorted.filter(o=>o.setor==='GLICERINA'||o.setor==='GERAL').map(o=>`<p style="color:${grCor[o.grau]}">${grEmoji[o.grau]} ${o.descricao}</p>`).join('');
      const ocE=sorted.filter(o=>o.setor==='ESTUFAS'||o.setor==='GERAL').map(o=>`<p style="color:${grCor[o.grau]}">${grEmoji[o.grau]} ${o.descricao}</p>`).join('');
      if(ocG)t.ocorrGlic=ocG;
      if(ocE)t.ocorrEst=ocE;
      sorted.slice(0,4).forEach((oc,i)=>{t.ocLabels[i]=oc.descricao.substring(0,22);t.ocVals[i]=i===0?3:i===1?2:1;});
    }
  }

  // Carrega o primeiro turno importado e atualiza tudo
  if(firstId){
    turnoId=firstId;
    carregarTurno(firstId);
  }
  renderHist();
  recalc();
  atuCharts();

  // Gera recomendações e planos automaticamente
  notif('Gerando recomendações e planos...','🤖');
  await gerarRecomendacoes();
  // Gera planos silenciosamente (sem abrir modal)
  await gerarPlanosSilencioso();

  salvar();
  fM('m-import');
  const n=importTurnos.length;
  notif(`${n} turno${n>1?'s':''} importado${n>1?'s':''} com sucesso!`,'✅');
}

// ADMIN SYNC
function sincAdmin(){
  const t=curT();if(!t.id)return;
  setAdm('a-lider',t.lider);setAdm('a-data',t.data);setAdm('a-turno',t.turno);
  setAdm('a-me',t.metaEst);setAdm('a-re',t.realEst);setAdm('a-rfest',t.refugoEst);
  setAdm('a-mg',t.metaGlic);setAdm('a-rg',t.realGlic);setAdm('a-rfglic',t.refugoGlic);
  const ocDiv=document.getElementById('a-oc-div');if(ocDiv){ocDiv.innerHTML='';
    (t.ocLabels||[]).forEach((l,i)=>{
      ocDiv.innerHTML+=`<div class="flex gap-1 items-center">
        <input class="inp inp-sm flex-1" value="${l}" placeholder="Label" oninput="curT().ocLabels[${i}]=this.value;const el=document.getElementById('oc-lbl-${i}');if(el)el.innerText=this.value;atuCharts()">
        <input type="number" class="inp inp-sm w-14 text-center" value="${(t.ocVals||[])[i]||0}" oninput="curT().ocVals[${i}]=+this.value;const vi=document.getElementById('oc-inp-${i}');if(vi)vi.value=this.value;atuCharts()">
      </div>`;
    });
  }
  const epDiv=document.getElementById('a-ep-div');if(epDiv){epDiv.innerHTML='';
    (t.epLabels||[]).forEach((l,i)=>{
      epDiv.innerHTML+=`<div class="flex gap-1 items-center">
        <input class="inp inp-sm flex-1" value="${l}" placeholder="Label" oninput="curT().epLabels[${i}]=this.value;atuCharts()">
        <input type="number" class="inp inp-sm w-14 text-center" value="${(t.epVals||[])[i]||0}" oninput="curT().epVals[${i}]=+this.value;atuCharts()">
      </div>`;
    });
  }
  const rfDiv=document.getElementById('a-rf-div');if(rfDiv){rfDiv.innerHTML='';
    (t.rfLabels||[]).forEach((l,i)=>{
      rfDiv.innerHTML+=`<div class="flex gap-1 items-center">
        <input id="rf-lbl-${i}" class="inp inp-sm flex-1" value="${l}" placeholder="Fonte" oninput="curT().rfLabels[${i}]=this.value;atuCharts()">
        <input id="rf-val-${i}" type="number" class="inp inp-sm w-14 text-center" value="${(t.rfVals||[])[i]||0}" oninput="curT().rfVals[${i}]=+this.value;atuCharts()">
      </div>`;
    });
  }
  const aa=document.getElementById('a-alertas');if(aa)aa.innerHTML=t.alertas||'';
  const ag=document.getElementById('a-goc');if(ag)ag.innerHTML=t.ocorrGlic||'';
  const ae=document.getElementById('a-eoc');if(ae)ae.innerHTML=t.ocorrEst||'';
  const ar=document.getElementById('a-recom');if(ar)ar.innerHTML=t.recomendacoes||'';
}

function aSync(idDisp,idAdm){const el=document.getElementById(idDisp);const adm=document.getElementById(idAdm);if(el&&adm)el.innerText=adm.value;}
function syncHTML(idDisp,idAdm){const el=document.getElementById(idDisp);const adm=document.getElementById(idAdm);if(el&&adm)el.innerHTML=adm.innerHTML;}

// LINK
function gerarLink(){
  salvarAtual();salvar();
  const payload=JSON.stringify({turnos,turnoId,ro:true});
  const b64=btoa(unescape(encodeURIComponent(payload)));
  const url=`${location.href.split('?')[0]}?d=${b64}`;
  document.getElementById('a-link').value=url.length>8000?'Dados grandes demais — use Exportar JSON':url;
}
function copiarLink(){
  const v=document.getElementById('a-link').value;
  navigator.clipboard.writeText(v).then(()=>{
    const el=document.getElementById('link-ok');el.classList.remove('hidden');
    setTimeout(()=>el.classList.add('hidden'),2500);
  });
}
function verificarLink(){
  const p=new URLSearchParams(location.search).get('d');
  if(!p)return;
  try{
    const d=JSON.parse(decodeURIComponent(escape(atob(p))));
    if(d.turnos)turnos=d.turnos;
    if(d.turnoId)turnoId=d.turnoId;
    else if(Object.keys(turnos).length)turnoId=Object.keys(turnos)[0];
    if(turnoId)carregarTurno(turnoId);
    renderHist();notif('Relatório compartilhado!','📊');
  }catch(e){}
}

// PDF
async function gerarPDF(){
  notif('Gerando PDF...','📄');
  const {jsPDF}=window.jspdf;

  // Força todos os gráficos a renderizar antes de capturar
  Object.values(charts).forEach(c=>{try{c.resize();c.update();}catch(e){}});
  await new Promise(r=>setTimeout(r,400));

  const secs=[
    {id:'aba-consolidado',titulo:'Visão Consolidada'},
    {id:'aba-glicerina',   titulo:'Setor Glicerina'},
    {id:'aba-estufas',     titulo:'Setor Estufas'},
    {id:'aba-planos',      titulo:'Plano de Ação'},
    {id:'aba-comparacao',  titulo:'Comparativo de Turnos'}
  ];

  const pdf=new jsPDF({orientation:'landscape',unit:'mm',format:'a4'});
  const pW=pdf.internal.pageSize.getWidth(); // 297mm
  const pH=pdf.internal.pageSize.getHeight(); // 210mm
  const HDR=12; // altura do cabeçalho
  const MARGIN=2;
  const areaH=pH-HDR-MARGIN;
  let first=true;

  const addHeader=(titulo)=>{
    pdf.setFillColor(6,16,32);
    pdf.rect(0,0,pW,HDR,'F');
    pdf.setTextColor(147,197,253);
    pdf.setFontSize(9);
    pdf.setFont('helvetica','bold');
    pdf.text('SISTEMA PCP',MARGIN+1,8);
    pdf.setFont('helvetica','normal');
    pdf.setTextColor(100,116,139);
    pdf.setFontSize(8);
    pdf.text(`${titulo}  ·  ${getText('h-lider')}  ·  ${getText('h-data')}  ·  ${getText('h-turno')}`,38,8);
    // linha separadora
    pdf.setDrawColor(19,47,88);
    pdf.line(0,HDR,pW,HDR);
  };

  for(const s of secs){
    const el=document.getElementById(s.id);
    if(!el)continue;
    const wasH=el.classList.contains('hidden');

    // Mostra o elemento, espera render completo
    el.classList.remove('hidden');
    // Redimensiona gráficos dentro dessa aba
    Object.values(charts).forEach(c=>{try{c.resize();c.update();}catch(e){}});
    await new Promise(r=>setTimeout(r,300));

    try{
      // Captura em alta resolução
      const canvas=await html2canvas(el,{
        scale:2,
        backgroundColor:'#020b16',
        useCORS:true,
        allowTaint:true,
        logging:false,
        imageTimeout:8000,
        onclone:(doc)=>{
          // Garante que canvas dos gráficos estejam visíveis no clone
          doc.querySelectorAll('canvas').forEach(cv=>{
            cv.style.display='block';
          });
        }
      });

      const imgData=canvas.toDataURL('image/png'); // PNG para melhor qualidade nos gráficos
      const imgW=canvas.width;
      const imgH=canvas.height;
      // Proporção real em mm dentro da área disponível
      const ratio=Math.min(pW/imgW, areaH/(imgH/2));
      const drawW=imgW*ratio;
      const drawH=imgH*ratio;

      // Se a imagem cabe em uma página
      if(drawH<=areaH){
        if(!first)pdf.addPage();first=false;
        addHeader(s.titulo);
        // Centraliza horizontalmente
        const xOff=(pW-drawW)/2;
        pdf.addImage(imgData,'PNG',xOff,HDR+MARGIN,drawW,drawH,'','FAST');
      }else{
        // Divide em múltiplas páginas — corta por blocos de areaH
        const totalPages=Math.ceil(drawH/areaH);
        for(let pg=0;pg<totalPages;pg++){
          if(!first||pg>0)pdf.addPage();first=false;
          addHeader(pg===0?s.titulo:`${s.titulo} (${pg+1}/${totalPages})`);
          // Calcula qual fatia da imagem original corresponde a esta página
          const srcY=Math.round((pg*areaH/ratio));
          const srcH=Math.round(areaH/ratio);
          // Cria canvas temporário com a fatia
          const slice=document.createElement('canvas');
          slice.width=imgW;
          slice.height=Math.min(srcH,imgH-srcY);
          const ctx2=slice.getContext('2d');
          ctx2.drawImage(canvas,0,srcY,imgW,slice.height,0,0,imgW,slice.height);
          const sliceData=slice.toDataURL('image/png');
          const sliceDrawH=(slice.height*ratio);
          const xOff=(pW-drawW)/2;
          pdf.addImage(sliceData,'PNG',xOff,HDR+MARGIN,drawW,sliceDrawH,'','FAST');
        }
      }
    }catch(e){console.error('PDF erro em '+s.titulo,e);}

    if(wasH)el.classList.add('hidden');
    await new Promise(r=>setTimeout(r,80));
  }

  const lider=getText('h-lider').replace(/\s+/g,'_')||'PCP';
  const dt=getText('h-data').replace(/\//g,'-')||new Date().toLocaleDateString('pt-BR').replace(/\//g,'-');
  pdf.save(`Relatorio_PCP_${lider}_${dt}.pdf`);
  notif('PDF gerado com sucesso!','✅');
}

// EXPORT/IMPORT
function exportJSON(){salvarAtual();const blob=new Blob([JSON.stringify({turnos,turnoId},null,2)],{type:'application/json'});const a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download='backup_pcp.json';a.click();}
function importJSON(file){if(!file)return;const r=new FileReader();r.onload=e=>{try{const d=JSON.parse(e.target.result);if(d.turnos)turnos=d.turnos;if(d.turnoId)turnoId=d.turnoId||Object.keys(d.turnos||{})[0];if(turnoId)carregarTurno(turnoId);renderHist();salvar();notif('Backup restaurado!','✅');}catch(ex){notif('Erro JSON','❌');}};r.readAsText(file);}

// STORAGE
async function salvar(){
  salvarAtual();
  const p=JSON.stringify({turnos,turnoId});
  try{await window.storage.set(SK,p);}catch(e){try{localStorage.setItem(SK,p);}catch(ex){}}
  document.getElementById('r-ts').innerText='Salvo: '+new Date().toLocaleTimeString('pt-BR');
}
async function carregar(){
  let d=null;
  try{const r=await window.storage.get(SK);if(r?.value)d=JSON.parse(r.value);}
  catch(e){try{const v=localStorage.getItem(SK);if(v)d=JSON.parse(v);}catch(ex){}}
  if(d?.turnos){turnos=d.turnos;turnoId=d.turnoId||Object.keys(d.turnos)[0]||null;if(turnoId){carregarTurno(turnoId);renderHist();}return true;}
  return false;
}

// UTILS
const oM=id=>document.getElementById(id)?.classList.remove('hidden');
const fM=id=>document.getElementById(id)?.classList.add('hidden');
const setText=(id,v)=>{const e=document.getElementById(id);if(e)e.innerText=v;};
const setHTML=(id,v)=>{const e=document.getElementById(id);if(e)e.innerHTML=v;};
const getText=id=>document.getElementById(id)?.innerText||'';
const gN=id=>{const el=document.getElementById(id);return el?(parseInt(el.value)||0):0;};
const setAdm=(id,v)=>{const e=document.getElementById(id);if(e&&v!=null)e.value=v;};
const upd=(id,fn)=>{try{if(charts[id]){fn(charts[id].data);charts[id].update('none');}}catch(e){}};
const toB64=f=>new Promise((res,rej)=>{const r=new FileReader();r.onload=()=>res(r.result.split(',')[1]);r.onerror=rej;r.readAsDataURL(f);});
async function callAI(content,maxTok=2000){const res=await fetch('https://api.anthropic.com/v1/messages',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({model:'claude-sonnet-4-20250514',max_tokens:maxTok,messages:[{role:'user',content}]})});const d=await res.json();if(d.error)throw new Error(d.error.message||'API error');return d.content.map(b=>b.text||'').join('');}
function notif(msg,icon='💾'){document.getElementById('n-icon').innerText=icon;document.getElementById('n-msg').innerText=msg;const n=document.getElementById('notif-box');n.classList.remove('hidden');setTimeout(()=>n.classList.add('hidden'),2500);}

// INIT
window.addEventListener('DOMContentLoaded',async()=>{
  document.getElementById('nt-data').value=new Date().toISOString().split('T')[0];
  initCharts();
  const ok=await carregar();
  if(!ok){
    const id='t_demo';const t=mkTurno();
    t.id=id;t.lider='Esequiel';t.data='02/06/2026';t.turno='3º Turno (22:00–06:00)';
    t.metaEst=6725;t.realEst=6141;t.refugoEst=45;t.metaGlic=7852;t.realGlic=4901;t.refugoGlic=56;
    t.ocVals=[3,2,5,1];t.epVals=[2,1,1,1];t.rfVals=[33,9,9,50];
    t.alertas='<p style="color:#f87171">🔴 [CRÍTICO] Robô 1 QUEBROU às 20:20 (1 Op. e 4 Gabs).</p><p style="color:#fb923c">🟠 [ALTO] Yuri deslocado do Robô 1 para Lavagem.</p><p style="color:#fbbf24">🟡 [MÉDIO] Operadores em treinamento listados.</p><p style="color:#60a5fa">🔵 [BAIXO] BA0300 bloqueado por qualidade (9 peças).</p>';
    t.recomendacoes='<p class="text-emerald-300">— Priorizar manutenção corretiva do Robô 1.</p><p class="text-emerald-300">— Revisão técnica no Robô 5 (refugo alto).</p><p class="text-emerald-300">— Suporte para treinamento intensivo.</p><p class="text-emerald-300">— Revisar logística de gabaritos Estufa 01-02.</p>';
    t.ocorrGlic='<p style="color:#f87171">🔴 Robô 1 quebrou às 20:20 — operação limitada.</p><p style="color:#fb923c">🟠 Robô 5 com refugo elevado — Hiago.</p>';
    t.ocorrEst='<p style="color:#f87171">🔴 Manutenção Estufa 02 — barramentos térmicos.</p><p style="color:#fbbf24">🟡 Ajuste de gabarito de engenharia.</p>';
    t.glicRobos[0]={nome:'R-01',operador:'João',produto:'BA0283',meta:500,real:420};
    t.glicRobos[1]={nome:'R-02A',operador:'Yuri',produto:'BA0295',meta:300,real:270};
    t.glicRobos[4]={nome:'R-03',operador:'Maria',produto:'BA0257',meta:200,real:56};
    t.glicRobos[5]={nome:'R-04',operador:'Hiago',produto:'BA0318',meta:400,real:434};
    t.est02[0]={nome:'Est02-01',operador:'Pedro',produto:'BA0290',meta:280,real:250};
    t.est02[1]={nome:'Est02-02',operador:'Ana',produto:'BA0297',meta:250,real:200};
    t.ops02=['Pedro','Ana','Carlos','Fernanda','Lucas','Julia'];
    t.ops03=['Roberto','Camila','Diego','Patrícia'];
    turnos[id]=t;turnoId=id;carregarTurno(id);renderHist();salvar();
  }
  verificarLink();aba('consolidado');
  window.addEventListener('beforeunload',()=>salvar());
  setInterval(()=>salvar(),30000);
  document.addEventListener('click',e=>{
    if(!document.getElementById('btn3pts').contains(e.target)&&!document.getElementById('menu3pts').contains(e.target))
      document.getElementById('menu3pts').classList.add('hidden');
  });
  document.getElementById('r-ts').innerText='Carregado: '+new Date().toLocaleTimeString('pt-BR');
});
</script>
</body>
</html>
