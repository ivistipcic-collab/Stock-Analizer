<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Stock Analyzer Pro</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
:root {
  --bg: #080c14;
  --surface: #0d1421;
  --surface2: #111927;
  --border: #1e2d42;
  --accent: #00d4ff;
  --accent2: #7c3aed;
  --green: #00e676;
  --red: #ff4444;
  --yellow: #ffd600;
  --text: #e8f0fe;
  --text-dim: #4a6080;
  --text-mid: #8aa0b8;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Space Grotesk', sans-serif;
  min-height: 100vh;
  padding: 24px 16px 60px;
}
.container { max-width: 860px; margin: 0 auto; }
.header { display: flex; align-items: center; gap: 16px; margin-bottom: 32px; }
.logo { width: 42px; height: 42px; background: linear-gradient(135deg, var(--accent), var(--accent2)); border-radius: 12px; display: flex; align-items: center; justify-content: center; font-size: 20px; flex-shrink: 0; }
.header-text h1 { font-size: clamp(18px, 4vw, 26px); font-weight: 700; background: linear-gradient(90deg, var(--accent), var(--accent2)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.header-text p { font-size: 13px; color: var(--text-dim); margin-top: 2px; }
.search-wrap { display: flex; gap: 10px; margin-bottom: 28px; }
.search-input { flex: 1; background: var(--surface); border: 1.5px solid var(--border); border-radius: 14px; padding: 15px 20px; color: var(--text); font-family: 'JetBrains Mono', monospace; font-size: 18px; font-weight: 600; letter-spacing: 2px; text-transform: uppercase; outline: none; transition: border-color 0.2s, box-shadow 0.2s; }
.search-input::placeholder { color: var(--text-dim); font-size: 14px; letter-spacing: 1px; font-weight: 400; }
.search-input:focus { border-color: var(--accent); box-shadow: 0 0 0 3px rgba(0,212,255,0.1); }
.search-btn { background: linear-gradient(135deg, var(--accent), var(--accent2)); border: none; border-radius: 14px; padding: 15px 24px; color: white; font-family: 'Space Grotesk', sans-serif; font-size: 14px; font-weight: 700; cursor: pointer; transition: opacity 0.15s, transform 0.1s; white-space: nowrap; }
.search-btn:hover { opacity: 0.88; }
.search-btn:active { transform: scale(0.97); }
.search-btn:disabled { opacity: 0.5; cursor: not-allowed; }
.loading { display: none; text-align: center; padding: 60px 20px; }
.loading.show { display: block; }
.loading-ring { width: 56px; height: 56px; margin: 0 auto 20px; border: 3px solid var(--border); border-top-color: var(--accent); border-radius: 50%; animation: spin 0.8s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
.loading-text { color: var(--text-mid); font-size: 14px; }
.loading-sub { color: var(--text-dim); font-size: 12px; margin-top: 6px; }
.error-box { display: none; background: rgba(255,68,68,0.08); border: 1px solid rgba(255,68,68,0.3); border-radius: 14px; padding: 20px; text-align: center; color: var(--red); font-size: 14px; }
.error-box.show { display: block; }
.result { display: none; }
.result.show { display: block; }
.ticker-hero { background: var(--surface); border: 1px solid var(--border); border-radius: 20px; padding: 24px; margin-bottom: 16px; display: flex; justify-content: space-between; align-items: flex-start; flex-wrap: wrap; gap: 16px; }
.ticker-left .ticker-symbol { font-family: 'JetBrains Mono', monospace; font-size: clamp(28px, 6vw, 42px); font-weight: 600; color: var(--accent); letter-spacing: 2px; }
.ticker-left .company-name { font-size: 15px; color: var(--text-mid); margin-top: 4px; }
.ticker-right { text-align: right; }
.current-price { font-family: 'JetBrains Mono', monospace; font-size: clamp(24px, 5vw, 36px); font-weight: 600; }
.price-change { font-size: 14px; margin-top: 4px; font-weight: 600; }
.price-change.up { color: var(--green); }
.price-change.down { color: var(--red); }
.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 14px; }
.grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; margin-bottom: 14px; }
@media (max-width: 600px) { .grid-2 { grid-template-columns: 1fr; } .grid-3 { grid-template-columns: 1fr 1fr; } }
.card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 20px; }
.card-title { font-size: 10px; font-weight: 700; text-transform: uppercase; letter-spacing: 2px; color: var(--text-dim); margin-bottom: 14px; display: flex; align-items: center; gap: 8px; }
.card-title::before { content: ''; display: inline-block; width: 3px; height: 12px; border-radius: 2px; background: var(--accent); }
.val-row { display: flex; justify-content: space-between; align-items: center; padding: 8px 0; border-bottom: 1px solid var(--border); font-size: 13px; }
.val-row:last-child { border-bottom: none; }
.val-label { color: var(--text-mid); }
.val-value { font-weight: 600; font-family: 'JetBrains Mono', monospace; font-size: 13px; }
.val-value.highlight { color: var(--accent); }
.analyst-score { display: flex; align-items: center; justify-content: center; flex-direction: column; padding: 10px 0; }
.analyst-badge { font-size: clamp(18px, 4vw, 26px); font-weight: 700; padding: 8px 20px; border-radius: 30px; margin-bottom: 10px; }
.strong-buy { background: rgba(0,230,118,0.15); color: #00e676; }
.buy { background: rgba(100,220,100,0.12); color: #69db7c; }
.hold { background: rgba(255,214,0,0.12); color: #ffd600; }
.sell { background: rgba(255,100,100,0.12); color: #ff6b6b; }
.strong-sell { background: rgba(255,68,68,0.15); color: #ff4444; }
.pt-box { text-align: center; margin-top: 10px; }
.pt-label { font-size: 11px; color: var(--text-dim); text-transform: uppercase; letter-spacing: 1px; }
.pt-value { font-family: 'JetBrains Mono', monospace; font-size: 22px; font-weight: 600; color: var(--yellow); margin-top: 2px; }
.pt-upside { font-size: 12px; margin-top: 4px; font-weight: 600; }
.earnings-item { padding: 10px 0; border-bottom: 1px solid var(--border); font-size: 13px; line-height: 1.5; }
.earnings-item:last-child { border-bottom: none; }
.earnings-label { font-size: 10px; text-transform: uppercase; letter-spacing: 1px; color: var(--text-dim); margin-bottom: 3px; }
.earnings-value { font-weight: 600; color: var(--text); }
.beat { color: var(--green); }
.miss { color: var(--red); }
.news-item { padding: 12px 0; border-bottom: 1px solid var(--border); }
.news-item:last-child { border-bottom: none; }
.news-headline { font-size: 13px; font-weight: 600; color: var(--text); margin-bottom: 4px; line-height: 1.4; }
.news-meta { font-size: 11px; color: var(--text-dim); }
.news-tag { display: inline-block; background: rgba(0,212,255,0.1); color: var(--accent); border-radius: 4px; padding: 2px 7px; font-size: 10px; font-weight: 700; margin-right: 6px; text-transform: uppercase; }
.news-tag.red { background: rgba(255,68,68,0.1); color: var(--red); }
.news-tag.yellow { background: rgba(255,214,0,0.1); color: var(--yellow); }
.news-tag.green { background: rgba(0,230,118,0.1); color: var(--green); }
.chart-card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 20px; margin-bottom: 14px; }
.chart-tabs { display: flex; gap: 6px; margin-bottom: 20px; flex-wrap: wrap; }
.chart-tab { padding: 6px 14px; border-radius: 20px; font-size: 12px; font-weight: 700; cursor: pointer; border: 1px solid var(--border); background: transparent; color: var(--text-mid); transition: all 0.15s; }
.chart-tab.active { background: var(--accent); color: var(--bg); border-color: var(--accent); }
.chart-wrap { position: relative; height: 240px; }
.ai-card { background: linear-gradient(135deg, rgba(0,212,255,0.05), rgba(124,58,237,0.05)); border: 1px solid rgba(0,212,255,0.2); border-radius: 16px; padding: 20px; margin-bottom: 14px; }
.ai-label { font-size: 10px; font-weight: 700; text-transform: uppercase; letter-spacing: 2px; color: var(--accent); margin-bottom: 12px; display: flex; align-items: center; gap: 8px; }
.ai-label::before { content: '✦'; }
.ai-text { font-size: 14px; line-height: 1.7; color: var(--text-mid); }
.ai-text strong { color: var(--text); }
.disclaimer { font-size: 11px; color: var(--text-dim); text-align: center; margin-top: 24px; line-height: 1.6; opacity: 0.6; }
</style>
</head>
<body>
<div class="container">
  <div class="header">
    <div class="logo">📈</div>
    <div class="header-text">
      <h1>Stock Analyzer Pro</h1>
      <p>Análisis financiero inteligente en segundos</p>
    </div>
  </div>
  <div class="search-wrap">
    <input class="search-input" id="tickerInput" type="text" placeholder="Ingresa un ticker: AAPL, MSFT, TSLA..." maxlength="8"/>
    <button class="search-btn" id="analyzeBtn" onclick="analyze()">ANALIZAR</button>
  </div>
  <div class="loading" id="loading">
    <div class="loading-ring"></div>
    <div class="loading-text">Analizando empresa...</div>
    <div class="loading-sub" id="loadingSub">Buscando datos financieros</div>
  </div>
  <div class="error-box" id="errorBox"></div>
  <div class="result" id="result">
    <div class="ticker-hero">
      <div class="ticker-left">
        <div class="ticker-symbol" id="heroTicker"></div>
        <div class="company-name" id="heroName"></div>
      </div>
      <div class="ticker-right">
        <div class="current-price" id="heroPrice"></div>
        <div class="price-change" id="heroPriceChange"></div>
      </div>
    </div>
    <div class="ai-card">
      <div class="ai-label">Resumen ejecutivo IA</div>
      <div class="ai-text" id="aiSummary"></div>
    </div>
    <div class="grid-2">
      <div class="card">
        <div class="card-title">Valuación</div>
        <div id="valuationContent"></div>
      </div>
      <div class="card">
        <div class="card-title">Consenso de Analistas</div>
        <div class="analyst-score">
          <div class="analyst-badge" id="analystBadge"></div>
          <div style="font-size:12px;color:var(--text-dim)" id="analystCount"></div>
          <div class="pt-box">
            <div class="pt-label">Price Target Promedio</div>
            <div class="pt-value" id="priceTarget"></div>
            <div class="pt-upside" id="priceUpside"></div>
          </div>
        </div>
      </div>
    </div>
    <div class="card" style="margin-bottom:14px">
      <div class="card-title">Último Balance (Earnings)</div>
      <div id="earningsContent"></div>
    </div>
    <div class="chart-card">
      <div class="card-title">Rendimiento Histórico</div>
      <div class="chart-tabs">
        <button class="chart-tab active" onclick="switchTab(this,'1D')">1D</button>
        <button class="chart-tab" onclick="switchTab(this,'1W')">1S</button>
        <button class="chart-tab" onclick="switchTab(this,'1M')">1M</button>
        <button class="chart-tab" onclick="switchTab(this,'1Y')">1A</button>
        <button class="chart-tab" onclick="switchTab(this,'5Y')">5A</button>
      </div>
      <div class="chart-wrap">
        <canvas id="priceChart"></canvas>
      </div>
    </div>
    <div class="card" style="margin-bottom:14px">
      <div class="card-title">Noticias & Eventos Relevantes</div>
      <div id="newsContent"></div>
    </div>
    <div class="disclaimer">
      ⚠️ Esta información es generada con IA y tiene fines educativos únicamente.<br>
      No constituye asesoramiento financiero. Verifica siempre los datos antes de tomar decisiones de inversión.
    </div>
  </div>
</div>
<script>
let chartInstance = null;
let chartDataStore = {};

async function analyze() {
  const ticker = document.getElementById('tickerInput').value.trim().toUpperCase();
  if (!ticker) return;
  document.getElementById('result').classList.remove('show');
  document.getElementById('errorBox').classList.remove('show');
  document.getElementById('loading').classList.add('show');
  document.getElementById('analyzeBtn').disabled = true;
  const steps = ['Buscando datos financieros...','Analizando balance y resultados...','Consultando consenso de analistas...','Generando análisis con IA...'];
  let step = 0;
  const stepInterval = setInterval(() => { step=(step+1)%steps.length; document.getElementById('loadingSub').textContent=steps[step]; }, 2500);
  try {
    const prompt = `Eres un analista financiero experto. El usuario quiere un análisis completo de la empresa con ticker: ${ticker}.

Responde ÚNICAMENTE con un JSON válido (sin markdown, sin backticks, sin texto extra) con esta estructura exacta:

{
  "ticker": "${ticker}",
  "companyName": "Nombre completo de la empresa",
  "sector": "Sector",
  "currentPrice": 150.00,
  "priceChange": 2.50,
  "priceChangePct": 1.67,
  "marketCap": "2.3T",
  "peRatio": 28.5,
  "forwardPE": 24.1,
  "pbRatio": 8.2,
  "evEbitda": 18.4,
  "fairValue": 165.00,
  "fairValueMethod": "DCF + Comparables",
  "analystRating": "Strong Buy",
  "analystCount": 42,
  "priceTarget": 210.00,
  "earnings": {
    "quarter": "Q3 2024",
    "revenue": "125.3B",
    "revenueVsEst": "+3.2%",
    "eps": 1.64,
    "epsVsEst": "+5.8%",
    "netIncome": "29.6B",
    "grossMargin": "46.2%",
    "guidance": "Descripción del guidance dado por la empresa"
  },
  "chartData": {
    "1D": { "labels": ["9:30","10:00","10:30","11:00","11:30","12:00","12:30","13:00","13:30","14:00","14:30","15:00","15:30","16:00"], "values": [148.5,149.2,149.8,150.1,149.9,150.5,151.0,150.8,151.3,151.8,152.1,151.9,152.3,150.0] },
    "1W": { "labels": ["Lun","Mar","Mié","Jue","Vie"], "values": [146.0,147.5,149.0,148.3,150.0] },
    "1M": { "labels": ["S1","S2","S3","S4"], "values": [142.0,145.0,148.0,150.0] },
    "1Y": { "labels": ["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"], "values": [120.0,125.0,132.0,128.0,135.0,140.0,138.0,143.0,147.0,145.0,149.0,150.0] },
    "5Y": { "labels": ["2020","2021","2022","2023","2024"], "values": [73.0,129.0,95.0,130.0,150.0] }
  },
  "news": [
    { "tag": "ACUERDO", "tagColor": "blue", "headline": "Evento o noticia más importante con cifras concretas", "detail": "Detalle adicional" },
    { "tag": "RESULTADOS", "tagColor": "green", "headline": "Segunda noticia relevante", "detail": "Detalle" },
    { "tag": "RIESGO", "tagColor": "red", "headline": "Riesgo o punto de atención importante", "detail": "Detalle" }
  ],
  "aiSummary": "Resumen ejecutivo de 3-4 oraciones sobre la empresa, situación actual, fortalezas y contexto de mercado. Menciona acuerdos importantes y eventos recientes relevantes. Usa <strong>negritas</strong> para datos clave."
}

Usa los datos más precisos y actualizados posibles. Si el ticker no existe devuelve: {"error": "Ticker inválido"}`;

    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 1000,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await response.json();
    let rawText = '';
    for (const block of data.content) { if (block.type === 'text') rawText += block.text; }
    let jsonStr = rawText.trim().replace(/```json|```/g, '').trim();
    const jsonMatch = jsonStr.match(/\{[\s\S]*\}/);
    if (!jsonMatch) throw new Error('No se pudo procesar la respuesta');
    const d = JSON.parse(jsonMatch[0]);
    if (d.error) throw new Error(d.error);
    clearInterval(stepInterval);
    renderResult(d);
  } catch(e) {
    clearInterval(stepInterval);
    document.getElementById('loading').classList.remove('show');
    document.getElementById('errorBox').classList.add('show');
    document.getElementById('errorBox').textContent = `❌ ${e.message || 'Error al analizar. Verifica el ticker e intenta nuevamente.'}`;
  }
  document.getElementById('analyzeBtn').disabled = false;
}

function renderResult(d) {
  document.getElementById('loading').classList.remove('show');
  document.getElementById('heroTicker').textContent = d.ticker;
  document.getElementById('heroName').textContent = d.companyName + (d.sector ? ` · ${d.sector}` : '');
  document.getElementById('heroPrice').textContent = `$${Number(d.currentPrice).toFixed(2)}`;
  const chg = d.priceChange >= 0;
  document.getElementById('heroPriceChange').textContent = `${chg?'+':''}${Number(d.priceChange).toFixed(2)} (${chg?'+':''}${Number(d.priceChangePct).toFixed(2)}%) hoy`;
  document.getElementById('heroPriceChange').className = `price-change ${chg?'up':'down'}`;
  document.getElementById('aiSummary').innerHTML = d.aiSummary || '—';
  const valRows = [
    ['Precio actual', `$${Number(d.currentPrice).toFixed(2)}`],
    ['Valor intrínseco (IA)', `$${Number(d.fairValue).toFixed(2)}`, true],
    ['Método', d.fairValueMethod || 'DCF'],
    ['Market Cap', d.marketCap],
    ['P/E Ratio', d.peRatio],
    ['P/E Forward', d.forwardPE],
    ['P/B Ratio', d.pbRatio],
    ['EV/EBITDA', d.evEbitda],
  ];
  document.getElementById('valuationContent').innerHTML = valRows.map(([l,v,h])=>`<div class="val-row"><span class="val-label">${l}</span><span class="val-value ${h?'highlight':''}">${v}</span></div>`).join('');
  const rating = d.analystRating || 'Hold';
  const ratingClass = rating.toLowerCase().replace(' ','-');
  document.getElementById('analystBadge').textContent = rating;
  document.getElementById('analystBadge').className = `analyst-badge ${ratingClass}`;
  document.getElementById('analystCount').textContent = d.analystCount ? `${d.analystCount} analistas` : '';
  const pt = Number(d.priceTarget), cp = Number(d.currentPrice);
  const upside = cp > 0 ? ((pt-cp)/cp*100).toFixed(1) : 0;
  document.getElementById('priceTarget').textContent = `$${pt.toFixed(2)}`;
  const upsideEl = document.getElementById('priceUpside');
  upsideEl.textContent = `${upside>0?'+':''}${upside}% potencial`;
  upsideEl.style.color = upside >= 0 ? 'var(--green)' : 'var(--red)';
  const e = d.earnings || {};
  document.getElementById('earningsContent').innerHTML = `
    <div class="earnings-item"><div class="earnings-label">Período</div><div class="earnings-value">${e.quarter||'—'}</div></div>
    <div class="earnings-item"><div class="earnings-label">Ingresos</div><div class="earnings-value">${e.revenue||'—'} <span class="${(e.revenueVsEst||'').startsWith('+')?'beat':'miss'}">${e.revenueVsEst||''} vs estimado</span></div></div>
    <div class="earnings-item"><div class="earnings-label">EPS</div><div class="earnings-value">$${e.eps||'—'} <span class="${(e.epsVsEst||'').startsWith('+')?'beat':'miss'}">${e.epsVsEst||''} vs estimado</span></div></div>
    <div class="earnings-item"><div class="earnings-label">Ganancia Neta</div><div class="earnings-value">${e.netIncome||'—'}</div></div>
    <div class="earnings-item"><div class="earnings-label">Margen Bruto</div><div class="earnings-value">${e.grossMargin||'—'}</div></div>
    <div class="earnings-item"><div class="earnings-label">Guidance</div><div class="earnings-value">${e.guidance||'—'}</div></div>`;
  chartDataStore = d.chartData || {};
  buildChart('1D');
  const tagColorMap = { blue:'', green:'green', red:'red', yellow:'yellow' };
  document.getElementById('newsContent').innerHTML = (d.news||[]).map(n=>`
    <div class="news-item">
      <div class="news-headline"><span class="news-tag ${tagColorMap[n.tagColor]||''}">${n.tag}</span>${n.headline}</div>
      ${n.detail?`<div class="news-meta">${n.detail}</div>`:''}
    </div>`).join('');
  document.getElementById('result').classList.add('show');
}

function buildChart(period) {
  const d = chartDataStore[period];
  if (!d) return;
  const ctx = document.getElementById('priceChart').getContext('2d');
  if (chartInstance) chartInstance.destroy();
  const first = d.values[0], last = d.values[d.values.length-1];
  const isUp = last >= first;
  const color = isUp ? '#00e676' : '#ff4444';
  const gradient = ctx.createLinearGradient(0,0,0,240);
  gradient.addColorStop(0, isUp ? 'rgba(0,230,118,0.25)' : 'rgba(255,68,68,0.25)');
  gradient.addColorStop(1, 'rgba(0,0,0,0)');
  chartInstance = new Chart(ctx, {
    type: 'line',
    data: { labels: d.labels, datasets: [{ data: d.values, borderColor: color, backgroundColor: gradient, borderWidth: 2, pointRadius: 0, pointHoverRadius: 4, fill: true, tension: 0.4 }] },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { backgroundColor: '#0d1421', borderColor: '#1e2d42', borderWidth: 1, titleColor: '#8aa0b8', bodyColor: '#e8f0fe', callbacks: { label: ctx => `$${ctx.parsed.y.toFixed(2)}` } } },
      scales: {
        x: { grid: { color: 'rgba(30,45,66,0.5)' }, ticks: { color: '#4a6080', font: { size: 11 } } },
        y: { grid: { color: 'rgba(30,45,66,0.5)' }, ticks: { color: '#4a6080', font: { size: 11 }, callback: v => `$${v}` }, position: 'right' }
      }
    }
  });
}

function switchTab(btn, period) {
  document.querySelectorAll('.chart-tab').forEach(t => t.classList.remove('active'));
  btn.classList.add('active');
  buildChart(period);
}

document.getElementById('tickerInput').addEventListener('keydown', e => { if (e.key==='Enter') analyze(); });
</script>
</body>
</html>
# Stock-Analizer
