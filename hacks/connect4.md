---
layout: opencs
title: Connect 4
description: Play Connect 4 in the browser
permalink: /connect4/
---

<div id="app" class="wrap">

  <!-- ===== START SCREEN ===== -->
  <section id="start" class="card center">
    <h1 class="title">Connect 4</h1>
    <p class="muted">Choose a timer per player</p>
    <div class="row">
      <button class="btn" data-time="180">3 Minutes</button>
      <button class="btn" data-time="300">5 Minutes</button>
      <button class="btn" data-time="600">10 Minutes</button>
    </div>
    <p class="press">…or press <kbd>Enter</kbd> to start with 5:00</p>
  </section>

  <!-- ===== GAME SCREEN ===== -->
  <section id="game" class="hidden game-overlay">
    <div class="hud">

      <div class="panel red-side">
        <h2>Red</h2>
        <div class="timer" id="tRed">05:00</div>
        <div class="stash">
          <div class="dot red"></div>
          <span>Coins: <b id="cRed">21</b></span>
        </div>
      </div>

      <div id="boardWrap">
        <div id="board"></div>
        <div id="fall" class="coin hidden"></div>
      </div>

      <div class="panel yellow-side">
        <h2>Yellow</h2>
        <div class="timer" id="tYellow">05:00</div>
        <div class="stash">
          <div class="dot yellow"></div>
          <span>Coins: <b id="cYellow">21</b></span>
        </div>
      </div>

    </div>

    <div class="row center" style="margin-top:20px;">
      <button id="restart" class="btn danger">Restart</button>
    </div>
  </section>

</div>

<!-- ================== STYLES ================== -->
<style>
:root{
  --bg:#0f0f10;
  --card:#17181c;
  --muted:#a9b0be;
  --blue:#1658e5;
  --red:#ef4444;
  --yellow:#facc15;
  --cell:76px;
  --gap:12px;
  --radius:50%;
  --boardPad:18px;
  --boardCols:7;
  --boardRows:6;
}

*{box-sizing:border-box}

.wrap{
  min-height:100vh;
  display:flex;
  align-items:center;
  justify-content:center;
  background:var(--bg);
  color:#fff;
  font-family:system-ui,Segoe UI,Roboto,Inter,Arial;
  padding:24px;
}

.hidden{display:none}
.center{text-align:center; justify-content:center}
.row{display:flex; gap:12px; align-items:center; flex-wrap:wrap}

.card{
  background:var(--card);
  padding:28px;
  border-radius:18px;
  box-shadow:0 10px 30px #0006;
  width:min(720px,95vw);
}

.title{font-size:42px; margin:0 0 8px}
.muted{color:var(--muted)}
.press{color:var(--muted); font-size:14px}

.btn{
  background:#2b2f3a;
  border:1px solid #3a4151;
  color:#fff;
  padding:10px 16px;
  border-radius:12px;
  cursor:pointer;
  font-weight:600;
}

.btn.danger{
  background:#b71c1c;
  border-color:#c72a2a;
}

.game-overlay{
  position:fixed;
  inset:0;
  background:var(--bg);
  z-index:1000;
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
}

.hud{
  display:grid;
  grid-template-columns:1fr auto 1fr;
  gap:18px;
  align-items:center;
}

.panel{
  background:#14161b;
  border:1px solid #272c38;
  border-radius:16px;
  padding:16px;
  display:flex;
  flex-direction:column;
  align-items:center;
  gap:10px;
}

.timer{font-size:38px}

.dot{
  width:18px;
  height:18px;
  border-radius:50%;
}

.red{background:var(--red)}
.yellow{background:var(--yellow)}

#boardWrap{
  position:relative;
  padding:var(--boardPad);
  background:var(--blue);
  border-radius:22px;
}

#board{
  display:grid;
  gap:var(--gap);
  grid-template-columns:repeat(7,var(--cell));
  grid-template-rows:repeat(6,var(--cell));
}

.hole{
  width:var(--cell);
  height:var(--cell);
  border-radius:50%;
  background:#f1f5f9;
  box-shadow:inset 0 0 0 6px #0e3ea3;
  cursor:pointer;
}

.hole.filled.red{background:var(--red)}
.hole.filled.yellow{background:var(--yellow)}

.coin{
  position:absolute;
  width:var(--cell);
  height:var(--cell);
  border-radius:50%;
}
</style>

<!-- ================== GAME SCRIPT ================== -->
<script>
/* YOUR GAME LOGIC IS UNCHANGED */
(() => {
  const game = new Connect4Game();
})();
</script>
