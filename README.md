<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>6-Digit Code Generator (Supabase)</title>

<!-- Supabase SDK -->
<script src="https://unpkg.com/@supabase/supabase-js@2.46.1/dist/umd/supabase.js"></script>
<!-- SheetJS for Excel -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
:root {
  --accent: #7c5cff;
  --muted: #9aa4b2;
}

body {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial;
  background: linear-gradient(180deg, #061021 0%, #071428 100%);
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
  max-width: 640px;
  padding: 20px;
  border-radius: 14px;
  background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
  box-shadow: 0 8px 30px rgba(2,6,23,0.6);
  box-sizing: border-box;
}

/* ---------- Controls ---------- */
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.controls select,
.controls input[type="number"] {
  flex: 1 1 45%;
  min-width: 130px;
  max-width: 200px;
  border-radius: 10px;
  padding: 10px;
  font-size: 15px;
  border: 1px solid rgba(255,255,255,0.08);
  background: rgba(255,255,255,0.05);
  color: #e6eef6;
  height: 44px;
  box-sizing: border-box;
}

.controls select:focus,
.controls input:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(124, 92, 255, 0.3);
}

/* ---------- Buttons ---------- */
button {
  background: linear-gradient(180deg, var(--accent), #6046e6);
  color: white;
  border: 0;
  font-weight: 600;
  cursor: pointer;
  padding: 10px 0;
  border-radius: 10px;
  font-size: 15px;
  height: 44px;
  transition: transform 0.1s ease, opacity 0.2s;
}

button:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

button.ghost {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.08);
  color: var(--muted);
}

/* Footer export button */
.footer button {
  width: 60%;
  min-width: 140px;
  height: 44px;
}

