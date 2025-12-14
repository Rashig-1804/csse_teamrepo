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

<!-- ================== FULL GAME SCRIPT ================== -->
<script>
// ========= PLAYER CLASS =========
class Player {
  constructor(name, color) {
    this.name = name;
    this.color = color;
    this.time = 300; // default 5 minutes
    this.coins = 21;
  }

  setTime(seconds){ this.time = seconds; }
  usesCoin(){ if(this.coins>0){ this.coins--; return true; } return false; }
  hasTimeLeft(){ return this.time>0; }
  decrementTime(){ if(this.time>0) this.time--; }
  reset(time){ this.time=time; this.coins=21; }
}

// ========= GAME BOARD =========
class GameBoard {
  constructor(rows=6, cols=7){
    this.rows=rows; this.cols=cols;
    this.initialize();
  }
  initialize(){
    this.grid = Array.from({length:this.rows},()=>Array(this.cols).fill(null));
  }
  isValidColumn(col){ return col>=0 && col<this.cols && this.grid[0][col]===null; }
  getDropRow(col){
    if(!this.isValidColumn(col)) return -1;
    for(let r=this.rows-1;r>=0;r--) if(this.grid[r][col]===null) return r;
    return -1;
  }
  placePiece(row,col,color){ if(row>=0&&row<this.rows&&col>=0&&col<this.cols){ this.grid[row][col]=color; return true;} return false; }
  checkWin(row,col){
    const color=this.grid[row][col]; if(!color) return false;
    const dirs=[[1,0],[0,1],[1,1],[1,-1]];
    for(const [dr,dc] of dirs){
      let count=1;
      for(const dir of [-1,1]){
        let r=row+dr*dir,c=col+dc*dir;
        while(r>=0&&r<this.rows&&c>=0&&c<this.cols&&this.grid[r][c]===color){
          count++; r+=dr*dir; c+=dc*dir;
        }
      }
      if(count>=4) return true;
    }
    return false;
  }
  isFull(){ return this.grid.every(row=>row.every(cell=>cell!==null)); }
  reset(){ this.initialize(); }
}

// ========= GAME TIMER =========
class GameTimer{
  constructor(){ this.intervalId=null; this.onTick=null; this.onTimeUp=null; }
  start(tickCb,timeUpCb){
    this.onTick=tickCb; this.onTimeUp=timeUpCb;
    this.intervalId=setInterval(()=>{if(this.onTick) this.onTick();},1000);
  }
  stop(){ if(this.intervalId){clearInterval(this.intervalId); this.intervalId=null;} }
  reset(){ this.stop(); }
}

// ========= GAME UI =========
class GameUI {
  constructor(){
    this.elements={
      start:document.getElementById('start'),
      game:document.getElementById('game'),
      board:document.getElementById('board'),
      fallCoin:document.getElementById('fall'),
      redTimer:document.getElementById('tRed'),
      yellowTimer:document.getElementById('tYellow'),
      redCoins:document.getElementById('cRed'),
      yellowCoins:document.getElementById('cYellow'),
      restartBtn:document.getElementById('restart')
    };
  }
  showStartScreen(){ this.elements.start.classList.remove('hidden'); this.elements.game.classList.add('hidden'); }
  showGameScreen(){ this.elements.start.classList.add('hidden'); this.elements.game.classList.remove('hidden'); }
  createBoard(rows,cols){
    this.elements.board.innerHTML='';
    for(let r=0;r<rows;r++) for(let c=0;c<cols;c++){
      const hole=document.createElement('div');
      hole.className='hole'; hole.dataset.col=c;
      this.elements.board.appendChild(hole);
    }
  }
  updateBoard(board){
    const holes=[...this.elements.board.children];
    holes.forEach((hole,index)=>{
      const row=Math.floor(index/board.cols);
      const col=index%board.cols;
      const piece=board.grid[row][col];
      hole.classList.remove('filled','red','yellow');
      if(piece) hole.classList.add('filled',piece);
    });
  }
  updatePlayerInfo(red,yellow){
    this.elements.redTimer.textContent=this.formatTime(red.time);
    this.elements.yellowTimer.textContent=this.formatTime(yellow.time);
    this.elements.redCoins.textContent=red.coins;
    this.elements.yellowCoins.textContent=yellow.coins;
  }
  formatTime(s){ const m=Math.floor(Math.max(0,s)/60), sec=Math.max(0,s)%60; return `${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`; }
  animateFallingCoin(col,row,color){
    const cell=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--cell'));
    const gap=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--gap'));
    const pad=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--boardPad'));
    const x=pad+col*(cell+gap);
    const y=pad+row*(cell+gap);
    const el=this.elements.fallCoin;
    el.className=`coin ${color}`;
    el.style.left=x+'px';
    el.style.transform='translate(0,-100%)';
    el.classList.remove('hidden');
    void el.offsetWidth;
    const dur=Math.min(120+row*120,520);
    el.style.transitionDuration=`${dur}ms`;
    el.style.transform=`translate(0,${y}px)`;
    return new Promise(resolve=>{
      const f=()=>{el.removeEventListener('transitionend',f); el.classList.add('hidden'); resolve();}
      el.addEventListener('transitionend',f,{once:true});
    });
  }
  showWinMessage(msg){
    const overlay=document.createElement('div'); overlay.className='win-overlay';
    overlay.innerHTML=`<div class="win-card"><h2 class="win-title">${msg}</h2><button class="btn win-btn" onclick="this.parentElement.parentElement.remove()">Continue</button></div>`;
    document.body.appendChild(overlay);
  }
}

