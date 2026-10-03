
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>金科一甲象棋｜完整版</title>
<style>
:root{--wood:#f1d49a;--ink:#65401f;--paper:#fff8e9;--red:#bd2929;--black:#24211e}
*{box-sizing:border-box}
body{margin:0;padding:18px 10px 30px;text-align:center;font-family:"Microsoft JhengHei","PingFang TC",sans-serif;color:#422719;background:radial-gradient(circle at top,#fff4d9,#ead3ad 75%)}
h1{font-size:clamp(24px,5vw,34px);margin:8px 0 5px}
.subtitle{color:#765738;font-size:13px;margin-bottom:14px}
.panel{width:min(96vw,620px);margin:0 auto 12px;background:#fff9edc9;border:1px solid #d8b47a;border-radius:14px;padding:12px;box-shadow:0 5px 16px #5b35151a}
.mode{display:flex;justify-content:center;flex-wrap:wrap;gap:12px;font-weight:700}
.mode label{cursor:pointer}
#status{font-size:19px;font-weight:800;margin:8px}
.board-wrap{width:min(94vw,560px);margin:0 auto;padding:10px;border:4px solid #633b1c;border-radius:12px;background:linear-gradient(135deg,#dba85e,#f2d49a,#d7a15a);box-shadow:0 12px 28px #59371644}
#board{position:relative;width:100%;aspect-ratio:9/10;display:grid;grid-template-columns:repeat(9,1fr);grid-template-rows:repeat(10,1fr);background:var(--wood);border:1px solid #6b4829;overflow:hidden}
.cell{position:relative;display:flex;align-items:center;justify-content:center;min-width:0;min-height:0;cursor:pointer}
.cell:before{content:"";position:absolute;left:0;top:0;width:100%;height:100%;border-right:1px solid #7a512b;border-bottom:1px solid #7a512b;pointer-events:none}
.cell:nth-child(9n+1):after{content:"";position:absolute;left:0;top:0;height:100%;border-left:1px solid #7a512b;pointer-events:none}
.river-text{position:absolute;z-index:1;top:45%;left:0;width:100%;display:flex;justify-content:space-evenly;align-items:center;color:#80582e;font-family:serif;font-weight:bold;font-size:clamp(15px,3.5vw,23px);letter-spacing:3px;pointer-events:none;background:var(--wood)}
.palace{position:absolute;left:33.333%;width:33.333%;height:30%;pointer-events:none;z-index:1}
.palace.top{top:0}.palace.bottom{bottom:0}
.palace line{stroke:#7a512b;stroke-width:1.1}
.piece{position:relative;z-index:2;width:86%;height:auto;aspect-ratio:1;border-radius:50%;display:flex;align-items:center;justify-content:center;font-family:"DFKai-SB","標楷體",serif;font-size:clamp(17px,4.1vw,31px);font-weight:bold;background:radial-gradient(circle at 32% 25%,#fffbe8,#e8c98d 68%,#cda56a);border:2px solid currentColor;box-shadow:1px 3px 4px #0004,inset 0 0 0 2px #fff8;user-select:none}
.piece.red{color:var(--red)}
.piece.black{color:var(--black)}
.selected .piece{outline:3px solid #168bd2;outline-offset:1px;transform:scale(1.04)}
.target:after{content:"";position:absolute;width:23%;aspect-ratio:1;border-radius:50%;background:#208b51;opacity:.9;z-index:3;pointer-events:none}
.target.capture:after{width:82%;background:transparent;border:3px dashed #d52e2e}
.last-from .piece,.last-to .piece{box-shadow:0 0 0 3px #f5b942,1px 3px 4px #0004,inset 0 0 0 2px #fff8}
#message{min-height:24px;font-weight:700;margin:12px 0 4px}
.controls{display:flex;align-items:center;justify-content:center;flex-wrap:wrap;gap:5px;margin:8px auto}
button,select{font:inherit;font-size:14px;font-weight:700;border:0;border-radius:8px;padding:10px 13px;background:#67401f;color:white;cursor:pointer;box-shadow:0 3px 7px #0002}
button:hover{background:#89552a}
button:disabled,select:disabled{opacity:.55;cursor:wait}
.secondary{background:#8a6a47}
.meta{display:flex;justify-content:center;gap:18px;flex-wrap:wrap;font-size:13px;color:#725334;margin-top:8px}
.history{width:min(96vw,620px);margin:12px auto;background:var(--paper);border:1px solid #e1c99e;border-radius:10px;padding:12px;text-align:left;white-space:pre-wrap;max-height:190px;overflow:auto;font-size:14px;line-height:1.7}
.history b{color:#70411d}
.note{max-width:620px;margin:12px auto;font-size:12px;line-height:1.7;color:#765a3a}
@media(max-width:390px){.board-wrap{padding:6px;border-width:3px}.piece{border-width:1px}.controls button{padding:9px 10px}}
</style>
</head>
<body>
<h1>🏮 金科一甲象棋</h1>
<div class="subtitle">完整走棋判定・人機對弈・棋譜紀錄</div>

<div class="panel">
  <div class="mode">
    <label><input type="radio" name="mode" value="ai" checked> 單人對弈（紅方對電腦）</label>
    <label><input type="radio" name="mode" value="pvp"> 雙人同屏</label>
  </div>
  <div id="status">紅方先行</div>
  <div class="board-wrap">
    <div id="board">
      <svg class="palace top" viewBox="0 0 180 180" preserveAspectRatio="none">
        <line x1="0" y1="0" x2="180" y2="180"/>
        <line x1="180" y1="0" x2="0" y2="180"/>
      </svg>
      <svg class="palace bottom" viewBox="0 0 180 180" preserveAspectRatio="none">
        <line x1="0" y1="0" x2="180" y2="180"/>
        <line x1="180" y1="0" x2="0" y2="180"/>
      </svg>
      <div class="river-text"><span>楚 河</span><span>漢 界</span></div>
    </div>
  </div>
  <div id="message">紅方先行，點擊棋子查看合法落點。</div>
  <div class="controls">
    <label>電腦難度
      <select id="difficulty">
        <option value="1">入門</option>
        <option value="2" selected>普通</option>
        <option value="3">困難</option>
        <option value="4">專家</option>
      </select>
    </label>
    <button id="restart">重新開始</button>
    <button id="undo">悔棋</button>
    <button id="hint">提示棋步</button>
    <button id="flip" class="secondary">翻轉棋盤</button>
    <button id="export" class="secondary">匯出棋譜</button>
  </div>
  <div class="meta">
    <span id="move-count">回合：0</span>
    <span id="think-info">AI：待命</span>
    <span id="check-info">局面：正常</span>
  </div>
</div>

<div class="history" id="history"><b>棋譜</b><br>尚未開始</div>
<div class="note">
說明：支援車、馬、炮、相／象、仕／士、帥／將、兵／卒的基本合法走法，
包含馬腿、象眼、炮架、過河、九宮、將帥照面與不得讓己方將帥受攻擊。
包含將軍、將死、困斃、三次重複局面及連續 120 半回合未吃子和棋判定。
長將、長捉等正式競賽裁判規則較複雜，本程式以一般休閒對弈規則處理。
</div>

<script>
'use strict';
const RED='red',BLACK='black';
const NAMES={K:'帥',A:'仕',E:'相',R:'俥',H:'傌',C:'炮',P:'兵',k:'將',a:'士',e:'象',r:'車',h:'馬',c:'砲',p:'卒'};
const VALUE={k:30000,r:1000,c:510,h:440,a:220,e:220,p:100};
const $=id=>document.getElementById(id),boardEl=$('board');
let board,turn,selected,legalMoves,moveLog,undoStack,gameOver,vsAI,thinking,flipped,positionCounts,halfMoves,lastMove,searchNodes;

function initialBoard(){
  const b=Array.from({length:10},()=>Array(9).fill(null));
  const back=['r','h','e','a','k','a','e','h','r'];
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
function clone(b){return b.map(row=>row.map(p=>p?{...p}:null))}
function inside(r,c){return r>=0&&r<10&&c>=0&&c<9}
function palace(r,c,color){return c>=3&&c<=5&&(color===RED?r>=7&&r<=9:r>=0&&r<=2)}
function pathCount(b,r1,c1,r2,c2){
  let n=0;
  if(r1===r2){
    const d=Math.sign(c2-c1);
    for(let c=c1+d;c!==c2;c+=d)if(b[r1][c])n++;
  }else if(c1===c2){
    const d=Math.sign(r2-r1);
    for(let r=r1+d;r!==r2;r+=d)if(b[r][c1])n++;
  }else return -1;
  return n;
}
function pseudo(b,r1,c1,r2,c2){
  if(!inside(r1,c1)||!inside(r2,c2)||(r1===r2&&c1===c2))return false;
  const p=b[r1][c1],t=b[r2][c2];
  if(!p||(t&&t.color===p.color))return false;
  const type=p.type.toLowerCase(),dr=r2-r1,dc=c2-c1;
  const ar=Math.abs(dr),ac=Math.abs(dc);
  const forward=p.color===RED?-1:1;
  if(type==='r')return(dr===0||dc===0)&&pathCount(b,r1,c1,r2,c2)===0;
  if(type==='c'){
    if(dr!==0&&dc!==0)return false;
    const n=pathCount(b,r1,c1,r2,c2);
    return t?n===1:n===0;
  }
  if(type==='h'){
    if(!((ar===2&&ac===1)||(ar===1&&ac===2)))return false;
    const lr=ar===2?r1+dr/2:r1;
    const lc=ac===2?c1+dc/2:c1;
    return !b[lr][lc];
  }
  if(type==='e')
    return ar===2&&ac===2&&!b[r1+dr/2][c1+dc/2]&&(p.color===RED?r2>=5:r2<=4);
  if(type==='a')return ar===1&&ac===1&&palace(r2,c2,p.color);
  if(type==='k'){
    if(c1===c2&&t&&t.type.toLowerCase()==='k'&&pathCount(b,r1,c1,r2,c2)===0)return true;
    return ar+ac===1&&palace(r2,c2,p.color);
  }
  if(type==='p'){
    if(dc===0&&dr===forward)return true;
    const crossed=p.color===RED?r1<=4:r1>=5;
    return crossed&&dr===0&&ac===1;
  }
  return false;
}
function findKing(b,color){
  for(let r=0;r<10;r++)for(let c=0;c<9;c++){
    const p=b[r][c];
    if(p&&p.color===color&&p.type.toLowerCase()==='k')return[r,c];
  }
  return null;
}
function attacked(b,r,c,by){
  for(let y=0;y<10;y++)for(let x=0;x<9;x++){
    const p=b[y][x];
    if(p&&p.color===by&&pseudo(b,y,x,r,c))return true;
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
  for(let r=0;r<10;r++)for(let c=0;c<9;c++){
    const p=b[r][c];
    if(!p||p.color!==color)continue;
    for(let tr=0;tr<10;tr++)for(let tc=0;tc<9;tc++){
      if(!pseudo(b,r,c,tr,tc))continue;
      const target=b[tr][tc];
      if(target&&target.type.toLowerCase()==='k')continue;
      const m={fr:r,fc:c,tr,tc};
      const cap=apply(b,m);
      const ok=!inCheck(b,color);
      unapply(b,m,cap);
      if(ok)out.push(m);
    }
  }
  return out;
}
function posKey(b=board,s=turn){
  let k=s+':';
  for(const row of b)for(const p of row)k+=p?p.color[0]+p.type.toLowerCase():'.';
  return k;
}
function notation(m,p,cap){
  const file=c=>String.fromCharCode(65+c);
  return(p.color===RED?'紅':'黑')+NAMES[p.type]+' '+file(m.fc)+(10-m.fr)+'→'+file(m.tc)+(10-m.tr)+(cap?' 吃'+NAMES[cap.type]:'');
}
function render(){
  boardEl.querySelectorAll('.cell').forEach(x=>x.remove());
  for(let vr=0;vr<10;vr++)for(let vc=0;vc<9;vc++){
    const r=flipped?9-vr:vr,c=flipped?8-vc:vc;
    const cell=document.createElement('div');
    cell.className='cell';
    cell.style.gridRow=String(vr+1);
    cell.style.gridColumn=String(vc+1);
    const p=board[r][c];
    if(p){
      const el=document.createElement('div');
      el.className='piece '+p.color;
      el.textContent=NAMES[p.type];
      cell.appendChild(el);
    }
    if(selected&&selected[0]===r&&selected[1]===c)cell.classList.add('selected');
    if(legalMoves.some(m=>m[0]===r&&m[1]===c)){
      cell.classList.add('target');
      if(p)cell.classList.add('capture');
    }
    if(lastMove){
      if(lastMove.fr===r&&lastMove.fc===c)cell.classList.add('last-from');
      if(lastMove.tr===r&&lastMove.tc===c)cell.classList.add('last-to');
    }
    cell.addEventListener('click',()=>clickCell(r,c));
    boardEl.appendChild(cell);
  }
  if(!gameOver)$('status').textContent=(turn===RED?'紅方':'黑方')+'回合'+(inCheck(board,turn)?'（將軍！）':'')+(thinking?' · 電腦思考中':'');
  $('move-count').textContent='回合：'+Math.ceil(moveLog.length/2);
  $('check-info').textContent='局面：'+(inCheck(board,turn)?'被將軍':'正常');
  $('history').innerHTML='<b>棋譜</b><br>'+(moveLog.length?moveLog.map((m,i)=>(i%2===0?Math.floor(i/2)+1+'. ':'')+escapeHTML(m)+(i%2===0?'　':'<br>')).join(''):'尚未開始');
}
function escapeHTML(s){
  return s.replace(/[&<>"']/g,ch=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[ch]));
}
function setMessage(s){$('message').textContent=s}
function finishIfNeeded(){
  const moves=allLegal(board,turn);
  if(!moves.length){
    gameOver=true;
    $('status').textContent=inCheck(board,turn)?(turn===RED?'將死！黑方獲勝':'將死！紅方獲勝'):(turn===RED?'紅方困斃，黑方獲勝':'黑方困斃，紅方獲勝');
    setMessage('本局結束，可重新開始或悔棋。');
    return true;
  }
  const key=posKey();
  if((positionCounts.get(key)||0)>=3){
    gameOver=true;
    $('status').textContent='三次重複局面：和棋';
    setMessage('局面重複三次，本局判和。');
    return true;
  }
  if(halfMoves>=120){
    gameOver=true;
    $('status').textContent='連續 120 半回合未吃子：和棋';
    setMessage('連續 120 半回合未吃子，本局判和。');
    return true;
  }
  return false;
}
function saveUndo(){
  undoStack.push({
    board:clone(board),turn,halfMoves,counts:new Map(positionCounts),
    lastMove:lastMove?{...lastMove}:null,gameOver,moveLog:moveLog.slice()
  });
}
function makeMove(m){
  const p=board[m.fr][m.fc],cap=board[m.tr][m.tc];
  saveUndo();
  moveLog.push(notation(m,p,cap));
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
  if(!ended&&vsAI&&turn===BLACK)aiMakeMove();
  else if(!ended)setMessage(inCheck(board,turn)?'將軍！請處理將帥危機。':'走棋成功。');
}
function clickCell(r,c){
  if(gameOver||thinking||(vsAI&&turn===BLACK))return;
  const p=board[r][c];
  if(selected&&legalMoves.some(m=>m[0]===r&&m[1]===c)){
    const m=allLegal(board,turn).find(m=>m.fr===selected[0]&&m.fc===selected[1]&&m.tr===r&&m.tc===c);
    if(m){makeMove(m);return}
  }
  if(p&&p.color===turn){
    selected=[r,c];
    legalMoves=allLegal(board,turn).filter(m=>m.fr===r&&m.fc===c).map(m=>[m.tr,m.tc]);
    setMessage('已選擇棋子，綠點為合法落點。');
  }else{
    selected=null;
    legalMoves=[];
  }
  render();
}
function evaluate(b){
  let score=0;
  for(let r=0;r<10;r++)for(let c=0;c<9;c++){
    const p=b[r][c];
    if(!p)continue;
    const t=p.type.toLowerCase();
    let v=VALUE[t]||0;
    const advance=p.color===RED?9-r:r;
    if(t==='p'){
      v+=advance*7;
      if(p.color===RED?r<=4:r>=5)v+=38;
      if(c>=2&&c<=6)v+=8;
    }
    if(t==='h'||t==='c'||t==='r'){
      v+=Math.max(0,4-Math.abs(4-c))*3;
      if(t==='h'&&advance>=3)v+=12;
    }
    if(t==='a'||t==='e')v+=10;
    score+=p.color===BLACK?v:-v;
  }
  return score;
}
let transposition=new Map();
const INF=1e9,MATE=900000;
function moveOrder(b,ms){
  return ms.sort((a,z)=>{
    const score=m=>{
      const cap=b[m.tr][m.tc],att=b[m.fr][m.fc];
      return(cap?10*VALUE[cap.type.toLowerCase()]-VALUE[att.type.toLowerCase()]:0)+(att&&att.type.toLowerCase()==='p'?5:0);
    };
    return score(z)-score(a);
  });
}
function search(b,side,depth,alpha,beta,ply,path,deadline){
  searchNodes++;
  if((searchNodes&511)===0&&performance.now()>deadline)throw new Error('timeout');
  if(depth===0)return evaluate(b);
  const key=posKey(b,side);
  if(path.has(key))return 0;
  const cached=transposition.get(key);
  if(cached&&cached.depth>=depth){
    if(cached.flag==='exact')return cached.score;
    if(cached.flag==='lower')alpha=Math.max(alpha,cached.score);
    if(cached.flag==='upper')beta=Math.min(beta,cached.score);
    if(alpha>=beta)return cached.score;
  }
  const ms=moveOrder(b,allLegal(b,side));
  if(!ms.length)return side===BLACK?-MATE+ply:MATE-ply;
  const oldA=alpha,oldB=beta;
  let best=side===BLACK?-INF:INF;
  path.add(key);
  for(const m of ms){
    const cap=apply(b,m);
    let value;
    try{
      value=search(b,side===BLACK?RED:BLACK,depth-1,alpha,beta,ply+1,path,deadline);
    }finally{
      unapply(b,m,cap);
    }
    if(side===BLACK){
      if(value>best)best=value;
      if(best>alpha)alpha=best;
    }else{
      if(value<best)best=value;
      if(best<beta)beta=best;
    }
    if(alpha>=beta)break;
  }
  path.delete(key);
  transposition.set(key,{depth,score:best,flag:best<=oldA?'upper':best>=oldB?'lower':'exact'});
  if(transposition.size>60000)transposition.clear();
  return best;
}
function bestMove(side,maxDepth){
  const start=performance.now(),deadline=start+1700;
  searchNodes=0;
  transposition.clear();
  const moves=moveOrder(board,allLegal(board,side));
  if(!moves.length)return null;
  let bestMove=moves[0],completed=0;
  for(let depth=1;depth<=maxDepth;depth++){
    let best=side===BLACK?-INF:INF,choice=bestMove;
    try{
      for(const m of moves){
        const cap=apply(board,m);
        let value;
        try{
          value=search(board,side===BLACK?RED:BLACK,depth-1,-INF,INF,1,new Set(),deadline);
        }finally{
          unapply(board,m,cap);
        }
        if((side===BLACK&&value>best)||(side===RED&&value<best)){
          best=value;
          choice=m;
        }
      }
      bestMove=choice;
      completed=depth;
    }catch(e){
      if(e.message!=='timeout')throw e;
      break;
    }
  }
  $('think-info').textContent=`AI：深度 ${completed}／${searchNodes.toLocaleString()} 節點`;
  return bestMove;
}
function lockControls(lock){
  document.querySelectorAll('button,select,input').forEach(el=>el.disabled=lock);
}
function aiMakeMove(){
  if(gameOver||turn!==BLACK)return;
  thinking=true;
  lockControls(true);
  render();
  setTimeout(()=>{
    try{
      const m=bestMove(BLACK,Number($('difficulty').value)+1);
      thinking=false;
      lockControls(false);
      if(m)makeMove(m);
      else{
        gameOver=true;
        finishIfNeeded();
        render();
      }
    }catch(e){
      thinking=false;
      lockControls(false);
      setMessage('AI 發生錯誤，請重新開始。');
      console.error(e);
    }
  },30);
}
function resetGame(){
  board=initialBoard();
  turn=RED;
  selected=null;
  legalMoves=[];
  moveLog=[];
  undoStack=[];
  gameOver=false;
  thinking=false;
  flipped=false;
  positionCounts=new Map();
  halfMoves=0;
  lastMove=null;
  positionCounts.set(posKey(),1);
  transposition.clear();
  $('think-info').textContent='AI：待命';
  setMessage(vsAI?'單人模式：紅方先行。':'雙人模式：紅方先行。');
  render();
}
function undoMove(){
  if(thinking){
    setMessage('請等電腦完成思考後再悔棋。');
    return;
  }
  if(!undoStack.length){
    setMessage('目前沒有可悔棋的步數。');
    return;
  }
  let n=vsAI&&turn===RED?2:1;
  while(n-->0&&undoStack.length){
    const s=undoStack.pop();
    board=s.board;
    turn=s.turn;
    halfMoves=s.halfMoves;
    positionCounts=s.counts;
    lastMove=s.lastMove;
    gameOver=s.gameOver;
    moveLog=s.moveLog;
  }
  selected=null;
  legalMoves=[];
  setMessage('已悔棋。');
  render();
}
function hint(){
  if(gameOver||thinking||(vsAI&&turn===BLACK))return;
  thinking=true;
  lockControls(true);
  setMessage('正在分析最佳棋步…');
  setTimeout(()=>{
    const m=bestMove(turn,Math.max(2,Number($('difficulty').value)));
    thinking=false;
    lockControls(false);
    if(m){
      selected=[m.fr,m.fc];
      legalMoves=[[m.tr,m.tc]];
      setMessage('提示：將選取棋子移至綠點。');
      render();
    }
  },30);
}
function exportMoves(){
  const text='金科一甲象棋棋譜\n'+(moveLog.length?moveLog.map((m,i)=>(i%2===0?Math.floor(i/2)+1+'. ':'')+m).join('\n'):'尚無棋步');
  const blob=new Blob([text],{type:'text/plain;charset=utf-8'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  a.href=url;
  a.download='金科一甲象棋棋譜.txt';
  a.click();
  URL.revokeObjectURL(url);
}
document.querySelectorAll('input[name="mode"]').forEach(el=>el.addEventListener('change',e=>{
  vsAI=e.target.value==='ai';
  resetGame();
}));
$('restart').addEventListener('click',resetGame);
$('undo').addEventListener('click',undoMove);
$('hint').addEventListener('click',hint);
$('flip').addEventListener('click',()=>{
  flipped=!flipped;
  render();
});
$('export').addEventListener('click',exportMoves);
vsAI=true;
resetGame();
</script>
</body>
</html>