/* ---------- Display & History ---------- */
.display {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.codebox {
  background: #051025;
  border-radius: 10px;
  padding: 18px;
  font-size: 22px;
  text-align: center;
  width: 100%;
  word-break: break-word;
}

.history {
  margin-top: 16px;
  max-height: 300px;
  overflow-y: auto;
  font-family: monospace;
  font-size: 14px;
  border-top: 1px solid rgba(255,255,255,0.05);
  padding-top: 8px;
}

.history-entry {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid rgba(255,255,255,0.04);
  padding: 6px 0;
}

/* ---------- Mobile Optimization ---------- */
@media (max-width: 600px) {
  body {
    padding: 10px;
  }

  .card {
    padding: 16px;
  }

  .controls {
    flex-direction: column;
    align-items: stretch;
  }

  .controls select,
  .controls input[type="number"],
  .controls button {
    flex: 1 1 100%;
    width: 100%;
    max-width: 100%;
    font-size: 16px;
  }

  .footer button {
    width: 100%;
  }

  h1 {
    font-size: 1.4rem;
  }

  .codebox {
    font-size: 20px;
  }

  .history {
    max-height: 200px;
  }
}

</style>
</head>

<body>
<div class="card">
  <h1 style="text-align:center;">6-Digit Code Generator</h1>
  <p class="lead" style="text-align:center;color:var(--muted)">All users share one synced history.</p>

  <div class="controls">
    <select id="prefix">
      <option value="LR">LR</option><option value="ML">ML</option>
      <option value="SY">SY</option><option value="CM">CM</option><option value="NM">NM</option>
    </select>

    <!-- Quantity moved before Grade -->
    <input id="quantity" type="number" min="1" max="50" value="1">

    <!-- Grade moved after Quantity -->
    <select id="grade" required>
      <option value="">Select Grade *</option>
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

    <button id="genBtn">Generate</button>
    <button id="copyBtn" class="ghost">Copy</button>
  </div>

  <div class="display"><div class="codebox" id="codebox">—</div></div>

  <div class="footer" style="margin-top:16px;text-align:center;">
    <button id="exportBtn" class="ghost">Export to Excel</button>
  </div>

  <div class="history" id="history">Loading shared history...</div>
</div>

<script>
/* Supabase Setup */
const supabaseUrl = 'https://regoucscslemhbvurekt.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJlZ291Y3Njc2xlbWhidnVyZWt0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjAzODcxNjYsImV4cCI6MjA3NTk2MzE2Nn0.TKPxKfj70S-BarDNuWrpnmLMEl55XABwhIq-DvBxvAA';
const supabase = window.supabase.createClient(supabaseUrl, supabaseKey);

/* Generator Logic */
const M = 1000000;
function randInt(max) { return Math.floor(Math.random() * max); }
function gcd(a, b) { while (b) { [a, b] = [b, a % b]; } return a; }
function chooseA() { while (true) { const a = 1 + randInt(M - 1); if (gcd(a, M) === 1) return a; } }
function fmt6(n) { return n.toString().padStart(6, "0"); }
let a = chooseA(), b = randInt(M), c = randInt(M);
function nextVal() { const val = ((a * c) + b) % M; c = (c + 1) % M; return val; }

/* Elements */
const prefixEl = document.getElementById('prefix');
const gradeEl = document.getElementById('grade');
const homeEl = document.getElementById('home');
const qtyEl = document.getElementById('quantity');
const codebox = document.getElementById('codebox');
const historyEl = document.getElementById('history');
const copyBtn = document.getElementById('copyBtn');
const exportBtn = document.getElementById('exportBtn');
let lastBatch = [];

/* Grade Abbreviations */
const gradeAbbr = {
  "Cook": "CO",
  "HCA - Day": "HD",
  "HCA - Night": "HN",
  "RGN - Day": "RD",
  "RGN - Night": "RN",
  "SHCA - Day": "SD",
  "SHCA - Night": "SN"
};

/* Load History */
async function loadHistory() {
  const { data, error } = await supabase
    .from('codes')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(500);
  if (error) {
    historyEl.textContent = "Error loading history.";
    return;
  }
  historyEl.innerHTML = '';
  if (!data.length) {
    historyEl.textContent = 'No codes yet.';
    return;
  }
  data.forEach(r => {
    const div = document.createElement('div');
    div.className = 'history-entry';
    const t = new Date(r.created_at).toLocaleString();
    div.innerHTML = `
      <div>
        <strong>${r.full_code}</strong><br>
        <small>Grade: ${r.grade || '-'} | Home: ${r.home || '-'}</small>
      </div>
      <div style="color:var(--muted);font-size:12px">${t}</div>`;
    historyEl.appendChild(div);
  });
}

/* Insert Codes */
async function insertCodes(codes) {
  await supabase.from('codes').insert(codes);
}

/* Generate */
document.getElementById('genBtn').addEventListener('click', async () => {
  const prefix = prefixEl.value || '';
  const grade = gradeEl.value.trim();
  const home = homeEl.value.trim();
  if (!grade || !home) {
    alert('Please select both Grade and Home before generating.');
    return;
  }
  let qty = parseInt(qtyEl.value) || 1;
  if (qty > 50) qty = 50;

  const abbr = gradeAbbr[grade] || '';

  const newCodes = [];
  for (let i = 0; i < qty; i++) {
    const v = fmt6(nextVal());
    const fullCode = `${prefix}${v}${abbr}`;
    newCodes.push({ full_code: fullCode, grade, home });
  }

  lastBatch = newCodes;
  codebox.textContent = qty === 1 ? newCodes[0].full_code : `${qty} codes generated`;
  await insertCodes(newCodes);
  await loadHistory();
});

/* Copy */
copyBtn.addEventListener('click', async () => {
  if (!lastBatch.length) return alert('Generate a code first.');
  await navigator.clipboard.writeText(lastBatch.map(c => c.full_code).join('\n'));
  copyBtn.textContent = 'Copied!';
  setTimeout(() => copyBtn.textContent = 'Copy', 1200);
});

/* Export */
exportBtn.addEventListener('click', async () => {
  const { data, error } = await supabase
    .from('codes')
    .select('*')
    .order('created_at', { ascending: false });
  if (error) { alert("Failed to export."); return; }
  const rows = data.map(r => ({
    Full_Code: r.full_code,
    Grade: r.grade || '',
    Home: r.home || '',
    Timestamp: new Date(r.created_at).toLocaleString()
  }));
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Codes");
  XLSX.writeFile(wb, "shared_codes.xlsx");
});

/* Live Updates */
supabase.channel('codes-changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'codes' }, loadHistory)
  .subscribe();

/* Initial Load */
loadHistory();
</script>
</body>
</html>
