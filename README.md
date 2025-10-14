<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Shared 6-Digit Code Generator (Supabase)</title>

<!-- Supabase JS SDK -->
<script src="https://unpkg.com/@supabase/supabase-js@2.46.1/dist/umd/supabase.js"></script>

<!-- SheetJS for Excel Export -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

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
.footer { margin-top: 16px; display: flex; justify-content: center; }
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
  <p class="lead">All users share one synced history (Supabase backend).</p>

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

  <div class="footer">
    <button id="exportBtn" class="ghost">Export to Excel</button>
  </div>

  <div class="history" id="history">Loading shared history...</div>
</div>

<script>
/* ================================
   Supabase Setup
================================ */
const supabaseUrl = 'https://regoucscslemhbvurekt.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJlZ291Y3Njc2xlbWhidnVyZWt0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjAzODcxNjYsImV4cCI6MjA3NTk2MzE2Nn0.TKPxKfj70S-BarDNuWrpnmLMEl55XABwhIq-DvBxvAA'; // ← replace this
const supabase = window.supabase.createClient(supabaseUrl, supabaseKey);

/* ================================
   Generator Logic
================================ */
const M = 1000000;
function randInt(max){ return Math.floor(Math.random()*max); }
function gcd(a,b){ while(b){[a,b]=[b,a%b];} return a; }
function chooseA(){ while(true){ const a=1+randInt(M-1); if(gcd(a,M)===1)return a; } }
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
const exportBtn=document.getElementById('exportBtn');

let lastBatch=[];

/* ================================
   Supabase Functions
================================ */
async function loadHistory(){
  const { data, error } = await supabase
    .from('codes')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(500);
  if (error) {
    historyEl.textContent = "Error loading history.";
    console.error(error);
    return [];
  }
  renderHistory(data);
  return data;
}

function renderHistory(rows){
  historyEl.innerHTML = '';
  if (!rows.length) {
    historyEl.textContent = 'No codes yet.';
    return;
  }
  rows.forEach(row => {
    const div=document.createElement('div');
    div.className='history-entry';
    const t=new Date(row.created_at).toLocaleString();
    div.innerHTML=`<div>${row.full_code}</div><div style="color:var(--muted);font-size:12px">${t}</div>`;
    historyEl.appendChild(div);
  });
}

async function insertCodes(codes){
  const { error } = await supabase.from('codes').insert(codes);
  if (error) console.error('Insert failed', error);
}

/* ================================
   Excel Export
================================ */
async function exportToExcel(){
  const { data, error } = await supabase
    .from('codes')
    .select('*')
    .order('created_at',{ascending:false});
  if(error){ alert("Failed to export."); return; }
  const rows = data.map(r => ({
    Full_Code: r.full_code,
    Digits: r.digits,
    Timestamp: new Date(r.created_at).toLocaleString()
  }));
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Codes");
  XLSX.writeFile(wb, "shared_codes.xlsx");
}

/* ================================
   UI Handlers
================================ */
genBtn.addEventListener('click', async()=>{
  const prefix=prefixEl.value||'';
  const pad=padEl.value||'';
  let qty=parseInt(qtyEl.value)||1;
  if(qty>50)qty=50;

  const newCodes=[];
  for(let i=0;i<qty;i++){
    const v=fmt6(nextVal());
    newCodes.push({
      full_code: prefix + pad + v,
      digits: v
    });
  }
  lastBatch=newCodes;
  codebox.textContent = qty===1 ? newCodes[0].full_code : `${qty} codes generated`;

  await insertCodes(newCodes);
  await loadHistory();
});

copyBtn.addEventListener('click', async()=>{
  if(!lastBatch.length) return alert('Generate a code first.');
  const text=lastBatch.map(c=>c.full_code).join('\n');
  await navigator.clipboard.writeText(text);
  copyBtn.textContent='Copied!';
  setTimeout(()=>copyBtn.textContent='Copy',1200);
});

exportBtn.addEventListener('click', exportToExcel);

/* ================================
   Live Updates (Realtime)
================================ */
supabase
  .channel('codes-changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'codes' }, () => {
    loadHistory();
  })
  .subscribe();

/* ================================
   Initial Load
================================ */
loadHistory();
</script>
</body>
</html>

