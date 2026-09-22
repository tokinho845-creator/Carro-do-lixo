<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Onde Está o Caminhão do Lixo? • Rastreamento Urbano</title>
  
  <!-- LIGAÇÃO AO MANIFESTO E SERVICE WORKER -->
  <link rel="manifest" href="./manifest.json"/>
  <meta name="theme-color" content="#0f172a"/>
  
  <link href="data:image/svg+xml;utf8,&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 512 512&quot;&gt;&lt;circle cx=&quot;256&quot; cy=&quot;256&quot; fill=&quot;%23059669&quot; r=&quot;240&quot;/&gt;&lt;path d=&quot;M110 210h190v120H110z&quot; fill=&quot;%23ffffff&quot;/&gt;&lt;path d=&quot;M300 230l80 50v60H300z&quot; fill=&quot;%231e293b&quot;/&gt;&lt;circle cx=&quot;160&quot; cy=&quot;350&quot; r=&quot;30&quot; fill=&quot;%231e293b&quot;/&gt;&lt;circle cx=&quot;330&quot; cy=&quot;350&quot; r=&quot;30&quot; fill=&quot;%231e293b&quot;/&gt;&lt;/svg&gt;" rel="icon" type="image/svg+xml"/>
  <link href="data:image/svg+xml;utf8,&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 512 512&quot;&gt;&lt;circle cx=&quot;256&quot; cy=&quot;256&quot; fill=&quot;%23059669&quot; r=&quot;240&quot;/&gt;&lt;path d=&quot;M110 210h190v120H110z&quot; fill=&quot;%23ffffff&quot;/&gt;&lt;path d=&quot;M300 230l80 50v60H300z&quot; fill=&quot;%231e293b&quot;/&gt;&lt;circle cx=&quot;160&quot; cy=&quot;350&quot; r=&quot;30&quot; fill=&quot;%231e293b&quot;/&gt;&lt;circle cx=&quot;330&quot; cy=&quot;350&quot; r=&quot;30&quot; fill=&quot;%231e293b&quot;/&gt;&lt;/svg&gt;" rel="apple-touch-icon"/>
  
  <meta name="apple-mobile-web-app-capable" content="yes"/>
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
  <meta name="apple-mobile-web-app-title" content="Caminhão Lixo"/>

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome para Ícones -->
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet"/>

  <!-- Leaflet CSS & JS para o Mapa -->
  <link href="https://unpkg.com/leaflet/dist/leaflet.css" rel="stylesheet"/>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>

  <style>
    body { background-color: #0f172a; margin: 0; padding: 0; font-family: sans-serif; }
    #toast-container { position: fixed; bottom: 20px; right: 20px; z-index: 99999; display: flex; flex-direction: column; gap: 10px; }
    .toast-msg { background: #1e293b; color: #f8fafc; border: 1px solid #334155; padding: 12px 18px; border-radius: 14px; font-size: 13px; box-shadow: 0 10px 25px -5px rgba(0,0,0,0.5); display: flex; align-items: center; gap: 10px; animation: slideIn 0.3s ease forwards; }
    @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
    #mapaContainerPrincipal { height: 420px; width: 100%; border-radius: 1rem; z-index: 10; }
    .custom-truck-icon, .custom-bin-icon { background: transparent; border: none; }
  </style>
</head>

<body>
  <div id="toast-container"></div>

  <div class="min-h-screen bg-slate-900 text-slate-100 font-sans p-4 md:p-8 flex flex-col justify-between" id="app">
    <div class="max-w-3xl mx-auto w-full space-y-6 my-auto">
      
      <div class="flex justify-between items-center px-2">
        <span class="text-[10px] text-slate-500 font-mono">Plataforma SaaS • v5.9 (GitHub PWA)</span>
        <button class="text-slate-500 hover:text-teal-400 text-xs transition flex items-center gap-1.5 bg-slate-800/50 px-3 py-1.5 rounded-xl border border-slate-700/50" onclick="abrirModalMasterAdmin()">
          <i class="fa-solid fa-user-shield text-teal-400"></i> Painel Master (Criador)
        </button>
      </div>

      <div class="bg-gradient-to-r from-teal-600 to-emerald-600 text-white p-4 rounded-3xl shadow-2xl flex items-center justify-between border border-teal-400/40">
        <div class="flex items-center gap-3">
          <div class="bg-white/20 p-3 rounded-2xl">
            <i class="fa-solid fa-truck text-2xl text-white"></i>
          </div>
          <div>
            <b class="text-xs md:text-sm block font-bold">Adicionar App ao Telemóvel</b>
            <span class="text-[11px] text-teal-100">Instale no ecrã inicial para acesso rápido.</span>
          </div>
        </div>
        <button class="bg-white text-teal-900 font-bold px-4 py-2.5 rounded-2xl text-xs shadow hover:bg-slate-100 transition" onclick="abrirModalInstrucoes()">
          Como Instalar
        </button>
      </div>

      <div class="text-center space-y-3 bg-slate-800/80 backdrop-blur border border-slate-700/50 p-6 rounded-3xl shadow-2xl">
        <div class="inline-block bg-gradient-to-tr from-emerald-500 to-teal-500 p-4 rounded-2xl text-white shadow-lg mb-1">
          <i class="fa-solid fa-truck text-3xl"></i>
        </div>
        <h1 class="text-2xl md:text-3xl font-bold tracking-wide text-white">Onde Está o Caminhão do Lixo?</h1>
        <p class="text-xs md:text-sm text-teal-400 font-medium">Acompanhe em tempo real a limpeza urbana e os pontos de descarte da sua cidade.</p>
      </div>

      <div class="bg-slate-800/90 border border-teal-500/30 p-4 rounded-3xl text-slate-300 text-xs md:text-sm space-y-3 shadow-xl">
        <p class="font-bold text-teal-400 flex items-center gap-2 text-sm">
          <i class="fa-solid fa-location-crosshairs text-base"></i> Rastreamento Automático por Região
        </p>
        <p class="text-slate-400 leading-relaxed">
          Toque no botão abaixo para permitir o acesso à sua localização. O sistema deteta automaticamente a sua cidade para mostrar os camiões ativos e as lixeiras na sua zona.
        </p>
        <button class="w-full bg-gradient-to-r from-teal-600 to-emerald-600 hover:opacity-95 text-white font-semibold py-3.5 rounded-2xl transition shadow-lg text-sm flex items-center justify-center space-x-2" onclick="ativarGeolocalizacaoCidadao()">
          <i class="fa-solid fa-crosshairs"></i>
          <span>Ativar Minha Localização e Ver o Mapa</span>
        </button>
      </div>

      <div class="bg-slate-800 border border-slate-700 p-4 rounded-3xl shadow-2xl space-y-3">
        <div class="flex justify-between items-center px-1">
          <span class="text-xs font-bold text-slate-300 uppercase tracking-wider"><i class="fa-solid fa-map mr-1 text-teal-400"></i> Mapa ao Vivo &amp; Lixeiras</span>
          <span class="text-[10px] text-emerald-400 bg-emerald-950 px-2.5 py-1 rounded-full border border-emerald-800/50 flex items-center gap-1 font-medium">
            <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span> Sistema Ativo
          </span>
        </div>
        <div id="mapaContainerPrincipal"></div>
      </div>

    </div>

    <footer class="text-center pt-8 space-y-2">
      <div class="flex justify-center gap-4 text-xs">
        <button class="text-slate-400 hover:text-teal-400 transition" onclick="abrirModalLoginEmpresa()">
          <i class="fa-solid fa-building mr-1"></i> Painel da Empresa
        </button>
        <span class="text-slate-700">•</span>
        <button class="text-slate-400 hover:text-emerald-400 transition" onclick="abrirModalMotorista()">
          <i class="fa-solid fa-truck-fast mr-1"></i> Área do Motorista
        </button>
      </div>
      <p class="text-slate-600 text-[10px]">
        Sistema de Rastreamento Urbano &copy; 2026 • Desenvolvido para Prefeituras e Empresas de Limpeza.
      </p>
    </footer>
  </div>

  <!-- MODAIS -->
  <div class="hidden fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center p-4 z-50" id="modalInstrucoes">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-md p-6 rounded-3xl shadow-2xl space-y-4">
      <div class="flex justify-between items-center border-b border-slate-700 pb-3">
        <h3 class="text-base font-bold text-teal-400"><i class="fa-solid fa-mobile-screen mr-2"></i> Como Adicionar ao Ecrã</h3>
        <button class="text-slate-400 hover:text-white px-2 py-1" onclick="fecharModalInstrucoes()"><i class="fa-solid fa-xmark text-lg"></i></button>
      </div>
      <div class="space-y-3 text-xs text-slate-300 leading-relaxed">
        <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 space-y-2">
          <b class="text-teal-400 block"><i class="fa-brands fa-android mr-1"></i> No Android (Google Chrome):</b>
          <p>1. Toque nos três pontos (<i class="fa-solid fa-ellipsis-vertical"></i>) no canto superior direito.</p>
          <p>2. Selecione <b class="text-white">"Instalar aplicação"</b> ou <b class="text-white">"Adicionar ao ecrã principal"</b>.</p>
        </div>
      </div>
      <button class="w-full bg-teal-600 hover:bg-teal-500 text-white py-3 rounded-xl transition text-xs font-semibold" onclick="fecharModalInstrucoes()">Entendido</button>
    </div>
  </div>

  <div class="hidden fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center p-4 z-50" id="modalMasterAdmin">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-lg p-6 rounded-3xl shadow-2xl space-y-4 max-h-[90vh] overflow-y-auto">
      <div class="flex justify-between items-center border-b border-slate-700 pb-3">
        <h3 class="text-base font-bold text-teal-400"><i class="fa-solid fa-user-shield mr-2"></i> Painel Master</h3>
        <button class="text-slate-400 hover:text-white px-2 py-1" onclick="fecharModalMasterAdmin()"><i class="fa-solid fa-xmark text-lg"></i></button>
      </div>
      <div class="space-y-3" id="ecraLoginMaster">
        <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2.5 text-white text-xs" id="masterLoginEmail" placeholder="E-mail" type="email"/>
        <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2.5 text-white text-xs" id="masterLoginSenha" placeholder="Senha" type="password"/>
        <button class="w-full bg-teal-600 hover:bg-teal-500 text-white font-semibold py-3 rounded-xl text-xs" onclick="fazerLoginMaster()">Entrar</button>
      </div>
      <div class="hidden space-y-4" id="ecraPainelMaster">
        <p class="text-xs text-teal-400">Painel Master Ativo</p>
      </div>
      <button class="w-full bg-slate-700 text-slate-300 py-2 rounded-xl text-xs" onclick="fecharModalMasterAdmin()">Fechar</button>
    </div>
  </div>

  <div class="hidden fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center p-4 z-50" id="modalLoginEmpresa">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-md p-6 rounded-3xl shadow-2xl space-y-4">
      <div class="flex justify-between items-center border-b border-slate-700 pb-3">
        <h3 class="text-base font-bold text-teal-400">Painel da Empresa</h3>
        <button class="text-slate-400 hover:text-white px-2 py-1" onclick="fecharModalLoginEmpresa()"><i class="fa-solid fa-xmark text-lg"></i></button>
      </div>
      <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2.5 text-white text-xs" id="emailEmpresaInput" placeholder="E-mail" type="email"/>
      <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2.5 text-white text-xs" id="senhaEmpresaInput" placeholder="Senha" type="password"/>
      <button class="w-full bg-teal-600 text-white font-semibold py-3 rounded-xl text-xs" onclick="fazerLoginEmpresa()">Entrar</button>
      <button class="w-full bg-slate-700 text-slate-300 py-2 rounded-xl text-xs" onclick="fecharModalLoginEmpresa()">Voltar</button>
    </div>
  </div>

  <div class="hidden fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center p-4 z-50" id="modalPainelEmpresa">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-lg p-6 rounded-3xl shadow-2xl space-y-4 max-h-[90vh] overflow-y-auto">
      <h3 class="text-base font-bold text-teal-400" id="tituloEmpresaLogada">Gestão</h3>
      <button class="w-full bg-slate-700 text-slate-300 py-2 rounded-xl text-xs" onclick="fecharPainelEmpresa()">Fechar</button>
    </div>
  </div>

  <div class="hidden fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center p-4 z-50" id="modalMotorista">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-md p-6 rounded-3xl shadow-2xl space-y-4">
      <h3 class="text-base font-bold text-emerald-400">Área do Motorista</h3>
      <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white text-xs" id="nomeMotoristaInput" placeholder="Seu Nome"/>
      <input class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white text-xs" id="codigoMotoristaInput" placeholder="PIN do Veículo"/>
      <button class="w-full bg-emerald-600 text-white py-3 rounded-xl text-xs" onclick="fazerLoginMotorista()">Entrar</button>
      <button class="w-full bg-slate-700 text-slate-300 py-2 rounded-xl text-xs" onclick="fecharModalMotorista()">Voltar</button>
    </div>
  </div>

  <!-- SCRIPT DE REGISTO DO SERVICE WORKER E LÓGICA -->
  <script>
    // Registo do Service Worker para habilitar o PWA no GitHub Pages
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', function() {
        navigator.serviceWorker.register('./sw.js').catch(function(err) {
          console.log('Falha ao registar o Service Worker: ', err);
        });
      });
    }

    window.abrirModalInstrucoes = function() { document.getElementById('modalInstrucoes').classList.remove('hidden'); };
    window.fecharModalInstrucoes = function() { document.getElementById('modalInstrucoes').classList.add('hidden'); };
    window.abrirModalMasterAdmin = function() { document.getElementById('modalMasterAdmin').classList.remove('hidden'); };
    window.fecharModalMasterAdmin = function() { document.getElementById('modalMasterAdmin').classList.add('hidden'); };
    window.abrirModalLoginEmpresa = function() { document.getElementById('modalLoginEmpresa').classList.remove('hidden'); };
    window.fecharModalLoginEmpresa = function() { document.getElementById('modalLoginEmpresa').classList.add('hidden'); };
    window.fecharPainelEmpresa = function() { document.getElementById('modalPainelEmpresa').classList.add('hidden'); };
    window.abrirModalMotorista = function() { document.getElementById('modalMotorista').classList.remove('hidden'); };
    window.fecharModalMotorista = function() { document.getElementById('modalMotorista').classList.add('hidden'); };

    let db = null;
    let auth = null;
    let mapInstance = null;
    let markerInstances = {};
    let binMarkerInstances = {};
    let unsubscribeFirestore = null;
    let unsubscribeBins = null;
    let motoristaAtual = null;
    let empresaAtual = null;
    const ADMIN_UID_MASTER = "PLpqWf4jbAbsgI7IQ4alnYt1ZaR2";

    window.mostrarToast = function(mensagem, tipo = 'sucesso') {
      const container = document.getElementById('toast-container');
      if(!container) return;
      const toast = document.createElement('div');
      toast.className = 'toast-msg';
      toast.innerHTML = '<span>' + mensagem + '</span>';
      container.appendChild(toast);
      setTimeout(() => toast.remove(), 3500);
    };

    window.addEventListener('DOMContentLoaded', function() {
      try {
        const firebaseConfig = {
          apiKey: "AIzaSyDDWaIje-Yo2y_dS67eJLaUiZ9dY_cWIs0",
          authDomain: "cndbx1bet-72b3e.firebaseapp.com",
          databaseURL: "https://cndbx1bet-default-rtdb.firebaseio.com",
          projectId: "cndbx1bet",
          storageBucket: "cndbx1bet.firebasestorage.app",
          messagingSenderId: "211001459800",
          appId: "1:211001459800:web:ce4bbde8bf265d61db762d"
        };
        if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);
        db = firebase.firestore();
        auth = firebase.auth();
        window.inicializarMapaPrincipal("geral");
      } catch (err) { console.error(err); }
    });

    window.inicializarMapaPrincipal = function(cidadeFiltro = "geral") {
      const container = document.getElementById('mapaContainerPrincipal');
      if(!container) return;
      if(!mapInstance) {
        mapInstance = L.map('mapaContainerPrincipal').setView([-9.665, -35.735], 14);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(mapInstance);
      } else {
        mapInstance.invalidateSize();
      }
    };
  </script>
</body>
</html>
