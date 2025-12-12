---
layout: opencs
title: Ping Pong
description: Use JavaScript to code a ping pong game
permalink: /pong
---
## 🎮 Pong Game Demo (Enhanced)

<div class="game-canvas-container" style="text-align:center;">
  <canvas id="pongCanvas" width="800" height="500"></canvas>
  <br>
  <button id="restartBtn">Restart Game</button>
</div>

<style>
  .game-canvas-container {
    margin-top: 20px;
  }
  #pongCanvas {
    border: 2px solid #fff;
    background: #061426;
    display: block;
    margin: 0 auto;
  }
  #restartBtn {
    display: none;
    margin-top: 15px;
    padding: 10px 20px;
    font-size: 18px;
    border: none;
    border-radius: 6px;
    background: #4caf50;
    color: white;
    cursor: pointer;
  }
  #restartBtn:hover { background: #45a049; }
</style>

<script>
// =====================================================
// FIXED VERSION — Player 1 paddle now properly bounces ball
// =====================================================

// --------------------
// CONFIG
// --------------------
const Config = {
  canvas: { width: 800, height: 500 },
  paddle: { width: 12, height: 100, speed: 7 },
  ball: {
    radius: 10,
    baseSpeedX: 5,
    maxRandomY: 2,
    spinFactor: 0.28,
    speedupFactor: 1.06,
    maxSpeedX: 18
  },
  rules: { winningScore: 10 },
  keys: { p1Up: "w", p1Down: "s", p2Up: "i", p2Down: "k", pause: " " },
  visuals: {
    bg: "#061426",
    fg: "#e6f7ff",
    text: "#e6f7ff",
    gameOver: "#ff4d4f",
    win: "#ffe66d",
    centerLine: "#2b728f"
  },
  ai: { enabled: true, difficulty: 0.9 },
  powerups: { spawnInterval: 8000, duration: 6000, size: 18 }
};

// --------------------
// Helper Classes
// --------------------
class Vector2 { constructor(x=0,y=0){this.x=x;this.y=y;} }

class Paddle {
  constructor(x,y,w,h,s,bh){
    this.position=new Vector2(x,y);
    this.width=w;
    this.height=h;
    this.speed=s;
    this.boundsHeight=bh;
  }
  move(dy){
    this.position.y = Math.max(0, Math.min(this.boundsHeight - this.height, this.position.y + dy));
  }
  rect(){
    return { x:this.position.x, y:this.position.y, w:this.width, h:this.height };
  }
}

class Ball {
  constructor(radius,bw,bh){
    this.radius=radius;
    this.boundsWidth=bw;
    this.boundsHeight=bh;
    this.position=new Vector2();
    this.velocity=new Vector2();
    this.reset(true);
  }
  reset(randomDirection=false){
    this.position.x = this.boundsWidth/2;
    this.position.y = this.boundsHeight/2;

    const dir = randomDirection && Math.random() > 0.5 ? 1 : -1;
    this.velocity.x = dir * Config.ball.baseSpeedX;
    this.velocity.y = Math.random() * (2 * Config.ball.maxRandomY) - Config.ball.maxRandomY;
  }
  update(){
    this.position.x += this.velocity.x;
    this.position.y += this.velocity.y;

    // Wall bounce
    if (this.position.y - this.radius < 0 || this.position.y + this.radius > this.boundsHeight) {
      this.velocity.y *= -1;
    }
  }
}

class Input {
  constructor(){
    this.keys = {};
    document.addEventListener("keydown", e => this.keys[e.key] = true);
    document.addEventListener("keyup", e => this.keys[e.key] = false);
  }
  isDown(k){ return !!this.keys[k]; }
}

class Renderer {
  constructor(ctx){ this.ctx=ctx; }
  clear(w,h){
    this.ctx.fillStyle = Config.visuals.bg;
    this.ctx.fillRect(0,0,w,h);
  }
  rect(r){
    this.ctx.fillStyle = Config.visuals.fg;
    this.ctx.fillRect(r.x,r.y,r.w,r.h);
  }
  circle(ball){
    this.ctx.fillStyle = Config.visuals.fg;
    this.ctx.beginPath();
    this.ctx.arc(ball.position.x, ball.position.y, ball.radius, 0, Math.PI*2);
    this.ctx.fill();
  }
  text(t,x,y,color=Config.visuals.text,size=30){
    this.ctx.fillStyle=color;
    this.ctx.font = size + "px Arial";
    this.ctx.fillText(t,x,y);
  }
  dashedCenter(w,h){
    const ctx=this.ctx;
    ctx.strokeStyle = Config.visuals.centerLine;
    ctx.setLineDash([10,10]);
    ctx.beginPath();
    ctx.moveTo(w/2,0);
    ctx.lineTo(w/2,h);
    ctx.stroke();
    ctx.setLineDash([]);
  }
}

