<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>金科一甲象棋</title>
<style>
  * {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    padding: 20px 10px;
    text-align: center;
    font-family: "Kaiti TC", "KaiTi", "標楷體", "Microsoft JhengHei", sans-serif;
    background: #f0e2c8;
    color: #422719;
  }
  h1 { margin: 8px 0; font-size: 28px; }
  #status { font-size: 20px; margin: 12px; font-weight: bold; }
  
  /* 外框容器 */
  .board-container {
    width: min(92vw, 540px);
    margin: 0 auto;
    padding: 12px;
    background: #e6c587;
    border: 4px solid #5a3518;
    border-radius: 8px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.25);
  }

  #board {
    width: 100%;
    aspect-ratio: 9 / 10;
    position: relative;
    background: #f1d49a;
    border: 2px solid #5a3518;
    display: grid;
    grid-template-columns: repeat(9, 1fr);
    grid-template-rows: repeat(10, 1fr);
  }

  .cell {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    min-width: 0;
    min-height: 0;
  }

  /* 傳統棋盤格線 */
  .cell::before {
    content: "";
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    border-right: 1px solid #754b2a;
    border-bottom: 1px solid #754b2a;
    pointer-events: none;
  }
  .cell:nth-child(9n+1)::after {
    content: "";
    position: absolute;
    height: 100%;
    left: 0;
    top: 0;
    border-left: 1px solid #754b2a;
    pointer-events: none;
  }

  /* 楚河漢界 (第5、6列中間不繪製豎線) */
  .river-cell::before {
    border-bottom: 1px solid #754b2a;
    border-right: none;
  }
  .river-text {
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: space-around;
    font-size: clamp(16px, 3.5vw, 22px);
    font-weight: bold;
    color: #754b2a;
    pointer-events: none;
    letter-spacing: 4px;
    opacity: 0.85;
  }

  /* 九宮格斜線 (SVG 畫出 X 線條) */
  .palace-svg {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    pointer-events: none;
    z-index: 1;
  }
  .palace-svg line {
    stroke: #754b2a;
    stroke-width: 1;
  }

  /* 棋子樣式 */
  .piece {
    position: relative;
    z-index: 2;
    width: 86%;
    aspect-ratio: 1;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: clamp(18px, 4.2vw, 32px);
    font-weight: bold;
    background: radial-gradient(circle at 35% 35%, #fff6dd, #ebd1a0);
    border: 2px solid currentColor;
    box-shadow: 2px 3px 5px rgba(0,0,0,0.3);
    cursor: pointer;
    user-select: none;
    transition: transform 0.1s ease;
  }
  .piece:hover {
    transform: scale(1.05);
  }
  .red { color: #c62828; }
  .black { color: #1a1a1a; }

  .selected .piece {
    transform: scale(1.1);
  }
  .selected::after {
    content: "";
    position: absolute;
    width: 90%;
    height: 90%;
    border-radius: 50%;
    border: 3px solid #36a9e1;
    z-index: 3;
    animation: pulse 1.5s infinite;
  }

  @keyframes pulse {
    0% { opacity: 0.6; }
    50% { opacity: 1; }
    100% { opacity: 0.6; }
  }

  .target::after {
    content: "";
    position: absolute;
    width: 26%;
    aspect-ratio: 1;
    border-radius: 50%;
    background: #2e9d55;
    opacity: .85;
    z-index: 3;
  }
  .target.capture::after {
    width: 84%;
    background: transparent;
    border: 3px dashed #e33b32;
    animation: rotate 6s linear infinite;
  }

  @keyframes rotate {
    100% { transform: rotate(360deg); }
  }

  /* 模式切換與控制選項 */
  .mode-select {
    margin: 10px 0;
    font-size: 16px;
    font-weight: bold;
  }
  .mode-select label {
    margin: 0 10px;
    cursor: pointer;
  }

  .controls { margin-top: 15px; }
  button {
    margin: 5px 8px;
    padding: 10px 24px;
    border: 0;
    border-radius: 6px;
    background: #5a3518;
    color: white;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 0 4px 6px rgba(0,0,0,0.2);
    transition: background 0.2s;
  }
  button:hover { background: #824f26; }
  #message { min-height: 24px; margin: 10px; font-weight: bold; }
  .hint { font-size: 14px; color: #6b4d32; }
</style>
</head>
<body>

<h1>🏮 金科一甲象棋</h1>

<div class="mode-select">
  <label><input type="radio" name="gameMode" value="ai" checked onchange="changeMode('ai')"> 單人對弈 (人機大戰)</label>
  <label><input type="radio" name="gameMode" value="pvp" onchange="changeMode('pvp')"> 雙人同屏對弈</label>
</div>

<div id="status">紅方先行</div>

<div class="board-container">
  <div id="board">
    <!-- 九宮格斜線 (黑方 Top Palace) -->
    <svg class="palace-svg" style="grid-area: 1 / 4 / 4 / 7;">
      <line x1="0" y1="0" x2="100%" y2="100%" />
      <line x1="100%" y1="0" x2="0" y2="100%" />
    </svg>
    <!-- 九宮格斜線 (紅方 Bottom Palace) -->
    <svg class="palace-svg" style="grid-area: 8 / 4 / 11 / 7;">
      <line x1="0" y1="0" x2="100%" y2="100%" />
      <line x1="100%" y1="0" x2="0" y2="100%" />
    </svg>
    <!-- 楚河漢界文字層 -->
    <div class="river-text" style="grid-area: 5 / 1 / 6 / 6;">楚 河</div>
    <div class="river-text" style="grid-area: 5 / 5 / 6 / 10;">漢 界</div>
  </div>
</div>

<div id="message">點擊紅方棋子開始走棋。</div>

<div class="controls">
  <button onclick="resetGame()">重新開始</button>
  <button onclick="undoMove()">悔棋</button>
</div>
<p class="hint">紅方在下（帥）、黑方在上（將）。單人模式下玩家執紅子。</p>

<script>
const NAMES = {
  K: "帥", A: "仕", E: "相", R: "俥",
  H: "傌", C: "炮", P: "兵",
  k: "將", a: "士", e: "象", r: "車",
  h: "馬", c: "砲", p: "卒"
};

// 棋子價值估算（AI 用）
const PIECE_VALUES = {
  k: 10000, r: 90, c: 45, h: 40, a: 20, e: 20, p: 10
};

const RED = "red";
const BLACK = "black";

let board = [];
let turn = RED;
let selected = null;
let legalMoves = [];
let history = [];
let gameOver = false;
let vsAI = true;

function changeMode(mode) {
  vsAI = (mode === 'ai');
  resetGame();
}

function initialBoard() {
  const b = Array.from({length: 10}, () => Array(9).fill(null));
  const back = ["r","h","e","a","k","a","e","h","r"];

  for (let c = 0; c < 9; c++) {
    b[0][c] = {type: back[c], color: BLACK};
    b[9][c] = {type: back[c].toUpperCase(), color: RED};
  }

  b[2][1] = {type:"c", color:BLACK};
  b[2][7] = {type:"c", color:BLACK};
  b[7][1] = {type:"C", color:RED};
  b[7][7] = {type:"C", color:RED};

  for (const c of [0,2,4,6,8]) {
    b[3][c] = {type:"p", color:BLACK};
    b[6][c] = {type:"P", color:RED};
  }
  return b;
}

function inside(r,c) {
  return r >= 0 && r < 10 && c >= 0 && c < 9;
}

function palace(r,c,color) {
  if (c < 3 || c > 5) return false;
  return color === RED ? r >= 7 && r <= 9 : r >= 0 && r <= 2;
}

function sameSide(a,b) {
  return a && b && a.color === b.color;
}

function pathCount(r1,c1,r2,c2) {
  let count = 0;
  if (r1 === r2) {
    const step = c2 > c1 ? 1 : -1;
    for (let c = c1 + step; c !== c2; c += step)
      if (board[r1][c]) count++;
  } else if (c1 === c2) {
    const step = r2 > r1 ? 1 : -1;
    for (let r = r1 + step; r !== r2; r += step)
      if (board[r][c1]) count++;
  } else {
    return -1;
  }
  return count;
}

function pseudoMove(r1,c1,r2,c2) {
  if (!inside(r1,c1) || !inside(r2,c2)) return false;
  if (r1 === r2 && c1 === c2) return false;

  const p = board[r1][c1];
  const target = board[r2][c2];
  if (!p || sameSide(p,target)) return false;

  const type = p.type.toLowerCase();
  const dr = r2-r1, dc = c2-c1;
  const ar = Math.abs(dr), ac = Math.abs(dc);
  const forward = p.color === RED ? -1 : 1;

  if (type === "r") {
    return (dr === 0 || dc === 0) && pathCount(r1,c1,r2,c2) === 0;
  }

  if (type === "c") {
    if (dr !== 0 && dc !== 0) return false;
    const screens = pathCount(r1,c1,r2,c2);
    return target ? screens === 1 : screens === 0;
  }

  if (type === "h") {
    if (!((ar===2 && ac===1)||(ar===1 && ac===2))) return false;
    const legR = ar === 2 ? r1 + dr/2 : r1;
    const legC = ac === 2 ? c1 + dc/2 : c1;
    return !board[legR][legC];
  }

  if (type === "e") {
    if (ar !== 2 || ac !== 2) return false;
    if (board[r1+dr/2][c1+dc/2]) return false;
    return p.color === RED ? r2 >= 5 : r2 <= 4;
  }

  if (type === "a") {
    return ar === 1 && ac === 1 && palace(r2,c2,p.color);
  }

  if (type === "k") {
    if (c1 === c2 && target && target.type.toLowerCase() === "k" && pathCount(r1,c1,r2,c2) === 0) {
      return true;
    }
    return ar + ac === 1 && palace(r2,c2,p.color);
  }

  if (type === "p") {
    if (dc === 0 && dr === forward) return true;
    const crossed = p.color === RED ? r1 <= 4 : r1 >= 5;
    return crossed && dr === 0 && ac === 1;
  }

  return false;
}

function findKing(color) {
  for (let r=0; r<10; r++)
    for (let c=0; c<9; c++) {
      const p = board[r][c];
      if (p && p.color===color && p.type.toLowerCase()==="k") return [r,c];
    }
  return null;
}

function inCheck(color) {
  const king = findKing(color);
  if (!king) return true;
  const [kr,kc] = king;

  for (let r=0; r<10; r++)
    for (let c=0; c<9; c++) {
      const p = board[r][c];
      if (p && p.color !== color && pseudoMove(r,c,kr,kc)) return true;
    }
  return false;
}

function legalMove(r1,c1,r2,c2) {
  if (!pseudoMove(r1,c1,r2,c2)) return false;

  const moving = board[r1][c1];
  const captured = board[r2][c2];

  board[r2][c2] = moving;
  board[r1][c1] = null;
  const safe = !inCheck(moving.color);
  board[r1][c1] = moving;
  board[r2][c2] = captured;

  return safe;
}

function getMoves(r,c) {
  const moves = [];
  for (let r2=0; r2<10; r2++)
    for (let c2=0; c2<9; c2++)
      if (legalMove(r,c,r2,c2))
        moves.push([r2,c2]);
  return moves;
}

function getAllLegalMoves(color) {
  const allMoves = [];
  for (let r=0; r<10; r++) {
    for (let c=0; c<9; c++) {
      const p = board[r][c];
      if (p && p.color === color) {
        const moves = getMoves(r,c);
        moves.forEach(([r2,c2]) => {
          allMoves.push({from: [r,c], to: [r2,c2]});
        });
      }
    }
  }
  return allMoves;
}

function hasAnyMove(color) {
  return getAllLegalMoves(color).length > 0;
}

function render() {
  const el = document.getElementById("board");
  
  const cells = el.querySelectorAll(".cell");
  cells.forEach(c => c.remove());

  for (let r=0; r<10; r++) {
    for (let c=0; c<9; c++) {
      const cell = document.createElement("div");
      cell.className = "cell";
      if (r === 4) cell.classList.add("river-cell");

      const p = board[r][c];
      if (p) {
        const piece = document.createElement("div");
        piece.className = "piece " + p.color;
        piece.textContent = NAMES[p.type];
        cell.appendChild(piece);
      }

      if (selected && selected[0]===r && selected[1]===c) {
        cell.classList.add("selected");
      }

      const move = legalMoves.find(m => m[0]===r && m[1]===c);
      if (move) {
        cell.classList.add("target");
        if (p) cell.classList.add("capture");
      }

      cell.addEventListener("click", () => clickCell(r,c));
      el.appendChild(cell);
    }
  }

  const status = document.getElementById("status");
  if (gameOver) return;

  const checked = inCheck(turn);
  status.textContent = (turn===RED ? "紅方" : "黑方") + "回合" + (checked ? "（將軍！）" : "");
}

function makeMove(r1, c1, r2, c2) {
  history.push({
    board: board.map(row => row.map(x => x ? {...x} : null)),
    turn
  });

  const captured = board[r2][c2];
  board[r2][c2] = board[r1][c1];
  board[r1][c1] = null;

  selected = null;
  legalMoves = [];

  if (captured && captured.type.toLowerCase()==="k") {
    gameOver = true;
    render();
    document.getElementById("status").textContent = (turn===RED ? "紅方" : "黑方") + "獲勝！";
    document.getElementById("message").textContent = "對方的將帥已被吃掉，遊戲結束。";
    return true;
  }

  turn = turn===RED ? BLACK : RED;

  if (!hasAnyMove(turn)) {
    gameOver = true;
    render();
    document.getElementById("status").textContent = (turn===RED ? "紅方" : "黑方") + "無合法走法";
    document.getElementById("message").textContent = (turn===RED ? "黑方" : "紅方") + "獲勝！";
    return true;
  }

  document.getElementById("message").textContent = "走棋成功！";
  render();

  // 若為單人模式且輪到 AI (黑方)
  if (vsAI && turn === BLACK && !gameOver) {
    document.getElementById("message").textContent = "電腦思考中...";
    setTimeout(aiMakeMove, 300);
  }

  return false;
}

function clickCell(r,c) {
  if (gameOver) return;
  if (vsAI && turn === BLACK) return; // AI 思考時防止玩家點擊

  const p = board[r][c];

  if (selected) {
    const [r1,c1] = selected;
    const found = legalMoves.some(m => m[0]===r && m[1]===c);

    if (found) {
      makeMove(r1, c1, r, c);
      return;
    }

    if (p && p.color===turn) {
      selected = [r,c];
      legalMoves = getMoves(r,c);
    } else {
      selected = null;
      legalMoves = [];
    }
  } else if (p && p.color===turn) {
    selected = [r,c];
    legalMoves = getMoves(r,c);
  }

  render();
}

// 簡單 AI 思考邏輯（貪婪 + 評分法）
function aiMakeMove() {
  const moves = getAllLegalMoves(BLACK);
  if (moves.length === 0) return;

  let bestMove = null;
  let bestScore = -Infinity;

  for (const move of moves) {
    const [r1, c1] = move.from;
    const [r2, c2] = move.to;

    const moving = board[r1][c1];
    const target = board[r2][c2];

    let score = 0;

    // 1. 優先吃子價值評分
    if (target) {
      score += (PIECE_VALUES[target.type.toLowerCase()] || 0) * 10;
    }

    // 模擬移動進行評分
    board[r2][c2] = moving;
    board[r1][c1] = null;

    // 2. 將軍獎勵
    if (inCheck(RED)) {
      score += 50;
    }

    // 3. 防守評分（避開紅方攻擊）
    if (inCheck(BLACK)) {
      score -= 30;
    }

    // 復原棋盤
    board[r1][c1] = moving;
    board[r2][c2] = target;

    // 加入小隨機性避免重複盤局
    score += Math.random() * 5;

    if (score > bestScore) {
      bestScore = score;
      bestMove = move;
    }
  }

  if (bestMove) {
    makeMove(bestMove.from[0], bestMove.from[1], bestMove.to[0], bestMove.to[1]);
  }
}

function resetGame() {
  board = initialBoard();
  turn = RED;
  selected = null;
  legalMoves = [];
  history = [];
  gameOver = false;
  document.getElementById("message").textContent = vsAI ? "單人模式開局，請紅方先行。" : "雙人對弈開局，紅方先行。";
  render();
}

function undoMove() {
  if (history.length === 0) {
    document.getElementById("message").textContent = "目前沒有可以悔棋的步數。";
    return;
  }

  if (gameOver) gameOver = false;

  // 單人模式下悔棋需要回退 2 步（玩家 + AI）
  const steps = (vsAI && history.length >= 2) ? 2 : 1;
  let prev = null;
  for (let i = 0; i < steps; i++) {
    prev = history.pop();
  }

  if (prev) {
    board = prev.board;
    turn = prev.turn;
    selected = null;
    legalMoves = [];
    document.getElementById("message").textContent = "已悔棋，請重新走棋。";
    render();
  }
}

resetGame();
</script>
</body>
</html>
