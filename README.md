
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>金科一甲象棋｜完整版</title>
<style>
:root{--wood:#e8bd70;--wood-light:#f8dfa0;--ink:#6c421d;--paper:#fffaf0;--red:#d52b2b;--black:#292521}
*{box-sizing:border-box}
body{margin:0;padding:22px 10px 34px;text-align:center;font-family:"Microsoft JhengHei","PingFang TC",sans-serif;color:#432817;background:radial-gradient(ellipse at 50% 0%,#fff8e5 0%,#f0dfbf 48%,#d9bd91 100%);min-height:100vh}
h1{font-size:clamp(25px,5vw,36px);margin:8px 0 5px;letter-spacing:2px;text-shadow:0 1px #fff8}
.subtitle{color:#87603a;font-size:13px;margin-bottom:16px;letter-spacing:1px}
.panel{width:min(96vw,640px);margin:0 auto 14px;background:#fffaf0e8;border:1px solid #d5b47e;border-radius:18px;padding:14px;box-shadow:0 8px 24px #59371620,inset 0 1px #fff}
.mode{display:flex;justify-content:center;flex-wrap:wrap;gap:14px;font-weight:700}
.mode label{cursor:pointer}
#status{font-size:19px;font-weight:800;margin:8px;color:#60391d}
.board-wrap{width:min(94vw,560px);margin:0 auto;padding:10px;border:5px solid #6b3e1d;border-radius:8px;background:linear-gradient(135deg,#c78a42,#f2d08a 22%,#d39a4c 48%,#f4d28d 72%,#bb7938);box-shadow:0 12px 25px #4d2a1640,inset 0 0 0 2px #f7dfaa,inset 0 0 0 4px #8c5727}
#board{position:relative;width:100%;aspect-ratio:8/9;background-color:#efc66e;background-image:repeating-linear-gradient(to right,transparent 0,transparent calc(12.5% - 1px),#70471f calc(12.5% - 1px),#70471f 12.5%),repeating-linear-gradient(to bottom,transparent 0,transparent calc(11.111111% - 1px),#70471f calc(11.111111% - 1px),#70471f 11.111111%),repeating-linear-gradient(90deg,#9c652110 0,#9c652110 1px,transparent 1px,transparent 5px),linear-gradient(90deg,#e9b958,#f9db8a 48%,#e8b958);border:2px solid #71451e;overflow:visible;isolation:isolate}
.cell{position:absolute;left:0;top:0;width:12.5%;height:11.111%;transform:translate(-50%,-50%);display:flex;align-items:center;justify-content:center;cursor:pointer;z-index:3}
.river-text{position:absolute;z-index:2;top:44.444%;left:0;width:100%;height:11.112%;display:flex;justify-content:space-evenly;align-items:center;color:#6b3c14;font-family:"DFKai-SB","標楷體",serif;font-weight:bold;font-size:clamp(15px,3.7vw,24px);letter-spacing:4px;pointer-events:none;background:linear-gradient(90deg,#efc66eF2,#f7d989F7,#efc66eF2);text-shadow:0 1px #fff4}
.palace{position:absolute;left:37.5%;width:25%;height:22.222%;pointer-events:none;z-index:1}
.palace.top{top:0}.palace.bottom{bottom:0}
.palace line{stroke:#795027;stroke-width:1.2}
.piece{position:relative;z-index:5;width:min(10vw,56px);max-width:92%;height:auto;aspect-ratio:1;border-radius:50%;display:flex;align-items:center;justify-content:center;font-family:"DFKai-SB","標楷體","KaiTi",serif;font-size:clamp(17px,4.1vw,31px);font-weight:bold;background:radial-gradient(circle at 30% 22%,#fffef7 0%,#fff9e7 48%,#f1dfb7 72%,#c8a66a 100%);background-color:#fff5dc;opacity:1;border:2px solid #9a6a2e;box-shadow:0 3px 0 #603813,0 6px 9px #321a0d80,inset 0 0 0 2px #fff9e9,inset 0 -4px 6px #a77a3c70;user-select:none;transition:transform .12s,box-shadow .12s;isolation:isolate}
.piece.red{color:#c51f1f;border-color:#b66b35;text-shadow:0 1px #fff3}
.piece.black{color:#211b15;border-color:#9a6a2e}
.selected .piece{outline:3px solid #46c8ff;outline-offset:1px;transform:scale(1.06);z-index:8}
.target:after{content:"";position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:23%;aspect-ratio:1;border-radius:50%;background:#218b52;opacity:.95;z-index:6;pointer-events:none;box-shadow:0 0 0 2px #fff8}
.target.capture:after{width:min(10vw,56px);background:#fff2;border:3px dashed #d52e2e;box-shadow:0 0 0 1px #fff8;z-index:6}
.last-from .piece,.last-to .piece{box-shadow:0 0 0 3px #f6c449,0 6px 9px #321a0d80,inset 0 0 0 2px #fff8}
#message{min-height:24px;font-weight:700;margin:12px 0 4px;color:#ffe0a0}
.controls{display:flex;align-items:center;justify-content:center;flex-wrap:wrap;gap:7px;margin:10px auto}
button,select{font:inherit;font-size:14px;font-weight:700;border:1px solid #c28b3c;border-radius:9px;padding:10px 13px;background:linear-gradient(#6e451f,#301a0b);color:#fff1ce;cursor:pointer;box-shadow:0 3px 7px #0006;transition:filter .15s,transform .15s}
button:hover{filter:brightness(1.3);transform:translateY(-1px)}
button:disabled,select:disabled{opacity:.55;cursor:wait}
.secondary{background:linear-gradient(#a17d51,#795735)}
.meta{display:flex;justify-content:center;gap:18px;flex-wrap:wrap;font-size:13px;color:#e3c28c;margin-top:10px}
.history{width:min(96vw,640px);margin:14px auto;background:linear-gradient(145deg,#29180d,#120d08 70%,#34200f);border:2px solid #9c6a2b;border-radius:14px;padding:14px;text-align:left;white-space:pre-wrap;max-height:200px;overflow:auto;font-size:14px;line-height:1.85;box-shadow:0 10px 28px #0008;color:#f8e9c9}
.history b{color:#ffe0a0}
.note{max-width:640px;margin:14px auto;font-size:12px;line-height:1.8;color:#e3c28c}
.panel{width:min(96vw,760px);background:linear-gradient(145deg,#29180d,#120d08 70%,#34200f);border:2px solid #9c6a2b;border-radius:14px;box-shadow:0 10px 28px #0008,inset 0 0 0 1px #f4d28b35;color:#f8e9c9}
body{background:radial-gradient(ellipse at 50% 0%,#6b421f 0%,#2a180d 75%,#170e08 100%);color:#f8e5b5}
h1{color:#f9d77f;text-shadow:0 2px #321706,0 0 14px #d9a64b;letter-spacing:4px}
.subtitle{color:#e4c18a}
@media(min-width:900px){.board-wrap{width:min(94vw,620px)}.piece{width:min(6vw,60px)}}
@media(max-width:390px){body{padding:14px 5px 24px}.board-wrap{padding:6px;border-width:4px}.piece{width:9.5vw;font-size:clamp(16px,4.5vw,23px)}.controls button{padding:9px 10px}.panel{padding:10px}}
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
      <svg class="palace top" viewBox="0 0 90 90" preserveAspectRatio="none">
        <line x1="0" y1="0" x2="90" y2="90"/>
        <line x1="90" y1="0" x2="0" y2="90"/>
      </svg>
      <svg class="palace bottom" viewBox="0 0 90 90" preserveAspectRatio="none">
        <line x1="0" y1="0" x2="90" y2="90"/>
        <line x1="90" y1="0" x2="0" y2="90"/>
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
<div class="note">說明：支援車、馬、炮、相／象、仕／士、帥／將、兵／卒的基本合法走法，包含馬腿、象眼、炮架、過河、九宮、將帥照面與不得讓己方將帥受攻擊。棋譜採傳統記譜方式：紅方用中文數字、黑方用阿拉伯數字，記錄進、退、平及同線棋子的前後區分。包含將軍、將死、困斃、三次重複局面及連續 120 半回合未吃子和棋判定。長將、長捉等正式競賽裁判規則較複雜，本程式以一般休閒對弈規則處理。</div>

<script>
'use strict';
const RED='red',BLACK='black';
const NAMES={K:'帥',A:'仕',E:'相',R:'俥',H:'傌',C:'炮',P:'兵',k:'將',a:'士',e:'象',r:'車',h:'馬',c:'砲',p:'卒'};
const VALUE={k:30000,r:1000,c:510,h:440,a:220,e:220,p:100};
const $=id=>document.getElementById(id), boardEl=$('board');
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
  const type=p.type.toLowerCase(),dr=r2-r1,dc=c2-c1,ar=Math.abs(dr),ac=Math.abs(dc),forward=p.color===RED?-1:1;
  if(type==='r')return(dr===0||dc===0)&&pathCount(b,r1,c1,r2,c2)===0;
  if(type==='c'){
    if(dr!==0&&dc!==0)return false;
    const n=pathCount(b,r1,c1,r2,c2);
    return t?n===1:n===0;
  }
  if(type==='h'){
    if(!((ar===2&&ac===1)||(ar===1&&ac===2)))return false;
    const lr=ar===2?r1+dr/2:r1,lc=ac===2?c1+dc/2:c1;
    return !b[lr][lc];
  }
  if(type==='e')return ar===2&&ac===2&&!b[r1+dr/2][c1+dc/2]&&(p.color===RED?r2>=5:r2<=4);
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
  const nums=['','一','二','三','四','五','六','七','八','九'];
  const fileNum=c=>p.color===RED?9-c:c+1;
  const fmt=n=>p.color===RED?nums[n]:String(n);
  const type=p.type.toLowerCase();
  const forward=p.color===RED?-1:1;
  let pieceName=NAMES[p.type];
  const same=[];
  for(let r=0;r<10;r++){
    const q=board[r][m.fc];
    if(q&&q.color===p.color&&q.type.toLowerCase()===type)same.push({r,p:q});
  }
  if(same.length>1){
    same.sort((a,b)=>p.color===RED?a.r-b.r:b.r-a.r);
    const idx=same.findIndex(x=>x.r===m.fr);
    const labels=same.length===2?['前','後']:same.length===3?['前','中','後']:['前','前二','前一','後一','後二','後'];
    pieceName=(labels[idx]||('第'+(idx+1)))+pieceName;
  }
  const dr=m.tr-m.fr,dc=m.tc-m.fc;
  let action,argument;
  if(dc===0){
    action=dr*forward<0?'退':'進';
    if(['h','e','a'].includes(type))argument=fmt(fileNum(m.tc));
    else argument=String(Math.abs(dr));
  }else if(dr===0){
    action='平';
    argument=fmt(fileNum(m.tc));
  }else{
    action=dr*forward<0?'退':'進';
    argument=fmt(fileNum(m.tc));
  }
  return pieceName+fmt(fileNum(m.fc))+action+argument+(cap?'（吃'+NAMES[cap.type]+'）':'');
}
function render(){
  boardEl.querySelectorAll('.cell').forEach(x=>x.remove());
  for(let vr=0;vr<10;vr++)for(let vc=0;vc<9;vc++){
    const r=flipped?9-vr:vr,c=flipped?8-vc:vc;
    const cell=document.createElement('div');
    cell.className='cell';
    cell.style.left=(vc/8*100)+'%';
    cell.style.top=(vr/9*100)+'%';
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
  undoStack.push({board:clone(board),turn,halfMoves,counts:new Map(positionCounts),lastMove:lastMove?{...lastMove}:null,gameOver,moveLog:moveLog.slice()});
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
      return (cap?10*VALUE[cap.type.toLowerCase()]-VALUE[att.type.toLowerCase()]:0)+(att&&att.type.toLowerCase()==='p'?5:0);
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
