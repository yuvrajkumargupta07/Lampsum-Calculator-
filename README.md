# Lampsum-Calculator--
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Lumpsum Mutual Fund Calculator</title>
  <style>
    :root{--bg:#0f172a;--card:#0b1220;--accent:#ef4444;--muted:#9aa4b2;--glass:rgba(255,255,255,0.03)}
    *{box-sizing:border-box}
    body{font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; margin:0; background:linear-gradient(180deg,#071029 0%, #071a2a 100%); color:#e6eef6; padding:28px}
    .wrap{max-width:980px;margin:0 auto}
    header{display:flex;gap:12px;align-items:center;margin-bottom:18px}
    header h1{font-size:20px;margin:0}
    .card{background:var(--card);padding:18px;border-radius:14px;box-shadow:0 6px 30px rgba(2,6,23,0.6);}
    .grid{display:grid;grid-template-columns:repeat(12,1fr);gap:12px;margin-top:12px}
    label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px}
    input[type="number"], input[type="text"]{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:var(--glass);color:inherit}
    select{padding:10px;border-radius:8px;background:var(--glass);border:1px solid rgba(255,255,255,0.06);color:inherit}
    .col-4{grid-column:span 4}
    .col-6{grid-column:span 6}
    .col-12{grid-column:span 12}
    .actions{display:flex;gap:10px;align-items:center}
    button{padding:10px 14px;border-radius:10px;border:0;cursor:pointer;background:var(--accent);color:white}
    button.secondary{background:transparent;border:1px solid rgba(255,255,255,0.06)}
    .result{margin-top:16px;display:flex;gap:12px;flex-wrap:wrap}
    .stat{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:12px;border-radius:10px;min-width:180px}
    .stat h3{margin:0;font-size:14px;color:var(--muted)}
    .stat p{margin:6px 0 0;font-weight:700;font-size:18px}
    table{width:100%;border-collapse:collapse;margin-top:12px}
    th,td{padding:8px;border-bottom:1px solid rgba(255,255,255,0.03);text-align:right}
    th{text-align:left;color:var(--muted);font-weight:600}
    .center{text-align:center}
    footer{margin-top:16px;color:var(--muted);font-size:13px}
    @media(max-width:700px){.col-4,.col-6{grid-column:span 12}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div style="width:52px;height:52px;background:linear-gradient(135deg,#ef4444,#f97316);border-radius:12px;display:flex;align-items:center;justify-content:center;font-weight:700">LM</div>
      <div>
        <h1>Lumpsum Mutual Fund Calculator</h1>
        <div style="color:var(--muted);font-size:13px">Simple calculator to find future value of a one-time investment</div>
      </div>
    </header><div class="card">
  <div style="font-weight:700">Input</div>
  <div class="grid">
    <div class="col-4">
      <label>Investment Amount (₹)</label>
      <input id="amount" type="number" min="0" step="any" placeholder="e.g. 100000" />
    </div>

    <div class="col-4">
      <label>Expected Annual Return (%)</label>
      <input id="rate" type="number" min="0" step="any" placeholder="e.g. 12" />
    </div>

    <div class="col-4">
      <label>Investment Period (years)</label>
      <input id="years" type="number" min="1" step="1" placeholder="e.g. 5" />
    </div>

    <div class="col-6">
      <label>Compounding Frequency</label>
      <select id="freq">
        <option value="1">Annually</option>
        <option value="4">Quarterly</option>
        <option value="12">Monthly</option>
        <option value="365">Daily</option>
      </select>
    </div>

    <div class="col-6 actions">
      <button id="calc">Calculate</button>
      <button id="reset" class="secondary">Reset</button>
      <button id="download" class="secondary">Download CSV</button>
    </div>

    <div class="col-12 result">
      <div class="stat">
        <h3>Future Value</h3>
        <p id="fv">-</p>
      </div>
      <div class="stat">
        <h3>Total Gain</h3>
        <p id="gain">-</p>
      </div>
      <div class="stat">
        <h3>Annualised Return (CAGR)</h3>
        <p id="cagr">-</p>
      </div>
    </div>

    <div class="col-12" style="margin-top:10px">
      <canvas id="chart" height="120"></canvas>
    </div>

    <div class="col-12">
      <div style="font-weight:700;margin-top:10px">Year-wise projection</div>
      <div style="overflow:auto;max-height:280px">
        <table id="table">
          <thead>
            <tr><th class="center">Year</th><th>Start Value</th><th>Interest Earned</th><th>End Value</th></tr>
          </thead>
          <tbody></tbody>
        </table>
      </div>
    </div>

  </div>

  <footer>
    Tip: For lumpsum, formula used = PV × (1 + r/n)^(n×t). Values are indicative and for education only.
  </footer>
</div>

  </div>  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>  <script>
    const fmt = (v)=>{
      if(isNaN(v) || !isFinite(v)) return '-';
      return v.toLocaleString('en-IN', {style:'currency', currency:'INR', maximumFractionDigits:0});
    }

    const amountEl = document.getElementById('amount');
    const rateEl = document.getElementById('rate');
    const yearsEl = document.getElementById('years');
    const freqEl = document.getElementById('freq');
    const fvEl = document.getElementById('fv');
    const gainEl = document.getElementById('gain');
    const cagrEl = document.getElementById('cagr');
    const tableBody = document.querySelector('#table tbody');
    const calcBtn = document.getElementById('calc');
    const resetBtn = document.getElementById('reset');
    const downloadBtn = document.getElementById('download');
    const ctx = document.getElementById('chart').getContext('2d');
    let chart = null;
    
    function compute(){
      const pv = parseFloat(amountEl.value);
      const annual = parseFloat(rateEl.value);
      const years = parseInt(yearsEl.value);
      const freq = parseInt(freqEl.value);
      if(isNaN(pv) || pv<=0 || isNaN(annual) || isNaN(years) || years<=0){
        alert('Please enter valid values'); return;
      }
      const r = annual/100;
      const n = freq;
      const totalPeriods = n * years;
      // Effective rate per period
      const ratePerPeriod = r / n;
      // Future value using compound interest
      const fv = pv * Math.pow(1 + ratePerPeriod, totalPeriods);
      const gain = fv - pv;
      // CAGR = (FV/PV)^(1/years) -1
      const cagr = Math.pow(fv / pv, 1/years) - 1;

      fvEl.textContent = fmt(Math.round(fv));
      gainEl.textContent = fmt(Math.round(gain));
      cagrEl.textContent = (cagr*100).toFixed(2) + ' %';

      // Year-wise breakdown (compute values at the end of each year)
      tableBody.innerHTML = '';
      const yearsArr = [];
      const endValues = [];

      for(let y=1;y<=years;y++){
        const periods = n * y;
        const end = pv * Math.pow(1 + ratePerPeriod, periods);
        const start = (y===1)? pv : pv * Math.pow(1 + ratePerPeriod, n*(y-1));
        const interest = end - start;
        const tr = document.createElement('tr');
        tr.innerHTML = `<td class="center">${y}</td><td style="text-align:right">${fmt(Math.round(start))}</td><td style="text-align:right">${fmt(Math.round(interest))}</td><td style="text-align:right">${fmt(Math.round(end))}</td>`;
        tableBody.appendChild(tr);
        yearsArr.push('Year ' + y);
        endValues.push(Math.round(end));
      }

      // draw chart
      if(chart) chart.destroy();
      chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: yearsArr,
          datasets: [{
            label: 'End Value (₹)',
            data: endValues,
            tension: 0.35,
            fill: true,
            backgroundColor: 'rgba(239,68,68,0.12)',
            borderColor: 'rgba(239,68,68,0.95)',
            pointRadius:4
          }]
        },
        options: {
          responsive:true,
          plugins:{legend:{display:false}},
          scales:{y:{beginAtZero:false,ticks:{callback:function(val){return val.toLocaleString('en-IN');}}}}
        }
      });

      // store last result for download
      lastResult = {pv, annual, years, freq, rows: tableBody.innerHTML, fv: Math.round(fv)};
    }

    let lastResult = null;

    calcBtn.addEventListener('click', compute);
    resetBtn.addEventListener('click', ()=>{
      amountEl.value=''; rateEl.value=''; yearsEl.value=''; freqEl.value='1'; fvEl.textContent='-'; gainEl.textContent='-'; cagrEl.textContent='-'; tableBody.innerHTML=''; if(chart) chart.destroy(); lastResult=null;
    });

    downloadBtn.addEventListener('click', ()=>{
      if(!lastResult){ alert('Please calculate first'); return; }
      // build CSV
      let csv = 'Year,Start Value,Interest Earned,End Value\n';
      // parse rows from table
      const rows = document.querySelectorAll('#table tbody tr');
      rows.forEach(r=>{
        const cols = r.querySelectorAll('td');
        const year = cols[0].textContent.trim();
        const start = cols[1].textContent.trim().replace(/₹|,/g,'');
        const interest = cols[2].textContent.trim().replace(/₹|,/g,'');
        const end = cols[3].textContent.trim().replace(/₹|,/g,'');
        csv += `${year},${start},${interest},${end}\n`;
      });
      const blob = new Blob([csv], {type: 'text/csv'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = 'lumpsum-projection.csv';
      document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
    });

    // small convenience: prefill example values
    amountEl.value = 100000;
    rateEl.value = 12;
    yearsEl.value = 5;
  </script></body>
</html>