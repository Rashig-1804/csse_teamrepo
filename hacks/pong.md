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
    background: #061426; /* darker bluish background */
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
// Enhanced Pong — implements full student checklist
// Features added:
// 1) Config visuals tweaks
// 2) Right-paddle AI (toggleable, difficulty multiplier)
// 3) Rally speed-up on paddle hit (with max speed)
// 4) Dashed center line + WebAudio score SFX
// 5) Power-ups (spawn, collect, temporary effects)
// 6) Pause/Resume (Space key)
// 7) Win screen with replay countdown and auto-restart

const Config = {
  canvas: { width: 800, height: 500 },
  paddle: { width: 12, height: 100, speed: 7 },
  ball: { radius: 10, baseSpeedX: 5, maxRandomY: 2, spinFactor: 0.28, speedupFactor: 1.06, maxSpeedX: 18 },
  rules: { winningScore: 10 },
  keys: { p1Up: 'w', p1Down: 's', p2Up: 'i', p2Down: 'k', pause: ' ' },
  visuals: { bg: '#061426', fg: '#e6f7ff', text: '#e6f7ff', gameOver: '#ff4d4f', win: '#ffe66d', centerLine: '#2b728f' },
  ai: { enabled: true, difficulty: 0.9 }, // difficulty: 0 (easy) to 1 (perfect)
  powerups: { spawnInterval: 8000, duration: 6000, size: 18 }
};

class Vector2 { constructor(x=0,y=0){this.x=x;this.y=y;} }

class Paddle {
  constructor(x,y,w,h,s,bh){ this.position=new Vector2(x,y); this.width=w; this.height=h; this.speed=s; this.boundsHeight=bh; }
  move(dy){ this.position.y = Math.min(this.boundsHeight - this.height, Math.max(0, this.position.y + dy)); }
  rect(){ return { x:this.position.x, y:this.position.y, w:this.width, h:this.height }; }
}

class Ball {
  constructor(radius,bw,bh){ this.radius=radius; this.boundsWidth=bw; this.boundsHeight=bh; this.position=new Vector2(); this.velocity=new Vector2(); this.reset(true); }
  reset(randomDirection=false){ this.position.x = this.boundsWidth/2; this.position.y = this.boundsHeight/2; const dir = randomDirection && Math.random() > 0.5 ? 1 : -1; this.velocity.x = dir * Config.ball.baseSpeedX; this.velocity.y = (Math.random() * (2 * Config.ball.maxRandomY)) - Config.ball.maxRandomY; }
  update(){ this.position.x += this.velocity.x; this.position.y += this.velocity.y; if(this.position.y + this.radius > this.boundsHeight || this.position.y - this.radius < 0){ this.velocity.y *= -1; } }
}

class Input { constructor(){ this.keys={}; document.addEventListener('keydown', e => { this.keys[e.key] = true; }); document.addEventListener('keyup', e => { this.keys[e.key] = false; }); } isDown(k){ return !!this.keys[k]; } }

class Renderer {
  constructor(ctx){ this.ctx = ctx; }
  clear(w,h){ this.ctx.fillStyle = Config.visuals.bg; this.ctx.fillRect(0,0,w,h); }
  rect(r,color=Config.visuals.fg){ this.ctx.fillStyle=color; this.ctx.fillRect(r.x, r.y, r.w, r.h); }
  circle(ball,color=Config.visuals.fg){ this.ctx.fillStyle=color; this.ctx.beginPath(); this.ctx.arc(ball.position.x, ball.position.y, ball.radius, 0, Math.PI*2); this.ctx.closePath(); this.ctx.fill(); }
  text(t,x,y,color=Config.visuals.text, size=30){ this.ctx.fillStyle = color; this.ctx.font = size+'px Arial'; this.ctx.fillText(t,x,y); }
  dashedCenter(w,h){ const ctx=this.ctx; ctx.strokeStyle = Config.visuals.centerLine; ctx.lineWidth = 2; ctx.setLineDash([10,10]); ctx.beginPath(); ctx.moveTo(w/2, 0); ctx.lineTo(w/2, h); ctx.stroke(); ctx.setLineDash([]); }
}

// Simple WebAudio SFX (score sound and powerup sound)
class SFX {
  constructor(){ this.ctx = null; this.gain = null; }
  _ensure() { if(!this.ctx){ this.ctx = new (window.AudioContext || window.webkitAudioContext)(); this.gain = this.ctx.createGain(); this.gain.connect(this.ctx.destination); this.gain.gain.value = 0.12; } }
  playTone(freq, time=0.12, type='sine'){ this._ensure(); const o = this.ctx.createOscillator(); o.type = type; o.frequency.value = freq; o.connect(this.gain); o.start(); o.stop(this.ctx.currentTime + time); }
  score(){ this.playTone(880,0.08,'sine'); setTimeout(()=>this.playTone(660,0.08,'sine'), 90); }
  power(){ this.playTone(1320,0.12,'triangle'); }
}

