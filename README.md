# Facility-Location-MILP[index.html](https://github.com/user-attachments/files/32999409/index.html)
<!DOCTYPE html>
<html lang="en"><head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Facility Location MILP</title>
<style>
:root{--bg:#eef2f6;--card:#fff;--fg:#12263a;--mut:#5b6b7e;--acc:#2563a8;--flow:#c77700;--bd:#d3dbe5;--ok:#1f7a4d;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1a28;--card:#142435;--fg:#e6edf5;--mut:#92a3b6;--acc:#6ab0ff;--flow:#ffb347;--bd:#27405a;--ok:#5fd39a}}
:root[data-theme="dark"]{--bg:#0e1a28;--card:#142435;--fg:#e6edf5;--mut:#92a3b6;--acc:#6ab0ff;--flow:#ffb347;--bd:#27405a;--ok:#5fd39a}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--fg);font:15px/1.5 system-ui,-apple-system,Segoe UI,Roboto,sans-serif}
main{max-width:980px;margin:0 auto;padding:28px 16px 48px}
h1{font:600 clamp(30px,6vw,46px)/1.1 "Iowan Old Style",Georgia,serif;margin:0 0 6px;letter-spacing:-.01em}
h1 i{color:var(--acc)}
.sub{color:var(--mut);margin:0 0 4px}
.tags{display:flex;flex-wrap:wrap;gap:6px;margin:12px 0 22px}
.tags span{border:1px solid var(--bd);border-radius:99px;padding:2px 11px;font-size:13px;color:var(--mut)}
h2{font:600 20px/1.2 "Iowan Old Style",Georgia,serif;margin:0 0 10px}
h3{font-size:14px;margin:16px 0 6px;color:var(--mut);font-weight:600}
.modes{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:14px}
.mode{all:unset;box-sizing:border-box;cursor:pointer;padding:14px;border:1.5px solid var(--bd);border-radius:10px;background:var(--card)}
.mode b{display:block}.mode small{color:var(--mut)}
.mode.on{border-color:var(--acc);box-shadow:inset 4px 0 0 var(--acc)}
.mode:focus-visible,button:focus-visible,input:focus-visible{outline:2px solid var(--acc);outline-offset:2px}
.panel{background:var(--card);border:1px solid var(--bd);border-radius:10px;padding:16px;margin-bottom:14px}
.row{display:flex;flex-wrap:wrap;gap:12px;align-items:end}
label{display:flex;flex-direction:column;font-size:13px;color:var(--mut);gap:3px}
input{font:inherit;color:var(--fg);background:var(--bg);border:1px solid var(--bd);border-radius:6px;padding:6px 8px;width:76px}
button.b{font:inherit;font-weight:600;cursor:pointer;border:1.5px solid var(--acc);background:transparent;color:var(--acc);border-radius:8px;padding:8px 14px}
button.b.p{background:var(--acc);color:var(--bg)}
.sc{overflow-x:auto}
table{border-collapse:collapse}
th{font-weight:600;font-size:13px;color:var(--mut);padding:3px 6px;text-align:left;white-space:nowrap}
td{padding:2px}
td input{width:68px}
pre{margin:0;overflow-x:auto;font:12.5px/1.55 ui-monospace,Menlo,Consolas,monospace;white-space:pre}
.muted{color:var(--mut)}
svg{width:100%;height:auto;display:block}
.kpis{display:flex;flex-wrap:wrap;gap:10px;margin:8px 0 12px}
.kpi{flex:1 1 130px;border:1px solid var(--bd);border-radius:8px;padding:8px 12px}
.kpi small{color:var(--mut);display:block}.kpi b{font-size:21px}
.err{color:#c0392b;font-weight:600}
</style></head>
<body><main>
<h1>Facility Location <i>MILP</i></h1>
<p class="sub">Choose where to open facilities and how to ship from them to customers at the lowest total cost.</p>
<div class="tags"><span>Capacitated plant location</span><span>Binary open indicators</span><span>Flow variables Xij</span><span>Exact solver</span></div>

<div class="modes">
<button class="mode on" data-m="single"><b>Single facility</b><small>One capacity level per facility</small></button>
<button class="mode" data-m="dual"><b>Dual facility</b><small>Low or high capacity per facility</small></button>
</div>

<div class="panel">
<div class="row">
<label>Customers (m)<input id="m" type="number" min="1" max="8" value="3"></label>
<label>Facilities (n)<input id="n" type="number" min="1" max="8" value="3"></label>
<button class="b" id="bt">Build tables</button>
</div>
<div id="tables"></div>
</div>

<div class="row" style="margin-bottom:14px">
<button class="b" id="bf">Generate formulation</button>
<button class="b" id="bn">Draw network</button>
<button class="b p" id="bs">Solve MILP</button>
<button class="b" id="bc">Copy report</button>
</div>

<div class="panel"><h2>Mathematical formulation</h2><div class="sc"><pre id="form" class="muted">Press “Generate formulation” to show the model for your data.</pre></div></div>
<div class="panel"><h2>Network diagram</h2><div id="net" class="muted">Press “Draw network” to see facilities and customers.</div></div>
<div class="panel"><h2>Optimal solution</h2><div id="sol" class="muted">Press “Solve MILP”. The solver checks every opening plan and routes flow at minimum cost for each.</div></div>
<p class="muted" style="font-size:13px">Xij = units shipped from facility i to customer j. Yi = 1 if facility i is open. Costs are per unit shipped; fixed costs are paid once per opened facility.</p>
</main>
<script>
const $=id=>document.getElementById(id);
let mode='single',report='';
const cl=(v,a,b)=>Math.max(a,Math.min(b,Math.round(+v)||a));
const I=(id,v)=>`<input id="${id}" type="number" step="any" value="${v}">`;
document.querySelectorAll('.mode').forEach(b=>b.onclick=()=>{mode=b.dataset.m;document.querySelectorAll('.mode').forEach(x=>x.classList.toggle('on',x===b));build()});
$('bt').onclick=build;$('bf').onclick=()=>{$('form').className='';$('form').textContent=formulation(read())};
$('bn').onclick=()=>draw(read());$('bs').onclick=solve;$('bc').onclick=copy;

function build(){
 const m=cl($('m').value,1,8),n=cl($('n').value,1,8);$('m').value=m;$('n').value=n;
 let h='<h3>Unit cost matrix c<sub>ij</sub> (facility → customer)</h3><div class="sc"><table><tr><th></th>';
 for(let j=0;j<m;j++)h+=`<th>C${j+1}</th>`;h+='</tr>';
 for(let i=0;i<n;i++){h+=`<tr><th>F${i+1}</th>`;for(let j=0;j<m;j++)h+=`<td>${I(`c${i}_${j}`,(i*7+j*5)%9+3)}</td>`;h+='</tr>'}
 h+='</table></div><h3>Customer demand D<sub>j</sub></h3><div class="sc"><table><tr>';
 for(let j=0;j<m;j++)h+=`<th>C${j+1}</th>`;h+='</tr><tr>';
 for(let j=0;j<m;j++)h+=`<td>${I('d'+j,50+10*(j%3))}</td>`;
 h+='</tr></table></div><h3>Facility data</h3><div class="sc"><table><tr><th></th>'+(mode==='single'?'<th>Fixed cost</th><th>Capacity</th>':'<th>Fixed (low)</th><th>Cap (low)</th><th>Fixed (high)</th><th>Cap (high)</th>')+'</tr>';
 for(let i=0;i<n;i++){h+=`<tr><th>F${i+1}</th>`+(mode==='single'?`<td>${I('f'+i,400+50*(i%3))}</td><td>${I('k'+i,100+10*(i%3))}</td>`:`<td>${I('fl'+i,250+30*(i%3))}</td><td>${I('kl'+i,90)}</td><td>${I('fh'+i,420+40*(i%3))}</td><td>${I('kh'+i,180)}</td>`)+'</tr>'}
 $('tables').innerHTML=h+'</table></div>';
 ['form','net','sol'].forEach(k=>{});
}
function read(){
 const m=+$('m').value,n=+$('n').value,g=id=>+$(id).value||0;
 const c=[...Array(n)].map((_,i)=>[...Array(m)].map((_,j)=>g(`c${i}_${j}`)));
 const D=[...Array(m)].map((_,j)=>g('d'+j));
 const opts=[...Array(n)].map((_,i)=>mode==='single'?[{f:g('f'+i),cap:g('k'+i),n:'open'}]:[{f:g('fl'+i),cap:g('kl'+i),n:'low'},{f:g('fh'+i),cap:g('kh'+i),n:'high'}]);
 return{m,n,c,D,opts};
}
function formulation(d){
 const{n,m,c,D,opts}=d,S=mode==='single';
 let s='Minimize  Z = Σi Σj cij·Xij + '+(S?'Σi fi·Yi':'Σi (fLi·YLi + fHi·YHi)')+'\n\nsubject to\n  Σi Xij = Dj   for every customer j\n  '+(S?'Σj Xij ≤ Capi·Yi':'Σj Xij ≤ CapLi·YLi + CapHi·YHi')+'   for every facility i\n  '+(S?'':'YLi + YHi ≤ 1   for every facility i\n  ')+'Xij ≥ 0,  '+(S?'Yi':'YLi, YHi')+' ∈ {0,1}\n\nWith your data\n  Z = ';
 const t=[];for(let i=0;i<n;i++)for(let j=0;j<m;j++)t.push(`${c[i][j]}·X${i+1}${j+1}`);
 for(let i=0;i<n;i++)opts[i].forEach((o,k)=>t.push(`${o.f}·Y${S?'':'LH'[k]}${i+1}`));
 s+=t.join(' + ')+'\n';
 for(let j=0;j<m;j++)s+=`\n  ${[...Array(n)].map((_,i)=>`X${i+1}${j+1}`).join(' + ')} = ${D[j]}`;
 for(let i=0;i<n;i++)s+=`\n  ${[...Array(m)].map((_,j)=>`X${i+1}${j+1}`).join(' + ')} ≤ ${opts[i].map((o,k)=>`${o.cap}·Y${S?'':'LH'[k]}${i+1}`).join(' + ')}`;
 return s;
}
function mcf(c,D,caps){
 const n=c.length,m=D.length,N=n+m+2,Sx=n+m,T=Sx+1,E=[],adj=Array.from({length:N},()=>[]),idx=[];
 const add=(u,v,cap,cost)=>{adj[u].push(E.length);E.push({v,cap,cost});adj[v].push(E.length);E.push({v:u,cap:0,cost:-cost})};
 for(let i=0;i<n;i++){idx.push([]);if(caps[i]>0){add(Sx,i,caps[i],0);for(let j=0;j<m;j++){idx[i][j]=E.length;add(i,n+j,1e9,c[i][j])}}}
 for(let j=0;j<m;j++)add(n+j,T,D[j],0);
 let need=D.reduce((a,b)=>a+b,0),cost=0;
 while(need>1e-9){
  const dist=Array(N).fill(Infinity),pe=Array(N).fill(-1),inq=Array(N).fill(false);dist[Sx]=0;const q=[Sx];
  while(q.length){const u=q.shift();inq[u]=false;for(const id of adj[u]){const e=E[id];if(e.cap>1e-9&&dist[u]+e.cost<dist[e.v]-1e-12){dist[e.v]=dist[u]+e.cost;pe[e.v]=id;if(!inq[e.v]){inq[e.v]=true;q.push(e.v)}}}}
  if(dist[T]===Infinity)return null;
  let f=need,v=T;while(v!==Sx){const id=pe[v];f=Math.min(f,E[id].cap);v=E[id^1].v}
  v=T;while(v!==Sx){const id=pe[v];E[id].cap-=f;E[id^1].cap+=f;v=E[id^1].v}
  need-=f;cost+=f*dist[T];
 }
 const x=idx.map(r=>r.map(k=>E[k+1].cap));
 return{cost,x};
}
function best(d){
 const{n,c,D,opts}=d,tot=D.reduce((a,b)=>a+b,0),st=Array(n).fill(0);let b=null;
 (function rec(i,fix,cap){
  if(b&&fix>=b.total)return;
  if(i===n){if(cap<tot-1e-9)return;
   const caps=st.map((s,k)=>s?opts[k][s-1].cap:0),r=mcf(c,D,caps);
   if(r&&(!b||fix+r.cost<b.total))b={total:fix+r.cost,fix,tr:r.cost,st:[...st],x:r.x};return}
  for(let s=0;s<=opts[i].length;s++){st[i]=s;rec(i+1,fix+(s?opts[i][s-1].f:0),cap+(s?opts[i][s-1].cap:0))}
  st[i]=0;
 })(0,0,0);
 return b;
}
function solve(){
 const d=read(),r=best(d),el=$('sol');
 $('form').className='';$('form').textContent=formulation(d);
 if(!r){el.className='err';el.textContent='Infeasible: total capacity cannot cover total demand. Raise capacities or lower demand.';return}
 el.className='';
 const f=v=>(Math.round(v*100)/100).toLocaleString();
 let h=`<div class="kpis"><div class="kpi"><small>Total cost</small><b>${f(r.total)}</b></div><div class="kpi"><small>Fixed cost</small><b>${f(r.fix)}</b></div><div class="kpi"><small>Transport cost</small><b>${f(r.tr)}</b></div></div>`;
 const txt=[`Total cost ${f(r.total)} (fixed ${f(r.fix)}, transport ${f(r.tr)})`];
 h+='<div class="sc"><table><tr><th>Facility</th><th>Status</th>'+d.D.map((_,j)=>`<th>→ C${j+1}</th>`).join('')+'</tr>';
 r.st.forEach((s,i)=>{const lab=s?(mode==='single'?'Open':'Open ('+d.opts[i][s-1].n+')'):'Closed';
  h+=`<tr><th>F${i+1}</th><td style="padding:2px 8px;color:${s?'var(--ok)':'var(--mut)'}"><b>${lab}</b></td>`+d.D.map((_,j)=>`<td style="padding:2px 8px">${r.x[i][j]?f(r.x[i][j]):'–'}</td>`).join('')+'</tr>';
  txt.push(`F${i+1}: ${lab}`+(s?' ships '+d.D.map((_,j)=>`${f(r.x[i][j]||0)} to C${j+1}`).join(', '):''))});
 el.innerHTML=h+'</table></div>';
 report=formulation(d)+'\n\nOptimal solution\n'+txt.join('\n');
 draw(d,r);
}
function draw(d,r){
 const{n,m,c}=d,H=Math.max(n,m)*62+30,fy=i=>15+(H-30)*(i+.5)/n,cy=j=>15+(H-30)*(j+.5)/m;
 let s=`<svg viewBox="0 0 640 ${H}" role="img" aria-label="Facility to customer network">`;
 for(let i=0;i<n;i++)for(let j=0;j<m;j++){
  const fl=r&&r.x[i][j]>1e-9,x1=110,y1=fy(i),x2=530,y2=cy(j);
  s+=`<line x1="${x1}" y1="${y1}" x2="${x2}" y2="${y2}" stroke="${fl?'var(--flow)':'var(--bd)'}" stroke-width="${fl?3.2:1}"/>`;
  if(fl||n*m<=16)s+=`<text x="${x1+(x2-x1)*.3}" y="${y1+(y2-y1)*.3-4}" font-size="11" fill="${fl?'var(--flow)':'var(--mut)'}" font-weight="${fl?700:400}">${fl?Math.round(r.x[i][j]*100)/100+' @ ':''}${c[i][j]}</text>`}
 for(let i=0;i<n;i++){const o=r?r.st[i]>0:false;
  s+=`<circle cx="110" cy="${fy(i)}" r="22" fill="${o?'var(--acc)':'var(--card)'}" stroke="var(--acc)" stroke-width="2"/><text x="110" y="${fy(i)+4}" text-anchor="middle" font-size="13" font-weight="700" fill="${o?'var(--card)':'var(--fg)'}">F${i+1}</text>`}
 for(let j=0;j<m;j++)s+=`<rect x="508" y="${cy(j)-17}" width="44" height="34" rx="6" fill="var(--card)" stroke="var(--fg)" stroke-width="1.5"/><text x="530" y="${cy(j)+4}" text-anchor="middle" font-size="13" font-weight="700" fill="var(--fg)">C${j+1}</text><text x="562" y="${cy(j)+4}" font-size="11" fill="var(--mut)">D=${d.D[j]}</text>`;
 $('net').className='';$('net').innerHTML=s+'</svg>'+(r?'':'<p class="muted" style="font-size:13px;margin:6px 0 0">Labels show unit cost. Solve to highlight the optimal flows.</p>');
}
async function copy(){
 if(!report){$('bs').click()}
 const b=$('bc');
 try{await navigator.clipboard.writeText(report);b.textContent='Copied'}catch(e){const t=document.createElement('textarea');t.value=report;document.body.appendChild(t);t.select();try{document.execCommand('copy');b.textContent='Copied'}catch(e2){b.textContent='Copy failed'}t.remove()}
 setTimeout(()=>b.textContent='Copy report',1500);
}
build();
</script></body></html>
