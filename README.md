
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Snake Arena</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;padding:20px;background:#10151c;color:white;
 font-family:Arial,sans-serif;text-align:center
}
h1{color:#5cff9d}
.game{
 max-width:440px;margin:auto;background:#1b2530;
 padding:18px;border-radius:18px
}
canvas{
 width:100%;background:#080e13;
 border:2px solid #5cff9d;border-radius:8px
}
.info{display:flex;justify-content:space-around;margin:15px;font-size:18px}
button{
 background:#5cff9d;color:#102018;border:0;
 padding:12px 20px;border-radius:8px;font-weight:bold;
 cursor:pointer;margin:5px;touch-action:manipulation
}
.controls{
 display:grid;grid-template-columns:repeat(3,65px);
 justify-content:center;gap:7px;margin-top:18px
}
.controls button{height:48px;font-size:20px;margin:0}
</style>
</head>
<body>
<h1>🐍 SNAKE ARENA</h1>
<div class="game">
 <div class="info">
  <span>Pontos: <b id="score">0</b></span>
  <span>Recorde: <b id="best">0</b></span>
 </div>
 <canvas id="game" width="400" height="400"></canvas>
 <p id="status">Clique em iniciar para jogar!</p>
 <button onclick="startGame()">INICIAR JOGO</button>
 <div class="controls">
  <span></span><button onclick="change(0,-1)">⬆</button><span></span>
  <button onclick="change(-1,0)">⬅</button>
  <button onclick="change(0,1)">⬇</button>
  <button onclick="change(1,0)">➡</button>
 </div>
</div>
<h2>🏆 Ranking</h2>
<p>As pontuações enviadas ficam registradas na planilha.</p>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");
const size=20,total=20;
const API_URL="COLE_A_URL_DO_APPS_SCRIPT";

let snake,food,dx,dy,score,timer;
let running=false,locked=false;

let best=Number(localStorage.getItem("snakeBest"))||0;
document.getElementById("best").textContent=best;

function startGame(){
 clearInterval(timer);
 snake=[{x:10,y:10}];
 dx=1;dy=0;score=0;running=true;locked=false;
 document.getElementById("score").textContent=0;
 document.getElementById("status").textContent="Boa sorte!";
 placeFood();draw();
 timer=setInterval(update,130);
}

function placeFood(){
 do{
  food={x:Math.floor(Math.random()*total),y:Math.floor(Math.random()*total)};
 }while(snake.some(p=>p.x===food.x&&p.y===food.y));
}

function change(x,y){
 if(!running||locked)return;
 if(x===-dx&&y===-dy)return;
 dx=x;dy=y;locked=true;
}

function update(){
 const head={x:snake[0].x+dx,y:snake[0].y+dy};
 const ate=head.x===food.x&&head.y===food.y;

 if(head.x<0||head.y<0||head.x>=total||head.y>=total||
 snake.slice(0,ate?undefined:-1).some(p=>p.x===head.x&&p.y===head.y)){
  endGame();return;
 }

 snake.unshift(head);
 if(ate){
  score+=10;
  document.getElementById("score").textContent=score;
  placeFood();
 }else snake.pop();

 locked=false;draw();
}

function draw(){
 ctx.fillStyle="#080e13";
 ctx.fillRect(0,0,400,400);
 ctx.fillStyle="#ff5b67";
 ctx.fillRect(food.x*size,food.y*size,size-2,size-2);
 snake.forEach((p,i)=>{
  ctx.fillStyle=i===0?"#aaffc5":"#5cff9d";
  ctx.fillRect(p.x*size,p.y*size,size-2,size-2);
 });
}

function endGame(){
 running=false;clearInterval(timer);
 document.getElementById("status").textContent="Fim de jogo! Pontuação: "+score;

 if(score>best){
  best=score;
  localStorage.setItem("snakeBest",best);
  document.getElementById("best").textContent=best;
 }
 saveScore();
}

async function saveScore(){
 const nome=prompt("Digite seu nome para registrar a pontuação:");
 if(!nome||!nome.trim())return;

 try{
  await fetch(API_URL,{
   method:"POST",
   mode:"no-cors",
   headers:{"Content-Type":"text/plain;charset=utf-8"},
   body:JSON.stringify({nome:nome.trim(),pontos:score})
  });
  document.getElementById("status").textContent=
   "Envio realizado! Confira a planilha.";
 }catch(e){
  document.getElementById("status").textContent=
   "Não foi possível enviar a pontuação.";
 }
}

document.addEventListener("keydown",e=>{
 const keys={
  ArrowUp:[0,-1],ArrowDown:[0,1],
  ArrowLeft:[-1,0],ArrowRight:[1,0],
  w:[0,-1],s:[0,1],a:[-1,0],d:[1,0]
 };
 if(keys[e.key]){
  e.preventDefault();
  change(...keys[e.key]);
 }
});
</script>
</body>
</html>
