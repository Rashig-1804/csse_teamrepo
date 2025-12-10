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
