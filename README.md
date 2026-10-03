<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>象棋</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;padding:20px;
  font-family:Arial,"Microsoft JhengHei",sans-serif;
  background:#f4ead5;color:#3c2415;text-align:center
}
h1{margin:5px 0 15px}
.game{max-width:650px;margin:auto}
canvas{
  width:100%;max-width:540px;height:auto;
  background:#f4d99a;border:3px solid #75451f;
  border-radius:5px;touch-action:manipulation
}
.controls{
  display:flex;justify-content:center;flex-wrap:wrap;
  gap:8px;margin:12px 0
}
button,select{
  padding:9px 13px;border:1px solid #8b5a2b;
  border-radius:6px;background:#fff8e8;
  font-size:15px;cursor:pointer
}
button:hover{background:#ead1a1}
#status{font-weight:bold;margin:12px}
#history{
  text-align:left;background:#fff8e8;padding:12px;
  border-radius:8px;min-height:50px;white-space:pre-wrap
}
</style>
</head>
<body>
<div class="game">
<h1>🏮 象棋</h1>
<canvas id="board" width="540" height="600"></canvas>
<div id="status">紅方先行</div>

<div class="controls">
  <select id="difficulty">
    <option value="1">簡單</option>
    <option value="2" selected>普通</option>
    <option value="3">困難</option>
  </select>
  <button onclick="newGame()">重新開始</button>
  <button onclick="undo()">悔棋</button>
  <button onclick="flipBoard()">翻轉棋盤</button>
  <button onclick="hint()">提示</button>
</div>

<h3>棋譜紀錄</h3>
<div id="history">尚未開始</div>
</div>

<script>
const canvas=document.getElementById("board");
const ctx=canvas.getContext("2d");
const N=9, M=10, step=60, ox=30, oy=30;

const initial=[
 ["車","馬","象","士","將","士","象","馬","車"],
 ["","","","","","","","",""],
 ["","炮","","","","","","炮",""],
 ["卒","","卒","","卒","","卒","","卒"],
 ["","","","","","","","",""],
 ["","","","","","","","",""],
 ["兵","","兵","","兵","","兵","","兵"],
 ["","砲","","","","","","砲",""],
 ["","","","","","","","",""],
 ["俥","傌","相","仕","帥","仕","相","傌","俥"]
];

// 使用標準紅黑棋子配置
function startBoard(){
 const b=Array.from({length:10},()=>Array(9).fill(null));
 const back=["R","N","B","A","K","A","B","N","R"];
 for(let x=0;x<9;x++){
  b[0][x]={c:"b",t:back[x]};
  b[9][x]={c:"r",t:back[x]};
 }
 b[2][1]={c:"b",t:"C"}; b[2][7]={c:"b",t:"C"};
 b[7][1]={c:"r",t:"C"}; b[7][7]={c:"r",t:"C"};
 for(let x=0;x<9;x+=2){
  b[3][x]={c:"b",t:"P"};
  b[6][x]={c:"r",t:"P"};
 }
 return b;
}
const names={
 K:{r:"帥",b:"將"},A:{r:"仕",b:"士"},
 B:{r:"相",b:"象"},N:{r:"傌",b:"馬"},
 R:{r:"俥",b:"車"},C:{r:"砲",b:"炮"},
 P:{r:"兵",b:"卒"}
};
let board,turn,selected,legal,history,records,flipped,ended;

