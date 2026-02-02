<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Agency Code Generator (Supabase)</title>

<!-- Supabase SDK -->
<script src="https://unpkg.com/@supabase/supabase-js@2.46.1/dist/umd/supabase.js"></script>
<!-- SheetJS for Excel -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
:root {
  --accent: #7c5cff;
  --accent-dark: #6046e6;
  --muted: #9aa4b2;
  --bg: #0a1020;
}

* { box-sizing: border-box; }

body {
  font-family: "Inter", system-ui, sans-serif;
  background: var(--bg);
  color: #e6eef6;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  margin: 0;
  padding: 16px;
}

.card {
  width: 100%;
  max-width: 420px;
  background: #0e1528;
  border-radius: 12px;
  padding: 28px 26px;
  box-shadow: 0 6px 20px rgba(0,0,0,0.4);
  display: flex;
  flex-direction: column;
  gap: 30px;
}

h1 {
  font-size: 1.5rem;
  text-align: center;
  margin: 0;
  padding-bottom: 10px;
  border-bottom: 2px solid rgba(124, 92, 255, 0.8); /* thicker + accent color */
  display: inline-block;
  width: 100%;
}

.lead {
  text-align: center;
  color: var(--muted);
  font-size: 0.9rem;
  margin-top: 8px;
}

/* ---------- Layout ---------- */
.controls {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

.controls select,
.controls input[type="number"] {
  width: 100%;
  border-radius: 8px;
  padding: 10px;
  font-size: 15px;
  border: 1px solid rgba(255,255,255,0.12);
  background: #121b32;
  color: #f1f4fa;
  height: 44px;
  appearance: none;
}

.controls select:focus,
.controls input:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(124, 92, 255, 0.3);
}

/* ---------- Buttons ---------- */
button {
  background: linear-gradient(180deg, var(--accent), var(--accent-dark));
  color: white;
  border: none;
  font-weight: 600;
  cursor: pointer;
  border-radius: 8px;
  font-size: 15px;
  height: 44px;
  transition: transform 0.1s ease, opacity 0.2s;
  width: 100%;
}

button:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

button.ghost {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.12);
  color: var(--muted);
}

