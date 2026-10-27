
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ME TIME GAMES - Play Games</title>

<style>
*{box-sizing:border-box;margin:0;padding:0}
body{
  background:#080b12;color:white;
  font-family:Arial,sans-serif;min-height:100vh
}
header{
  background:#101522;padding:20px 6%;
  display:flex;justify-content:space-between;
  align-items:center;gap:15px;flex-wrap:wrap;
  border-bottom:1px solid #263247
}
.logo{color:#39ff88;font-size:25px;font-weight:bold}
header span{color:#aab4c5;font-size:13px}
.hero{text-align:center;padding:42px 15px 25px}
.hero h1{font-size:clamp(30px,6vw,52px);margin-bottom:12px}
.hero h1 span{color:#39ff88}
.hero p{color:#aab4c5;line-height:1.6}
main{max-width:1000px;margin:auto;padding:20px}
.selector{
  display:grid;grid-template-columns:repeat(3,1fr);
  gap:12px;margin:12px 0 22px
}
.game-choice{
  padding:17px 8px;background:#111827;color:white;
  border:1px solid #303b50;border-radius:10px;
  cursor:pointer;font-size:15px;font-weight:bold
}
.game-choice.active,.game-choice:hover{
  border-color:#39ff88;color:#39ff88;
  background:#14231e
}
.panel{
  background:#101522;border:1px solid #283449;
  border-radius:14px;padding:18px
}
.game-heading{
  display:flex;justify-content:space-between;
  align-items:center;gap:12px;flex-wrap:wrap;margin-bottom:15px
}
.game-heading h2{font-size:22px}
.score{color:#39ff88;font-weight:bold}
canvas{
  display:block;width:100%;max-width:600px;
  margin:auto;background:#05080e;
  border:1px solid #2a3a4d;border-radius:6px;
  touch-action:none
}
.controls{
  display:flex;justify-content:center;gap:10px;
  flex-wrap:wrap;margin-top:17px
}
button.action{
  padding:12px 19px;border:0;border-radius:7px;
  background:#39ff88;color:#07110b;font-weight:bold;
  cursor:pointer;font-size:14px
}
button.action:hover{filter:brightness(1.1)}
.instructions{
  text-align:center;color:#aab4c5;
  font-size:14px;line-height:1.8;margin-top:15px
}
.mobile-pad{
  display:grid;grid-template-columns:repeat(3,48px);
  gap:6px;justify-content:center;margin-top:15px
}
.mobile-pad button{
  height:43px;background:#202c3e;color:white;
  border:1px solid #3b4b60;border-radius:8px;
  font-size:19px;touch-action:manipulation
}
.mobile-pad button:active{background:#39ff88;color:#07110b}
.mobile-pad .empty{visibility:hidden}
footer{text-align:center;padding:28px 12px;color:#768398;font-size:13px}
@media(max-width:500px){
  main{padding:12px}
  .panel{padding:10px}
  .game-choice{font-size:13px}
  .hero{padding-top:32px}
}
</style>
</head>

<body>
<header>
  <div class="logo">⚡ NEON ARCADE</div>
  <span>FREE GAMES · PLAY IN YOUR BROWSER</span>
</header>

<section class="hero">
  <h1>LET'S <span>PLAY!</span></h1>
  <p>Choose your game. Beat your high score. Become a legend.</p>
</section>

<main>
  <div class="selector">
    <button class="game-choice active" data-game="snake">🐍 Snake</button>
    <button class="game-choice" data-game="breakout">🧱 Breakout</button>
    <button class="game-choice" data-game="pong">🏓 Pong</button>
  </div>

  <section class="panel">
    <div class="game-heading">
      <h2 id="gameTitle">🐍 Snake</h2>
      <div class="score">Score: <span id="score">0</span></div>
    </div>

    <canvas id="game" width="600" height="400"
      aria-label="Game playing area"></canvas>

    <div class="controls">
      <button class="action" id="start">▶ START GAME</button>
      <button class="action" id="pause">Ⅱ PAUSE</button>
      <button class="action" id="restart">↻ RESTART</button>
    </div>

    <div class="mobile-pad" id="mobilePad">
      <button class="empty"> </button>
      <button data-dir="up">▲</button>
      <button class="empty"> </button>
      <button data-dir="left">◀</button>
      <button data-dir="down">▼</button>
      <button data-dir="right">▶</button>
    </div>

    <p class="instructions" id="instructions">
      Use arrow keys or WASD to move. Eat the food and don't hit the walls!
    </p>
  </section>
</main>

<footer>NEON ARCADE © 2026 · Made for gamers 🎮</footer>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const scoreEl = document.getElementById("score");
const titleEl = document.getElementById("gameTitle");
const instructionsEl = document.getElementById("instructions");

const W = 600, H = 400;
const CELL = 20;

let game = "snake";
let timer = null;
let running = false;
let score = 0;
let best = {snake:0, breakout:0, pong:0};
let state = {};

function stopTimer(){
  if(timer !== null) clearInterval(timer);
  timer = null;
  running = false;
}

function setScore(n){
  score = n;
  scoreEl.textContent = score;
  if(score > best[game]) best[game] = score;
}

function text(message, sub=""){
  ctx.fillStyle="#05080e";
  ctx.fillRect(0,0,W,H);
  ctx.textAlign="center";
  ctx.fillStyle="#39ff88";
  ctx.font="bold 30px Arial";
  ctx.fillText(message,W/2,H/2-8);
  ctx.fillStyle="#c1cada";
  ctx.font="16px Arial";
  ctx.fillText(sub,W/2,H/2+25);
}

function roundedRect(x,y,w,h,r,color){
  ctx.fillStyle=color;
  ctx.beginPath();
  ctx.roundRect(x,y,w,h,r);
  ctx.fill();
}

function selectGame(name){
  stopTimer();
  game = name;
  document.querySelectorAll(".game-choice").forEach(b=>{
    b.classList.toggle("active",b.dataset.game===name);
  });

  const titles = {
    snake:"🐍 Snake",
    breakout:"🧱 Breakout",
    pong:"🏓 Pong"
  };
  const help = {
    snake:"Arrow keys / WASD to move. On mobile, use the direction pad. Eat food and avoid crashing!",
    breakout:"Move your paddle with ← → or your mouse. On mobile, drag on the canvas. Break every brick!",
    pong:"Move with ↑ ↓ or W / S. On mobile, drag on the canvas. First to 5 points wins!"
  };

  titleEl.textContent=titles[name];
  instructionsEl.textContent=help[name];
  resetGame();
}

function resetGame(){
  stopTimer();
  setScore(0);

  if(game==="snake"){
    state={
      snake:[{x:200,y:200},{x:180,y:200},{x:160,y:200}],
      dir:{x:CELL,y:0},next:{x:CELL,y:0},
      food:{x:300,y:200},over:false
    };
    placeFood();
  } else if(game==="breakout"){
    state={
      x:W/2,y:H-55,dx:3,dy:-4,
      paddle:80, bricks:[],over:false
    };
    for(let r=0;r<5;r++){
      for(let c=0;c<10;c++){
        state.bricks.push({
          x:10+c*59,y:35+r*25,w:52,h:16,
          color:["#39ff88","#00d9ff","#a970ff","#ffce56","#ff6584"][r],
          alive:true
        });
      }
    }
  } else {
    state={
      x:W/2,y:H/2,dx:4,dy:3,
      p1:H/2-40,p2:H/2-40,
      left:0,right:0,over:false
    };
  }
  draw();
}

function placeFood(){
  do{
    state.food={
      x:Math.floor(Math.random()*(W/CELL))*CELL,
      y:Math.floor(Math.random()*(H/CELL))*CELL
    };
  }while(state.snake.some(s=>s.x===state.food.x&&s.y===state.food.y));
}

function startGame(){
  if(running)return;
  if(state.over)resetGame();
  running=true;
  timer=setInterval(tick,game==="snake"?115:16);
}

function endGame(message){
  stopTimer();
  state.over=true;
  draw();
  text(message,"Score: "+score+"  ·  Press RESTART to play again");
}

function tick(){
  if(game==="snake")snakeTick();
  if(game==="breakout")breakoutTick();
  if(game==="pong")pongTick();
}

function snakeTick(){
  const s=state;
  s.dir={...s.next};
  const head={
    x:s.snake[0].x+s.dir.x,
    y:s.snake[0].y+s.dir.y
  };

  const eating=head.x===s.food.x&&head.y===s.food.y;
  const body=eating?s.snake:s.snake.slice(0,-1);

  if(head.x<0||head.x>=W||head.y<0||head.y>=H||
     body.some(p=>p.x===head.x&&p.y===head.y)){
    endGame("GAME OVER!");
    return;
  }

  s.snake.unshift(head);
  if(eating){
    setScore(score+10);
    placeFood();
  }else{
    s.snake.pop();
  }
  draw();
}

function breakoutTick(){
  const s=state;
  s.x+=s.dx;
  s.y+=s.dy;

  if(s.x<8||s.x>W-8)s.dx*=-1;
  if(s.y<8)s.dy=Math.abs(s.dy);

  if(s.y>H-35&&s.y<H-20&&
     s.x>s.paddle&&s.x<s.paddle+100&&s.dy>0){
    s.dy=-Math.abs(s.dy);
    s.dx+=((s.x-(s.paddle+50))/50)*0.7;
  }

  for(const b of s.bricks){
    if(b.alive&&s.x>b.x&&s.x<b.x+b.w&&
       s.y>b.y&&s.y<b.y+b.h&&s.dy<0){
      b.alive=false;
      s.dy=Math.abs(s.dy);
      setScore(score+10);
      break;
    }
  }

  if(s.bricks.every(b=>!b.alive)){
    stopTimer();
    state.over=true;
    draw();
    text("YOU WIN!","All bricks destroyed · Score: "+score);
    return;
  }

  if(s.y>H){
    endGame("GAME OVER!");
    return;
  }
  draw();
}

function pongTick(){
  const s=state;
  s.y+=s.dy;
  s.x+=s.dx;

  if(s.y<8||s.y>H-8)s.dy*=-1;

  if(s.x<25&&s.y>s.p1&&s.y<s.p1+80&&s.dx<0){
    s.dx=Math.abs(s.dx)*1.04;
    s.dy+=((s.y-(s.p1+40))/40)*1.5;
  }
  if(s.x>W-25&&s.y>s.p2&&s.y<s.p2+80&&s.dx>0){
    s.dx=-Math.abs(s.dx)*1.04;
    s.dy+=((s.y-(s.p2+40))/40)*1.5;
  }

  s.p2+=(s.y-40-s.p2)*0.055;
  s.p2=Math.max(0,Math.min(H-80,s.p2));

  if(s.x<0){
    s.right++;
    pointPong();
    return;
  }
  if(s.x>W){
    s.left++;
    pointPong();
    return;
  }
  draw();
}

function pointPong(){
  const s=state;
  setScore(s.left);
  if(s.left>=5||s.right>=5){
    stopTimer();
    s.over=true;
    draw();
    text(s.left>=5?"YOU WIN!":"CPU WINS!",
      "Final score "+s.left+" - "+s.right+" · Restart to play");
    return;
  }
  s.x=W/2;s.y=H/2;
  s.dx=s.left>s.right?-4:4;
  s.dy=Math.random()*4-2;
  draw();
}

function draw(){
  ctx.fillStyle="#05080e";
  ctx.fillRect(0,0,W,H);

  if(game==="snake"){
    for(let x=0;x<W;x+=CELL){
      for(let y=0;y<H;y+=CELL){
        ctx.strokeStyle="#111a26";
        ctx.strokeRect(x,y,CELL,CELL);
      }
    }
    roundedRect(state.food.x+2,state.food.y+2, CELL-4,CELL-4,5,"#ff5577");
    state.snake.forEach((p,i)=>{
      roundedRect(p.x+1,p.y+1,CELL-2,CELL-2,5,
        i===0?"#39ff88":"#159b61");
    });
  }

  if(game==="breakout"){
    state.bricks.forEach(b=>{
      if(b.alive)roundedRect(b.x,b.y,b.w,b.h,3,b.color);
    });
    roundedRect(state.paddle,H-25,100,10,5,"#39ff88");
    ctx.beginPath();
    ctx.arc(state.x,state.y,8,0,Math.PI*2);
    ctx.fillStyle="#fff";
    ctx.fill();
  }

  if(game==="pong"){
    ctx.setLineDash([8,10]);
    ctx.strokeStyle="#304054";
    ctx.beginPath();ctx.moveTo(W/2,0);ctx.lineTo(W/2,H);ctx.stroke();
    ctx.setLineDash([]);
    roundedRect(12,state.p1,10,80,4,"#39ff88");
    roundedRect(W-22,state.p2,10,80,4,"#00d9ff");
    ctx.beginPath();ctx.arc(state.x,state.y,8,0,Math.PI*2);
    ctx.fillStyle="#fff";ctx.fill();
    ctx.fillStyle="#dce5f5";ctx.font="24px Arial";
    ctx.textAlign="center";
    ctx.fillText(state.left,W/2-45,35);
    ctx.fillText(state.right,W/2+45,35);
  }
}

function moveSnake(dx,dy){
  if(game!=="snake")return;
  if(state.dir.x===-dx&&state.dir.y===-dy)return;
  state.next={x:dx,y:dy};
}

document.addEventListener("keydown",e=>{
  const k=e.key.toLowerCase();
  if(["arrowup","arrowdown","arrowleft","arrowright"," "].includes(k))
    e.preventDefault();

  if(k==="arrowup"||k==="w")moveSnake(0,-CELL);
  if(k==="arrowdown"||k==="s")moveSnake(0,CELL);
  if(k==="arrowleft"||k==="a")moveSnake(-CELL,0);
  if(k==="arrowright"||k==="d")moveSnake(CELL,0);

  if(game==="breakout"){
    if(k==="arrowleft"||k==="a")state.paddle=Math.max(0,state.paddle-25);
    if(k==="arrowright"||k==="d")state.paddle=Math.min(W-100,state.paddle+25);
  }

  if(game==="pong"){
    if(k==="arrowup"||k==="w")state.p1=Math.max(0,state.p1-25);
    if(k==="arrowdown"||k==="s")state.p1=Math.min(H-80,state.p1+25);
  }

  if(k===" "&&!running)startGame();
});

document.querySelectorAll(".game-choice").forEach(btn=>{
  btn.addEventListener("click",()=>selectGame(btn.dataset.game));
});

document.getElementById("start").addEventListener("click",startGame);
document.getElementById("pause").addEventListener("click",()=>{
  if(running)stopTimer();
  else if(!state.over)startGame();
});

document.getElementById("restart").addEventListener("click",resetGame);

document.querySelectorAll("[data-dir]").forEach(btn=>{
  btn.addEventListener("click",()=>{
    const dirs={
      up:[0,-CELL],down:[0,CELL],
      left:[-CELL,0],right:[CELL,0]
    };
    const d=dirs[btn.dataset.dir];
    moveSnake(d[0],d[1]);
    if(game==="breakout"){
      if(btn.dataset.dir==="left")state.paddle=Math.max(0,state.paddle-35);
      if(btn.dataset.dir==="right")state.paddle=Math.min(W-100,state.paddle+35);
    }
    if(game==="pong"){
      if(btn.dataset.dir==="up")state.p1=Math.max(0,state.p1-30);
      if(btn.dataset.dir==="down")state.p1=Math.min(H-80,state.p1+30);
    }
    draw();
  });
});

function pointerMove(e){
  const rect=canvas.getBoundingClientRect();
  const x=(e.clientX-rect.left)*W/rect.width;
  const y=(e.clientY-rect.top)*H/rect.height;
  if(game==="breakout")state.paddle=Math.max(0,Math.min(W-100,x-50));
  if(game==="pong")state.p1=Math.max(0,Math.min(H-80,y-40));
}

canvas.addEventListener("pointermove",pointerMove);
canvas.addEventListener("pointerdown",e=>{
  pointerMove(e);
  if(!running&&!state.over)startGame();
  if(game==="snake"){
    const r=canvas.getBoundingClientRect();
    const x=(e.clientX-r.left)/r.width;
    const y=(e.clientY-r.top)/r.height;
    if(Math.abs(x-.5)>Math.abs(y-.5))
      moveSnake(x<.5?-CELL:CELL,0);
    else moveSnake(0,y<.5?-CELL:CELL);
  }
});

selectGame("snake");
</script>
</body>
</html>
