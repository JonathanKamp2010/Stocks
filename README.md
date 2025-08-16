<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Stockman — 3-Stock Trading Game</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    :root{
      --bg:#f5f7fb; --card:#ffffff; --muted:#6b7280; --accent:#0b69ff;
      --good:#137c3a; --bad:#c0283e;
      --glass: rgba(11,105,255,0.06);
    }
    *{box-sizing:border-box}
    body {
      margin:0; font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: linear-gradient(180deg,var(--bg),#eef3fb 200px);
      color:#0b1220;
      padding:18px;
    }
    #app { max-width:1200px; margin:0 auto; }
    header { display:flex; align-items:center; justify-content:space-between; gap:12px; margin-bottom:14px; }
    h1 { margin:0; font-size:20px; }
    .muted { color:var(--muted); font-size:13px }
    .grid { display:grid; gap:14px; }
    .cols-3 { grid-template-columns: 1fr 380px; }
    .panel { background:var(--card); border-radius:12px; padding:12px; box-shadow: 0 8px 30px rgba(10,20,40,0.06); }
    .row { display:flex; gap:10px; align-items:center; }
    .controls { display:flex; gap:10px; flex-wrap:wrap; margin-top:8px; }
    canvas { width:100%; height:160px; background:linear-gradient(180deg,#fff,#fbfdff); border-radius:8px; display:block; }
    .mini-canvas { height:110px; }
    .ticker-card { padding:10px; border-radius:10px; background:linear-gradient(180deg,#fff,#fbfdff); box-shadow:0 4px 12px rgba(12,25,60,0.04); }
    .big { font-weight:700; font-size:18px; }
    .small { font-size:13px; color:var(--muted); }
    button { padding:8px 12px; border-radius:8px; border:1px solid #e6eefc; background:white; cursor:pointer; }
    button.primary { background:var(--accent); color:white; border:0; box-shadow:0 6px 18px rgba(11,105,255,0.14) }
    table{width:100%; border-collapse:collapse; font-size:13px}
    th,td{padding:8px; text-align:left; border-bottom:1px solid #f0f4fb}
    .green{color:var(--good); font-weight:700}
    .red{color:var(--bad); font-weight:700}
    .log { height:140px; overflow:auto; background:#fbfcff; border-radius:8px; padding:8px; font-size:13px; }
    footer { margin-top:12px; color:var(--muted); }
    .flex { display:flex; gap:12px; align-items:center; }
    .grow { flex:1; }
    .stock-selector { width:100%; padding:8px; border-radius:8px; border:1px solid #e9f0ff; }
    @media (max-width:920px){
      .cols-3 { grid-template-columns: 1fr; }
      canvas { height:140px }
    }
  </style>
</head>
<body>
  <div id="app">
    <header>
      <div>
        <h1>Stockman — 3-Stock Trading Game</h1>
        <div class="muted">Trade ABC, XYZ, and LMN. Try to grow your capital — no real money involved.</div>
      </div>
      <div class="muted">Starting cash: <strong>$10,000</strong></div>
    </header>

    <div class="grid cols-3">
      <!-- Left: Market & Charts -->
      <div class="panel">
        <div style="display:flex; gap:12px; align-items:center; margin-bottom:10px;">
          <div style="display:flex; gap:10px;">
            <div class="ticker-card">
              <div class="small">Stock</div>
              <div id="ticker0" class="big">ABC</div>
              <div id="price0" class="small">$0.00</div>
            </div>
            <div class="ticker-card">
              <div class="small">Stock</div>
              <div id="ticker1" class="big">XYZ</div>
              <div id="price1" class="small">$0.00</div>
            </div>
            <div class="ticker-card">
              <div class="small">Stock</div>
              <div id="ticker2" class="big">LMN</div>
              <div id="price2" class="small">$0.00</div>
            </div>
          </div>

          <div style="margin-left:auto" class="small">
            <label>Speed:
              <select id="speed">
                <option value="900">1x</option>
                <option value="450">2x</option>
                <option value="200">5x</option>
                <option value="100">10x</option>
              </select>
            </label>
            &nbsp;
            <label>Volatility:
              <select id="vol">
                <option value="1">Normal</option>
                <option value="1.6">High</option>
                <option value="0.6">Low</option>
              </select>
            </label>
            &nbsp;
            <button id="pauseBtn">Pause</button>
            <button id="resetBtn" class="primary">Reset</button>
          </div>
        </div>

        <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:8px;">
          <div class="panel" style="padding:8px;">
            <canvas id="chartA" class="mini-canvas"></canvas>
            <div class="small">ABC — Tech / Growth</div>
          </div>
          <div class="panel" style="padding:8px;">
            <canvas id="chartB" class="mini-canvas"></canvas>
            <div class="small">XYZ — Consumer / Staples</div>
          </div>
          <div class="panel" style="padding:8px;">
            <canvas id="chartC" class="mini-canvas"></canvas>
            <div class="small">LMN — Energy / Commodities</div>
          </div>
        </div>

        <hr style="margin:10px 0">

        <div style="display:flex; gap:12px; align-items:flex-start;">
          <div style="width:280px;">
            <div class="small">Select stock to trade</div>
            <select id="stockSelect" class="stock-selector" size="1">
              <option value="0">ABC</option>
              <option value="1">XYZ</option>
              <option value="2">LMN</option>
            </select>

            <div style="margin-top:8px;">
              <div class="small">Quantity</div>
              <input id="qty" type="number" min="1" value="1" style="width:120px; padding:8px; border-radius:8px; border:1px solid #e9f0ff;" />
            </div>

            <div class="controls" style="margin-top:10px;">
              <button id="buyBtn" class="primary">Buy</button>
              <button id="sellBtn">Sell</button>
              <button id="buyAllBtn">Buy Max</button>
            </div>
            <div style="margin-top:8px" class="small">Tip: use Buy Max to spend all available cash on the selected stock.</div>
          </div>

          <div class="grow panel" style="padding:10px;">
            <div style="display:flex; justify-content:space-between; align-items:center;">
              <div>
                <div class="small">Cash</div>
                <div id="cash" class="big">$0.00</div>
              </div>
              <div>
                <div class="small">Total Portfolio</div>
                <div id="totalVal" class="big">$0.00</div>
              </div>
              <div>
                <div class="small">Unrealized P/L</div>
                <div id="unrealPL" class="big">—</div>
              </div>
            </div>

            <hr style="margin:10px 0">

            <div class="small">Holdings</div>
            <table>
              <thead><tr><th>Ticker</th><th>Shares</th><th>Avg Cost</th><th>Market</th><th>P/L</th></tr></thead>
              <tbody>
                <tr><td>ABC</td><td id="h_shares_0">0</td><td id="h_cost_0">$0.00</td><td id="h_market_0">$0.00</td><td id="h_pl_0">$0.00</td></tr>
                <tr><td>XYZ</td><td id="h_shares_1">0</td><td id="h_cost_1">$0.00</td><td id="h_market_1">$0.00</td><td id="h_pl_1">$0.00</td></tr>
                <tr><td>LMN</td><td id="h_shares_2">0</td><td id="h_cost_2">$0.00</td><td id="h_market_2">$0.00</td><td id="h_pl_2">$0.00</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Right: News & Log -->
      <div class="panel">
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <div>
            <div class="small">Market News</div>
            <div class="big">Events that move prices</div>
          </div>
          <div class="small">Tick: <span id="tickCount">0</span></div>
        </div>

        <hr style="margin:10px 0">

        <div style="display:flex; gap:8px; margin-bottom:8px;">
          <button id="generateNews">Generate News</button>
          <button id="clearLog">Clear Log</button>
          <div style="margin-left:auto" class="small">Vol factor: <span id="volFactor">1</span></div>
        </div>

        <div class="log" id="log"></div>

        <hr style="margin:10px 0">

        <div class="small">Price details</div>
        <div style="display:grid; gap:8px; margin-top:8px;">
          <div class="row"><div style="width:90px" class="small">ABC</div><div id="detail0" class="small">—</div></div>
          <div class="row"><div style="width:90px" class="small">XYZ</div><div id="detail1" class="small">—</div></div>
          <div class="row"><div style="width:90px" class="small">LMN</div><div id="detail2" class="small">—</div></div>
        </div>

        <footer class="muted">This game runs entirely in your browser — no internet required. Save & share the HTML file to play anywhere.</footer>
      </div>
    </div>
  </div>

<script>
/*
  Stockman — 3-stock trading game
  - Stocks: ABC (tech), XYZ (consumer), LMN (energy)
  - Each stock has price history and independent volatility.
  - Single portfolio, cash, buy/sell trading.
  - News events can affect one or more stocks.
*/

// --- Utility helpers ---
function fmt(n) {
  return n.toLocaleString(undefined, {minimumFractionDigits:2, maximumFractionDigits:2});
}
function nowTime() {
  return new Date().toLocaleTimeString();
}
function clamp(v, a, b){ return Math.max(a, Math.min(b, v)); }

// --- Game state ---
const STARTING_CASH = 10000;
const stocks = [
  { id:0, ticker:'ABC', sector:'Tech', price: 120 + Math.random()*20, history: [], volBase:1.2, newsBias: 1.1 },
  { id:1, ticker:'XYZ', sector:'Consumer', price: 55 + Math.random()*10, history: [], volBase:0.9, newsBias: 0.8 },
  { id:2, ticker:'LMN', sector:'Energy', price: 42 + Math.random()*12, history: [], volBase:1.0, newsBias: 1.0 }
];
const portfolio = {
  cash: STARTING_CASH,
  positions: [
    { shares:0, avgCost:0 },
    { shares:0, avgCost:0 },
    { shares:0, avgCost:0 }
  ]
};
let global = {
  tick: 0,
  speedMs: 900,
  volFactor: 1,
  running: true,
  priceTimer: null
};

// --- DOM refs ---
const chartElA = document.getElementById('chartA');
const chartElB = document.getElementById('chartB');
const chartElC = document.getElementById('chartC');
const ctxA = chartElA.getContext('2d');
const ctxB = chartElB.getContext('2d');
const ctxC = chartElC.getContext('2d');

const cashEl = document.getElementById('cash');
const totalValEl = document.getElementById('totalVal');
const unrealEl = document.getElementById('unrealPL');
const tickEl = document.getElementById('tickCount');
const logEl = document.getElementById('log');
const volSel = document.getElementById('vol');
const speedSel = document.getElementById('speed');
const pauseBtn = document.getElementById('pauseBtn');
const resetBtn = document.getElementById('resetBtn');

const stockSelect = document.getElementById('stockSelect');
const qtyInput = document.getElementById('qty');
const buyBtn = document.getElementById('buyBtn');
const sellBtn = document.getElementById('sellBtn');
const buyAllBtn = document.getElementById('buyAllBtn');
const generateNewsBtn = document.getElementById('generateNews');
const clearLogBtn = document.getElementById('clearLog');

// small UI price labels
const priceLabels = [ document.getElementById('price0'), document.getElementById('price1'), document.getElementById('price2') ];
const tickerLabels = [ document.getElementById('ticker0'), document.getElementById('ticker1'), document.getElementById('ticker2') ];
const detailEls = [ document.getElementById('detail0'), document.getElementById('detail1'), document.getElementById('detail2') ];

// holdings UI
const h_shares = [ document.getElementById('h_shares_0'), document.getElementById('h_shares_1'), document.getElementById('h_shares_2') ];
const h_cost = [ document.getElementById('h_cost_0'), document.getElementById('h_cost_1'), document.getElementById('h_cost_2') ];
const h_market = [ document.getElementById('h_market_0'), document.getElementById('h_market_1'), document.getElementById('h_market_2') ];
const h_pl = [ document.getElementById('h_pl_0'), document.getElementById('h_pl_1'), document.getElementById('h_pl_2') ];

// init labels
for(let i=0;i<stocks.length;i++){
  tickerLabels[i].textContent = stocks[i].ticker;
}

// --- Price simulation ---
function stepStock(stock) {
  // random walk with occasional news shocks
  const v = stock.volBase * global.volFactor;
  // small gaussian-like shock: use sum of 3 rand - 1.5
  const randGaussian = (Math.random()+Math.random()+Math.random()) - 1.5;
  const changePct = randGaussian * 0.012 * v; // typical small move
  stock.price *= (1 + changePct);

  // occasional news shock
  if (Math.random() < 0.03 * v) {
    // magnitude depends on sector/newsBias
    const mag = (Math.random()*0.25 + 0.05) * (Math.random() < 0.5 ? -1 : 1) * stock.newsBias;
    stock.price *= (1 + mag);
    pushLog(`News: ${stock.ticker} ${mag>0? 'up':'down'} ${(mag*100).toFixed(1)}% (${stock.sector})`);
  }

  // clamp
  if (stock.price < 1) stock.price = 1;
  // push history
  stock.history.push(stock.price);
  if (stock.history.length > 200) stock.history.shift();
}

// --- Drawing charts ---
function drawMiniChart(ctx, stock) {
  const canvas = ctx.canvas;
  const w = canvas.width = canvas.clientWidth * devicePixelRatio;
  const h = canvas.height = canvas.clientHeight * devicePixelRatio;
  ctx.clearRect(0,0,w,h);
  ctx.save();
  ctx.scale(devicePixelRatio, devicePixelRatio);

  // background
  ctx.fillStyle = '#fff';
  ctx.fillRect(0,0,canvas.clientWidth,canvas.clientHeight);

  const data = stock.history.length ? stock.history : [stock.price];
  const dw = canvas.clientWidth, dh = canvas.clientHeight;
  const max = Math.max(...data), min = Math.min(...data);
  const pad = (max-min)*0.08 || max*0.05;
  const top = max + pad, bottom = Math.max(0, min - pad);

  // grid lines
  ctx.strokeStyle = '#eef2f7';
  ctx.lineWidth = 1;
  for (let i=0;i<3;i++){
    const y = (i/2)*(dh-10)+5;
    ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(dw,y); ctx.stroke();
  }

  // path
  ctx.beginPath();
  for (let i=0;i<data.length;i++){
    const x = (i/(data.length-1 || 1))*(dw-10)+5;
    const y = ((top - data[i])/(top-bottom))*(dh-12)+6;
    if (i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  // stroke style per sector (deterministic)
  let color = '#0b69ff';
  if (stock.id === 1) color = '#f59e0b';
  if (stock.id === 2) color = '#10b981';
  ctx.strokeStyle = color;
  ctx.lineWidth = 2;
  ctx.stroke();

  // last dot
  const last = data[data.length-1];
  const lastX = dw - 5;
  const lastY = ((top - last)/(top-bottom))*(dh-12)+6;
  ctx.beginPath(); ctx.arc(lastX, lastY, 3, 0, Math.PI*2);
  ctx.fillStyle = '#fff';
  ctx.fill();
  ctx.stroke();

  ctx.restore();
}

// --- UI updates ---
function updateUI() {
  // prices and small details
  for(let i=0;i<stocks.length;i++){
    priceLabels[i].textContent = '$' + fmt(stocks[i].price);
    detailEls[i].textContent = `Last: $${fmt(stocks[i].price)} • VolBase: ${stocks[i].volBase.toFixed(2)}`;
    // holdings row
    const pos = portfolio.positions[i];
    h_shares[i].textContent = pos.shares;
    h_cost[i].textContent = '$' + fmt(pos.avgCost || 0);
    h_market[i].textContent = '$' + fmt(pos.shares * stocks[i].price);
    const pl = pos.shares * (stocks[i].price - (pos.avgCost || stocks[i].price));
    h_pl[i].textContent = (pl>=0? ' + ':' - ') + '$' + fmt(Math.abs(pl));
    h_pl[i].className = pl>=0 ? 'green' : 'red';
  }

  // charts
  drawMiniChart(ctxA, stocks[0]);
  drawMiniChart(ctxB, stocks[1]);
  drawMiniChart(ctxC, stocks[2]);

  // portfolio totals
  const marketVal = portfolio.positions.reduce((s,p,i) => s + p.shares * stocks[i].price, 0);
  const total = portfolio.cash + marketVal;
  const unrealPL = portfolio.positions.reduce((s,p,i)=> s + p.shares * (stocks[i].price - (p.avgCost || stocks[i].price)), 0);

  cashEl.textContent = '$' + fmt(portfolio.cash);
  totalValEl.textContent = '$' + fmt(total);
  unrealEl.textContent = (unrealPL >= 0 ? '+ ' : '- ') + '$' + fmt(Math.abs(unrealPL));
  unrealEl.className = unrealPL>=0 ? 'green' : 'red';
  tickEl.textContent = global.tick;
  document.getElementById('volFactor').textContent = global.volFactor.toFixed(2);
}

// --- Logging ---
function pushLog(text) {
  const stamp = nowTime();
  logEl.innerHTML = `<div>[${stamp}] ${text}</div>` + logEl.innerHTML;
  // keep log reasonable
  const lines = logEl.childElementCount;
  if (lines > 400) {
    // remove last nodes
    for (let i=0;i<200;i++) logEl.removeChild(logEl.lastChild);
  }
}

// --- Trading logic ---
function buyStock(idx, qty) {
  qty = Math.floor(qty);
  if (qty <= 0) { pushLog('Buy failed: invalid quantity'); return; }
  const cost = qty * stocks[idx].price;
  if (cost > portfolio.cash + 1e-6) { pushLog('Buy failed: not enough cash'); return; }
  const pos = portfolio.positions[idx];
  const prevShares = pos.shares;
  const prevCost = (pos.avgCost || 0) * prevShares;
  const newShares = prevShares + qty;
  const newAvg = (prevCost + cost) / newShares;
  pos.shares = newShares;
  pos.avgCost = newAvg;
  portfolio.cash -= cost;
  pushLog(`Bought ${qty} ${stocks[idx].ticker} @ $${fmt(stocks[idx].price)} = $${fmt(cost)}`);
  updateUI();
}

function sellStock(idx, qty) {
  qty = Math.floor(qty);
  const pos = portfolio.positions[idx];
  if (qty <= 0) { pushLog('Sell failed: invalid quantity'); return; }
  if (qty > pos.shares) { pushLog('Sell failed: not enough shares'); return; }
  const proceed = qty * stocks[idx].price;
  pos.shares -= qty;
  if (pos.shares === 0) pos.avgCost = 0;
  portfolio.cash += proceed;
  pushLog(`Sold ${qty} ${stocks[idx].ticker} @ $${fmt(stocks[idx].price)} = $${fmt(proceed)}`);
  updateUI();
}

// --- News generator (manual) ---
function generateNews() {
  // randomly pick 1-2 stocks or a market-wide event
  if (Math.random() < 0.2) {
    // market-wide
    const mag = (Math.random()*0.12 + 0.03) * (Math.random()<0.5 ? -1 : 1);
    stocks.forEach(s => s.price *= (1 + mag));
    pushLog(`Market news: global ${mag>0? 'boost':'shock'} ${(mag*100).toFixed(1)}%`);
  } else {
    const idx = Math.floor(Math.random()*stocks.length);
    const mag = (Math.random()*0.25 + 0.05) * (Math.random()<0.5 ? -1 : 1);
    stocks[idx].price *= (1 + mag);
    pushLog(`News: ${stocks[idx].ticker} ${mag>0? 'positive':'negative'} ${(mag*100).toFixed(1)}% — ${stocks[idx].sector}`);
  }
  updateUI();
}

// --- Game loop / timer ---
function tick() {
  global.tick++;
  // step each stock
  for (let s of stocks) {
    stepStock(s);
  }
  // small price effect if large buy pressure exists (player holdings)
  for (let i=0;i<stocks.length;i++){
    const pos = portfolio.positions[i];
    if (pos.shares > 0) {
      const exposure = (pos.shares * stocks[i].price) / Math.max(1, portfolio.cash + 1);
      // tiny upward nudges if exposure large
      if (exposure > 2.5) {
        stocks[i].price *= 1 + 0.001 * Math.random();
      }
    }
  }

  updateUI();

  // win/lose messages
  const totalVal = portfolio.cash + portfolio.positions.reduce((s,p,i) => s + p.shares * stocks[i].price, 0);
  if (portfolio.cash < 1 && totalVal < 1) {
    pushLog('You are bankrupt — reset to try again.');
    pauseGame();
  }
  if (totalVal >= STARTING_CASH * 3) {
    pushLog('Nice! Portfolio tripled — you win. Reset to play again.');
    pauseGame();
  }
}

// timer control
function startTimer() {
  if (global.priceTimer) clearInterval(global.priceTimer);
  global.priceTimer = setInterval(() => { if (global.running) tick(); }, global.speedMs);
}
function pauseGame() {
  global.running = false;
  pauseBtn.textContent = 'Resume';
}
function resumeGame() {
  global.running = true;
  pauseBtn.textContent = 'Pause';
}

// --- Event wiring ---
buyBtn.addEventListener('click', () => {
  const idx = Number(stockSelect.value);
  const qty = Number(qtyInput.value) || 0;
  buyStock(idx, qty);
});
sellBtn.addEventListener('click', () => {
  const idx = Number(stockSelect.value);
  const qty = Number(qtyInput.value) || 0;
  sellStock(idx, qty);
});
buyAllBtn.addEventListener('click', () => {
  const idx = Number(stockSelect.value);
  const maxQ = Math.floor(portfolio.cash / stocks[idx].price);
  if (maxQ <= 0) { pushLog('Buy Max: no cash available'); return; }
  buyStock(idx, maxQ);
});
generateNewsBtn.addEventListener('click', generateNews);
clearLogBtn.addEventListener('click', ()=> logEl.innerHTML = '');
pauseBtn.addEventListener('click', ()=> {
  if (global.running) pauseGame(); else resumeGame();
});
resetBtn.addEventListener('click', initGame);

speedSel.addEventListener('change', ()=> {
  global.speedMs = Number(speedSel.value);
  startTimer();
});
volSel.addEventListener('change', ()=> {
  global.volFactor = Number(volSel.value);
  document.getElementById('volFactor').textContent = global.volFactor.toFixed(2);
});

// --- Initialization ---
function initGame() {
  // reset base prices and histories
  for (let i=0;i<stocks.length;i++){
    stocks[i].price = (i === 0 ? 120 : (i===1 ? 55 : 42)) + Math.random()*10;
    stocks[i].history = [stocks[i].price];
  }
  portfolio.cash = STARTING_CASH;
  for (let i=0;i<portfolio.positions.length;i++){
    portfolio.positions[i].shares = 0;
    portfolio.positions[i].avgCost = 0;
  }
  global.tick = 0;
  global.running = true;
  pauseBtn.textContent = 'Pause';
  pushLog('Game reset. Starting cash: $' + fmt(STARTING_CASH));
  updateUI();
}

// start the game
initGame();
startTimer();

// responsive redraw when canvas resized (e.g. mobile)
window.addEventListener('resize', updateUI);

// expose a couple helpers for debugging in console (optional)
window._stockman = { stocks, portfolio, pushLog, tick };

</script>
</body>
</html>