---
layout: opencs
title: Rock Paper Scissors Game
description: Playing rock paper scissors using javascript code
permalink: /rockpaper
---

<div id="mainGameBox" style="max-width:700px;margin:64px auto 48px auto;position:relative;z-index:2;">
  <div id="gameContainer">
    <canvas id='gameCanvas' style="display:none"></canvas>
  </div>
</div>

<script type="module">
  // === ADDED: shared shake animation CSS (inserted dynamically) ===
  (function addShakeStyles(){
    const style = document.createElement("style");
    style.innerHTML = `
      @keyframes shake {
        0% { transform: translateX(0); }
        20% { transform: translateX(-6px); }
        40% { transform: translateX(6px); }
        60% { transform: translateX(-4px); }
        80% { transform: translateX(4px); }
        100% { transform: translateX(0); }
      }
      .rps-shake {
        animation: shake 0.36s ease-in-out;
      }
    `;
    document.head.appendChild(style);
  })();

  // --- UI (purple box) ---
  const instructionsStyle = `
    position: relative;
    margin: 64px auto 48px auto;
    background: linear-gradient(135deg, black, purple);
    color: white;
    padding: 30px;
    border-radius: 15px;
    z-index: 1000;
    max-width: 600px;
    width: 90%;
    max-height: 80vh;
    overflow-y: auto;
    font-family: 'Press Start 2P', cursive;
    border: 3px solid purple;
    box-shadow: 0 0 20px rgba(128, 0, 128, 0.5);
    text-align: center;
  `;

  const instructionsHTML = `
    <h2 style="color: purple; margin-bottom: 20px;">Rock Paper Scissors SHOOT!</h2>
    <div style="margin-bottom: 20px;">
      <p>Play the game from your browser console!</p>
      <p>Type <code>playRPS("rock")</code>, <code>playRPS("paper")</code>, or <code>playRPS("scissors")</code></p>
    </div>
    <div id="images" style="display:flex; justify-content:center; gap:20px; margin-bottom:14px;">
      <button id="rock-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="rock-img" src="{{site.baseurl}}/images/rps/rock.jpg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
      <button id="paper-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="paper-img" src="{{site.baseurl}}/images/rps/paper.jpeg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
      <button id="scissors-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="scissors-img" src="{{site.baseurl}}/images/rps/scissors.jpeg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
    </div>
    <div style="margin-bottom:18px; font-size:1.1em; color:#ffd700;">
      Click any icon to customize using the console!
    </div>
    <!-- mount battle canvas INSIDE the purple box so you can see it -->
    <div id="battleMount" style="display:block; margin:12px auto;"></div>

    <div id="resultBox" style="margin-top: 16px; font-size: 16px; color: yellow;"></div>
  `;
  const container = document.createElement("div");
  container.setAttribute("style", instructionsStyle);
  container.innerHTML = instructionsHTML;
  document.getElementById("mainGameBox").appendChild(container);

  // --- helper: highlight chosen images (accepts string or array) ---
  function highlightImage(ids){
    // normalize to array
    const arr = Array.isArray(ids) ? ids : [ids];
    ["rock-img","paper-img","scissors-img"].forEach(i=>{
      const el = document.getElementById(i);
      if(el) {
        // clear previous highlight, keep original border
        el.style.boxShadow = "";
        el.style.filter = "";
      }
    });
    arr.forEach(id=>{
      const picked = document.getElementById(id);
      if(picked){
        picked.style.boxShadow = "0 0 30px 10px gold";
        // small pop for visibility
        picked.style.filter = "brightness(1.06) saturate(1.05)";
      }
    });
  }

  // --- OOP classes ---
  class BattleBackground {
    constructor(image, width, height, speedRatio=0.1){
      this.image = image;
      this.width = width;
      this.height = height;
      this.x = 0; this.y = 0;
      this.speed = 2 * speedRatio;
    }
    update(){ this.x = (this.x - this.speed) % this.width; }
    draw(ctx){
      if(!this.image.complete || this.image.naturalWidth===0) return;
      ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
      ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
    }
  }

  class BattleSprite {
    constructor(image, width, height, x, y){
      this.image = image;
      this.width = width; this.height = height;
      this.homeX = x; this.homeY = y;
      this.x = x; this.y = y;
      this.targetX = x; this.targetY = y;
      this.opacity = 1; this.scale = 1; this.rotation = 0;
      this.animating = false;
    }
    update(){
      if(this.animating){
        this.x += (this.targetX - this.x)*0.12;
        this.y += (this.targetY - this.y)*0.12;
      } else {
        // drift gently back to home
        this.x += (this.homeX - this.x)*0.08;
        this.y += (this.homeY - this.y)*0.08;
      }
    }
    draw(ctx){
      if(!this.image.complete || this.image.naturalWidth===0) return;
      ctx.save();
      ctx.globalAlpha = this.opacity;
      ctx.translate(this.x + this.width/2, this.y + this.height/2);
      ctx.rotate(this.rotation);
      ctx.scale(this.scale, this.scale);
      ctx.drawImage(this.image, -this.width/2, -this.height/2, this.width, this.height);
      ctx.restore();
    }
    resetVisuals(){
      this.opacity = 1; this.scale = 1; this.rotation = 0;
    }
    resetPosition(){
      this.x = this.homeX; this.y = this.homeY;
      this.targetX = this.homeX; this.targetY = this.homeY;
      this.animating = false;
    }
  }

  // --- Canvas mounted inside purple box ---
  const battleCanvas = document.createElement('canvas');
  battleCanvas.width = 360;
  battleCanvas.height = 180;
  battleCanvas.style.display = 'block';
  battleCanvas.style.margin = '0 auto';
  battleCanvas.style.background = '#111';
  battleCanvas.style.borderRadius = '12px';
  battleCanvas.style.boxShadow = '0 2px 12px rgba(0,0,0,0.18)';
  document.getElementById('battleMount').appendChild(battleCanvas);
  const ctx = battleCanvas.getContext('2d');

  // --- assets ---
  const bgImage = new Image();
  bgImage.src = '{{site.baseurl}}/images/platformer/backgrounds/alien_planet1.jpg';

  const rockImg = new Image();
  rockImg.src = '{{site.baseurl}}/images/rps/rock.jpg';
  const paperImg = new Image();
  paperImg.src = '{{site.baseurl}}/images/rps/paper.jpeg';
  const scissorsImg = new Image();
  scissorsImg.src = '{{site.baseurl}}/images/rps/scissors.jpeg';

  const bg = new BattleBackground(bgImage, battleCanvas.width, battleCanvas.height, 0.12);

  const sprites = {
    rock:     new BattleSprite(rockImg,     96, 96,  10, 42),
    paper:    new BattleSprite(paperImg,    96, 96, 132, 42),
    scissors: new BattleSprite(scissorsImg, 96, 96, 254, 42)
  };

  function resetAll(){
    Object.values(sprites).forEach(s=>{
      s.resetVisuals();
      s.resetPosition();
    });
    // ensure UI highlights cleared
    ["rock-img","paper-img","scissors-img"].forEach(i=>{
      const el = document.getElementById(i);
      if(el){ el.style.boxShadow = ""; el.style.filter = ""; }
    });
    battleCanvas.style.boxShadow = '0 2px 12px rgba(0,0,0,0.18)';
  }

  // --- global battle state, rendered by a continuous loop ---
  const battle = {
    active: false,
    winner: null,
    loser: null,
    frames: 0,
    max: 120,
    tie: null
  };

  // --- WebAudio simple tones for feedback (no files required) ---
  const audioCtx = (typeof window.AudioContext !== "undefined") ? new AudioContext() : null;
  function playTone(type){
    if(!audioCtx) return;
    const now = audioCtx.currentTime;
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.connect(g); g.connect(audioCtx.destination);
    // short envelope
    g.gain.setValueAtTime(0, now);
    g.gain.linearRampToValueAtTime(0.18, now + 0.01);
    g.gain.exponentialRampToValueAtTime(0.001, now + 0.25);

    if(type === 'win') o.frequency.value = 880;
    else if(type === 'tie') o.frequency.value = 440;
    else o.frequency.value = 220; // lose
    o.type = 'sine';
    o.start(now);
    o.stop(now + 0.26);
  }

  function startBattle(winner, loser){
    battle.active = true;
    battle.tie = null;
    battle.winner = winner;
    battle.loser = loser;
    battle.frames = 0;

    // set targets for "winner moves toward loser"
    sprites[winner].animating = true;
    sprites[winner].targetX = sprites[loser].homeX;
    sprites[winner].targetY = sprites[loser].homeY;

    // winner "jump" effect: move up slightly then return
    sprites[winner].targetY = sprites[loser].homeY - 10;
    setTimeout(()=>{ // return after a short delay
      sprites[winner].targetY = sprites[loser].homeY;
    }, 240);

    // loser will fade/scale/rotate in the render loop
    sprites[loser].animating = false; // stays put, gets affected visually

    // flash the canvas briefly (visible on any result)
    battleCanvas.style.boxShadow = "0 0 28px 6px cyan";
    setTimeout(()=> {
      // leave original style until battle ends; will be cleared in render end
      battleCanvas.style.boxShadow = "0 2px 12px rgba(0,0,0,0.18)";
    }, 260);
  }

  function startTie(choice){
    battle.active = true;
    battle.tie = choice;
    battle.winner = null;
    battle.loser = null;
    battle.frames = 0;

    // small wiggle, no target move
    Object.values(sprites).forEach(s=>{ s.animating = false; });
    // canvas flash for tie
    battleCanvas.style.boxShadow = "0 0 20px 5px purple";
    setTimeout(()=> battleCanvas.style.boxShadow = "0 2px 12px rgba(0,0,0,0.18)", 260);
  }

  // --- continuous render loop (always runs) ---
  function render(){
    ctx.clearRect(0,0,battleCanvas.width,battleCanvas.height);
    bg.update();  bg.draw(ctx);

    // Draw 'Animated Battle: OOP' text (smaller)
    ctx.save();
    ctx.font = "bold 14px 'Press Start 2P', cursive";
    ctx.fillStyle = "cyan";
    ctx.textAlign = "center";
    ctx.fillText("Animated Battle: OOP", battleCanvas.width/2, 24);
    ctx.restore();

    if(battle.active){
      const t = battle.frames / battle.max; // 0..1

      if(battle.tie){
        const wobble = Math.sin(battle.frames*0.3)*4;
        sprites[battle.tie].rotation = wobble * Math.PI/180;
      } else {
        // winner punch-in / pulse
        const w = sprites[battle.winner];
        const l = sprites[battle.loser];

        // winner pulse scale up then down
        const pulse = (battle.frames < battle.max/2)
          ? 1 + (battle.frames/(battle.max/2))*0.2
          : 1.2 - ((battle.frames - battle.max/2)/(battle.max/2))*0.2;
        w.scale = pulse;

        // loser fades & shrinks
        l.opacity = Math.max(0.15, 1 - t*0.85);
        l.scale   = Math.max(0.6, 1 - t*0.4);

        // matchup-specific flair
        if(battle.winner === "rock" && battle.loser === "scissors"){
          l.rotation = -t * (Math.PI/4);
        }
        if(battle.winner === "paper" && battle.loser === "rock"){
          // paper "covers" rock by moving slightly past center
          w.targetX = l.homeX - 6; w.targetY = l.homeY - 6;
        }
        if(battle.winner === "scissors" && battle.loser === "paper"){
          w.rotation =  t * (Math.PI/10);
          l.rotation = -t * (Math.PI/10);
        }
      }

      battle.frames++;
      if(battle.frames >= battle.max){
        battle.active = false;
        Object.values(sprites).forEach(s=>{ s.resetVisuals(); s.animating = false; s.resetPosition(); });
        // clear image highlights and canvas flash
        ["rock-img","paper-img","scissors-img"].forEach(i=>{
          const el = document.getElementById(i);
          if(el){ el.style.boxShadow = ""; el.style.filter = ""; }
        });
        battleCanvas.style.boxShadow = "0 2px 12px rgba(0,0,0,0.18)";
      }
    }

    // update/draw sprites every frame
    Object.values(sprites).forEach(s=>{ s.update(); s.draw(ctx); });

    requestAnimationFrame(render);
  }
  render(); // kick off the engine once

  // --- game logic + console entry point ---
  window.playRPS = function(playerChoice){
    const choices = ["rock","paper","scissors"];
    if(!choices.includes(playerChoice)){
      console.log("Invalid choice. Use 'rock', 'paper', or 'scissors'.");
      return;
    }

    // highlight player's choice
    highlightImage(playerChoice + "-img");

    const computerChoice = choices[Math.floor(Math.random()*choices.length)];
    // also highlight computer choice (we pass both)
    highlightImage([playerChoice + "-img", computerChoice + "-img"]);

    let resultText, winner=null, loser=null;

    if(playerChoice === computerChoice){
      resultText = "Tie!";
      startTie(playerChoice);
      playTone('tie');
    } else if(
      (playerChoice==="rock" && computerChoice==="scissors") ||
      (playerChoice==="paper" && computerChoice==="rock") ||
      (playerChoice==="scissors" && computerChoice==="paper")
    ){
      resultText = "You Win!";
      winner = playerChoice; loser = computerChoice;
      startBattle(winner, loser);
      playTone('win');
    } else {
      resultText = "You Lose!";
      winner = computerChoice; loser = playerChoice;
      startBattle(winner, loser);
      playTone('lose');
    }

    document.getElementById("resultBox").innerHTML = `
      <p>You chose: <b>${playerChoice.toUpperCase()}</b></p>
      <p>Computer chose: <b>${computerChoice.toUpperCase()}</b></p>
      <h3 style="color: cyan;">${resultText}</h3>
    `;

    if(winner && loser) {
      // ensure both highlighted while animation runs
      highlightImage([winner + "-img", loser + "-img"]);
    }

    console.log(`You chose: ${playerChoice.toUpperCase()}`);
    console.log(`Computer chose: ${computerChoice.toUpperCase()}`);
    console.log(`Result: ${resultText}`);
  };

  // === OOP: GameObject + subclasses (with new shake method) ===
  class GameObject {
    constructor(id) {
      this.el = document.getElementById(id);
      if (!this.el) throw new Error(`Element #${id} not found`);
    }

    rotate(deg) {
      this.el.style.transform = `rotate(${deg}deg)`;
      return this;
    }

    setBorder(style) {
      this.el.style.border = style;
      return this;
    }

    setWidth(px) {
      this.el.style.width = `${px}px`;
      return this;
    }

    setColor(color) {
      this.el.style.backgroundColor = color;
      return this;
    }

    reset() {
      this.el.style.transform = "";
      this.el.style.border = "";
      this.el.style.width = "";
      this.el.style.backgroundColor = "";
      return this;
    }

    // --- NEW: shake animation usable from console (returns this for chaining) ---
    shake() {
      if(!this.el) return this;
      // toggle a class so repeated calls still animate
      this.el.classList.remove('rps-shake');
      // force reflow to restart animation
      void this.el.offsetWidth;
      this.el.classList.add('rps-shake');
      // remove after animation ends so it's clean
      setTimeout(()=> this.el.classList.remove('rps-shake'), 420);
      return this;
    }
  }

  // --- Specialized classes (extend GameObject) ---
  class Rock extends GameObject {
    constructor() { super("rock-img"); }
  }

  class Paper extends GameObject {
    constructor() { super("paper-img"); }
  }

  class Scissors extends GameObject {
    constructor() { super("scissors-img"); }
  }

  // --- Instances (global) ---
  const rock = new Rock();
  const paper = new Paper();
  const scissors = new Scissors();

  // expose for console use
  window.rock = rock;
  window.paper = paper;
  window.scissors = scissors;

  // --- inspect-learning alerts (unchanged) ---
  document.getElementById("rock-btn").addEventListener("click", () => {
    alert("🪨 Try in the console:\n\nrock.setBorder('4px solid lime');\nrock.shake();");
  });
  document.getElementById("paper-btn").addEventListener("click", () => {
    alert("📄 Try in the console:\n\npaper.rotate(15);\npaper.shake();");
  });
  document.getElementById("scissors-btn").addEventListener("click", () => {
    alert("✂️ Try in the console:\n\nscissors.setWidth(150);\nscissors.shake();");
  });

  // Ensure reset when page/tab hidden or reloaded
  window.addEventListener('beforeunload', resetAll);
  document.addEventListener('visibilitychange', ()=>{ if(document.hidden) resetAll(); });

</script>
