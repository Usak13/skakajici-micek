<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="UTF-8" />
<title>Skákající míček</title>
<style>
  body {
    margin: 0; 
    background: #222;
    color: white;
    font-family: Arial, sans-serif;
    text-align: center;
  }
  canvas {
    background: #333;
    display: block;
    margin: 20px auto;
    border-radius: 10px;
  }
  #score {
    font-size: 24px;
    margin-top: 10px;
  }
  #startBtn {
    font-size: 20px;
    padding: 10px 20px;
    margin-top: 10px;
    cursor: pointer;
    background: #4CAF50;
    color: white;
    border: none;
    border-radius: 8px;
  }
  #startBtn:hover {
    background: #45a049;
  }
</style>
</head>
<body>

<h1>Skákající míček</h1>
<div id="score">Skóre: 0</div>
<button id="startBtn">Start</button>
<canvas id="gameCanvas" width="600" height="300"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const startBtn = document.getElementById('startBtn');

const gravity = 0.8;
const jumpPower = -15;

let ball = {
  x: 50,
  y: 250,
  radius: 20,
  vy: 0,
  grounded: false,
  colorToggle: false,
};

let obstacles = [];
let speed = 6;
let score = 0;
let gameRunning = false;

const jumpSound = new Audio('https://freesound.org/data/previews/66/66717_634166-lq.mp3');

function startGame() {
  obstacles = [];
  score = 0;
  speed = 6;
  gameRunning = true;
  ball.y = 250;
  ball.vy = 0;
  ball.grounded = false;
  scoreEl.textContent = 'Skóre: 0';
  startBtn.style.display = 'none';
  spawnObstacle();
  requestAnimationFrame(gameLoop);
}

function spawnObstacle() {
  let height = 30 + Math.random() * 40;
  obstacles.push({
    x: canvas.width,
    y: canvas.height - height,
    width: 20,
    height: height,
  });
  if(gameRunning){
    setTimeout(spawnObstacle, 2000);
  }
}

function jump() {
  if(ball.grounded){
    ball.vy = jumpPower;
    ball.grounded = false;
    jumpSound.currentTime = 0;
    jumpSound.play();
  }
}

function gameLoop(){
  if(!gameRunning) return;

  // Gravity + movement
  ball.vy += gravity;
  ball.y += ball.vy;
  if(ball.y + ball.radius > canvas.height){
    ball.y = canvas.height - ball.radius;
    ball.vy = 0;
    ball.grounded = true;
  }

  // Move obstacles
  for(let i = obstacles.length -1; i >= 0; i--){
    obstacles[i].x -= speed;

    // Remove off screen + score
    if(obstacles[i].x + obstacles[i].width < 0){
      obstacles.splice(i,1);
      score++;
      scoreEl.textContent = 'Skóre: ' + score;
      if(score % 5 === 0) speed += 0.5;
    }

    // Collision detection (circle-rectangle)
    if(circleRectCollision(ball, obstacles[i])){
      gameOver();
      return;
    }
  }

  // Color toggle pro animaci míčku
  ball.colorToggle = !ball.colorToggle;

  draw();

  requestAnimationFrame(gameLoop);
}

function circleRectCollision(circle, rect){
  // Nejbližší bod na rect k centru kruhu
  let nearestX = Math.max(rect.x, Math.min(circle.x, rect.x + rect.width));
  let nearestY = Math.max(rect.y, Math.min(circle.y, rect.y + rect.height));

  let dx = circle.x - nearestX;
  let dy = circle.y - nearestY;

  return (dx*dx + dy*dy) < (circle.radius * circle.radius);
}

function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Zem
  ctx.fillStyle = '#555';
  ctx.fillRect(0, canvas.height - 20, canvas.width, 20);

  // Míček (animace barvy)
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI*2);
  ctx.fillStyle = ball.colorToggle ? 'orange' : 'yellow';
  ctx.fill();
  ctx.closePath();

  // Překážky
  ctx.fillStyle = 'red';
  for(let obs of obstacles){
    ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
  }
}

function gameOver(){
  alert('Konec hry! Skóre: ' + score);
  gameRunning = false;
  startBtn.style.display = 'inline-block';
}

startBtn.addEventListener('click', startGame);
window.addEventListener('keydown', e => {
  if(e.code === 'Space') jump();
});
window.addEventListener('click', jump);
</script>

</body>
</html>