// ========= MAIN GAME =========
class Connect4Game{
  constructor(){
    this.board=new GameBoard();
    this.redPlayer=new Player('Red','red');
    this.yellowPlayer=new Player('Yellow','yellow');
    this.currentPlayer=this.redPlayer;
    this.timer=new GameTimer();
    this.ui=new GameUI();
    this.isRunning=false;
    this.isAnimating=false;
    this.initializeEventListeners();
  }
  initializeEventListeners(){
    this.ui.elements.start.querySelectorAll('.btn').forEach(btn=>{
      btn.addEventListener('click'=>this.startGame(parseInt(btn.dataset.time)));
    });
    window.addEventListener('keydown', e=>{
      if(!this.ui.elements.game.classList.contains('hidden') || e.key!=='Enter') return;
      this.startGame(300);
    });
    this.ui.elements.board.addEventListener('click', e=>this.handleBoardClick(e));
    this.ui.elements.restartBtn.addEventListener('click', ()=>this.restart());
  }
  startGame(time){
    this.board.reset();
    this.redPlayer.reset(time); this.yellowPlayer.reset(time);
    this.currentPlayer=this.redPlayer;
    this.isRunning=true; this.isAnimating=false;
    this.ui.showGameScreen();
    this.ui.createBoard(this.board.rows,this.board.cols);
    this.ui.updateBoard(this.board);
    this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
    this.timer.start(()=>this.handleTimerTick(), player=>this.handleTimeUp(player));
  }
  async handleBoardClick(e){
    const hole=e.target.closest('.hole'); if(!hole||!this.isRunning||this.isAnimating) return;
    const col=parseInt(hole.dataset.col), row=this.board.getDropRow(col);
    if(row<0) return; if(!this.currentPlayer.usesCoin()) return;
    this.isAnimating=true;
    await this.ui.animateFallingCoin(col,row,this.currentPlayer.color);
    this.board.placePiece(row,col,this.currentPlayer.color);
    this.ui.updateBoard(this.board);
    this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
    this.isAnimating=false;
    if(this.board.checkWin(row,col)){ this.endGame(`${this.currentPlayer.name} wins!`); return;}
    if(this.board.isFull()){ this.endGame('Draw!'); return;}
    this.switchPlayer();
  }
  handleTimerTick(){
    if(!this.isRunning) return;
    this.currentPlayer.decrementTime();
    if(!this.currentPlayer.hasTimeLeft()){
      const winner=this.currentPlayer===this.redPlayer?this.yellowPlayer:this.redPlayer;
      this.endGame(`${winner.name} wins on time!`); return;
    }
    this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
  }
  switchPlayer(){ this.currentPlayer=this.currentPlayer===this.redPlayer?this.yellowPlayer:this.redPlayer; }
  endGame(msg){ this.isRunning=false; this.timer.stop(); this.ui.showWinMessage(msg); }
  restart(){ this.timer.stop(); this.ui.showStartScreen(); this.isRunning=false; }
}

// ========= START GAME =========
(() => { const game=new Connect4Game(); })();
</script>