/* ---------- Display ---------- */
.display {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.codebox {
  background: #071020;
  border-radius: 8px;
  padding: 18px;
  font-size: 22px;
  text-align: center;
  width: 100%;
  word-break: break-word;
  font-family: monospace;
  min-height: 58px;
}

/* ---------- Toast ---------- */
.toast {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%) translateY(100px);
  background: rgba(0,0,0,0.7);
  color: #fff;
  padding: 10px 16px;
  border-radius: 8px;
  font-size: 14px;
  opacity: 0;
  transition: all 0.4s ease;
  z-index: 1000;
}
.toast.show {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

/* ---------- History ---------- */
.history {
  max-height: 250px;
  overflow-y: auto;
  font-family: monospace;
  font-size: 13.5px;
  border-top: 1px solid rgba(255,255,255,0.08);
  padding-top: 10px;
}

.history-entry {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid rgba(255,255,255,0.04);
  padding: 6px 0;
}

.history-entry strong { font-size: 15px; }
.history-entry small { color: var(--muted); }

/* ---------- Responsive ---------- */
@media (max-width: 600px) {
  .card { padding: 20px; gap: 26px; }
  .row { grid-template-columns: 1fr; gap: 16px; }
}
</style>
</head>
<body>
<div class="card">
  <div>
    <h1>Agency Code Generator</h1>
    <p class="lead">All users share one synced history.</p>
  </div>

  <div class="controls">
    <!-- Row 1: Prefix + Quantity -->
    <div class="row">
      <select id="prefix">
        <option value="LR">LR</option>
        <option value="AF">LR</option>
        <option value="ML">ML</option>
        <option value="SY">SY</option>
        <option value="CM">CM</option>
        <option value="NM">NM</option>
      </select>

      <input id="quantity" type="number" min="1" max="50" value="1">
    </div>

    <!-- Row 2: Grade + Home -->
    <div class="row">
      <select id="grade">
        <option value="">Select Grade (optional)</option>
        <option>Cook</option><option>HCA - Day</option><option>HCA - Night</option>
        <option>RGN - Day</option><option>RGN - Night</option>
        <option>SHCA - Day</option><option>SHCA - Night</option>
      </select>

      <select id="home" required>
        <option value="">Select Home *</option>
        <option>AHE</option><option>ANLW</option><option>BC</option><option>BG</option>
        <option>BH</option><option>BM</option><option>BMN</option><option>CRH</option>
        <option>CV</option><option>DH</option><option>HC</option><option>HH</option>
        <option>HM</option><option>KC</option><option>LP</option><option>MH</option>
        <option>MV</option><option>NH</option><option>OG</option><option>PHB</option>
        <option>PHP</option><option>RM</option><option>RWM</option><option>SL</option>
        <option>SP</option><option>TC</option><option>TG</option><option>TL</option>
        <option>WH</option><option>WK</option><option>WWH</option><option>YG</option>
      </select>
    </div>

    <!-- Generate Button (Full Width Row) -->
    <div>
      <button id="genBtn">Generate</button>
    </div>
  </div>

  <div class="display">
    <div class="codebox" id="codebox">—</div>
  </div>

  <div>
    <button id="exportBtn" class="ghost">Export to Excel</button>
  </div>

  <div class="history" id="history">Loading shared history...</div>
</div>

<div id="toast" class="toast">Copied to clipboard ✅</div>

<script>
const supabaseUrl = 'https://regoucscslemhbvurekt.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJlZ291Y3Njc2xlbWhidnVyZWt0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjAzODcxNjYsImV4cCI6MjA3NTk2MzE2Nn0.TKPxKfj70S-BarDNuWrpnmLMEl55XABwhIq-DvBxvAA';
const supabase = window.supabase.createClient(supabaseUrl, supabaseKey);

const M=1000000;
function randInt(max){return Math.floor(Math.random()*max);}
function gcd(a,b){while(b)[a,b]=[b,a%b];return a;}
function chooseA(){while(true){const a=1+randInt(M-1);if(gcd(a,M)===1)return a;}}
function fmt6(n){return n.toString().padStart(6,"0");}
let a=chooseA(),b=randInt(M),c=randInt(M);
function nextVal(){const val=((a*c)+b)%M;c=(c+1)%M;return val;}

const prefixEl=document.getElementById('prefix');
const gradeEl=document.getElementById('grade');
const homeEl=document.getElementById('home');
const qtyEl=document.getElementById('quantity');
const codebox=document.getElementById('codebox');
const toast=document.getElementById('toast');
const historyEl=document.getElementById('history');
const exportBtn=document.getElementById('exportBtn');

const gradeAbbr={
  "Cook":"CO","HCA - Day":"HD","HCA - Night":"HN",
  "RGN - Day":"RD","RGN - Night":"RN",
  "SHCA - Day":"SD","SHCA - Night":"SN"
};

async function loadHistory(){
  const {data,error}=await supabase.from('codes').select('*').order('created_at',{ascending:false}).limit(500);
  if(error){historyEl.textContent="Error loading history.";return;}
  historyEl.innerHTML='';
  if(!data.length){historyEl.textContent='No codes yet.';return;}
  data.forEach(r=>{
    const div=document.createElement('div');
    div.className='history-entry';
    const t=new Date(r.created_at).toLocaleString();
    div.innerHTML=`<div><strong>${r.full_code}</strong><br><small>Grade: ${r.grade||'-'} | Home: ${r.home||'-'}</small></div><div style="color:var(--muted);font-size:12px">${t}</div>`;
    historyEl.appendChild(div);
  });
}

async function insertCodes(codes){
  await supabase.from('codes').insert(codes);
}

function showToast(msg){
  toast.textContent=msg;
  toast.classList.add('show');
  setTimeout(()=>toast.classList.remove('show'),2500);
}

document.getElementById('genBtn').addEventListener('click',async()=>{
  const prefix=prefixEl.value||'';
  const grade=gradeEl.value.trim();
  const home=homeEl.value.trim();
  if(!home){alert('Please select a Home before generating.');return;}
  let qty=parseInt(qtyEl.value)||1;
  if(qty>50)qty=50;
  const abbr=gradeAbbr[grade]||'';
  const newCodes=[];
  for(let i=0;i<qty;i++){
    const v=fmt6(nextVal());
    const fullCode=`${prefix}${v}${abbr}`;
    newCodes.push({full_code:fullCode,grade,home});
  }
  codebox.textContent=qty===1?newCodes[0].full_code:`${qty} codes generated`;
  await navigator.clipboard.writeText(newCodes.map(c=>c.full_code).join('\n'));
  showToast('Copied to clipboard ✅');
  await insertCodes(newCodes);
  await loadHistory();
});

exportBtn.addEventListener('click',async()=>{
  const {data,error}=await supabase.from('codes').select('*').order('created_at',{ascending:false});
  if(error){alert("Failed to export.");return;}
  const rows=data.map(r=>({
    Full_Code:r.full_code,Grade:r.grade||'',Home:r.home||'',Timestamp:new Date(r.created_at).toLocaleString()
  }));
  const ws=XLSX.utils.json_to_sheet(rows);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"Codes");
  XLSX.writeFile(wb,"shared_codes.xlsx");
});

supabase.channel('codes-changes').on('postgres_changes',{event:'*',schema:'public',table:'codes'},loadHistory).subscribe();
loadHistory();
</script>
</body>
</html>
