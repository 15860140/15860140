
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>金科一甲象棋</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;padding:20px 10px;text-align:center;
  font-family:"Kaiti TC","KaiTi","標楷體","Microsoft JhengHei",sans-serif;
  background:#f0e2c8;color:#422719
}
h1{margin:8px 0;font-size:28px}
#status{font-size:20px;margin:12px;font-weight:bold}
.board-container{
  width:min(92vw,540px);margin:0 auto;padding:12px;
  background:#e6c587;border:4px solid #5a3518;border-radius:8px;
  box-shadow:0 8px 20px #0004
}
#board{
  width:100%;aspect-ratio:9/10;position:relative;
  background:#f1d49a;border:2px solid #5a3518;
  display:grid;grid-template-columns:repeat(9,1fr);
  grid-template-rows:repeat(10,1fr)
}
.cell{
  position:relative;display:flex;align-items:center;
  justify-content:center;min-width:0;min-height:0
}
.cell::before{
  content:"";position:absolute;inset:0;
  border-right:1px solid #754b2a;
  border-bottom:1px solid #754b2a;pointer-events:none
}
.cell:nth-child(9n+1)::after{
  content:"";position:absolute;height:100%;left:0;top:0;
  border-left:1px solid #754b2a;pointer-events:none
}
.river-cell::before{border-right:none}
.river-text{
  position:absolute;inset:40% 0 0;display:flex;
  justify-content:space-around;align-items:center;
  font-size:clamp(16px,3.5vw,22px);font-weight:bold;
  color:#754b2a;pointer-events:none;letter-spacing:4px;opacity:.85
}
.palace-svg{
  position:absolute;width:33.333%;height:30%;left:33.333%;
  pointer-events:none;z-index:1
}
.palace-svg.top{top:0}
.palace-svg.bottom{bottom:0}
.palace-svg line{stroke:#754b2a;stroke-width:1}
.piece{
  position:relative;z-index:2;width:86%;aspect-ratio:1;
  border-radius:50%;display:flex;justify-content:center;
  align-items:center;font-size:clamp(18px,4.2vw,32px);
  font-weight:bold;
  background:radial-gradient(circle at 35% 35%,#fff6dd,#ebd1a0);
  border:2px solid currentColor;box-shadow:2px 3px 5px #0005;
  cursor:pointer;user-select:none
}
.red{color:#c62828}
.black{color:#1a1a1a}
.selected .piece{outline:3px solid #36a9e1;outline-offset:1px}
.target::after{
  content:"";position:absolute;width:25%;aspect-ratio:1;
  border-radius:50%;background:#2e9d55;opacity:.9;
  z-index:3;pointer-events:none
}
.target.capture::after{
  width:84%;background:transparent;border:3px dashed #e33b32
}
.mode-select{margin:10px 0;font-size:16px;font-weight:bold}
.mode-select label{margin:0 10px;cursor:pointer}
.controls{
  margin-top:15px;display:flex;justify-content:center;
  flex-wrap:wrap;gap:6px
}
button,select{
  margin:4px;padding:10px 16px;border:0;border-radius:6px;
  background:#5a3518;color:white;font-size:15px;font-weight:bold;
  cursor:pointer;box-shadow:0 4px 6px #0003
}
button:hover{background:#824f26}
button:disabled,select:disabled{opacity:.55;cursor:wait}
#message{min-height:24px;margin:10px;font-weight:bold}
.hint{font-size:14px;color:#6b4d32}
.history{
  max-width:540px;margin:12px auto;padding:10px;
  background:#fff8e9;border-radius:6px;text-align:left;
  white-space:pre-wrap;max-height:180px;overflow:auto;font-size:14px
}
.small{
  font-size:12px;color:#755b41;max-width:560px;
  margin:10px auto;line-height:1.6
}
</style>
</head>
<body>

<h1>🏮 金科一甲象棋</h1>

<div class="mode-select">
  <label><input type="radio" name="gameMode" value="ai" checked>
    單人對弈（人機大戰）</label>
  <label><input type="radio" name="gameMode" value="pvp">
    雙人同屏對弈</label>
</div>

<div id="status">紅方先行</div>

<div class="board-container">
  <div id="board">
    <svg class="palace-svg top" viewBox="0 0 180 180" preserveAspectRatio="none">
      <line x1="0" y1="0" x2="180" y2="180"/>
      <line x1="180" y1="0" x2="0" y2="180"/>
    </svg>
    <svg class="palace-svg bottom" viewBox="0 0 180 180" preserveAspectRatio="none">
      <line x1="0" y1="0" x2="180" y2="180"/>
      <line x1="180" y1="0" x2="0" y2="180"/>
    </svg>
    <div class="river-text"><span>楚 河</span><span>漢 界</span></div>
  </div>
</div>

<div id="message">點擊紅方棋子開始走棋。</div>

<div class="controls">
  <label>電腦難度
    <select id="difficulty">
      <option value="2">簡單</option>
      <option value="3" selected>普通</option>
      <option value="4">困難</option>
      <option value="5">專家</option>
    </select>
  </label>
  <button id="restart">重新開始</button>
  <button id="undo">悔棋</button>
  <button id="hint">提示最佳棋步</button>
</div>

<div class="history" id="history">棋譜尚未開始</div>

<div class="small">
  支援基本合法走棋、將帥照面限制、將軍安全檢查、
  將死／困斃判定、簡化重複局面和棋判定，以及 Alpha-Beta 搜尋。
  搜尋深度會受裝置效能影響；長將、長捉等正式競賽裁定尚未完整涵蓋。
</div>

<script>
'use strict';

const RED = 'red', BLACK = 'black';
const NAMES = {
  K:'帥', A:'仕', E:'相', R:'俥', H:'傌', C:'炮', P:'兵',
  k:'將', a:'士', e:'象', r:'車', h:'馬', c:'砲', p:'卒'
};
const VALUES = {k:30000,r:1000,c:500,h:430,a:220,e:220,p:100};
const boardEl = document.getElementById('board');

let board, turn, selected, legalMoves, history, gameOver,
    vsAI, thinking, flipped, positionCounts, halfMoves, lastMove;

function initialBoard(){
  const b = Array.from({length:10},()=>Array(9).fill(null));
  const back = ['r','h','e','a','k','a','e','h','r'];

  for(let c=0;c<9;c++){
    b[0][c]={type:back[c],color:BLACK};
    b[9][c]={type:back[c].toUpperCase(),color:RED};
  }

  b[2][1]={type:'c',color:BLACK};
  b[2][7]={type:'c',color:BLACK};
  b[7][1]={type:'C',color:RED};
  b[7][7]={type:'C',color:RED};

  for(const c of [0,2,4,6,8]){
    b[3][c]={type:'p',color:BLACK};
    b[6][c]={type:'P',color:RED};
  }
  return b;
}

function clone(b){
  return b.map(row=>row.map(p=>p?{...p}:null));
}

function inside(r,c){
  return r>=0&&r<10&&c>=0&&c<9;
}

function palace(r,c,color){
  return c>=3&&c<=5&&
    (color===RED?r>=7&&r<=9:r>=0&&r<=2);
}

function pathCount(b,r1,c1,r2,c2){
  let n=0;
  if(r1===r2){
    const d=Math.sign(c2-c1);
    for(let c=c1+d;c!==c2;c+=d) if(b[r1][c]) n++;
  }else if(c1===c2){
    const d=Math.sign(r2-r1);
    for(let r=r1+d;r!==r2;r+=d) if(b[r][c1]) n++;
  }else{
    return -1;
  }
  return n;
}

function pseudo(b,r1,c1,r2,c2){
  if(!inside(r1,c1)||!inside(r2,c2)||
     (r1===r2&&c1===c2)) return false;

  const p=b[r1][c1], t=b[r2][c2];
  if(!p||(t&&t.color===p.color)) return false;

  const type=p.type.toLowerCase();
  const dr=r2-r1, dc=c2-c1;
  const ar=Math.abs(dr), ac=Math.abs(dc);
  const forward=p.color===RED?-1:1;

  // 車：直線移動，中間不可有棋子
  if(type==='r')
    return (dr===0||dc===0)&&pathCount(b,r1,c1,r2,c2)===0;

  // 炮：不吃子時不能隔子；吃子時必須隔一子
  if(type==='c'){
    if(dr!==0&&dc!==0) return false;
    const n=pathCount(b,r1,c1,r2,c2);
    return t?n===1:n===0;
  }

  // 馬：日字移動，檢查馬腿
  if(type==='h'){
    if(!((ar===2&&ac===1)||(ar===1&&ac===2))) return false;
    const lr=ar===2?r1+dr/2:r1;
    const lc=ac===2?c1+dc/2:c1;
    return !b[lr][lc];
  }

  // 象：田字移動，檢查象眼及過河限制
  if(type==='e'){
    return ar===2&&ac===2&&
      !b[r1+dr/2][c1+dc/2]&&
      (p.color===RED?r2>=5:r2<=4);
  }

  // 士：斜走一格，不可離開九宮
  if(type==='a')
    return ar===1&&ac===1&&palace(r2,c2,p.color);

  // 將帥：九宮內走一格；可沿直線吃對方將帥
  if(type==='k'){
    if(c1===c2&&t&&t.type.toLowerCase()==='k'&&
       pathCount(b,r1,c1,r2,c2)===0) return true;
    return ar+ac===1&&palace(r2,c2,p.color);
  }

  // 兵卒：前進一格，過河後可以左右移動
  if(type==='p'){
    if(dc===0&&dr===forward) return true;
    const crossed=p.color===RED?r1<=4:r1>=5;
    return crossed&&dr===0&&ac===1;
  }

  return false;
}

function findKing(b,color){
  for(let r=0;r<10;r++){
    for(let c=0;c<9;c++){
      const p=b[r][c];
      if(p&&p.color===color&&p.type.toLowerCase()==='k')
        return [r,c];
    }
  }
  return null;
}

function attacked(b,r,c,by){
  for(let y=0;y<10;y++){
    for(let x=0;x<9;x++){
      const p=b[y][x];
      if(p&&p.color===by&&pseudo(b,y,x,r,c)) return true;
    }
  }
  return false;
}

function inCheck(b,color){
  const k=findKing(b,color);
  return !k||attacked(b,k[0],k[1],color===RED?BLACK:RED);
}

function apply(b,m){
  const captured=b[m.tr][m.tc];
  b[m.tr][m.tc]=b[m.fr][m.fc];
  b[m.fr][m.fc]=null;
  return captured;
}

function unapply(b,m,captured){
  b[m.fr][m.fc]=b[m.tr][m.tc];
  b[m.tr][m.tc]=captured||null;
}

function allLegal(b,color){
  const out=[];
  for(let r=0;r<10;r++){
    for(let c=0;c<9;c++){
      const p=b[r][c];
      if(!p||p.color!==color) continue;

      for(let tr=0;tr<10;tr++){
        for(let tc=0;tc<9;tc++){
          if(!pseudo(b,r,c,tr,tc)) continue;
          const m={fr:r,fc:c,tr,tc};
          const cap=apply(b,m);
          const ok=!inCheck(b,color);
          unapply(b,m,cap);
          if(ok) out.push(m);
        }
      }
    }
  }
  return out;
}

function movesFrom(r,c){
  return allLegal(board,turn)
    .filter(m=>m.fr===r&&m.fc===c)
    .map(m=>[m.tr,m.tc]);
}

function posKey(b=board,s=turn){
  let k=s+':';
  for(const row of b){
    for(const p of row){
      k+=p?p.color[0]+p.type.toLowerCase():'.';
    }
  }
  return k;
}

function moveText(m,p,cap){
  return (p.color===RED?'紅':'黑')+NAMES[p.type]+' '+
    String.fromCharCode(A()+m.fc)+(10-m.fr)+'→'+
    String.fromCharCode(A()+m.tc)+(10-m.tr)+
    (cap?' 吃'+NAMES[cap.type]:'');
}

function A(){return 65}

function render(){
  boardEl.querySelectorAll('.cell').forEach(x=>x.remove());

  for(let r=0;r<10;r++){
    for(let c=0;c<9;c++){
      const cell=document.createElement('div');
      cell.className='cell'+(r===4?' river-cell':'');
      cell.style.gridRow=String(r+1);
      cell.style.gridColumn=String(c+1);

      const p=board[r][c];
      if(p){
        const el=document.createElement('div');
        el.className='piece '+p.color;
        el.textContent=NAMES[p.type];
        cell.appendChild(el);
      }

      if(selected&&selected[0]===r&&selected[1]===c)
        cell.classList.add('selected');

      if(legalMoves.some(m=>m[0]===r&&m[1]===c)){
        cell.classList.add('target');
        if(p) cell.classList.add('capture');
      }

      cell.addEventListener('click',()=>clickCell(r,c));
      boardEl.appendChild(cell);
    }
  }

  const status=document.getElementById('status');
  if(!gameOver){
    status.textContent=(turn===RED?'紅方':'黑方')+'回合'+
      (inCheck(board,turn)?'（將軍！）':'')+
      (thinking?' · 電腦思考中':'');
  }

  document.getElementById('history').textContent=
    history.length
      ?history.map((x,i)=>`${Math.floor(i/2)+1}. ${x}`).join('\n')
      :'棋譜尚未開始';
}

function setMessage(s){
  document.getElementById('message').textContent=s;
}

function finishIfNeeded(){
  const moves=allLegal(board,turn);

  if(!moves.length){
    gameOver=true;
    document.getElementById('status').textContent=
      inCheck(board,turn)
        ?(turn===RED?'將死！黑方獲勝':'將死！紅方獲勝')
        :(turn===RED?'紅方困斃，黑方獲勝':'黑方困斃，紅方獲勝');
    setMessage('本局結束。可重新開始或悔棋。');
    return true;
  }

  const k=posKey();
  if((positionCounts.get(k)||0)>=3){
    gameOver=true;
    document.getElementById('status').textContent=
      '三次重複局面：和棋（簡化判定）';
    setMessage('本局和棋。');
    return true;
  }

  if(halfMoves>=120){
    gameOver=true;
    document.getElementById('status').textContent=
      '連續 120 半回合未吃子：和棋';
    setMessage('本局和棋。');
    return true;
  }

  return false;
}

function makeMove(m){
  const p=board[m.fr][m.fc],cap=board[m.tr][m.tc];

  history.push({
    board:clone(board),
    turn,
    halfMoves,
    counts:new Map(positionCounts),
    lastMove:lastMove?{...lastMove}:null,
    gameOver
  });

  board[m.tr][m.tc]=p;
  board[m.fr][m.fc]=null;
  lastMove={...m};
  halfMoves=cap?0:halfMoves+1;
  turn=turn===RED?BLACK:RED;

  positionCounts.set(posKey(),(positionCounts.get(posKey())||0)+1);
  selected=null;
  legalMoves=[];

  const ended=finishIfNeeded();
  render();

  if(!ended&&vsAI&&turn===BLACK){
    aiMakeMove();
  }else if(!ended){
    setMessage('走棋成功。');
  }
}

function clickCell(r,c){
  if(gameOver||thinking||(vsAI&&turn===BLACK)) return;

  const p=board[r][c];

  if(selected&&legalMoves.some(m=>m[0]===r&&m[1]===c)){
    const m=allLegal(board,turn).find(m=>
      m.fr===selected[0]&&m.fc===selected[1]&&
      m.tr===r&&m.tc===c
    );
    if(m){
      makeMove(m);
      return;
    }
  }

  if(p&&p.color===turn){
    selected=[r,c];
    legalMoves=movesFrom(r,c);
    setMessage('已選擇棋子，綠點為合法落點。');
  }else{
    selected=null;
    legalMoves=[];
  }

  render();
}

function evaluate(b){
  let score=0;

  for(let r=0;r<10;r++){
    for(let c=0;c<9;c++){
      const p=b[r][c];
      if(!p) continue;

      const t=p.type.toLowerCase();
      let v=VALUES[t]||0;

      if(t==='p'){
        v+=(p.color===RED?9-r:r)*7;
        if(p.color===RED?r<=4:r>=5) v+=35;
      }

      if(['r','c','h'].includes(t)){
        v+=Math.max(0,4-Math.abs(4-c))*2;
      }

      score+=p.color===BLACK?v:-v;
    }
  }

  return score;
}

let nodes=0,table=new Map();
const INF=1e9,MATE=900000;

function ordered(b,ms){
  return ms.sort((a,z)=>{
    const av=b[a.tr][a.tc]
      ?VALUES[b[a.tr][a.tc].type.toLowerCase()]*10:0;
    const zv=b[z.tr][z.tc]
      ?VALUES[b[z.tr][z.tc].type.toLowerCase()]*10:0;
    return zv-av;
  });
}

function search(b,side,depth,alpha,beta,ply,path){
  nodes++;
  if(depth===0) return evaluate(b);

  const k=posKey(b,side);
  if(path.has(k)) return 0;

  const cached=table.get(k);
  if(cached&&cached.depth>=depth){
    if(cached.flag==='exact') return cached.score;
    if(cached.flag==='lower') alpha=Math.max(alpha,cached.score);
    if(cached.flag==='upper') beta=Math.min(beta,cached.score);
    if(alpha>=beta) return cached.score;
  }

  const ms=ordered(b,allLegal(b,side));
  if(!ms.length) return side===BLACK?-MATE+ply:MATE-ply;

  const oldAlpha=alpha,oldBeta=beta;
  let best=side===BLACK?-INF:INF;

  path.add(k);

  for(const m of ms){
    const cap=apply(b,m);
    const v=search(
      b,side===BLACK?RED:BLACK,depth-1,
      alpha,beta,ply+1,path
    );
    unapply(b,m,cap);

    if(side===BLACK){
      if(v>best) best=v;
      if(best>alpha) alpha=best;
    }else{
      if(v<best) best=v;
      if(best<beta) beta=best;
    }

    if(alpha>=beta) break;
  }

  path.delete(k);

  table.set(k,{
    depth,
    score:best,
    flag:best<=oldAlpha?'upper':best>=oldBeta?'lower':'exact'
  });

  if(table.size>70000) table.clear();
  return best;
}

function bestMove(side,depth){
  nodes=0;
  table.clear();

  const start=performance.now();
  const ms=ordered(board,allLegal(board,side));
  let choice=null;
  let best=side===BLACK?-INF:INF;

  for(const m of ms){
    const cap=apply(board,m);
    const score=search(
      board,side===BLACK?RED:BLACK,
      depth-1,-INF,INF,1,new Set()
    );
    unapply(board,m,cap);

    if((side===BLACK&&score>best)||
       (side===RED&&score<best)){
      best=score;
      choice=m;
    }

    if(performance.now()-start>2200) break;
  }

  document.getElementById('message').textContent=
    `搜尋完成：${nodes.toLocaleString()} 個節點，`+
    `${Math.round(performance.now()-start)} ms`;

  return choice;
}

function aiMakeMove(){
  if(gameOver||turn!==BLACK) return;

  thinking=true;
  document.querySelectorAll('button,select,input')
    .forEach(e=>e.disabled=true);
  render();

  setTimeout(()=>{
    const m=bestMove(
      BLACK,
      Number(document.getElementById('difficulty').value)
    );

    thinking=false;
    document.querySelectorAll('button,select,input')
      .forEach(e=>e.disabled=false);

    if(m){
      makeMove(m);
    }else{
      gameOver=true;
      render();
    }
  },30);
}

function resetGame(){
  board=initialBoard();
  turn=RED;
  selected=null;
  legalMoves=[];
  history=[];
  gameOver=false;
  thinking=false;
  flipped=false;
  positionCounts=new Map();
  halfMoves=0;
  lastMove=null;

  positionCounts.set(posKey(),1);
  table.clear();

  setMessage(vsAI
    ?'單人模式開局，請紅方先行。'
    :'雙人對弈開局，紅方先行。'
  );

  render();
}

function undoMove(){
  if(thinking){
    setMessage('請等電腦完成搜尋後再悔棋。');
    return;
  }

  if(!history.length){
    setMessage('目前沒有可悔棋的步數。');
    return;
  }

  let n=vsAI&&turn===RED?2:1;

  while(n-->0&&history.length){
    const s=history.pop();
    board=s.board;
    turn=s.turn;
    halfMoves=s.halfMoves;
    positionCounts=s.counts;
    lastMove=s.lastMove;
    gameOver=s.gameOver;
  }

  selected=null;
  legalMoves=[];
  setMessage('已悔棋。');
  render();
}

function hint(){
  if(gameOver||thinking) return;
  if(vsAI&&turn===BLACK) return;

  thinking=true;
  document.querySelectorAll('button,select,input')
    .forEach(e=>e.disabled=true);

  setTimeout(()=>{
    const m=bestMove(
      turn,
      Math.max(2,Number(document.getElementById('difficulty').value)-1)
    );

    thinking=false;
    document.querySelectorAll('button,select,input')
      .forEach(e=>e.disabled=false);

    if(m){
      selected=[m.fr,m.fc];
      legalMoves=[[m.tr,m.tc]];
      setMessage('最佳棋步提示：請將選取棋子移至綠點。');
      render();
    }
  },30);
}

document.querySelectorAll('input[name="gameMode"]')
  .forEach(el=>el.addEventListener('change',e=>{
    vsAI=e.target.value==='ai';
    resetGame();
  }));

document.getElementById('restart').addEventListener('click',resetGame);
document.getElementById('undo').addEventListener('click',undoMove);
document.getElementById('hint').addEventListener('click',hint);

vsAI=true;
resetGame();
</script>
</body>
</html>
