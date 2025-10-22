<!doctype html>
<html lang="de">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tic Tac Toe — Saemira</title>
  <style>
    :root{
      --bg:#0f1724;
      --card:#0b1220;
      --accent:#7dd3fc;
      --muted:#94a3b8;
      --win:#34d399;
      font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      min-height:100vh;
      display:flex;
      align-items:center;
      justify-content:center;
      background:linear-gradient(180deg,#071020 0%, #0f1724 100%);
      color:white;
      padding:20px;
    }
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:14px;
      padding:20px;
      width:360px;
      box-shadow: 0 8px 30px rgba(2,6,23,0.6);
    }
    h1{font-size:20px;margin:0 0 8px}
    .meta{color:var(--muted);font-size:13px;margin-bottom:14px}
    .board{
      display:grid;
      grid-template-columns: repeat(3, 1fr);
      gap:10px;
      margin-bottom:14px;
    }
    .cell{
      background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
      border:1px solid rgba(255,255,255,0.04);
      height:110px;
      display:flex;
      align-items:center;
      justify-content:center;
      font-size:46px;
      font-weight:700;
      border-radius:10px;
      cursor:pointer;
      user-select:none;
      transition:transform .08s ease, background .12s;
    }
    .cell:hover{transform:translateY(-4px)}
    .info{display:flex;align-items:center;justify-content:space-between;margin-bottom:12px;gap:8px}
    .left,.right{display:flex;gap:8px;align-items:center}
    .btn{
      background:transparent;
      border:1px solid rgba(255,255,255,0.06);
      padding:8px 10px;border-radius:8px;color:var(--accent);
      cursor:pointer;font-weight:600;
    }
    .btn.secondary{color:var(--muted)}
    .status{font-size:14px;color:var(--muted)}
    .score{font-size:13px;color:var(--muted)}
    .controls{display:flex;gap:8px;flex-wrap:wrap}
    .winner-line{
      height:6px;background:linear-gradient(90deg,var(--accent),var(--win));
      border-radius:999px;margin-top:10px;
    }
    .highlight{background:linear-gradient(180deg, rgba(52,211,153,0.12), rgba(52,211,153,0.06)); border-color:rgba(52,211,153,0.25)}
    .toggle{display:flex;gap:6px;align-items:center}
    .small{font-size:12px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="card" role="application" aria-label="Tic Tac Toe Spiel">
    <h1>Tic Tac Toe</h1>
    <div class="meta">Spiel für 2 Spieler oder gegen CPU (einfach)</div>

    <div class="info">
      <div class="left">
        <div class="status" id="status">Spieler X ist dran</div>
      </div>
      <div class="right">
        <div class="score" id="score">X: 0 • O: 0 • Unentschieden: 0</div>
      </div>
    </div>

    <div class="board" id="board" aria-live="polite">
      <!-- 9 Felder -->
      <div class="cell" data-index="0" role="button" aria-label="Feld 1"></div>
      <div class="cell" data-index="1" role="button" aria-label="Feld 2"></div>
      <div class="cell" data-index="2" role="button" aria-label="Feld 3"></div>
      <div class="cell" data-index="3" role="button" aria-label="Feld 4"></div>
      <div class="cell" data-index="4" role="button" aria-label="Feld 5"></div>
      <div class="cell" data-index="5" role="button" aria-label="Feld 6"></div>
      <div class="cell" data-index="6" role="button" aria-label="Feld 7"></div>
      <div class="cell" data-index="7" role="button" aria-label="Feld 8"></div>
      <div class="cell" data-index="8" role="button" aria-label="Feld 9"></div>
    </div>

    <div class="controls">
      <button class="btn" id="resetBtn">Neues Spiel</button>
      <button class="btn secondary" id="clearScoreBtn">Punkte zurücksetzen</button>

      <label class="toggle small" title="Gegen CPU spielen">
        <input type="checkbox" id="cpuCheckbox"> Gegen CPU
      </label>

      <label class="toggle small" title="Wenn CPU, ob CPU O sein soll">
        <input type="checkbox" id="cpuPlaysO" checked> CPU = O
      </label>
    </div>

    <div id="extras" style="margin-top:10px">
      <div class="small">Tipps: Klick aufs Feld. Bei CPU-Modus beginnt X (du) immer zuerst; CPU spielt zufällig (einfach).</div>
    </div>
  </div>

  <script>
    // Spiel-Logik
    const boardEl = document.getElementById('board');
    const cells = Array.from(document.querySelectorAll('.cell'));
    const statusEl = document.getElementById('status');
    const scoreEl = document.getElementById('score');
    const resetBtn = document.getElementById('resetBtn');
    const clearScoreBtn = document.getElementById('clearScoreBtn');
    const cpuCheckbox = document.getElementById('cpuCheckbox');
    const cpuPlaysO = document.getElementById('cpuPlaysO');

    let state; // Array(9) mit 'X'|'O'|null
    let current = 'X';
    let running = true;
    let scores = {X:0, O:0, draw:0};

    const wins = [
      [0,1,2],[3,4,5],[6,7,8],
      [0,3,6],[1,4,7],[2,5,8],
      [0,4,8],[2,4,6]
    ];

    function init(){
      state = Array(9).fill(null);
      current = 'X';
      running = true;
      cells.forEach(c => {
        c.textContent = '';
        c.classList.remove('highlight');
        c.removeAttribute('aria-disabled');
      });
      updateStatus(`${current === 'X' ? 'Spieler X' : 'Spieler O'} ist dran`);
    }

    function updateStatus(t){
      statusEl.textContent = t;
    }

    function updateScore(){
      scoreEl.textContent = `X: ${scores.X} • O: ${scores.O} • Unentschieden: ${scores.draw}`;
    }

    function checkWinner(){
      for (let combo of wins){
        const [a,b,c] = combo;
        if (state[a] && state[a] === state[b] && state[a] === state[c]){
          return {winner: state[a], combo};
        }
      }
      if (state.every(Boolean)) return {winner: 'draw'};
      return null;
    }

    function highlightCombo(combo){
      combo.forEach(i => cells[i].classList.add('highlight'));
    }

    function makeMove(i){
      if (!running) return;
      if (state[i]) return;
      state[i] = current;
      cells[i].textContent = current;
      cells[i].setAttribute('aria-disabled','true');
      const result = checkWinner();
      if (result){
        running = false;
        if (result.winner === 'draw'){
          scores.draw++;
          updateStatus('Unentschieden!');
        } else {
          scores[result.winner]++;
          updateStatus(`Spieler ${result.winner} gewinnt!`);
          if (result.combo) highlightCombo(result.combo);
        }
        updateScore();
        return;
      }
      // Wechsel
      current = current === 'X' ? 'O' : 'X';
      updateStatus(`Spieler ${current} ist dran`);
      // Falls CPU aktiviert und CPU ist am Zug, lasse CPU ziehen
      if (cpuCheckbox.checked){
        const cpuIsO = cpuPlaysO.checked;
        const cpuSymbol = cpuIsO ? 'O' : 'X';
        if (current === cpuSymbol && running){
          // kurze Verzögerung für natürliches Gefühl
          setTimeout(cpuMove, 300);
        }
      }
    }

    function cpuMove(){
      // einfacher Zufalls-Bot: wählt zufälliges freies Feld
      const free = state.map((v,i)=> v ? null : i).filter(Number.isInteger);
      if (!free.length) return;
      // einfache Heuristik: wenn möglich gewinnzug ausführen, sonst blockieren, sonst random
      // Prüfung: kann CPU gewinnen?
      const cpuIsO = cpuPlaysO.checked;
      const cpuSymbol = cpuIsO ? 'O' : 'X';
      const human = cpuSymbol === 'X' ? 'O' : 'X';

      // helper: test move
      function findWinningMove(symbol){
        for (let idx of free){
          const copy = state.slice();
          copy[idx] = symbol;
          for (let combo of wins){
            const [a,b,c] = combo;
            if (copy[a] && copy[a] === copy[b] && copy[a] === copy[c]) return idx;
          }
        }
        return null;
      }

      let move = findWinningMove(cpuSymbol) ?? findWinningMove(human) ?? free[Math.floor(Math.random()*free.length)];
      makeMove(move);
    }

    // Event listeners
    cells.forEach(c => c.addEventListener('click', () => {
      const i = Number(c.dataset.index);
      // If CPU is on and CPU plays as X, and it's X's turn but CPU is X -> block human clicks? We'll still allow human to click only when it's human's turn.
      const cpuOn = cpuCheckbox.checked;
      const cpuIsO = cpuPlaysO.checked;
      if (cpuOn){
        const cpuSymbol = cpuIsO ? 'O' : 'X';
        // If it's cpu's turn, ignore clicks
        if (current === cpuSymbol) return;
      }
      makeMove(i);
    }));

    resetBtn.addEventListener('click', init);
    clearScoreBtn.addEventListener('click', () => {
      scores = {X:0,O:0,draw:0};
      updateScore();
    });

    // Keyboard accessibility (Zahlen 1-9)
    document.addEventListener('keydown', (e)=>{
      if (!/^[1-9]$/.test(e.key)) return;
      const idx = Number(e.key) - 1;
      cells[idx].focus();
      cells[idx].click();
    });

    // Init on load
    init();
    updateScore();
  </script>
</body>
</html>
