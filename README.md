# faga-da-bolinha-<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>RPG 2D — Bosses a Cada 10 Ondas</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Georgia', serif;
    background: #0a0a15;
    color: #e0e0e0;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 15px;
    gap: 12px;
  }
  h1 {
    color: #c9a227;
    font-size: 1.3em;
    text-shadow: 0 0 10px rgba(201,162,39,0.5);
    text-align: center;
  }
  #hud {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
    justify-content: center;
    background: #1a1a2e;
    padding: 10px 18px;
    border-radius: 10px;
    border: 1px solid #4a3f6b;
    font-size: 0.88em;
  }
  #hud span { color: #f0c040; font-weight: bold; }

  #game-wrap {
    position: relative;
    border: 3px solid #6a4fbf;
    border-radius: 10px;
    box-shadow: 0 0 40px rgba(120,80,200,0.4);
    overflow: hidden;
    background: #000;
  }
  canvas { display: block; image-rendering: pixelated; }

  #controles {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    justify-content: center;
    font-size: 0.82em;
    color: #8888aa;
  }
  #controles kbd {
    background: #2a1f4b;
    border: 1px solid #6a4fbf;
    border-radius: 4px;
    padding: 2px 6px;
    color: #e0e0e0;
    font-family: monospace;
  }

  #hud-inferior {
    position: absolute;
    bottom: 10px;
    left: 10px;
    right: 10px;
    display: flex;
    gap: 10px;
    align-items: center;
    pointer-events: none;
  }
  .stat-bar {
    flex: 1;
    background: rgba(0,0,0,0.75);
    border: 2px solid #444;
    border-radius: 6px;
    height: 22px;
    overflow: hidden;
    position: relative;
  }
  .stat-bar > div { height: 100%; transition: width 0.15s; }
  .stat-bar .fill-hp { background: linear-gradient(90deg, #cc2222, #ff5555); }
  .stat-bar .fill-mp { background: linear-gradient(90deg, #2244cc, #5577ff); }
  .stat-bar span {
    position: absolute;
    inset: 0;
    text-align: center;
    font-size: 0.8em;
    line-height: 20px;
    font-weight: bold;
    text-shadow: 1px 1px 2px #000;
    color: #fff;
  }

  #msg {
    position: absolute;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0,0,0,0.85);
    border: 2px solid #c9a227;
    padding: 8px 20px;
    border-radius: 8px;
    font-size: 0.95em;
    color: #f0c040;
    opacity: 0;
    transition: opacity 0.3s;
    pointer-events: none;
    z-index: 5;
    white-space: nowrap;
  }
  #msg.ativo { opacity: 1; }

  /* ===== BARRA DE BOSS ===== */
  #boss-hud {
    position: absolute;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    width: 80%;
    max-width: 600px;
    display: none;
    z-index: 10;
    pointer-events: none;
  }
  #boss-hud.ativo { display: block; }
  #boss-nome {
    text-align: center;
    color: #ff4444;
    font-size: 1.1em;
    font-weight: bold;
    text-shadow: 0 0 10px #ff0000, 2px 2px 4px #000;
    margin-bottom: 4px;
    animation: bossPulse 2s infinite;
  }
  @keyframes bossPulse {
    0%, 100% { text-shadow: 0 0 10px #ff0000, 2px 2px 4px #000; }
    50% { text-shadow: 0 0 20px #ff0000, 2px 2px 4px #000; }
  }
  #boss-barra-outer {
    background: #000;
    border: 3px solid #8b3030;
    border-radius: 8px;
    height: 24px;
    overflow: hidden;
    box-shadow: 0 0 20px rgba(255, 0, 0, 0.6);
  }
  #boss-barra-inner {
    height: 100%;
    width: 100%;
    background: linear-gradient(90deg, #8b0000, #ff0000, #ff4444);
    transition: width 0.3s;
    box-shadow: inset 0 0 10px rgba(255, 200, 200, 0.5);
  }

  #upgrade-menu {
    position: absolute;
    inset: 0;
    background: rgba(0,0,0,0.94);
    display: none;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 30;
    padding: 20px;
    text-align: center;
    overflow-y: auto;
  }
  #upgrade-menu.ativo { display: flex; }
  #upgrade-titulo {
    font-size: 1.7em;
    color: #f0c040;
    text-shadow: 0 0 20px rgba(240,192,64,0.6);
    margin-bottom: 5px;
  }
  #upgrade-sub { color: #aaa; margin-bottom: 8px; font-size: 0.95em; }
  #ouro-loja { color: #f0c040; font-size: 1.1em; margin-bottom: 12px; font-weight: bold; }
  .upgrade-opcoes {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 8px;
    width: 100%;
    max-width: 720px;
    margin-bottom: 12px;
  }
  .upgrade-card {
    background: linear-gradient(180deg, #2a1f4b, #1a1030);
    border: 2px solid #6a4fbf;
    border-radius: 10px;
    padding: 10px 8px;
    cursor: pointer;
    transition: all 0.15s;
    text-align: center;
  }
  .upgrade-card:hover:not(.bloqueado) {
    border-color: #f0c040;
    transform: translateY(-3px);
    box-shadow: 0 6px 20px rgba(240,192,64,0.3);
  }
  .upgrade-card.bloqueado { opacity: 0.4; cursor: not-allowed; border-color: #444; }
  .upgrade-card .icone { font-size: 1.9em; display: block; margin-bottom: 4px; }
  .upgrade-card .nome { color: #f0c040; font-size: 0.82em; font-weight: bold; margin-bottom: 4px; }
  .upgrade-card .desc { color: #bbb; font-size: 0.7em; line-height: 1.3; margin-bottom: 5px; min-height: 32px; }
  .upgrade-card .nivel { color: #88ff88; font-size: 0.75em; font-weight: bold; }
  .upgrade-card .custo { color: #f0c040; font-size: 0.8em; font-weight: bold; margin-top: 3px; }
  .upgrade-card .custo.caro { color: #ff5555; }

  #btn-fechar {
    background: linear-gradient(180deg, #3a2f5b, #2a1f4b);
    color: #e0e0e0;
    border: 2px solid #c9a227;
    padding: 10px 30px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 1em;
    font-family: inherit;
    transition: all 0.15s;
  }
  #btn-fechar:hover { background: linear-gradient(180deg, #5a4f8b, #3a2f6b); transform: scale(1.05); }
  #btn-fechar.pulse { animation: pulseBtn 1.2s infinite; }
  @keyframes pulseBtn {
    0%, 100% { box-shadow: 0 0 0 0 rgba(240,192,64,0.6); }
    50% { box-shadow: 0 0 0 10px rgba(240,192,64,0); }
  }

  #overlay {
    position: absolute;
    inset: 0;
    background: rgba(0,0,0,0.9);
    display: none;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 20;
    padding: 30px;
    text-align: center;
  }
  #overlay.ativo { display: flex; }
  #overlay h2 { font-size: 2em; margin-bottom: 15px; }
  #overlay p { margin-bottom: 25px; line-height: 1.6; max-width: 500px; white-space: pre-line; font-size: 0.9em; }
  #overlay button {
    background: linear-gradient(180deg, #3a2f5b, #2a1f4b);
    color: #e0e0e0;
    border: 2px solid #c9a227;
    padding: 12px 28px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 1em;
    font-family: inherit;
  }
  #overlay button:hover { background: linear-gradient(180deg, #5a4f8b, #3a2f6b); }
</style>
</head>
<body>

<h1>⚔️ A Lenda do Cristal Sombrio — Bosses Épicos ⚔️</h1>

<div id="hud">
  <div>🪙 <span id="ouro">0</span></div>
  <div>⭐ <span id="xp">0</span></div>
  <div>📈 Nv <span id="nivel">1</span></div>
  <div>👹 <span id="inimigos">0</span></div>
  <div>🌊 Onda <span id="onda">0</span></div>
  <div>⚔️ Dano <span id="dano">9</span></div>
</div>

<div id="game-wrap">
  <canvas id="canvas" width="768" height="512"></canvas>

  <div id="boss-hud">
    <div id="boss-nome">👑 BOSS</div>
    <div id="boss-barra-outer">
      <div id="boss-barra-inner"></div>
    </div>
  </div>

  <div id="hud-inferior">
    <div class="stat-bar">
      <div class="fill-hp" id="barra-hp" style="width:100%"></div>
      <span id="txt-hp">80 / 80</span>
    </div>
    <div class="stat-bar" style="flex: 0 0 150px;">
      <div class="fill-mp" id="barra-mp" style="width:100%"></div>
      <span id="txt-mp">30 / 30</span>
    </div>
  </div>

  <div id="msg"></div>

  <div id="upgrade-menu">
    <div id="upgrade-titulo">🏪 LOJA DE UPGRADES</div>
    <div id="upgrade-sub">Escolha melhorias — clique várias vezes se tiver ouro!</div>
    <div id="ouro-loja">🪙 Ouro disponível: 0</div>
    <div class="upgrade-opcoes" id="opcoes"></div>
    <button id="btn-fechar" onclick="fecharLoja()">✅ FECHAR LOJA</button>
  </div>

  <div id="overlay">
    <h2 id="overlay-titulo"></h2>
    <p id="overlay-texto"></p>
    <button onclick="location.reload()">🔄 Recomeçar</button>
  </div>
</div>

<div id="controles">
  <span>Mover: <kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd> / <kbd>↑</kbd><kbd>↓</kbd><kbd>←</kbd><kbd>→</kbd></span>
  <span>Atacar: <kbd>Espaço</kbd></span>
  <span>Habilidade: <kbd>E</kbd></span>
  <span>Poção: <kbd>Q</kbd></span>
</div>

<script>
// ============================================================
//  RPG 2D — Bosses a Cada 10 Ondas
// ============================================================

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 32;
const COLS = canvas.width / TILE;
const ROWS = canvas.height / TILE;

// ===== JOGADOR =====
const jogador = {
  x: 384, y: 256,
  w: 24, h: 24,
  dir: 'down',
  hp: 80, hpMax: 80,
  mp: 30, mpMax: 30,
  forca: 9,
  ouro: 0, xp: 0, nivel: 1,
  pocoes: 2,
  armaBonus: 0,
  vel: 2.6,
  cooldownAtaque: 0,
  cooldownDano: 0,
  frameAnim: 0,
  atacando: 0,
  upgRegen: 0, upgVel: 0, upgAtk: 0, upgVida: 0,
  upgDano: 0, upgCrit: 0, upgEscudo: 0, upgColeta: 0,
  ultimoNivelLoja: 0,
  envenenado: 0, envenenadoDano: 0,
  lento: 0, queimando: 0
};

// ===== MAPA =====
const mapaTiles = [
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
];

// ===== MONSTROS NORMAIS =====
const TIPOS_MONSTROS = {
  slime:    { nome:'Slime',    icone:'🟢', hp:30, dano:6,  xp:25, ouro:10, vel:0.6, cor:'#55cc55', corBorda:'#228822', tamanho:22, alcance:26, tipoAtaque:'corpo' },
  goblin:   { nome:'Goblin',   icone:'👺', hp:50, dano:10, xp:45, ouro:20, vel:1.2, cor:'#88aa44', corBorda:'#446622', tamanho:24, alcance:28, tipoAtaque:'corpo' },
  esqueleto:{ nome:'Esqueleto',icone:'💀', hp:70, dano:14, xp:70, ouro:35, vel:0.9, cor:'#dddddd', corBorda:'#888888', tamanho:26, alcance:220, tipoAtaque:'projetil', projetilCor:'#eeeecc', projetilVel:3.5, projetilDano:12, cooldownAtaque:100 },
  aranha:   { nome:'Aranha',   icone:'🕷️', hp:55, dano:8,  xp:60, ouro:30, vel:1.8, cor:'#222222', corBorda:'#660066', tamanho:24, alcance:28, tipoAtaque:'veneno', venenoDano:3, venenoDuracao:180 },
  orc:      { nome:'Orc',      icone:'👹', hp:120,dano:18, xp:100,ouro:60, vel:1.1, cor:'#aa5555', corBorda:'#661111', tamanho:30, alcance:34, tipoAtaque:'corpo' },
  mago:     { nome:'Mago Negro',icone:'🧙', hp:80, dano:10, xp:130,ouro:80, vel:0.8, cor:'#8844cc', corBorda:'#441166', tamanho:26, alcance:280, tipoAtaque:'magia', projetilCor:'#ff44ff', projetilVel:2.5, projetilDano:15, cooldownAtaque:110 },
  bombardeiro:{nome:'Bombardeiro',icone:'💣',hp:90,dano:12,xp:140,ouro:90, vel:0.7, cor:'#665500', corBorda:'#332200', tamanho:28, alcance:240, tipoAtaque:'bomba', projetilCor:'#333333', projetilVel:2.8, projetilDano:20, explosaoRaio:70, cooldownAtaque:140 },
  necromante:{ nome:'Necromante',icone:'☠️',hp:130,dano:12, xp:200,ouro:130,vel:0.6, cor:'#663366', corBorda:'#220022', tamanho:28, alcance:220, tipoAtaque:'invocar', cooldownAtaque:300 },
  demonio:  { nome:'Demônio',  icone:'😈', hp:180,dano:22, xp:250,ouro:150,vel:1.3, cor:'#ff44aa', corBorda:'#aa0066', tamanho:32, alcance:36, tipoAtaque:'queimadura', queimaduraDano:5, queimaduraDuracao:200 }
};

// ============================================================
//  BOSSES — Um diferente a cada 10 ondas
// ============================================================
const BOSSES = {
  10: {
    nome: '🐛 Rei Slime',
    hp: 800, dano: 25, xp: 800, ouro: 500,
    vel: 0.7, cor: '#44aa44', corBorda: '#115511',
    tamanho: 90,
    tipoAtaque: 'boss_slime',
    fase1: 'lento_mas_tanque',
    descricao: 'Tanque gigante que divide ao morrer'
  },
  20: {
    nome: '💀 Lorde Esqueleto',
    hp: 1200, dano: 30, xp: 1500, ouro: 900,
    vel: 1.0, cor: '#cccccc', corBorda: '#666666',
    tamanho: 95,
    tipoAtaque: 'boss_esqueleto',
    descricao: 'Invoca hordas de esqueletos + atira ossos'
  },
  30: {
    nome: '🕷️ Viúva Negra',
    hp: 1600, dano: 35, xp: 2200, ouro: 1300,
    vel: 1.6, cor: '#111111', corBorda: '#ff00ff',
    tamanho: 100,
    tipoAtaque: 'boss_aranha',
    descricao: 'Muito rápida e envenena em área'
  },
  40: {
    nome: '🧙 Arquimago Sombrio',
    hp: 2000, dano: 40, xp: 3000, ouro: 1800,
    vel: 0.9, cor: '#6633cc', corBorda: '#220066',
    tamanho: 100,
    tipoAtaque: 'boss_mago',
    descricao: 'Teleguiados múltiplos + teleporte'
  },
  50: {
    nome: '🐲 Dragão Ancião',
    hp: 2800, dano: 50, xp: 4500, ouro: 2500,
    vel: 1.1, cor: '#cc2222', corBorda: '#660000',
    tamanho: 120,
    tipoAtaque: 'boss_dragao',
    descricao: 'Sopro de fogo em cone + voo'
  },
  60: {
    nome: '👑 Lorde das Sombras',
    hp: 4000, dano: 60, xp: 7000, ouro: 4000,
    vel: 1.3, cor: '#220044', corBorda: '#ff00ff',
    tamanho: 130,
    tipoAtaque: 'boss_final',
    descricao: 'TODOS os ataques + invoca lacaios'
  }
};

// Função que retorna boss baseado no número da onda
function getBossPorOnda(onda) {
  // Encontra o maior boss que já passou (onda / 10 * 10)
  const nivelBoss = Math.floor(onda / 10) * 10;
  if (BOSSES[nivelBoss]) return BOSSES[nivelBoss];
  // Fallback: escala boss do nível 60
  const base = BOSSES[60];
  const escala = onda / 60;
  return {
    ...base,
    nome: '👑 Lorde das Sombras Nv ' + onda,
    hp: Math.floor(base.hp * escala),
    dano: Math.floor(base.dano * escala * 0.7),
    xp: Math.floor(base.xp * escala),
    ouro: Math.floor(base.ouro * escala)
  };
}

// ===== UPGRADES =====
const UPGRADES = [
  { id:'regen', icone:'💚', nome:'Regeneração', desc:'+3 HP/s passivo por nível', custoBase: 50, custoMult: 1.6, chave:'upgRegen' },
  { id:'vel', icone:'👟', nome:'Rapidez', desc:'+15% velocidade por nível', custoBase: 40, custoMult: 1.5, chave:'upgVel' },
  { id:'atk', icone:'⚡', nome:'Ataque Rápido', desc:'-15% cooldown por nível', custoBase: 60, custoMult: 1.7, chave:'upgAtk' },
  { id:'vida', icone:'❤️', nome:'Vida Máxima', desc:'+15 HP máx por nível', custoBase: 45, custoMult: 1.4, chave:'upgVida' },
  { id:'dano', icone:'🗡️', nome:'Força Bruta', desc:'+2 dano por nível', custoBase: 70, custoMult: 1.8, chave:'upgDano' },
  { id:'crit', icone:'🎯', nome:'Crítico', desc:'+8% chance de dano x3', custoBase: 80, custoMult: 1.9, chave:'upgCrit' },
  { id:'escudo', icone:'🛡️', nome:'Escudo', desc:'-2 dano recebido por nível', custoBase: 55, custoMult: 1.6, chave:'upgEscudo' },
  { id:'coleta', icone:'💎', nome:'Coleta', desc:'+25% ouro/XP por nível', custoBase: 50, custoMult: 1.5, chave:'upgColeta' }
];

// ===== ESTADO =====
let monstros = [];
let projeteis = [];
let baus = [];
let particulas = [];
let onda = 0;
let tempoProximaOnda = 0;
let jogoAcabou = false;
let mensagemTimer = 0;
let frameCount = 0;
let lojaAberta = false;
let bossAtual = null;         // referência ao boss vivo
let bossAtivo = false;
let telaBossWarning = 0;      // frames do aviso

// ===== CONTROLES =====
const teclas = {};
window.addEventListener('keydown', e => {
  const k = e.key.toLowerCase();
  teclas[k] = true;
  if (lojaAberta) return;
  if (k === ' ' || e.key === 'Spacebar') { e.preventDefault(); atacar(); }
  if (k === 'e') usarHabilidade();
  if (k === 'q') usarPocao();
  if (['arrowup','arrowdown','arrowleft','arrowright'].includes(k)) e.preventDefault();
});
window.addEventListener('keyup', e => teclas[e.key.toLowerCase()] = false);

// ===== COLISÃO =====
function colideMapa(x, y, w, h) {
  const pontos = [
    [x + 2, y + 2], [x + w - 2, y + 2],
    [x + 2, y + h - 2], [x + w - 2, y + h - 2]
  ];
  for (const [px, py] of pontos) {
    const tx = Math.floor(px / TILE);
    const ty = Math.floor(py / TILE);
    if (ty < 0 || ty >= mapaTiles.length || tx < 0 || tx >= mapaTiles[0].length) return true;
    if (mapaTiles[ty][tx] === 1) return true;
  }
  return false;
}

// ===== MOVIMENTO =====
function moverJogador() {
  if (jogoAcabou || lojaAberta || telaBossWarning > 0) return;
  let dx = 0, dy = 0;

  if (teclas['w'] || teclas['arrowup']) dy = -1;
  else if (teclas['s'] || teclas['arrowdown']) dy = 1;
  if (teclas['a'] || teclas['arrowleft']) dx = -1;
  else if (teclas['d'] || teclas['arrowright']) dx = 1;

  if (dx === 0 && dy === 0) { jogador.frameAnim = 0; return; }

  if (dx < 0) jogador.dir = 'left';
  else if (dx > 0) jogador.dir = 'right';
  else if (dy < 0) jogador.dir = 'up';
  else if (dy > 0) jogador.dir = 'down';

  jogador.frameAnim += 0.2;
  let multVel = 1;
  if (jogador.lento > 0) multVel = 0.5;

  const vel = jogador.vel * (1 + jogador.upgVel * 0.15) * multVel;
  const nx = jogador.x + dx * vel;
  const ny = jogador.y + dy * vel;

  if (!colideMapa(nx, jogador.y, jogador.w, jogador.h)) jogador.x = nx;
  if (!colideMapa(jogador.x, ny, jogador.w, jogador.h)) jogador.y = ny;

  jogador.x = Math.max(0, Math.min(canvas.width - jogador.w, jogador.x));
  jogador.y = Math.max(0, Math.min(canvas.height - jogador.h, jogador.y));

  coletarBaus();
}

// ===== COLETA BAÚS =====
function coletarBaus() {
  const cx = jogador.x + jogador.w / 2;
  const cy = jogador.y + jogador.h / 2;
  for (let i = baus.length - 1; i >= 0; i--) {
    const b = baus[i];
    if (Math.hypot(cx - (b.x + 15), cy - (b.y + 15)) < 29) {
      abrirBau(b);
      baus.splice(i, 1);
    }
  }
}

function spawnarBaus(qtd) {
  for (let i = 0; i < qtd; i++) {
    let x, y, tentativas = 0;
    do {
      x = 80 + Math.random() * (canvas.width - 160);
      y = 80 + Math.random() * (canvas.height - 160);
      tentativas++;
    } while ((Math.hypot(x - jogador.x, y - jogador.y) < 120 || colideMapa(x, y, 30, 30)) && tentativas < 40);

    const r = Math.random();
    let tipo;
    if (r < 0.35) tipo = 'ouro';
    else if (r < 0.65) tipo = 'pocao';
    else if (r < 0.85) tipo = 'xp';
    else tipo = 'dano';

    baus.push({ x, y, tamanho: 30, tipo, brilho: 0 });
  }
}

function abrirBau(b) {
  let texto = '';
  const bonus = 1 + jogador.upgColeta * 0.25;
  if (b.tipo === 'ouro') {
    const qtd = Math.floor((30 + Math.random() * 50 + onda * 5) * bonus);
    jogador.ouro += qtd;
    texto = `+${qtd} 🪙`;
    criarParticula(b.x + 15, b.y, texto, '#f0c040', 100);
  } else if (b.tipo === 'pocao') {
    jogador.pocoes += 1;
    texto = '+1 🧪';
    criarParticula(b.x + 15, b.y, texto, '#55ff88', 100);
  } else if (b.tipo === 'xp') {
    const qtd = Math.floor((30 + Math.random() * 40 + onda * 4) * bonus);
    jogador.xp += qtd;
    texto = `+${qtd} ⭐`;
    criarParticula(b.x + 15, b.y, texto, '#88ccff', 100);
    checarLevelUp();
  } else {
    jogador.armaBonus += 2;
    texto = '+2 ⚔️';
    criarParticula(b.x + 15, b.y, texto, '#ff8888', 120);
  }
  criarOnda(b.x + 15, b.y + 15, 60);
  mostrarMsg(`🎁 ${texto}`, 1200);
  atualizarHud();
}

// ===== ATAQUE =====
function atacar() {
  if (jogoAcabou || lojaAberta || telaBossWarning > 0 || jogador.cooldownAtaque > 0) return;
  jogador.cooldownAtaque = Math.max(6, Math.floor(25 * Math.pow(0.85, jogador.upgAtk)));
  jogador.atacando = 12;

  const cx = jogador.x + jogador.w / 2;
  const cy = jogador.y + jogador.h / 2;
  const raio = 55;
  const danoBase = jogador.forca + jogador.armaBonus + jogador.upgDano * 2;

  let acertou = false;
  for (const m of monstros) {
    const mx = m.x + m.tamanho / 2;
    const my = m.y + m.tamanho / 2;
    if (Math.hypot(cx - mx, cy - my) < raio + m.tamanho / 2) {
      const isCrit = Math.random() < jogador.upgCrit * 0.08;
      const dano = isCrit ? danoBase * 3 : danoBase;
      m.hp -= dano;
      m.hitFlash = 6;
      acertou = true;
      criarParticula(mx, my - 10, (isCrit ? '💥' : '') + '-' + dano, isCrit ? '#ff00ff' : '#ff5555', isCrit ? 90 : 60);
      const ang = Math.atan2(my - cy, mx - cx);
      if (m.tamanho < 60) { // boss não é empurrado
        m.x += Math.cos(ang) * 15;
        m.y += Math.sin(ang) * 15;
      }
      if (m.hp <= 0) morrerMonstro(m);
    }
  }
  if (acertou) mostrarMsg('💥 Atingiu!', 400);
}

function usarHabilidade() {
  if (jogoAcabou || lojaAberta || telaBossWarning > 0) return;
  if (jogador.mp < 10) { mostrarMsg('⚡ MP insuficiente!', 700); return; }
  jogador.mp -= 10;
  jogador.cooldownAtaque = Math.max(10, Math.floor(40 * Math.pow(0.85, jogador.upgAtk)));
  jogador.atacando = 20;

  const cx = jogador.x + jogador.w / 2;
  const cy = jogador.y + jogador.h / 2;
  const danoBase = (jogador.forca + jogador.armaBonus + jogador.upgDano * 2) * 2;

  let acertou = 0;
  for (const m of monstros) {
    const mx = m.x + m.tamanho / 2;
    const my = m.y + m.tamanho / 2;
    if (Math.hypot(cx - mx, cy - my) < 130 + m.tamanho / 2) {
      const isCrit = Math.random() < jogador.upgCrit * 0.08;
      const dano = isCrit ? danoBase * 3 : danoBase;
      m.hp -= dano;
      m.hitFlash = 8;
      acertou++;
      criarParticula(mx, my - 10, (isCrit ? '💥' : '') + '-' + dano, isCrit ? '#ff00ff' : '#ffaa00', 80);
      if (m.hp <= 0) morrerMonstro(m);
    }
  }
  criarOnda(cx, cy, 130);
  mostrarMsg(acertou > 0 ? `💥 GOLPE PODEROSO! ${acertou} atingidos` : '💨 Habilidade!', 600);
  atualizarHud();
}

function usarPocao() {
  if (jogoAcabou || lojaAberta) return;
  if (jogador.pocoes <= 0) { mostrarMsg('🧪 Sem poções!', 700); return; }
  if (jogador.hp >= jogador.hpMax && jogador.envenenado <= 0 && jogador.queimando <= 0) {
    mostrarMsg('❤️ HP cheio!', 700); return;
  }
  jogador.pocoes--;
  jogador.hp = Math.min(jogador.hpMax, jogador.hp + 40);
  jogador.envenenado = 0; jogador.queimando = 0;
  criarParticula(jogador.x + 12, jogador.y, '+40 HP', '#55ff88', 90);
  mostrarMsg('🧪 +40 HP!', 900);
  atualizarHud();
}

// ============================================================
//  SPAWN DE ONDA + BOSS
// ============================================================
function spawnarOnda() {
  onda++;
  const dificuldade = Math.min(10, 1 + Math.floor(onda / 2));

  // ⚠️ SE FOR MÚLTIPLO DE 10 → BOSS!
  if (onda % 10 === 0) {
    iniciarAvisoBoss();
    return;
  }

  // Onda normal
  let tipos = ['slime', 'goblin'];
  if (onda >= 2) tipos.push('esqueleto');
  if (onda >= 3) tipos.push('aranha');
  if (onda >= 4) tipos.push('orc');
  if (onda >= 5) tipos.push('mago');
  if (onda >= 6) tipos.push('bombardeiro');
  if (onda >= 8) tipos.push('necromante');
  if (onda >= 10) tipos.push('demonio');

  const qtd = Math.min(20, 4 + onda * 2);

  for (let i = 0; i < qtd; i++) {
    const tipo = tipos[Math.floor(Math.random() * tipos.length)];
    const base = TIPOS_MONSTROS[tipo];

    let x, y, tentativas = 0;
    do {
      x = 60 + Math.random() * (canvas.width - 120);
      y = 60 + Math.random() * (canvas.height - 120);
      tentativas++;
    } while ((Math.hypot(x - jogador.x, y - jogador.y) < 200 || colideMapa(x, y, base.tamanho, base.tamanho)) && tentativas < 40);

    criarMonstro(base, tipo, x, y, dificuldade);
  }

  mostrarMsg(`🌊 ONDA ${onda}! ${qtd} inimigos!`, 1800);
  const qtdBaus = 1 + Math.floor(onda / 2);
  if (onda > 1) spawnarBaus(Math.min(4, qtdBaus));
  atualizarHud();
}

function criarMonstro(base, tipo, x, y, dificuldade, escala = 1) {
  const hpBonus = (dificuldade - 1) * 8;
  const m = {
    tipo, nome: base.nome, icone: base.icone,
    x, y, tamanho: base.tamanho * escala,
    hp: Math.floor((base.hp + hpBonus) * escala),
    hpMax: Math.floor((base.hp + hpBonus) * escala),
    dano: base.dano, xp: Math.floor(base.xp * escala), ouro: Math.floor(base.ouro * escala),
    vel: base.vel, cor: base.cor, corBorda: base.corBorda,
    alcance: base.alcance, tipoAtaque: base.tipoAtaque,
    projetilCor: base.projetilCor, projetilVel: base.projetilVel, projetilDano: base.projetilDano,
    cooldownBase: base.cooldownAtaque || 70,
    explosaoRaio: base.explosaoRaio,
    venenoDano: base.venenoDano, venenoDuracao: base.venenoDuracao,
    queimaduraDano: base.queimaduraDano, queimaduraDuracao: base.queimaduraDuracao,
    cooldownAtaque: 30 + Math.random() * 60,
    hitFlash: 0,
    isBoss: false
  };
  monstros.push(m);
  return m;
}

// ============================================================
//  SISTEMA DE BOSS
// ============================================================
function iniciarAvisoBoss() {
  telaBossWarning = 180; // 3 segundos
  const boss = getBossPorOnda(onda);
  mostrarMsg(`⚠️ BOSS SE APROXIMA: ${boss.nome}! ⚠️`, 3000);
}

function spawnarBoss() {
  const dados = getBossPorOnda(onda);
  // Boss aparece no canto oposto ao jogador
  const x = canvas.width - 150;
  const y = canvas.height - 150;

  const boss = {
    tipo: 'boss',
    nome: dados.nome,
    icone: '👑',
    x, y,
    tamanho: dados.tamanho,
    hp: dados.hp, hpMax: dados.hp,
    dano: dados.dano,
    xp: dados.xp, ouro: dados.ouro,
    vel: dados.vel,
    cor: dados.cor, corBorda: dados.corBorda,
    alcance: 250,
    tipoAtaque: dados.tipoAtaque,
    cooldownAtaque: 60,
    hitFlash: 0,
    isBoss: true,
    faseAtaque: 0,
    tempoFase: 0,
    // Fases especiais
    invocado: false,
    projeteisPorRajada: 0
  };

  monstros.push(boss);
  bossAtual = boss;
  bossAtivo = true;
  mostrarBossHud(true, boss.nome);
  mostrarMsg(`👑 ${boss.nome} apareceu!`, 2000);
}

function mostrarBossHud(ativo, nome) {
  const el = document.getElementById('boss-hud');
  el.classList.toggle('ativo', ativo);
  if (ativo && nome) document.getElementById('boss-nome').textContent = nome;
}

function atualizarBossHud() {
  if (!bossAtual) return;
  const pct = Math.max(0, (bossAtual.hp / bossAtual.hpMax) * 100);
  document.getElementById('boss-barra-inner').style.width = pct + '%';
}

// ============================================================
//  ATAQUES ESPECIAIS DO BOSS
// ============================================================
function ataqueBoss(boss) {
  const jx = jogador.x + jogador.w / 2;
  const jy = jogador.y + jogador.h / 2;
  const bx = boss.x + boss.tamanho / 2;
  const by = boss.y + boss.tamanho / 2;
  const ang = Math.atan2(jy - by, jx - bx);
  const dist = Math.hypot(jx - bx, jy - by);

  boss.faseAtaque++;
  boss.tempoFase++;

  switch (boss.tipoAtaque) {
    case 'boss_slime':
      // Ataque lento + invoca slimes pequenos a cada 5 ataques
      if (dist < 120) {
        aplicarDanoJogador(boss.dano, boss.nome);
      }
      if (boss.faseAtaque % 5 === 0 && monstros.length < 25) {
        for (let i = 0; i < 3; i++) {
          const a = Math.random() * Math.PI * 2;
          const x = bx + Math.cos(a) * 100;
          const y = by + Math.sin(a) * 100;
          criarMonstro(TIPOS_MONSTROS.slime, 'slime', x, y, 3, 1.2);
        }
        criarParticula(bx, by, '💚 DIVIDE!', '#55ff55', 100);
      }
      boss.cooldownAtaque = 70;
      break;

    case 'boss_esqueleto':
      // Rajada de 5 ossos em cone
      for (let i = -2; i <= 2; i++) {
        const a = ang + i * 0.25;
        projeteis.push({
          x: bx, y: by,
          vx: Math.cos(a) * 4,
          vy: Math.sin(a) * 4,
          dano: 20, cor: '#eeeecc', tamanho: 10, vida: 200, tipo: 'projetil'
        });
      }
      // Invoca esqueletos a cada 4 ataques
      if (boss.faseAtaque % 4 === 0 && monstros.length < 25) {
        for (let i = 0; i < 4; i++) {
          const a = Math.random() * Math.PI * 2;
          const x = bx + Math.cos(a) * 120;
          const y = by + Math.sin(a) * 120;
          criarMonstro(TIPOS_MONSTROS.esqueleto, 'esqueleto', x, y, 5);
        }
        criarParticula(bx, by, '💀 INVOCA!', '#cccccc', 100);
      }
      boss.cooldownAtaque = 90;
      break;

    case 'boss_aranha':
      // Ataque rápido corpo a corpo + teia que envenena em área
      if (dist < 130) {
        aplicarDanoJogador(boss.dano, boss.nome);
        jogador.envenenado = 240;
        jogador.envenenadoDano = 5;
      }
      // A cada 5 ataques, lança teia que deixa o jogador lento
      if (boss.faseAtaque % 5 === 0) {
        for (let i = 0; i < 8; i++) {
          const a = (Math.PI * 2 * i) / 8 + ang;
          projeteis.push({
            x: bx, y: by,
            vx: Math.cos(a) * 3,
            vy: Math.sin(a) * 3,
            dano: 15, cor: '#aa44ff', tamanho: 12, vida: 150, tipo: 'projetil',
            lento: true
          });
        }
        criarParticula(bx, by, '🕸️ TEIA!', '#aa44ff', 100);
      }
      boss.cooldownAtaque = 45;
      break;

    case 'boss_mago':
      // Teleporte + rajada de magias teleguiadas
      if (boss.faseAtaque % 3 === 0) {
        // Teleporta
        boss.x = 100 + Math.random() * (canvas.width - 200);
        boss.y = 100 + Math.random() * (canvas.height - 200);
        criarOnda(bx, by, 80);
        criarParticula(boss.x + boss.tamanho / 2, boss.y, '✨ TELEPORTE!', '#aa44ff', 80);
      } else {
        // Rajada de 3 magias teleguiadas
        for (let i = 0; i < 3; i++) {
          setTimeout(() => {
            if (jogoAcabou || !bossAtual) return;
            const bx2 = boss.x + boss.tamanho / 2;
            const by2 = boss.y + boss.tamanho / 2;
            const jx2 = jogador.x + jogador.w / 2;
            const jy2 = jogador.y + jogador.h / 2;
            const a = Math.atan2(jy2 - by2, jx2 - bx2);
            projeteis.push({
              x: bx2, y: by2,
              vx: Math.cos(a) * 3.5,
              vy: Math.sin(a) * 3.5,
              dano: 25, cor: '#ff44ff', tamanho: 12, vida: 300, tipo: 'magia', teleguiado: true
            });
          }, i * 150);
        }
      }
      boss.cooldownAtaque = 110;
      break;

    case 'boss_dragao':
      // Sopro de fogo em cone
      if (boss.faseAtaque % 4 === 0) {
        for (let i = -3; i <= 3; i++) {
          const a = ang + i * 0.15;
          projeteis.push({
            x: bx, y: by,
            vx: Math.cos(a) * 5,
            vy: Math.sin(a) * 5,
            dano: 30, cor: '#ff6600', tamanho: 14, vida: 180, tipo: 'projetil'
          });
        }
        criarParticula(bx, by, '🔥 SOPRO DE FOGO!', '#ff4400', 100);
        boss.cooldownAtaque = 130;
      } else if (dist < 150) {
        aplicarDanoJogador(boss.dano * 0.6, boss.nome);
        boss.cooldownAtaque = 60;
      } else {
        boss.cooldownAtaque = 30;
      }
      break;

    case 'boss_final':
      // MEGA BOSS: usa todos os ataques
      const tipoSorteado = Math.floor(Math.random() * 4);
      if (tipoSorteado === 0) {
        // Rajada de ossos
        for (let i = -2; i <= 2; i++) {
          const a = ang + i * 0.25;
          projeteis.push({ x: bx, y: by, vx: Math.cos(a) * 4, vy: Math.sin(a) * 4, dano: 25, cor: '#eeeecc', tamanho: 10, vida: 200, tipo: 'projetil' });
        }
      } else if (tipoSorteado === 1) {
        // Magias teleguiadas
        for (let i = 0; i < 4; i++) {
          const a = ang + (Math.random() - 0.5) * 0.8;
          projeteis.push({ x: bx, y: by, vx: Math.cos(a) * 3.5, vy: Math.sin(a) * 3.5, dano: 30, cor: '#ff00ff', tamanho: 12, vida: 300, tipo: 'magia', teleguiado: true });
        }
      } else if (tipoSorteado === 2) {
        // Invoca lacaios
        if (monstros.length < 30) {
          for (let i = 0; i < 5; i++) {
            const a = Math.random() * Math.PI * 2;
            const x = bx + Math.cos(a) * 130;
            const y = by + Math.sin(a) * 130;
            const tipos = ['esqueleto', 'orc', 'demonio'];
            const t = tipos[Math.floor(Math.random() * tipos.length)];
            criarMonstro(TIPOS_MONSTROS[t], t, x, y, 8);
          }
        }
      } else {
        // Teleporte perto do jogador + ataque forte
        boss.x = jogador.x + (Math.random() - 0.5) * 200;
        boss.y = jogador.y + (Math.random() - 0.5) * 200;
        if (dist < 150) aplicarDanoJogador(boss.dano, boss.nome);
      }
      boss.cooldownAtaque = 80;
      break;
  }
}

// ============================================================
//  COMPORTAMENTO DOS MONSTROS
// ============================================================
function moverMonstros() {
  for (const m of monstros) {
    if (m.cooldownAtaque > 0) m.cooldownAtaque--;
    if (m.hitFlash > 0) m.hitFlash--;

    const jx = jogador.x + jogador.w / 2;
    const jy = jogador.y + jogador.h / 2;
    const mx = m.x + m.tamanho / 2;
    const my = m.y + m.tamanho / 2;
    const dist = Math.hypot(jx - mx, jy - my);

    // Boss
    if (m.isBoss) {
      // Boss sempre persegue, mas com distância ideal
      const distIdeal = 60;
      if (dist > distIdeal) {
        const ang = Math.atan2(jy - my, jx - mx);
        const nx = m.x + Math.cos(ang) * m.vel;
        const ny = m.y + Math.sin(ang) * m.vel;
        if (!colideMapa(nx, m.y, m.tamanho, m.tamanho)) m.x = nx;
        if (!colideMapa(m.x, ny, m.tamanho, m.tamanho)) m.y = ny;
        m.x = Math.max(0, Math.min(canvas.width - m.tamanho, m.x));
        m.y = Math.max(0, Math.min(canvas.height - m.tamanho, m.y));
      }

      if (m.cooldownAtaque <= 0) {
        ataqueBoss(m);
      }
      continue;
    }

    // Monstros normais
    const isRanged = ['projetil', 'magia', 'bomba'].includes(m.tipoAtaque);
    const isInvocador = m.tipoAtaque === 'invocar';
    let distIdeal = 30;
    if (isRanged) distIdeal = m.alcance * 0.7;
    if (isInvocador) distIdeal = 150;

    if (dist > distIdeal) {
      const ang = Math.atan2(jy - my, jx - mx);
      const nx = m.x + Math.cos(ang) * m.vel;
      const ny = m.y + Math.sin(ang) * m.vel;
      if (!colideMapa(nx, m.y, m.tamanho, m.tamanho)) m.x = nx;
      if (!colideMapa(m.x, ny, m.tamanho, m.tamanho)) m.y = ny;
      m.x = Math.max(0, Math.min(canvas.width - m.tamanho, m.x));
      m.y = Math.max(0, Math.min(canvas.height - m.tamanho, m.y));
    } else if (isRanged && dist < distIdeal - 40) {
      const ang = Math.atan2(jy - my, jx - mx);
      const nx = m.x - Math.cos(ang) * m.vel;
      const ny = m.y - Math.sin(ang) * m.vel;
      if (!colideMapa(nx, m.y, m.tamanho, m.tamanho)) m.x = nx;
      if (!colideMapa(m.x, ny, m.tamanho, m.tamanho)) m.y = ny;
    }

    if (m.cooldownAtaque <= 0) {
      if (['corpo', 'veneno', 'queimadura'].includes(m.tipoAtaque)) {
        if (dist < m.alcance) {
          m.cooldownAtaque = m.cooldownBase;
          aplicarDanoJogador(m.dano, m.nome);
          if (m.tipoAtaque === 'veneno') {
            jogador.envenenado = m.venenoDuracao;
            jogador.envenenadoDano = m.venenoDano;
          } else if (m.tipoAtaque === 'queimadura') {
            jogador.queimando = m.queimaduraDuracao;
          }
        }
      } else if (['projetil', 'magia', 'bomba'].includes(m.tipoAtaque)) {
        if (dist < m.alcance) {
          m.cooldownAtaque = m.cooldownBase;
          const ang = Math.atan2(jy - my, jx - mx);
          projeteis.push({
            x: mx, y: my,
            vx: Math.cos(ang) * m.projetilVel,
            vy: Math.sin(ang) * m.projetilVel,
            dano: m.projetilDano || m.dano,
            cor: m.projetilCor || '#ffaa00',
            tamanho: 8, vida: 300, tipo: m.tipoAtaque,
            explosaoRaio: m.explosaoRaio,
            teleguiado: m.tipoAtaque === 'magia'
          });
        }
      } else if (m.tipoAtaque === 'invocar') {
        if (dist < m.alcance && monstros.length < 30) {
          m.cooldownAtaque = m.cooldownBase;
          for (let i = 0; i < 2; i++) {
            const a = Math.random() * Math.PI * 2;
            const x = m.x + Math.cos(a) * 50;
            const y = m.y + Math.sin(a) * 50;
            criarMonstro(TIPOS_MONSTROS.esqueleto, 'esqueleto', x, y, 3);
          }
          criarParticula(mx, my, '☠️', '#aa44ff', 100);
        }
      }
    }
  }
}

function aplicarDanoJogador(dano, fonte) {
  if (jogador.cooldownDano > 0) return;
  const danoFinal = Math.max(1, dano - jogador.upgEscudo * 2);
  jogador.hp -= danoFinal;
  jogador.cooldownDano = 30;
  const jx = jogador.x + jogador.w / 2;
  const jy = jogador.y + jogador.h / 2;
  criarParticula(jx, jy - 15, '-' + danoFinal, '#ff5555', 70);
  mostrarMsg(`💢 ${fonte} atacou! -${danoFinal} HP`, 500);
  atualizarHud();
  if (jogador.hp <= 0) {
    jogador.hp = 0;
    atualizarHud();
    fimDeJogo();
  }
}

// ============================================================
//  PROJÉTEIS
// ============================================================
function atualizarProjeteis() {
  const jx = jogador.x + jogador.w / 2;
  const jy = jogador.y + jogador.h / 2;

  for (let i = projeteis.length - 1; i >= 0; i--) {
    const p = projeteis[i];
    p.vida--;

    if (p.teleguiado && p.vida > 200) {
      const ang = Math.atan2(jy - p.y, jx - p.x);
      const velAtual = Math.hypot(p.vx, p.vy);
      p.vx = Math.cos(ang) * velAtual;
      p.vy = Math.sin(ang) * velAtual;
    }

    p.x += p.vx;
    p.y += p.vy;

    if (p.x < 0 || p.y < 0 || p.x > canvas.width || p.y > canvas.height) {
      if (p.tipo === 'bomba') explodirBomba(p);
      projeteis.splice(i, 1);
      continue;
    }

    if (p.vida <= 0) {
      if (p.tipo === 'bomba') explodirBomba(p);
      projeteis.splice(i, 1);
      continue;
    }

    const dist = Math.hypot(jx - p.x, jy - p.y);
    if (dist < 18) {
      if (p.tipo === 'bomba') explodirBomba(p);
      else {
        aplicarDanoJogador(p.dano, 'Projétil');
        if (p.lento) jogador.lento = 180;
      }
      projeteis.splice(i, 1);
    }
  }
}

function explodirBomba(p) {
  criarOnda(p.x, p.y, p.explosaoRaio || 70);
  const jx = jogador.x + jogador.w / 2;
  const jy = jogador.y + jogador.h / 2;
  if (Math.hypot(jx - p.x, jy - p.y) < (p.explosaoRaio || 70)) {
    aplicarDanoJogador(p.dano, 'Explosão');
  }
}

// ============================================================
//  MORTE E LEVEL UP
// ============================================================
function morrerMonstro(m) {
  const bonus = 1 + jogador.upgColeta * 0.25;
  const xpGanho = Math.floor(m.xp * bonus);
  const ouroGanho = Math.floor(m.ouro * bonus);
  jogador.xp += xpGanho;
  jogador.ouro += ouroGanho;
  if (Math.random() < 0.15) jogador.pocoes++;

  criarParticula(m.x + m.tamanho / 2, m.y + m.tamanho / 2, '+' + xpGanho + ' XP', '#88ccff', 80);
  criarParticula(m.x + m.tamanho / 2, m.y + m.tamanho / 2 - 20, '+' + ouroGanho + ' 🪙', '#f0c040', 80);

  // Morte de BOSS
  if (m.isBoss) {
    bossAtual = null;
    bossAtivo = false;
    mostrarBossHud(false);
    mostrarMsg(`👑 BOSS DERROTADO! +${xpGanho} XP +${ouroGanho} 🪙`, 3500);

    // Drop especial: baú lendário
    baus.push({ x: m.x + m.tamanho / 2 - 15, y: m.y + m.tamanho / 2 - 15, tamanho: 30, tipo: 'dano', brilho: 0 });
    baus.push({ x: m.x + m.tamanho / 2 + 40, y: m.y + m.tamanho / 2 - 15, tamanho: 30, tipo: 'pocao', brilho: 0 });
    baus.push({ x: m.x + m.tamanho / 2 - 70, y: m.y + m.tamanho / 2 - 15, tamanho: 30, tipo: 'ouro', brilho: 0 });

    // Efeito épico
    criarOnda(m.x + m.tamanho / 2, m.y + m.tamanho / 2, 300);
    criarOnda(m.x + m.tamanho / 2, m.y + m.tamanho / 2, 200);
    criarOnda(m.x + m.tamanho / 2, m.y + m.tamanho / 2, 100);

    // Remove os lacaios
    monstros = monstros.filter(mm => mm === m || mm.isBoss === false ? Math.random() > 0.3 : false);
  }

  checarLevelUp();
  monstros = monstros.filter(mm => mm !== m);
  atualizarHud();
}

function checarLevelUp() {
  const novoNivel = Math.floor(jogador.xp / 150) + 1;
  if (novoNivel > jogador.nivel) {
    const ganhos = novoNivel - jogador.nivel;
    jogador.nivel = novoNivel;
    jogador.hpMax += 8 * ganhos;
    jogador.hp = Math.min(jogador.hpMax, jogador.hp + 8 * ganhos);
    jogador.mpMax += 3 * ganhos;
    jogador.mp = Math.min(jogador.mpMax, jogador.mp + 3 * ganhos);
    jogador.forca += 1 * ganhos;
    mostrarMsg(`✨ LEVEL UP! Nível ${jogador.nivel}`, 1500);
    atualizarHud();
    if (jogador.nivel % 10 === 0 && jogador.nivel !== jogador.ultimoNivelLoja) {
      jogador.ultimoNivelLoja = jogador.nivel;
      abrirLoja();
    }
  }
}

// ============================================================
//  LOJA
// ============================================================
function calcularCusto(upg) {
  return Math.floor(upg.custoBase * Math.pow(upg.custoMult, jogador[upg.chave]));
}
function abrirLoja() {
  lojaAberta = true;
  document.getElementById('upgrade-menu').classList.add('ativo');
  document.getElementById('upgrade-titulo').textContent = `🏪 LOJA — NÍVEL ${jogador.nivel}`;
  renderizarLoja();
}
function renderizarLoja() {
  document.getElementById('ouro-loja').textContent = `🪙 Ouro disponível: ${jogador.ouro}`;
  const container = document.getElementById('opcoes');
  container.innerHTML = '';
  let podeComprar = false;
  UPGRADES.forEach(upg => {
    const nivel = jogador[upg.chave];
    const custo = calcularCusto(upg);
    const podePagar = jogador.ouro >= custo;
    if (podePagar) podeComprar = true;
    const card = document.createElement('div');
    card.className = 'upgrade-card' + (podePagar ? '' : ' bloqueado');
    card.innerHTML = `
      <span class="icone">${upg.icone}</span>
      <div class="nome">${upg.nome}</div>
      <div class="desc">${upg.desc}</div>
      <div class="nivel">Nível ${nivel}</div>
      <div class="custo ${podePagar ? '' : 'caro'}">🪙 ${custo}</div>`;
    if (podePagar) card.onclick = () => comprarUpgrade(upg);
    container.appendChild(card);
  });
  document.getElementById('btn-fechar').classList.toggle('pulse', !podeComprar);
}
function comprarUpgrade(upg) {
  const custo = calcularCusto(upg);
  if (jogador.ouro < custo) return;
  jogador.ouro -= custo;
  jogador[upg.chave]++;
  if (upg.id === 'vida') { jogador.hpMax += 15; jogador.hp += 15; }
  mostrarMsg(`✅ ${upg.nome} → Nv ${jogador[upg.chave]}!`, 1200);
  atualizarHud();
  renderizarLoja();
}
function fecharLoja() {
  lojaAberta = false;
  document.getElementById('upgrade-menu').classList.remove('ativo');
}

// ============================================================
//  PARTÍCULAS / STATUS
// ============================================================
function criarParticula(x, y, texto, cor, duracao) {
  particulas.push({ x, y, texto, cor, vida: duracao, vidaMax: duracao, vy: -0.6 });
}
function criarOnda(x, y, raio) {
  particulas.push({ x, y, onda: true, raioMax: raio, raio: 10, vida: 40, vidaMax: 40 });
}
function atualizarParticulas() {
  for (let i = particulas.length - 1; i >= 0; i--) {
    const p = particulas[i];
    p.vida--;
    if (p.onda) p.raio = p.raioMax * (1 - p.vida / p.vidaMax);
    else p.y += p.vy;
    if (p.vida <= 0) particulas.splice(i, 1);
  }
}
function atualizarStatus() {
  if (jogador.envenenado > 0) {
    jogador.envenenado--;
    if (frameCount % 30 === 0) {
      jogador.hp -= jogador.envenenadoDano;
      criarParticula(jogador.x + 12, jogador.y, '-' + jogador.envenenadoDano + ' ☠️', '#88ff44', 60);
      atualizarHud();
      if (jogador.hp <= 0) { jogador.hp = 0; atualizarHud(); fimDeJogo(); }
    }
  }
  if (jogador.queimando > 0) {
    jogador.queimando--;
    if (frameCount % 20 === 0) {
      jogador.hp -= 4;
      criarParticula(jogador.x + 12, jogador.y, '-4 🔥', '#ff8800', 60);
      atualizarHud();
      if (jogador.hp <= 0) { jogador.hp = 0; atualizarHud(); fimDeJogo(); }
    }
  }
  if (jogador.lento > 0) jogador.lento--;
}

// ============================================================
//  HUD
// ============================================================
function atualizarHud() {
  document.getElementById('ouro').textContent = jogador.ouro;
  document.getElementById('xp').textContent = jogador.xp;
  document.getElementById('nivel').textContent = jogador.nivel;
  document.getElementById('inimigos').textContent = monstros.length;
  document.getElementById('onda').textContent = onda;
  document.getElementById('dano').textContent = jogador.forca + jogador.armaBonus + jogador.upgDano * 2;
  const pctHP = Math.max(0, (jogador.hp / jogador.hpMax) * 100);
  const pctMP = Math.max(0, (jogador.mp / jogador.mpMax) * 100);
  document.getElementById('barra-hp').style.width = pctHP + '%';
  document.getElementById('barra-mp').style.width = pctMP + '%';
  document.getElementById('txt-hp').textContent = `${Math.max(0, Math.floor(jogador.hp))} / ${jogador.hpMax}`;
  document.getElementById('txt-mp').textContent = `${jogador.mp} / ${jogador.mpMax}`;
}
function mostrarMsg(texto, duracao) {
  const el = document.getElementById('msg');
  el.textContent = texto;
  el.classList.add('ativo');
  clearTimeout(mensagemTimer);
  mensagemTimer = setTimeout(() => el.classList.remove('ativo'), duracao);
}
function fimDeJogo() {
  jogoAcabou = true;
  const overlay = document.getElementById('overlay');
  document.getElementById('overlay-titulo').textContent = '💀 VOCÊ MORREU';
  document.getElementById('overlay-titulo').style.color = '#ff5555';
  document.getElementById('overlay-texto').textContent =
    `Ondas sobrevividas: ${onda - 1}
Nível: ${jogador.nivel}
XP: ${jogador.xp}
Ouro: ${jogador.ouro}
${bossAtivo ? '\n⚠️ BOSS ATUAL: ' + (bossAtual ? bossAtual.nome : '?') : ''}`;
  overlay.classList.add('ativo');
}

// ============================================================
//  RENDER
// ============================================================
function desenharMapa() {
  for (let y = 0; y < ROWS; y++) {
    for (let x = 0; x < COLS; x++) {
      const t = mapaTiles[y][x];
      const cx = x * TILE, cy = y * TILE;
      if (t === 1) {
        ctx.fillStyle = '#4a3f6b';
        ctx.fillRect(cx, cy, TILE, TILE);
        ctx.fillStyle = '#6a5f8b';
        ctx.fillRect(cx + 2, cy + 2, TILE - 4, TILE - 4);
        ctx.fillStyle = '#3a2f5b';
        ctx.fillRect(cx + 4, cy + 4, TILE - 8, TILE - 8);
      } else {
        ctx.fillStyle = (x + y) % 2 === 0 ? '#2a3f2a' : '#223422';
        ctx.fillRect(cx, cy, TILE, TILE);
        ctx.fillStyle = 'rgba(0,0,0,0.1)';
        ctx.fillRect(cx + (x * 7) % 20, cy + (y * 11) % 20, 3, 3);
      }
    }
  }
}

function desenharBaus() {
  for (const b of baus) {
    b.brilho = (b.brilho + 0.1) % (Math.PI * 2);
    const cx = b.x + b.tamanho / 2;
    const cy = b.y + b.tamanho / 2;
    const r = (b.tamanho / 2) * (1 + Math.sin(b.brilho) * 0.05);
    const grad = ctx.createRadialGradient(cx, cy, 0, cx, cy, r + 10);
    grad.addColorStop(0, 'rgba(255,200,50,0.5)');
    grad.addColorStop(1, 'rgba(255,200,50,0)');
    ctx.fillStyle = grad;
    ctx.beginPath(); ctx.arc(cx, cy, r + 10, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#8b5a1a';
    ctx.fillRect(b.x, b.y + 8, b.tamanho, b.tamanho - 8);
    ctx.fillStyle = '#a86a2a';
    ctx.beginPath();
    ctx.moveTo(b.x, b.y + 8);
    ctx.lineTo(b.x + 3, b.y + 3);
    ctx.lineTo(b.x + b.tamanho - 3, b.y + 3);
    ctx.lineTo(b.x + b.tamanho, b.y + 8);
    ctx.closePath(); ctx.fill();
    ctx.fillStyle = '#f0c040';
    ctx.fillRect(b.x + b.tamanho / 2 - 2, b.y + 3, 4, b.tamanho - 3);
    ctx.font = 'bold 14px serif';
    ctx.textAlign = 'center';
    ctx.fillStyle = '#fff';
    let icone = '?';
    if (b.tipo === 'ouro') icone = '🪙';
    if (b.tipo === 'pocao') icone = '🧪';
    if (b.tipo === 'xp') icone = '⭐';
    if (b.tipo === 'dano') icone = '⚔️';
    ctx.fillText(icone, cx, b.y + b.tamanho + 14);
  }
}

function desenharJogador() {
  const cx = jogador.x + jogador.w / 2;
  const cy = jogador.y + jogador.h / 2;
  if (jogador.cooldownDano > 0 && Math.floor(jogador.cooldownDano / 4) % 2 === 0) ctx.globalAlpha = 0.4;
  ctx.fillStyle = 'rgba(0,0,0,0.4)';
  ctx.beginPath(); ctx.ellipse(cx, cy + 14, 10, 4, 0, 0, Math.PI * 2); ctx.fill();
  if (jogador.upgEscudo > 0) {
    ctx.strokeStyle = `rgba(150,200,255,${0.5 + Math.sin(frameCount * 0.1) * 0.2})`;
    ctx.lineWidth = 3;
    ctx.beginPath(); ctx.arc(cx, cy, 17, 0, Math.PI * 2); ctx.stroke();
  }
  if (jogador.upgRegen > 0) {
    ctx.strokeStyle = `rgba(85,255,136,${0.3 + Math.sin(frameCount * 0.1) * 0.15})`;
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.arc(cx, cy, 20 + Math.sin(frameCount * 0.15) * 2, 0, Math.PI * 2); ctx.stroke();
  }
  const bal = Math.sin(jogador.frameAnim) * 2;
  ctx.fillStyle = '#f0c040';
  ctx.beginPath(); ctx.arc(cx, cy + bal, 12, 0, Math.PI * 2); ctx.fill();
  ctx.strokeStyle = '#8b6a00'; ctx.lineWidth = 2; ctx.stroke();
  ctx.fillStyle = '#000';
  let ox = 0, oy = 0;
  if (jogador.dir === 'up') oy = -3;
  else if (jogador.dir === 'down') oy = 3;
  else if (jogador.dir === 'left') ox = -3;
  else if (jogador.dir === 'right') ox = 3;
  ctx.beginPath();
  ctx.arc(cx + ox - 3, cy + oy + bal, 2, 0, Math.PI * 2);
  ctx.arc(cx + ox + 3, cy + oy + bal, 2, 0, Math.PI * 2);
  ctx.fill();
  if (jogador.atacando > 0) {
    const angs = { up: -Math.PI/2, down: Math.PI/2, left: Math.PI, right: 0 };
    const ang = angs[jogador.dir];
    const alc = 20 + (12 - jogador.atacando) * 2;
    ctx.strokeStyle = '#dddddd'; ctx.lineWidth = 4;
    ctx.beginPath(); ctx.moveTo(cx, cy); ctx.lineTo(cx + Math.cos(ang) * alc, cy + Math.sin(ang) * alc); ctx.stroke();
    ctx.strokeStyle = 'rgba(255,255,255,0.4)'; ctx.lineWidth = 8;
    ctx.beginPath(); ctx.arc(cx, cy, alc - 5, ang - 0.5, ang + 0.5); ctx.stroke();
  }
  ctx.globalAlpha = 1;
}

function desenharMonstros() {
  for (const m of monstros) {
    const cx = m.x + m.tamanho / 2;
    const cy = m.y + m.tamanho / 2;
    const r = m.tamanho / 2;

    ctx.fillStyle = 'rgba(0,0,0,0.4)';
    ctx.beginPath(); ctx.ellipse(cx, cy + r - 2, r * 0.8, r * 0.3, 0, 0, Math.PI * 2); ctx.fill();

    // Aura do boss
    if (m.isBoss) {
      const pulse = 1 + Math.sin(frameCount * 0.1) * 0.1;
      const grad = ctx.createRadialGradient(cx, cy, 0, cx, cy, r * 1.5 * pulse);
      grad.addColorStop(0, 'rgba(255,0,0,0.5)');
      grad.addColorStop(1, 'rgba(255,0,0,0)');
      ctx.fillStyle = grad;
      ctx.beginPath(); ctx.arc(cx, cy, r * 1.5 * pulse, 0, Math.PI * 2); ctx.fill();
    }

    ctx.fillStyle = m.hitFlash > 0 ? '#ffffff' : m.cor;
    ctx.beginPath(); ctx.arc(cx, cy, r, 0, Math.PI * 2); ctx.fill();
    ctx.strokeStyle = m.corBorda;
    ctx.lineWidth = m.isBoss ? 5 : 2;
    ctx.stroke();

    ctx.font = `${Math.floor(r * 1.2)}px serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(m.icone, cx, cy + 1);

    // HP do boss NÃO é mostrado em cima (está no HUD do topo)
    if (!m.isBoss) {
      const bW = m.tamanho + 6, bH = 4;
      const bX = cx - bW / 2, bY = cy - r - 10;
      ctx.fillStyle = '#000'; ctx.fillRect(bX - 1, bY - 1, bW + 2, bH + 2);
      ctx.fillStyle = '#4b1010'; ctx.fillRect(bX, bY, bW, bH);
      const pct = Math.max(0, m.hp / m.hpMax);
      ctx.fillStyle = pct > 0.5 ? '#33cc33' : pct > 0.25 ? '#ffcc00' : '#cc3333';
      ctx.fillRect(bX, bY, bW * pct, bH);
    }
  }
  ctx.textBaseline = 'alphabetic';
}

function desenharProjeteis() {
  for (const p of projeteis) {
    const grad = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.tamanho + 4);
    grad.addColorStop(0, '#fff');
    grad.addColorStop(0.5, p.cor);
    grad.addColorStop(1, 'rgba(0,0,0,0)');
    ctx.fillStyle = grad;
    ctx.beginPath(); ctx.arc(p.x, p.y, p.tamanho + 4, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = p.cor;
    ctx.beginPath(); ctx.arc(p.x, p.y, p.tamanho / 2, 0, Math.PI * 2); ctx.fill();
  }
}

function desenharParticulas() {
  for (const p of particulas) {
    const alpha = p.vida / p.vidaMax;
    if (p.onda) {
      ctx.strokeStyle = `rgba(255,200,50,${alpha})`;
      ctx.lineWidth = 4;
      ctx.beginPath(); ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2); ctx.stroke();
    } else {
      ctx.globalAlpha = alpha;
      ctx.font = 'bold 16px Georgia';
      ctx.textAlign = 'center';
      ctx.strokeStyle = '#000'; ctx.lineWidth = 3;
      ctx.strokeText(p.texto, p.x, p.y);
      ctx.fillStyle = p.cor;
      ctx.fillText(p.texto, p.x, p.y);
      ctx.globalAlpha = 1;
    }
  }
}

function desenharAvisoBoss() {
  if (telaBossWarning <= 0) return;
  const alpha = Math.abs(Math.sin(frameCount * 0.15));
  ctx.fillStyle = `rgba(255, 0, 0, ${0.3 * alpha})`;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = `rgba(255, 50, 50, ${alpha})`;
  ctx.font = 'bold 44px Georgia';
  ctx.textAlign = 'center';
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 6;
  ctx.strokeText('⚠️ BOSS SE APROXIMA ⚠️', canvas.width / 2, canvas.height / 2);
  ctx.fillText('⚠️ BOSS SE APROXIMA ⚠️', canvas.width / 2, canvas.height / 2);
}

// ============================================================
//  LOOP
// ============================================================
function loop() {
  if (!jogoAcabou && !lojaAberta) {
    if (telaBossWarning > 0) {
      telaBossWarning--;
      if (telaBossWarning === 0) spawnarBoss();
    } else {
      moverJogador();
      moverMonstros();
      atualizarProjeteis();
      atualizarStatus();
    }

    if (jogador.cooldownAtaque > 0) jogador.cooldownAtaque--;
    if (jogador.cooldownDano > 0) jogador.cooldownDano--;
    if (jogador.atacando > 0) jogador.atacando--;

    if (frameCount % 30 === 0 && jogador.mp < jogador.mpMax) {
      jogador.mp++;
      atualizarHud();
    }
    if (jogador.upgRegen > 0 && jogador.hp < jogador.hpMax && frameCount % 4 === 0) {
      jogador.hp = Math.min(jogador.hpMax, jogador.hp + jogador.upgRegen * 0.05 * 4);
      atualizarHud();
    }

    if (monstros.length === 0 && telaBossWarning <= 0) {
      tempoProximaOnda++;
      if (tempoProximaOnda > 45) {
        tempoProximaOnda = 0;
        spawnarOnda();
      }
    } else {
      tempoProximaOnda = 0;
    }

    if (bossAtual) atualizarBossHud();
  }

  atualizarParticulas();

  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  desenharMapa();
  desenharBaus();
  desenharMonstros();
  desenharProjeteis();
  desenharJogador();
  desenharParticulas();
  desenharAvisoBoss();

  frameCount++;
  requestAnimationFrame(loop);
}

// ===== INICIALIZAÇÃO =====
atualizarHud();
setTimeout(() => spawnarOnda(), 1000);
loop();
canvas.tabIndex = 0;
canvas.focus();
</script>
</body>
</html>
