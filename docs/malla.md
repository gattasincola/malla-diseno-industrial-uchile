---
title: Malla
---

<div class="controls">
  <label>Buscar:
    <input id="q" type="search" placeholder="Nombre del ramo o palabra clave" />
  </label>
  <label>Semestre:
    <select id="sem">
      <option value="">Todos</option>
    </select>
  </label>
  <label>Tipo:
    <select id="tipo">
      <option value="">Todos</option>
      <option>Proyectual</option>
      <option>Teoría e Investigación</option>
      <option>Comunicación y Gestión</option>
      <option>Ciencia y Tecnología</option>
      <option>Morfología</option>
      <option>Idioma / Transversal</option>
      <option>Electivo</option>
      <option>Práctica / Título</option>
    </select>
  </label>
</div>

<div id="grid" class="grid"></div>

<script>
const DATA_URL = '/data/cursos.json';


const STORAGE_KEY = 'mallaUChileEstados_v1';
const ESTADOS = ['Pendiente','En curso','Aprobado'];
const $ = (s)=>document.querySelector(s);
const norm = (s)=> (s||'').toLowerCase().normalize('NFD').replace(/\p{Diacritic}/gu,'');

function load(){ try{return JSON.parse(localStorage.getItem(STORAGE_KEY)||'{}')}catch(e){return {}}}
function save(m){ localStorage.setItem(STORAGE_KEY, JSON.stringify(m)) }
let estados = load();

async function init(){
  const res = await fetch(DATA_URL);
  const data = await res.json();
  renderSemestres(data);
  renderGrid(data);
  bind(data);
}
function renderSemestres(data){
  const sems = [...new Set(data.cursos.map(c=>c.semestre))].sort((a,b)=>a-b);
  const sel = document.getElementById('sem');
  sems.forEach(s=>{ const o=document.createElement('option'); o.value=s; o.textContent=s; sel.appendChild(o); });
}
function cardHTML(c){
  const id = c.id;
  const estado = estados[id] ?? 0;
  const badge = ESTADOS[estado];
  return `
  <label class="card state-${estado}" data-id="${id}">
    <input type="checkbox" aria-label="alternar estado" />
    <div class="card-top">
      <span class="badge">${badge}</span>
      <span class="creditos">${c.creditos ? c.creditos + ' SCT' : ''}</span>
    </div>
    <h3>${c.nombre}</h3>
    <div class="meta">
      <span class="pill">S${c.semestre}</span>
      ${c.tipo ? `<span class="pill">${c.tipo}</span>`: ''}
    </div>
    ${c.prerrequisitos && c.prerrequisitos.length ? `<div class="pre">Pre: ${c.prerrequisitos.join(', ')}</div>`:''}
  </label>`;
}
function renderGrid(data){
  const q = norm(document.getElementById('q').value);
  const sem = document.getElementById('sem').value;
  const tipo = document.getElementById('tipo').value;
  const grid = document.getElementById('grid');
  const items = data.cursos.filter(c=>{
    const okQ = !q || norm(c.nombre).includes(q);
    const okS = !sem || String(c.semestre)===String(sem);
    const okT = !tipo || c.tipo===tipo;
    return okQ && okS && okT;
  }).sort((a,b)=> a.semestre===b.semestre ? a.orden-b.orden : a.semestre-b.semestre);
  grid.innerHTML = items.map(cardHTML).join('');
  grid.querySelectorAll('.card').forEach(card=>{
    card.addEventListener('click', (ev)=>{
      ev.preventDefault();
      const id = card.getAttribute('data-id');
      const cur = estados[id] ?? 0;
      const next = (cur + 1) % 3;
      estados[id] = next; save(estados);
      card.classList.remove('state-0','state-1','state-2');
      card.classList.add('state-'+next);
      card.querySelector('.badge').textContent = ESTADOS[next];
    });
  });
}
function bind(data){
  document.getElementById('q').addEventListener('input', ()=>renderGrid(data));
  document.getElementById('sem').addEventListener('change', ()=>renderGrid(data));
  document.getElementById('tipo').addEventListener('change', ()=>renderGrid(data));
}
init();
</script>

  document.getElementById('sem').addEventListener('change', ()=>renderGrid(data));
  document.getElementById('tipo').addEventListener('change', ()=>renderGrid(data));
}
init();
</script>