function newGame(){
 board=startBoard();turn="r";selected=null;legal=[];
 history=[];records=[];flipped=false;ended=false;
 draw();updateStatus("紅方先行");
 document.getElementById("history").textContent="尚未開始";
}
function draw(){
 ctx.clearRect(0,0,540,600);
 ctx.fillStyle="#f4d99a";ctx.fillRect(0,0,540,600);
 ctx.strokeStyle="#70421f";ctx.lineWidth=1.5;
 for(let y=0;y<10;y++){
  ctx.beginPath();ctx.moveTo(ox,oy+y*step);ctx.lineTo(ox+8*step,oy+y*step);ctx.stroke();
 }
 for(let x=0;x<9;x++){
  ctx.beginPath();
  ctx.moveTo(ox+x*step,oy);ctx.lineTo(ox+x*step,oy+9*step);ctx.stroke();
 }
 // 河界
 ctx.fillStyle="#70421f";ctx.font="20px serif";ctx.textAlign="center";
 ctx.fillText("楚 河",150,oy+4.7*step);
 ctx.fillText("漢 界",390,oy+4.7*step);

 if(selected){
  const p=screen(selected.x,selected.y);
  ctx.fillStyle="#f6e58d";ctx.beginPath();
  ctx.arc(p.x,p.y,25,0,Math.PI*2);ctx.fill();
 }
 for(const m of legal){
  const p=screen(m.x,m.y);
  ctx.fillStyle="#378a54";ctx.beginPath();
  ctx.arc(p.x,p.y,7,0,Math.PI*2);ctx.fill();
 }
 for(let y=0;y<10;y++)for(let x=0;x<9;x++){
  const q=board[y][x];if(!q)continue;
  const p=screen(x,y);
  ctx.beginPath();ctx.arc(p.x,p.y,24,0,Math.PI*2);
  ctx.fillStyle="#fff0cc";ctx.fill();
  ctx.strokeStyle=q.c==="r"?"#bd2525":"#252525";
  ctx.lineWidth=2;ctx.stroke();
  ctx.fillStyle=q.c==="r"?"#bd2525":"#252525";
  ctx.font="bold 25px serif";ctx.textAlign="center";ctx.textBaseline="middle";
  ctx.fillText(names[q.t][q.c],p.x,p.y+1);
 }
}
function screen(x,y){
 return flipped?{x:ox+(8-x)*step,y:oy+(9-y)*step}
 :{x:ox+x*step,y:oy+y*step};
}
function logical(px,py){
 let x=Math.round((px-ox)/step),y=Math.round((py-oy)/step);
 if(flipped){x=8-x;y=9-y}
 return {x,y};
}
canvas.addEventListener("click",e=>{
 if(ended||turn!=="r")return;
 const rect=canvas.getBoundingClientRect();
 const px=(e.clientX-rect.left)*540/rect.width;
 const py=(e.clientY-rect.top)*600/rect.height;
 const {x,y}=logical(px,py);
 if(x<0||x>8||y<0||y>9)return;
 const q=board[y][x];
 if(selected&&legal.some(m=>m.x===x&&m.y===y)){
  move(selected.x,selected.y,x,y);
  selected=null;legal=[];draw();
  if(!ended)setTimeout(aiMove,180);
  return;
 }
 if(q&&q.c==="r"){
  selected={x,y};legal=moves(x,y);draw();
 }else{
  selected=null;legal=[];draw();
 }
});
function inside(x,y){return x>=0&&x<9&&y>=0&&y<10}
function addMove(out,x,y,c){
 if(!inside(x,y))return false;
 const q=board[y][x];
 if(q&&q.c===c)return false;
 out.push({x,y});
 return !q;
}
function moves(x,y){
 const q=board[y][x];if(!q)return [];
 let out=[],c=q.c,dir=c==="r"?-1:1;
 const add=(a,b)=>addMove(out,a,b,c);
 if(q.t==="K"){
  for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){
   let a=x+dx,b=y+dy;
   if(a>=3&&a<=5&&b>=(c==="r"?7:0)&&b<=(c==="r"?9:2))add(a,b);
  }
  // 將帥照面
  for(let yy=y+dir;inside(x,yy);yy+=dir){
   if(board[yy][x]){
    if(board[yy][x].t==="K"&&board[yy][x].c!==c)out.push({x,y:yy});
    break;
   }
  }
 }else if(q.t==="A"){
  for(const [dx,dy] of [[1,1],[1,-1],[-1,1],[-1,-1]]){
   let a=x+dx,b=y+dy;
   if(a>=3&&a<=5&&b>=(c==="r"?7:0)&&b<=(c==="r"?9:2))add(a,b);
  }
 }else if(q.t==="B"){
  for(const [dx,dy] of [[2,2],[2,-2],[-2,2],[-2,-2]]){
   let a=x+dx,b=y+dy;
   if(!inside(a,b))continue;
   if(c==="r"?b<5:b>4)continue;
   if(!board[y+dy/2][x+dx/2])add(a,b);
  }
 }else if(q.t==="N"){
  const jumps=[
   [1,2,0,1],[-1,2,0,1],[1,-2,0,-1],[-1,-2,0,-1],
   [2,1,1,0],[2,-1,1,0],[-2,1,-1,0],[-2,-1,-1,0]
  ];
  for(const [dx,dy,lx,ly] of jumps){
   const a=x+dx,b=y+dy;
   if(inside(a,b)&&!board[y+ly][x+lx])add(a,b);
  }
 }else if(q.t==="R"||q.t==="C"){
  for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){
   let a=x+dx,b=y+dy,screened=false;
   while(inside(a,b)){
    const target=board[b][a];
    if(q.t==="R"){
     if(!target){out.push({x:a,y:b})}
     else{if(target.c!==c)out.push({x:a,y:b});break}
    }else{
     if(!screened){
      if(!target)out.push({x:a,y:b});
      else screened=true;
     }else if(target){
      if(target.c!==c)out.push({x:a,y:b});
      break;
     }
    }
    a+=dx;b+=dy;
   }
  }
 }else if(q.t==="P"){
  add(x,y+dir);
  if(c==="r"?y<=4:y>=5){add(x-1,y);add(x+1,y)}
 }
 return out;
}
function move(x,y,a,b){
 const piece=board[y][x],captured=board[b][a];
 records.push({
  board:board.map(row=>row.map(p=>p?{...p}:null)),
  turn,history:history.slice(),ended
 });
 board[b][a]=piece;board[y][x]=null;
 history.push((piece.c==="r"?"紅":"黑")+names[piece.t][piece.c]+
  "："+String.fromCharCode(65+x)+(10-y)+" → "+
  String.fromCharCode(65+a)+(10-b));
 turn=turn==="r"?"b":"r";
 if(captured&&captured.t==="K"){
  ended=true;updateStatus((piece.c==="r"?"紅方":"黑方")+"獲勝！");
 }else{
  updateStatus(turn==="r"?"紅方回合":"黑方思考中…");
 }
 document.getElementById("history").textContent=history.join("\n");
}
function undo(){
 if(records.length===0)return;
 // 悔棋時若輪到黑方，連同電腦上一手一起復原
 let count=turn==="r"?2:1;
 while(count-->0&&records.length){
  const r=records.pop();board=r.board;turn=r.turn;
  history=r.history;ended=r.ended;
 }
 selected=null;legal=[];draw();
 updateStatus(turn==="r"?"紅方回合":"黑方回合");
 document.getElementById("history").textContent=history.join("\n")||"尚未開始";
}
function flipBoard(){flipped=!flipped;draw()}
function updateStatus(s){document.getElementById("status").textContent=s}
function hint(){
 if(turn!=="r"||ended)return;
 let all=[];
 for(let y=0;y<10;y++)for(let x=0;x<9;x++){
  if(board[y][x]&&board[y][x].c==="r")
   for(const m of moves(x,y))all.push({x,y,...m});
 }
 if(!all.length)return;
 const m=all[Math.floor(Math.random()*all.length)];
 selected={x:m.x,y:m.y};legal=moves(m.x,m.y);draw();
 updateStatus("提示：選擇高亮棋子，再點綠點");
}
function aiMove(){
 if(ended||turn!=="b")return;
 let all=[];
 for(let y=0;y<10;y++)for(let x=0;x<9;x++){
  if(board[y][x]&&board[y][x].c==="b")
   for(const m of moves(x,y))all.push({x,y,...m});
 }
 if(!all.length){ended=true;updateStatus("黑方無可走棋步");return}
 const depth=Number(document.getElementById("difficulty").value);
 all.sort((a,b)=>moveScore(b)-moveScore(a));
 let best=all.slice(0,Math.min(all.length,depth===1?12:depth===2?20:30));
 // 淺層搜尋加上隨機性，避免每次都走同一步
 let chosen=best[0],score=-Infinity;
 for(const m of best){
  const piece=board[m.y][m.x],captured=board[m.b][m.a];
  board[m.b][m.a]=piece;board[m.y][m.x]=null;
  let s=evaluate();
  if(captured)s+=value(captured.t)*1.1;
  if(depth>=2){
   turn="r";
   let reply=0;
   outer:for(let y=0;y<10;y++)for(let x=0;x<9;x++){
    if(board[y][x]&&board[y][x].c==="r"){
     for(const r of moves(x,y)){
      const old=board[r.y][r.x];
      if(old)reply=Math.max(reply,value(old.t));
     }
    }
    if(reply>0)break outer;
   }
   s-=reply*(depth===3?0.7:0.4);
   turn="b";
  }
  s+=Math.random()*3;
  board[m.y][m.x]=captured||null;board[m.x===m.a?m.y:m.y][m.x]=piece;
  // 恢復原始棋盤
  board[m.y][m.x]=piece;board[m.b][m.a]=captured||null;
  if(s>score){score=s;chosen=m}
 }
 move(chosen.x,chosen.y,chosen.a,chosen.b);
 draw();
}
function value(t){return {K:10000,R:900,C:450,N:400,B:200,A:200,P:100}[t]||0}
function moveScore(m){
 const target=board[m.b][m.a];
 return (target?value(target.t)*10:0)+Math.random()*10;
}
function evaluate(){
 let s=0;
 for(let y=0;y<10;y++)for(let x=0;x<9;x++){
  const p=board[y][x];if(!p)continue;
  let v=value(p.t);
  if(p.t==="P")v+=p.c==="r"?(9-y)*5:y*5;
  s+=p.c==="b"?v:-v;
 }
 return s;
}
newGame();
</script>
</body>
</html>
