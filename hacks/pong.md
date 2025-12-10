---
layout: opencs
title: Ping Pong
description: Use JavaScript to code a ping pong game
permalink: /pong
---

<div style="max-width:900px;margin:18px auto;text-align:center;">
  <h2>Ping Pong</h2>
  <p>Controls: Left paddle — W / S. Right paddle — Up / Down. Space to start.</p>

  <canvas id="pong" width="800" height="450" style="border:1px solid #ccc; display:block; margin:0 auto; background:#0b1220;">
    Your browser doesn't support canvas.
  </canvas>
</div>
<div style="margin-top:12px;">
  <label>Mode:
    <select id="mode">
      <option value="pvp">2-Player</option>
      <option value="pvai">Player vs AI</option>
    </select>
  </label>

  <label style="margin-left:12px;">Speed:
    <select id="speed">
      <option value="0.5">Slow</option>
      <option value="0.8" selected>Normal</option>
      <option value="1.1">Fast</option>
      <option value="1.4">Ultra</option>
    </select>
  </label>

  <button id="restart" style="margin-left:12px;">Restart</button>
</div>

<p id="status" style="margin-top:10px;font-family:monospace;color:#ddd;"></p>
<script>
(function(){
  const canvas = document.getElementById('pong');
  const ctx = canvas.getContext('2d');
  const W = canvas.width, H = canvas.height;

  const paddle = {w:12, h:90, speed:6};
  const left = {x:20, y:(H-90)/2, dy:0, score:0};
  const right = {x:W-32, y:(H-90)/2, dy:0, score:0};

  const ball = {x:W/2, y:H/2, r:9, vx:5, vy:3};

  function drawRect(x,y,w,h,c){ ctx.fillStyle=c; ctx.fillRect(x,y,w,h); }
  function drawCircle(x,y,r,c){ ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.fillStyle=c; ctx.fill(); }

  function drawNet(){
    for(let i=10;i<H;i+=22){
      drawRect(W/2 - 1, i, 2, 12, "#2ec4b6");
    }
  }

  function draw(){
    ctx.fillStyle = "#0b1220";
    ctx.fillRect(0,0,W,H);
    drawNet();
    drawRect(left.x,left.y,paddle.w,paddle.h,"#f0f3bd");
    drawRect(right.x,right.y,paddle.w,paddle.h,"#f0f3bd");
    drawCircle(ball.x,ball.y,ball.r,"#ff6b6b");
  }
  function clamp(v,a,b){ return Math.max(a, Math.min(b,v)); }

  function resetBall(side){
    ball.x = W/2;
    ball.y = H/2;
    const speed = 5 + Math.random()*3;
    const angle = (Math.random()*Math.PI/4) - Math.PI/8;

    ball.vx = (side === "left" ? 1 : -1) * speed;
    ball.vy = speed * Math.tan(angle);
  }

  function update(){
    ball.x += ball.vx * Number(speedSelect.value);
    ball.y += ball.vy * Number(speedSelect.value);

    if(ball.y - ball.r < 0 || ball.y + ball.r > H){
      ball.vy *= -1;
    }

    if(ball.x < 0){
      right.score++;
      resetBall("right");
    }

    if(ball.x > W){
      left.score++;
      resetBall("left");
    }
  }