class PowerUp {
  constructor(x,y,type,size){ this.x=x; this.y=y; this.type=type; this.size=size; this.active=true; }
  rect(){ return { x:this.x - this.size/2, y:this.y - this.size/2, w:this.size, h:this.size }; }
}

class Game {
  constructor(canvasEl, restartBtn){
    this.canvas = canvasEl; this.ctx = canvasEl.getContext('2d'); this.renderer = new Renderer(this.ctx); this.input = new Input();
    const { width, height, speed } = Config.paddle;
    this.paddleLeft = new Paddle(8, (Config.canvas.height - height)/2, width, height, speed, Config.canvas.height);
    this.paddleRight = new Paddle(Config.canvas.width - width - 8, (Config.canvas.height - height)/2, width, height, speed, Config.canvas.height);
    this.ball = new Ball(Config.ball.radius, Config.canvas.width, Config.canvas.height);
    this.scores = { p1:0, p2:0 };
    this.gameOver=false; this.paused=false; this.aiEnabled = Config.ai.enabled; this.restartBtn = restartBtn; this.restartBtn.addEventListener('click', ()=>this.restart());
    this.sfx = new SFX();

    // Powerups
    this.powerups = []; this.lastPowerSpawn = performance.now();

    // Win countdown
    this.winCountdown = -1; // seconds remaining

    // Bind loop
    this.loop = this.loop.bind(this);

    // Pause toggle
    document.addEventListener('keydown', e => { if(e.key === Config.keys.pause) { this.togglePause(); } });
  }

  togglePause(){ if(this.gameOver) return; this.paused = !this.paused; if(!this.paused) { /* continue */ } }

  handleInput(){ if(this.gameOver || this.paused) return;
    // Player 1
    if(this.input.isDown(Config.keys.p1Up)) this.paddleLeft.move(-this.paddleLeft.speed);
    if(this.input.isDown(Config.keys.p1Down)) this.paddleLeft.move(this.paddleLeft.speed);

    // Player 2: either human or AI
    if(this.aiEnabled){ this._handleAI(); }
    else { if(this.input.isDown(Config.keys.p2Up)) this.paddleRight.move(-this.paddleRight.speed); if(this.input.isDown(Config.keys.p2Down)) this.paddleRight.move(this.paddleRight.speed); }
  }

  _handleAI(){ // simple tracking AI with difficulty multiplier
    const centerY = this.paddleRight.position.y + this.paddleRight.height/2;
    // compute ideal movement direction
    const dir = (this.ball.position.y - centerY);
    const moveAmt = this.paddleRight.speed * Config.ai.difficulty;
    if(Math.abs(dir) > 10){ this.paddleRight.move(Math.sign(dir) * moveAmt); }
  }

  update(){ if(this.gameOver || this.paused) return; this.ball.update();
    // Paddle collisions
    const hitLeft = this.ball.position.x - this.ball.radius < this.paddleLeft.width + this.paddleLeft.position.x &&
      this.ball.position.y > this.paddleLeft.position.y && this.ball.position.y < this.paddleLeft.position.y + this.paddleLeft.height;
    if(hitLeft){ this.ball.velocity.x = Math.abs(this.ball.velocity.x) * Config.ball.speedupFactor; this.ball.velocity.x *= -1; const delta = this.ball.position.y - (this.paddleLeft.position.y + this.paddleLeft.height/2); this.ball.velocity.y = delta * Config.ball.spinFactor; if(Math.abs(this.ball.velocity.x) > Config.ball.maxSpeedX) this.ball.velocity.x = -Math.sign(this.ball.velocity.x) * Config.ball.maxSpeedX; }

    const hitRight = this.ball.position.x + this.ball.radius > this.paddleRight.position.x &&
      this.ball.position.y > this.paddleRight.position.y && this.ball.position.y < this.paddleRight.position.y + this.paddleRight.height;
    if(hitRight){ this.ball.velocity.x = -Math.abs(this.ball.velocity.x) * Config.ball.speedupFactor; const delta = this.ball.position.y - (this.paddleRight.position.y + this.paddleRight.height/2); this.ball.velocity.y = delta * Config.ball.spinFactor; if(Math.abs(this.ball.velocity.x) > Config.ball.maxSpeedX) this.ball.velocity.x = -Math.sign(this.ball.velocity.x) * Config.ball.maxSpeedX; }

    // Powerup collision with ball
    for(let i=this.powerups.length-1;i>=0;i--){ const p = this.powerups[i]; if(!p.active) continue; if(this._circleRectCollision(this.ball, p.rect())){ this._applyPowerup(p); this.powerups.splice(i,1); this.sfx.power(); } }

    // Scoring
    if(this.ball.position.x - this.ball.radius < 0){ this.scores.p2++; this.sfx.score(); if(this.checkWin()) return; this.ball.reset(true); }
    else if(this.ball.position.x + this.ball.radius > Config.canvas.width){ this.scores.p1++; this.sfx.score(); if(this.checkWin()) return; this.ball.reset(true); }

    // Spawn powerups periodically
    const now = performance.now(); if(now - this.lastPowerSpawn > Config.powerups.spawnInterval){ this._spawnPowerup(); this.lastPowerSpawn = now; }
  }

