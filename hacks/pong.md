---
layout: opencs
title: Ping Pong
description: Use JavaScript to code a ping pong game
permalink: /pong
---
<div style="max-width:900px;margin:18px auto;text-align:center;">
  <h2>Ping Pong</h2>
  <p>Controls: Left paddle — W / S. Right paddle — Up / Down. Press <strong>Space</strong> to pause or start.</p>

  <canvas id="pong" width="800" height="450" style="border:1px solid #ccc; display:block; margin:0 auto; background:#0b1220;">
    Your browser doesn't support canvas.
  </canvas>

  <div style="margin-top:12px;">

    <!-- Mode Selector -->
    <label style="margin-left:12px;">Mode:
      <select id="mode">
        <option value="pvp">2-Player</option>
        <option value="pvai">Player vs AI</option>
      </select>
    </label>

    <!-- Speed Selector (SLOWER VALUES) -->
    <label style="margin-left:12px;">Speed:
      <select id="speed">
        <option value="0.5">Slow</option>
        <option value="0.8" selected>Normal</option>
        <option value="1.1">Fast</option>
        <option value="1.4">Ultra</option>
      </select>
    </label>

    <!-- Restart Button -->
    <button id="restart" style="margin-left:12px;">Restart</button>
  </div>

  <p id="status" style="margin-top:10px;font-family:monospace;color:#ddd;"></p>
</div>

<script>
(function(){
  const canvas = document.getElementById('pong');
  const ctx = canvas.getContext('2d');
  const W = canvas.width, H = canvas.height;

  // Game objects
  const paddle = {w:12, h:90, speed:6};
  const left = {x:20, y:(H-90)/2, dy:0, score:0};
  const right = {x:W-20-12, y:(H-90)/2, dy:0, score:0};
  const ball = {x:W/2, y:H/2, r:9, vx:5, vy:3};

  let paused = true;

  const modeSelect = document.getElementById('mode');
  const speedSelect = document.getElementById('speed');
  const status = document.getElementById('status');

  function resetBall(winner){
    ball.x = W/2;
    ball.y = H/2;

    const base = 5 + Math.random() * 3;
    const angle = (Math.random()*Math.PI/4) - Math.PI/8;

    ball.vx = (winner === "left" ? 1 : -1) * base;
    ball.vy = base * Math.tan(angle);
  }

  function drawRect(x,y,w,h,fill){ ctx.fillStyle = fill; ctx.fillRect(x,y,w,h); }
  function drawCircle(x,y,r,fill){ ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.fillStyle = fill; ctx.fill(); }

  function drawNet(){
    for(let i=10; i<H; i+=22){
      drawRect(W/2 - 1, i, 2, 12, "#2ec4b6");
    }
  }

  function draw(){
    ctx.fillStyle = "#0b1220";
    ctx.fillRect(0,0,W,H);

    drawNet();
    drawRect(left.x, left.y, paddle.w, paddle.h, "#f0f3bd");
    drawRect(right.x, right.y, paddle.w, paddle.h, "#f0f3bd");
    drawCircle(ball.x, ball.y, ball.r, "#ff6b6b");

    ctx.fillStyle = "#c9d6df";
    ctx.font = "28px monospace";
    ctx.fillText(left.score, W*0.25, 40);
    ctx.fillText(right.score, W*0.75, 40);

    status.textContent = paused ? "Paused — press Space to start" : "Running — press Space to pause";
  }

  function clamp(v,a,b){ return Math.max(a, Math.min(b,v)); }

  function update(){
    if(paused) return;

    const SPEED = Number(speedSelect.value);

    // Move paddles (scaled)
    left.y += left.dy * SPEED;
    right.y += right.dy * SPEED;

    left.y = clamp(left.y, 0, H - paddle.h);
    right.y = clamp(right.y, 0, H - paddle.h);

    // AI movement
    if(modeSelect.value === "pvai"){
      const target = ball.y - paddle.h/2;
      const diff = target - right.y;

      right.dy = clamp(diff * 0.08 * SPEED, -paddle.speed * SPEED, paddle.speed * SPEED);
      right.y += right.dy;
    }

    // Ball movement (scaled)
    ball.x += ball.vx * SPEED;
    ball.y += ball.vy * SPEED;

    // Wall collisions
    if(ball.y - ball.r < 0){
      ball.y = ball.r;
      ball.vy *= -1;
    }
    if(ball.y + ball.r > H){
      ball.y = H - ball.r;
      ball.vy *= -1;
    }

    // LEFT paddle hit
    if(ball.x - ball.r < left.x + paddle.w && ball.x - ball.r > left.x){
      if(ball.y > left.y && ball.y < left.y + paddle.h){
        const rel = (ball.y - (left.y + paddle.h/2)) / (paddle.h/2);
        const speed = (Math.hypot(ball.vx, ball.vy) + 0.4) * SPEED;
        const angle = rel * Math.PI/3;

        ball.vx = Math.abs(Math.cos(angle) * speed);
        ball.vy = Math.sin(angle) * speed;
      }
    }

    // RIGHT paddle hit
    if(ball.x + ball.r > right.x && ball.x + ball.r < right.x + paddle.w){
      if(ball.y > right.y && ball.y < right.y + paddle.h){
        const rel = (ball.y - (right.y + paddle.h/2)) / (paddle.h/2);
        const speed = (Math.hypot(ball.vx, ball.vy) + 0.4) * SPEED;
        const angle = rel * Math.PI/3;

        ball.vx = -Math.abs(Math.cos(angle) * speed);
        ball.vy = Math.sin(angle) * speed;
      }
    }

    // Scoring
    if(ball.x < 0){
      right.score++;
      resetBall("right");
      paused = true;
    }
    if(ball.x > W){
      left.score++;
      resetBall("left");
      paused = true;
    }
  }

  // Keyboard controls
  document.addEventListener("keydown", e=>{
    if(e.key === "w") left.dy = -paddle.speed;
    if(e.key === "s") left.dy = paddle.speed;
    if(e.key === "ArrowUp") right.dy = -paddle.speed;
    if(e.key === "ArrowDown") right.dy = paddle.speed;

    if(e.code === "Space") paused = !paused;
  });

  document.addEventListener("keyup", e=>{
    if(e.key === "w" || e.key === "s") left.dy = 0;
    if(e.key === "ArrowUp" || e.key === "ArrowDown") right.dy = 0;
  });

  document.getElementById("restart").onclick = () => {
    left.score = 0;
    right.score = 0;
    resetBall("left");
    paused = true;
  };

  function loop(){
    update();
    draw();
    requestAnimationFrame(loop);
  }

  loop();
})();
</script>
