# sal-am-somalia-bank-demo

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Football Champions</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#071322;color:white;
 font-family:Arial,sans-serif;text-align:center;
}
header{padding:14px;background:#102b46}
h1{margin:0;color:#42d9ff;font-size:25px}
#score{font-size:27px;font-weight:bold;margin:12px}
#game{
 width:96%;max-width:900px;
 display:block;margin:auto;
 background:#168b45;border:4px solid white;
 border-radius:8px;touch-action:none;
}
.info{color:#b9d8e8;font-size:13px;margin:8px}
.controls{
 display:flex;justify-content:center;gap:35px;
 align-items:center;margin:15px auto;
}
.pad{display:grid;grid-template-columns:repeat(3,55px);gap:5px}
button{
 background:#087eb9;color:white;border:0;
 padding:15px;border-radius:12px;
 font-weight:bold;font-size:17px;
}
button:active{background:#28c9ff}
.shoot{background:#e34b34;width:90px;height:65px}
#start{background:#13a66a;margin:10px;padding:14px 35px}
</style>
</head>
<body>

<header>
<h1>⚽ FOOTBALL CHAMPIONS</h1>
<div class="info">ULTIMATE FOOTBALL EXPERIENCE</div>
</header>

<div id="score">BLUE 0 - 0 RED</div>

<canvas id="game" width="900" height="500"></canvas>

<div class="info" id="status">Press START to play!</div>
<button id="start" onclick="startGame()">▶ START MATCH</button>

<div class="controls">
 <div class="pad">
  <span></span>
  <button data-key="up">▲</button>
  <span></span>
  <button data-key="left">◀</button>
  <button data-key="down">▼</button>
  <button data-key="right">▶</button>
 </div>
 <button class="shoot" data-key="shoot">SHOOT ⚽</button>
</div>

<div class="info">
Keyboard: Arrow keys / WASD | Space: Shoot
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let scoreBlue=0,scoreRed=0;
let playing=false;
let keys={};
let time=90;
let lastTime=0;
let frame=0;

const player={
 x:220,y:250,r:17,speed:3.5,
 color:"#168cff",vx:0,vy:0
};

const enemy={
 x:680,y:250,r:17,speed:2.1,
 color:"#ff3434"
};

const ball={
 x:450,y:250,r:9,vx:0,vy:0
};

function field(){
 ctx.fillStyle="#168b45";
 ctx.fillRect(0,0,900,500);

 for(let i=0;i<9;i++){
  ctx.fillStyle=i%2?"#168b45":"#198f4b";
  ctx.fillRect(i*100,0,100,500);
 }

 ctx.strokeStyle="white";
 ctx.lineWidth=4;
 ctx.strokeRect(12,12,876,476);

 ctx.beginPath();
 ctx.moveTo(450,12);
 ctx.lineTo(450,488);
 ctx.stroke();

 ctx.beginPath();
 ctx.arc(450,250,75,0,Math.PI*2);
 ctx.stroke();

 ctx.strokeRect(12,145,110,210);
 ctx.strokeRect(778,145,110,210);

 ctx.fillStyle="#fff";
 ctx.fillRect(0,195,12,110);
 ctx.fillRect(888,195,12,110);
}

function drawPlayer(p){
 ctx.beginPath();
 ctx.fillStyle=p.color;
 ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
 ctx.fill();

 ctx.strokeStyle="white";
 ctx.lineWidth=3;
 ctx.stroke();

 ctx.fillStyle="white";
 ctx.font="bold 12px Arial";
 ctx.textAlign="center";
 ctx.fillText(p===player?"10":"9",p.x,p.y+4);
}

function drawBall(){
 ctx.beginPath();
 ctx.fillStyle="white";
 ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
 ctx.fill();

 ctx.strokeStyle="#111";
 ctx.lineWidth=2;
 ctx.stroke();
}

function resetBall(direction){
 ball.x=450;ball.y=250;
 ball.vx=direction||0;
 ball.vy=0;
 player.x=220;player.y=250;
 enemy.x=680;enemy.y=250;
}

function update(){
 if(!playing)return;

 player.vx=0;player.vy=0;

 if(keys.left)player.vx=-player.speed;
 if(keys.right)player.vx=player.speed;
 if(keys.up)player.vy=-player.speed;
 if(keys.down)player.vy=player.speed;

 player.x+=player.vx;
 player.y+=player.vy;

 player.x=Math.max(25,Math.min(875,player.x));
 player.y=Math.max(30,Math.min(470,player.y));

 // Opponent AI
 let dx=ball.x-enemy.x;
 let dy=ball.y-enemy.y;
 let d=Math.hypot(dx,dy)||1;

 enemy.x+=dx/d*enemy.speed;
 enemy.y+=dy/d*enemy.speed;

 // Player and ball collision
 let px=ball.x-player.x;
 let py=ball.y-player.y;
 let pd=Math.hypot(px,py)||1;

 if(pd<player.r+ball.r){
  ball.vx=px/pd*5;
  ball.vy=py/pd*5;
 }

 // Opponent and ball collision
 let ex=ball.x-enemy.x;
 let ey=ball.y-enemy.y;
 let ed=Math.hypot(ex,ey)||1;

 if(ed<enemy.r+ball.r){
  ball.vx=ex/ed*4;
  ball.vy=ey/ed*4;
 }

 // Shoot
 if(keys.shoot){
  if(pd<65){
   ball.vx=7;
   ball.vy=(ball.y-player.y)*0.08;
   keys.shoot=false;
  }
 }

 // Ball movement
 ball.x+=ball.vx;
 ball.y+=ball.vy;
 ball.vx*=0.985;
 ball.vy*=0.985;

 if(ball.y<22||ball.y>478)ball.vy*=-1;

 // Goals
 if(ball.x<5 && ball.y>195 && ball.y<305){
  scoreRed++;
  resetBall(0);
 }
 if(ball.x>895 && ball.y>195 && ball.y<305){
  scoreBlue++;
  resetBall(0);
 }

 ball.x=Math.max(5,Math.min(895,ball.x));

 document.getElementById("score").textContent=
 "BLUE "+scoreBlue+" - "+scoreRed+" RED";
}

function draw(){
 ctx.clearRect(0,0,900,500);
 field();
 drawPlayer(player);
 drawPlayer(enemy);
 drawBall();
}

function loop(){
 update();
 draw();
 requestAnimationFrame(loop);
}

function startGame(){
 scoreBlue=0;
 scoreRed=0;
 time=90;
 resetBall(0);
 playing=true;

 document.getElementById("status").textContent=
 "MATCH STARTED! Score goals to win!";

 document.getElementById("score").textContent=
 "BLUE 0 - 0 RED";

 clearInterval(window.clock);
 window.clock=setInterval(()=>{
  if(!playing)return;
  time--;
  document.getElementById("status").textContent=
  "MATCH TIME: "+time+" seconds";
  if(time<=0){
   playing=false;
   document.getElementById("status").textContent=
   scoreBlue>scoreRed?"🏆 BLUE WINS!":
   scoreRed>scoreBlue?"🏆 RED WINS!":"🤝 DRAW!";
   clearInterval(window.clock);
  }
 },1000);
}

// Keyboard
document.addEventListener("keydown",e=>{
 const k=e.key.toLowerCase();
 if(k==="arrowup"||k==="w")keys.up=true;
 if(k==="arrowdown"||k==="s")keys.down=true;
 if(k==="arrowleft"||k==="a")keys.left=true;
 if(k==="arrowright"||k==="d")keys.right=true;
 if(k===" ")keys.shoot=true;
});

document.addEventListener("keyup",e=>{
 const k=e.key.toLowerCase();
 if(k==="arrowup"||k==="w")keys.up=false;
 if(k==="arrowdown"||k==="s")keys.down=false;
 if(k==="arrowleft"||k==="a")keys.left=false;
 if(k==="arrowright"||k==="d")keys.right=false;
});

document.querySelectorAll("[data-key]").forEach(btn=>{
 const k=btn.dataset.key;
 btn.addEventListener("pointerdown",e=>{
  e.preventDefault();
  keys[k]=true;
 });
 ["pointerup","pointerleave","pointercancel"].forEach(event=>{
  btn.addEventListener(event,()=>keys[k]=false);
 });
});

loop();
</script>
</body>
</html>
 Banking Demo Website