  _circleRectCollision(ball, rect){ const cx = ball.position.x; const cy = ball.position.y; const rx = rect.x; const ry = rect.y; const rw = rect.w; const rh = rect.h; const nearestX = Math.max(rx, Math.min(cx, rx+rw)); const nearestY = Math.max(ry, Math.min(cy, ry+rh)); const dx = cx - nearestX; const dy = cy - nearestY; return (dx*dx + dy*dy) < (ball.radius * ball.radius); }

  _spawnPowerup(){ // spawn near center random Y
    const x = Math.random() < 0.5 ? Config.canvas.width * 0.25 : Config.canvas.width * 0.75; const y = 40 + Math.random() * (Config.canvas.height - 80);
    const types = ['bigPaddle','fastBall']; const type = types[Math.floor(Math.random()*types.length)]; this.powerups.push(new PowerUp(x,y,type, Config.powerups.size)); }

  _applyPowerup(pu){ if(pu.type === 'bigPaddle'){ // make the paddle that last hit ball bigger temporarily
      // Decide which player gets benefit — the side the ball was moving toward when collected
      const target = this.ball.velocity.x > 0 ? this.paddleRight : this.paddleLeft; target.height *= 1.4; setTimeout(()=>{ target.height = Math.max(40, target.height / 1.4); }, Config.powerups.duration);
    } else if(pu.type === 'fastBall'){ const prev = this.ball.velocity.x; this.ball.velocity.x *= 1.6; setTimeout(()=>{ this.ball.velocity.x = (this.ball.velocity.x > 0 ? Math.abs(prev) : -Math.abs(prev)); }, Config.powerups.duration); }
  }

  checkWin(){ if(this.scores.p1 >= Config.rules.winningScore || this.scores.p2 >= Config.rules.winningScore){ this.gameOver = true; this.restartBtn.style.display = 'inline-block'; // start countdown to auto-restart
      this.winCountdown = 5; // seconds
      // start countdown timer
      const tick = () => { if(this.winCountdown <= 0) { this.restart(); return; } this.winCountdown -= 1; setTimeout(tick, 1000); };
      setTimeout(tick, 1000);
      return true; } return false; }

  draw(){ this.renderer.clear(Config.canvas.width, Config.canvas.height); this.renderer.dashedCenter(Config.canvas.width, Config.canvas.height); // paddles
    this.renderer.rect(this.paddleLeft.rect()); this.renderer.rect(this.paddleRight.rect()); // ball
    this.renderer.circle(this.ball);
    // powerups
    for(const p of this.powerups){ if(!p.active) continue; this.ctx.fillStyle = p.type === 'bigPaddle' ? '#9ad3bc' : '#ffb86b'; const r = p.rect(); this.ctx.fillRect(r.x, r.y, r.w, r.h); }

    // scores
    this.renderer.text(this.scores.p1, Config.canvas.width/4, 50); this.renderer.text(this.scores.p2, 3*Config.canvas.width/4, 50);

    // paused overlay
    if(this.paused){ this.renderer.text('PAUSED', Config.canvas.width/2 - 70, Config.canvas.height/2, '#ffd166', 42); }

    // Game over / win display
    if(this.gameOver){ this.renderer.text('Game Over', Config.canvas.width/2 - 100, Config.canvas.height/2 - 40, Config.visuals.gameOver, 36); const msg = this.scores.p1 >= Config.rules.winningScore ? 'Player 1 Wins!' : 'Player 2 Wins!'; this.renderer.text(msg, Config.canvas.width/2 - 120, Config.canvas.height/2, Config.visuals.win, 32); if(this.winCountdown > 0){ this.renderer.text('Restarting in ' + Math.ceil(this.winCountdown) + '...', Config.canvas.width/2 - 120, Config.canvas.height/2 + 50, Config.visuals.text, 22); } }
  }

  restart(){ this.scores.p1 = 0; this.scores.p2 = 0; this.paddleLeft.height = Config.paddle.height; this.paddleRight.height = Config.paddle.height; this.paddleLeft.position.y = (Config.canvas.height - this.paddleLeft.height)/2; this.paddleRight.position.y = (Config.canvas.height - this.paddleRight.height)/2; this.ball.reset(true); this.gameOver=false; this.paused=false; this.powerups = []; this.restartBtn.style.display = 'none'; this.winCountdown = -1; }

  loop(){ this.handleInput(); this.update(); this.draw(); requestAnimationFrame(this.loop); }
}

// Boot
const canvas = document.getElementById('pongCanvas'); const restartBtn = document.getElementById('restartBtn'); canvas.width = Config.canvas.width; canvas.height = Config.canvas.height; const game = new Game(canvas, restartBtn); game.loop();

// Controls hint: W/S for left player; I/K for right player (if human); Space to pause/resume; Restart button appears on win.

</script>
