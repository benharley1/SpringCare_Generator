<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Shared 6-Digit Code Generator</title>

<style>
:root {
  --bg: #0f1724;
  --panel: #0b1220;
  --accent: #7c5cff;
  --muted: #9aa4b2;
  --glass: rgba(255,255,255,0.03);
}
body {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial;
  background: linear-gradient(180deg,#061021 0%, #071428 100%);
  color: #e6eef6;
  margin: 0;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}
.card {
  width: 100%;
  max-width: 640px;
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border-radius: 14px;
  padding: 20px;
  box-shadow: 0 8px 30px rgba(2,6,23,0.6);
  border: 1px solid rgba(255,255,255,0.03);
}
h1 { font-size: 22px; margin-bottom: 8px; text-align: center; }
p.lead { color: var(--muted); font-size: 14px; text-align: center; margin-bottom: 20px; }
.controls {
  display: flex; flex-wrap: wrap; gap: 10px; justify-content: center;
}
.controls input, .controls select, .controls button {
  flex: 1 1 45%;
  min-width: 120px;
  border-radius: 10px;
  padding: 12px;
  font-size: 16px;
}
button {
  background: linear-gradient(180deg,var(--accent),#6046e6);
  color: white;
  border: 0;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.15s;
}
button:active { transform: scale(0.97); }
button.ghost {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.06);
  color: var(--muted);
}
.display {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}
.codebox {
  background: #051025;
  border-radius: 10px;
  padding: 16px;
  font-size: 22px;
  text-align: center;
  width: 100%;
  user-select: all;
}
.history {
  margin-top: 16px;
  max-height: 250px;
  overflow-y: auto;
  background: rgba(255,255,255,0.015);
  border-radius: 8px;
  padding: 10px;
  font-family: ui-monospace, monospace;
  font-size: 14px;
}
.history-entry {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  padding-bottom: 6px;
  border-bottom: 1px solid rgba(255,255,255,0.04);
}
@media (max-width: 600px) {
  .controls { flex-direction: column; align-items: stretch; }
  button, input, select { width: 100%; font-size: 17px; }
  .codebox { font-size: 20px; }
}
</style>
</head>

<body>
<div class="card">
  <h1>Shared 6-Digit Code Generator</h1>
  <p class="lead">Generate unique codes — shared history updates live for everyone.</p>

  <div class="controls">
    <input id="prefix" type="text" placeholder="Prefix (e.g. INV-)" value="CODE-">
    <select id="padChar">
      <option value="">(none)</option>
      <option value="-">-</option>
      <option value="_">_</option>
    </select>
    <input id="quantity" type="number" min="1" max="50" value="1">
    <button id="genBtn">Generate</button>
    <button id="copyBtn" class="ghost">Copy</button>
  </div>

  <div class="display">
    <div class="codebox" id="codebox">—</div>
  </div>

  <div class="history" id="history">Loading history...</div>
</div>

<script>
/* ================================
   JSONBin Configuration
================================ */
const BIN_ID = "68ee20ea43b1c97be9671f6b"; // e.g. 66f11234567890abcd1234ef
const API_KEY = "68ee214b43b1c97be967204b";
const BIN_URL = `https://api.jsonbin.io/v3/b/${BIN_ID}`;

/* ================================
   Code Generator Logic
================================ */
const M = 1000000;
function randInt(max){ return Math.floor(Math.random()*max); }
function gcd(a,b){ while(b){[a,b]=[b,a%b];} return a; }
function chooseA(){ while(true){ const a=1+randInt(M-1); if(gcd(a,M)===1) return a; } }
function fmt6(n){ return n.toString().padStart(6,"0"); }

let a=chooseA(), b=randInt(M), c=randInt(M), used=0;
function nextVal(){ const val=((a*c)+b)%M; c=(c+1)%M; used++; return val; }

/* ================================
   DOM Elements
================================ */
const prefixEl=document.getElementById('prefix');
const padEl=document.getElementById('padChar');
const qtyEl=document.getElementById('quantity');
const genBtn=document.getElementById('genBtn');
const copyBtn=document.getElementById('copyBtn');
const codebox=document.getElementById('codebox');
const historyEl=document.getElementById('history');
let lastBatch=[];

/* ================================
   JSONBin Functions
================================ */
async function fetchHistory(){
  const res=await fetch(BIN_URL,{headers:{'X-Master-Key':API_KEY}});
  const data=await res.json();
  return data.record.codes||[];
}

async function saveHistory(codes){
  await fetch(BIN_URL,{
    method:'PUT',
    headers:{
      'Content-Type':'application/json',
      'X-Master-Key':API_KEY
    },
    body:JSON.stringify({codes})
  });
}

/* ================================
   UI Functions
================================ */
function renderHistory(codes){
  historyEl.innerHTML='';
  codes.slice().reverse().forEach(c=>{
    const div=document.createElement('div');
    div.className='history-entry';
    div.innerHTML=`<div>${c.full}</div><div style="color:var(--muted);font-size:12px">${new Date(c.timestamp).toLocaleString()}</div>`;
    historyEl.appendChild(div);
  });
}

async function updateHistory(){
  try{
    const codes=await fetchHistory();
    renderHistory(codes);
  }catch(e){
    historyEl.textContent='Failed to load shared history.';
  }
}

/* ================================
   Main Handlers
================================ */
genBtn.addEventListener('click', async()=>{
  const prefix=prefixEl.value||'';
  const pad=padEl.value||'';
  let qty=parseInt(qtyEl.value)||1;
  if(qty>50)qty=50;
  const newCodes=[];
  for(let i=0;i<qty;i++){
    const v=fmt6(nextVal());
    newCodes.push({full:prefix+pad+v, digits:v, timestamp:Date.now()});
  }
  lastBatch=newCodes;
  codebox.textContent=qty===1?newCodes[0].full:`${qty} codes generated`;

  // Load current shared log
  const current=await fetchHistory();
  const updated=[...current,...newCodes].slice(-500);
  await saveHistory(updated);
  await updateHistory();
});

copyBtn.addEventListener('click', async()=>{
  if(!lastBatch.length) return alert('Generate a code first.');
  const text=lastBatch.map(c=>c.full).join('\n');
  await navigator.clipboard.writeText(text);
  copyBtn.textContent='Copied!';
  setTimeout(()=>copyBtn.textContent='Copy',1200);
});

/* ================================
   Initialize
================================ */
updateHistory();
setInterval(updateHistory, 10000); // auto-refresh every 10s
</script>
</body>
</html>