// --------------------
// GAME CLASS
// --------------------
class Game {
  constructor(canvasEl,restartBtn){

    this.canvas = canvasEl;
    this.ctx = canvasEl.getContext("2d");
    this.renderer = new Renderer(this.ctx);
    this.input = new Input();

    // paddles
    let P = Config.paddle;
    this.paddleLeft = new Paddle(8, (Config.canvas.height - P.height)/2, P.width, P.height, P.speed, Config.canvas.height);
    this.paddleRight = new Paddle(Config.canvas.width - P.width - 8, (Config.canvas.height - P.height)/2, P.width, P.height, P.speed, Config.canvas.height);

    // ball
    this.ball = new Ball(Config.ball.radius, Config.canvas.width, Config.canvas.height);

    this.scores = {p1:0, p2:0};
    this.gameOver = false;

    this.restartBtn = restartBtn;
    this.restartBtn.addEventListener("click",()=>this.restart());

    this.loop = this.loop.bind(this);
  }

  handleInput(){
    if (this.gameOver) return;

    if (this.input.isDown(Config.keys.p1Up)) this.paddleLeft.move(-this.paddleLeft.speed);
    if (this.input.isDown(Config.keys.p1Down)) this.paddleLeft.move(this.paddleLeft.speed);

    if (this.input.isDown(Config.keys.p2Up)) this.paddleRight.move(-this.paddleRight.speed);
    if (this.input.isDown(Config.keys.p2Down)) this.paddleRight.move(this.paddleRight.speed);
  }

  update(){
    if (this.gameOver) return;
    this.ball.update();

    // ---------------------------
    // FIXED PLAYER 1 COLLISION
    // ---------------------------
    const hitLeft =
      this.ball.position.x - this.ball.radius <= this.paddleLeft.position.x + this.paddleLeft.width &&
      this.ball.position.x - this.ball.radius >= this.paddleLeft.position.x && // ensures ball is actually near paddle
      this.ball.position.y >= this.paddleLeft.position.y &&
      this.ball.position.y <= this.paddleLeft.position.y + this.paddleLeft.height;

    if (hitLeft){
      this.ball.velocity.x = Math.abs(this.ball.velocity.x);
      this.ball.position.x = this.paddleLeft.position.x + this.paddleLeft.width + this.ball.radius;

      const delta = this.ball.position.y - (this.paddleLeft.position.y + this.paddleLeft.height/2);
      this.ball.velocity.y = delta * Config.ball.spinFactor;
    }

    // Right paddle collision
    const hitRight =
      this.ball.position.x + this.ball.radius >= this.paddleRight.position.x &&
      this.ball.position.x + this.ball.radius <= this.paddleRight.position.x + this.paddleRight.width &&
      this.ball.position.y >= this.paddleRight.position.y &&
      this.ball.position.y <= this.paddleRight.position.y + this.paddleRight.height;

    if (hitRight){
      this.ball.velocity.x = -Math.abs(this.ball.velocity.x);
      this.ball.position.x = this.paddleRight.position.x - this.ball.radius;

      const delta = this.ball.position.y - (this.paddleRight.position.y + this.paddleRight.height/2);
      this.ball.velocity.y = delta * Config.ball.spinFactor;
    }

    // scoring
    if (this.ball.position.x - this.ball.radius < 0){
      this.scores.p2++;
      this.ball.reset(true);
    }
    if (this.ball.position.x + this.ball.radius > Config.canvas.width){
      this.scores.p1++;
      this.ball.reset(true);
    }
  }

  draw(){
    this.renderer.clear(Config.canvas.width, Config.canvas.height);
    this.renderer.dashedCenter(Config.canvas.width, Config.canvas.height);
    this.renderer.rect(this.paddleLeft.rect());
    this.renderer.rect(this.paddleRight.rect());
    this.renderer.circle(this.ball);

    this.renderer.text(this.scores.p1, Config.canvas.width/4, 50);
    this.renderer.text(this.scores.p2, 3*Config.canvas.width/4, 50);
  }

  restart(){
    this.scores={p1:0,p2:0};
    this.ball.reset(true);
    this.restartBtn.style.display="none";
  }

  loop(){
    this.handleInput();
    this.update();
    this.draw();
    requestAnimationFrame(this.loop);
  }
}

// --------------------
// Boot
// --------------------
const canvas=document.getElementById("pongCanvas");
const restartBtn=document.getElementById("restartBtn");

canvas.width = Config.canvas.width;
canvas.height = Config.canvas.height;

const game=new Game(canvas,restartBtn);
game.loop();
</script>
