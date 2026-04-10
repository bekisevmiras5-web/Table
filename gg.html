<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tourist Risks Reflection</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background: linear-gradient(135deg, #eff6ff, #f1f5f9);
    min-height: 100vh;
    padding: 40px 16px;
  }

  .container { max-width: 900px; margin: auto; }

  h1 {
    text-align: center;
    font-size: 26px;
    font-weight: 800;
    color: #1e293b;
    margin-bottom: 6px;
  }

  .subtitle {
    text-align: center;
    color: #94a3b8;
    font-size: 14px;
    margin-bottom: 28px;
  }

  .card {
    background: white;
    border-radius: 16px;
    box-shadow: 0 2px 12px rgba(0,0,0,0.08);
    border: 1px solid #e2e8f0;
    overflow: hidden;
    margin-bottom: 20px;
  }

  table { width: 100%; border-collapse: collapse; }

  thead tr { background: #334155; color: white; }

  th {
    padding: 12px 18px;
    text-align: left;
    font-size: 13px;
    font-weight: 600;
  }

  th:last-child { text-align: center; width: 80px; }

  tbody tr:nth-child(even) { background: #f8fafc; }
  tbody tr:nth-child(odd)  { background: white; }

  td { padding: 10px 18px; vertical-align: middle; }

  .name-cell {
    font-weight: 600;
    color: #1e293b;
    font-size: 14px;
    white-space: nowrap;
    width: 190px;
  }

  .grid { display: flex; gap: 6px; flex-wrap: wrap; }

  .box {
    width: 32px; height: 32px;
    border: 2px solid #cbd5e1;
    border-radius: 8px;
    background: white;
    cursor: pointer;
    font-size: 12px;
    font-weight: 700;
    color: #94a3b8;
    transition: transform 0.1s, box-shadow 0.1s;
  }

  .box:hover { transform: scale(1.15); border-color: #94a3b8; }
  .box:active { transform: scale(0.95); }

  .box.red    { background: #f87171; border-color: #ef4444; color: #1e293b; transform: scale(1.1); box-shadow: 0 2px 8px rgba(239,68,68,0.3); }
  .box.yellow { background: #fde047; border-color: #eab308; color: #1e293b; transform: scale(1.1); box-shadow: 0 2px 8px rgba(234,179,8,0.3); }
  .box.green  { background: #4ade80; border-color: #22c55e; color: #1e293b; transform: scale(1.1); box-shadow: 0 2px 8px rgba(34,197,94,0.3); }

  .score-badge {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 36px; height: 36px;
    border-radius: 50%;
    font-size: 13px;
    font-weight: 700;
    border: 2px solid transparent;
    color: #1e293b;
  }

  .score-badge.red    { background: #f87171; border-color: #ef4444; }
  .score-badge.yellow { background: #fde047; border-color: #eab308; }
  .score-badge.green  { background: #4ade80; border-color: #22c55e; }
  .score-badge.empty  { color: #cbd5e1; font-size: 20px; }

  .score-cell { text-align: center; }

  .bottom {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 16px;
  }

  .legend { display: flex; gap: 16px; font-size: 13px; color: #64748b; align-items: center; }

  .legend-dot {
    width: 14px; height: 14px;
    border-radius: 4px;
    display: inline-block;
    border: 2px solid;
  }

  .legend-dot.red    { background: #f87171; border-color: #ef4444; }
  .legend-dot.yellow { background: #fde047; border-color: #eab308; }
  .legend-dot.green  { background: #4ade80; border-color: #22c55e; }

  .legend-item { display: flex; align-items: center; gap: 6px; }

  .result-block {
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 14px;
    padding: 14px 28px;
    text-align: center;
    box-shadow: 0 1px 6px rgba(0,0,0,0.06);
  }

  .result-label { font-size: 11px; color: #94a3b8; text-transform: uppercase; letter-spacing: 0.05em; }
  .result-avg   { font-size: 36px; font-weight: 800; color: #1e293b; line-height: 1.1; }
  .result-sub   { font-size: 11px; color: #94a3b8; margin-top: 2px; }

  .reset-btn {
    background: #334155;
    color: white;
    border: none;
    border-radius: 12px;
    padding: 12px 22px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.15s;
  }

  .reset-btn:hover { background: #1e293b; }
</style>
</head>
<body>

<div class="container">
  <h1>🌍 The Main Risks for Tourists</h1>
  <p class="subtitle">Reflection — выберите оценку для каждого студента (1–10)</p>

  <div class="card">
    <table>
      <thead>
        <tr>
          <th>Студент</th>
          <th>Оценка (1–10)</th>
          <th>Балл</th>
        </tr>
      </thead>
      <tbody id="tbody"></tbody>
    </table>
  </div>

  <div class="bottom">
    <div class="legend">
      <div class="legend-item"><span class="legend-dot red"></span> 1–5</div>
      <div class="legend-item"><span class="legend-dot yellow"></span> 6–7</div>
      <div class="legend-item"><span class="legend-dot green"></span> 8–10</div>
    </div>

    <div style="display:flex; gap:12px; align-items:center;">
      <div class="result-block">
        <div class="result-label">Средний балл</div>
        <div class="result-avg" id="avg">—</div>
        <div class="result-sub" id="sub">0/12 оценено</div>
      </div>
      <button class="reset-btn" onclick="resetAll()">🔄 Сбросить</button>
    </div>
  </div>
</div>

<script>
const students = [
  "Ботагөз", "Жуманова Медина", "Жасұлан Ансар", "Ануар Ансар",
  "Ануар", "Мөлдір", "Е. Медина", "Жансулу",
  "Акбота", "Айлана", "Мансур", "Асанали"
];

let scores = new Array(students.length).fill(0);

function colorClass(n) {
  if (n >= 1 && n <= 5)  return "red";
  if (n >= 6 && n <= 7)  return "yellow";
  if (n >= 8 && n <= 10) return "green";
  return "";
}

function playSound() {
  try {
    new Audio("https://www.soundjay.com/buttons/sounds/button-16.mp3").play();
  } catch(e) {}
}

function render() {
  const tbody = document.getElementById("tbody");
  tbody.innerHTML = "";

  students.forEach((name, i) => {
    const tr = document.createElement("tr");

    const tdName = document.createElement("td");
    tdName.className = "name-cell";
    tdName.textContent = name;

    const tdGrid = document.createElement("td");
    const grid = document.createElement("div");
    grid.className = "grid";

    for (let s = 1; s <= 10; s++) {
      const btn = document.createElement("button");
      btn.className = "box" + (scores[i] === s ? " " + colorClass(s) : "");
      btn.textContent = s;
      btn.onclick = () => { scores[i] = s; playSound(); render(); };
      grid.appendChild(btn);
    }
    tdGrid.appendChild(grid);

    const tdScore = document.createElement("td");
    tdScore.className = "score-cell";
    if (scores[i] > 0) {
      const badge = document.createElement("span");
      badge.className = "score-badge " + colorClass(scores[i]);
      badge.textContent = scores[i];
      tdScore.appendChild(badge);
    } else {
      tdScore.innerHTML = '<span class="score-badge empty">–</span>';
    }

    tr.appendChild(tdName);
    tr.appendChild(tdGrid);
    tr.appendChild(tdScore);
    tbody.appendChild(tr);
  });

  const filled = scores.filter(s => s > 0).length;
  const total = scores.reduce((a, b) => a + b, 0);
  document.getElementById("avg").textContent = filled > 0
    ? (total / students.length).toFixed(1) : "—";
  document.getElementById("sub").textContent = filled + "/" + students.length + " оценено";
}

function resetAll() {
  scores = scores.map(() => 0);
  render();
}

render();
</script>
</body>
</html>