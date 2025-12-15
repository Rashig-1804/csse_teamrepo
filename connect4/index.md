---
layout: opencs
title: Connect 4
description: Play Connect 4 in the browser
permalink: /connect4/
---
 

<div id="app" class="wrap">c

	<!-- ===== START SCREEN ===== -->
	<section id="start" class="card center">
		<h1 class="title">Connect 4</h1>
		<p class="muted">Choose a timer per player</p>
		<div class="row">
			<button class="btn" data-time="180">3 Minutes</button>
			<button class="btn" data-time="300">5 Minutes</button>
			<button class="btn" data-time="600">10 Minutes</button>
		</div>
		<div class="row" style="margin-top:8px; align-items:center">
			<label style="display:flex; gap:8px; align-items:center; font-size:14px; color:var(--muted)">
				<input type="checkbox" id="vsRobot"> Play vs Computer
			</label>
			<label style="display:flex; gap:8px; align-items:center; font-size:14px; color:var(--muted)">
				Theme:
				<select id="themeSelect" style="margin-left:6px; padding:6px; border-radius:6px; background:#222; color:#fff; border:1px solid #333">
					<option value="dark">Dark</option>
					<option value="light">Light</option>
					<option value="retro">Retro</option>
				</select>
			</label>
		</div>
		<p class="press">Click a time to start (or press <kbd>Enter</kbd>). During the game click any column to drop a coin.</p>
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
.wrap{min-height:100vh; display:flex; align-items:center; justify-content:center; background:var(--bg); color:#fff; font-family:system-ui,Segoe UI,Roboto,Inter,Arial; padding:24px;}
.hidden{display:none}
.center{text-align:center; justify-content:center}
.row{display:flex; gap:12px; align-items:center; flex-wrap:wrap}
.card{background:var(--card); padding:28px; border-radius:18px; box-shadow:0 10px 30px #0006; width:min(720px,95vw);}
.title{font-size:42px; margin:0 0 8px}
.muted{color:var(--muted)}
.press{color:var(--muted); font-size:14px}
kbd{background:#222; padding:2px 6px; border-radius:6px; border:1px solid #333}
.btn{background:#2b2f3a; border:1px solid #3a4151; color:#fff; padding:10px 16px; border-radius:12px; cursor:pointer; font-weight:600; transition:transform .06s ease, background .2s, border .2s;}
.btn:hover{transform:translateY(-1px); background:#323849}
.btn:active{transform:translateY(0)}
.btn.danger{background:#b71c1c; border-color:#c72a2a}
.btn.danger:hover{background:#d32626}
.game-overlay{position:fixed; inset:0; background:var(--bg); z-index:1000; display:flex; flex-direction:column; align-items:center; justify-content:center; padding:24px;}
.hud{display:grid; grid-template-columns:1fr auto 1fr; gap:18px; align-items:center;}
.panel{background:#14161b; border:1px solid #272c38; border-radius:16px; padding:16px; display:flex; flex-direction:column; align-items:center; gap:10px; min-width:160px;}
.panel h2{margin:0; letter-spacing:.5px}
.red-side h2{color:var(--red)} .yellow-side h2{color:var(--yellow)}
.timer{font-size:38px}
.stash{display:flex; align-items:center; gap:10px; color:#d6dbe6}
.dot{width:18px; height:18px; border-radius:50%}
.red{background:var(--red)}
.yellow{background:var(--yellow)}
#boardWrap{position:relative; padding:var(--boardPad); background:var(--blue); border-radius:22px;}
#board{display:grid; gap:var(--gap); grid-template-columns:repeat(var(--boardCols),var(--cell)); grid-template-rows:repeat(var(--boardRows),var(--cell));}
.hole{width:var(--cell); height:var(--cell); border-radius:var(--radius); background:#f1f5f9; position:relative; overflow:hidden; box-shadow:inset 0 0 0 6px #0e3ea3; cursor:pointer; transition:filter .2s;}
.hole:hover{filter:brightness(0.95)}
.hole.filled::after{content:""; position:absolute; inset:0; border-radius:var(--radius);}
.hole.filled.red::after{background:var(--red)}
.hole.filled.yellow::after{background:var(--yellow)}
.coin{position:absolute; width:var(--cell); height:var(--cell); border-radius:50%; left:0; top:0; transform:translate(0,-100%); will-change:transform; box-shadow:0 4px 10px #0007, inset 0 -6px 0 #0003; transition:transform .28s cubic-bezier(.2,.9,.2,1);} 
.coin.red{background:var(--red)}
.coin.yellow{background:var(--yellow)}
.win-overlay{position: fixed; top:0; left:0; width:100vw; height:100vh; background: rgba(0,0,0,0.8); z-index:2000; display:flex; align-items:center; justify-content:center; backdrop-filter: blur(8px); animation: fadeIn .3s ease-out;}
.win-card{background: var(--card); padding:40px; border-radius:20px; text-align:center; box-shadow:0 20px 60px rgba(0,0,0,.5); transform:scale(.9); animation: popIn .4s ease-out .1s forwards; border:2px solid #3a4151;}
.win-title{font-size:36px; margin:0 0 24px; color:#fff; text-shadow:0 2px 4px rgba(0,0,0,.3);}
.win-btn{font-size:18px; padding:12px 24px; background:var(--blue); border-color:#1658e5;}
.win-btn:hover{background:#1d6bf0}
/* Theme variants (applied to document via data-theme attribute) */
[data-theme="light"]{
	--bg:#f6f8fb; --card:#ffffff; --muted:#6b7280; --blue:#0f62fe; --red:#d32f2f; --yellow:#f59e0b; --cell:76px;
}
[data-theme="retro"]{
	--bg:#f4ecd8; --card:#fff7e6; --muted:#6b4f2b; --blue:#2b6f7e; --red:#b22222; --yellow:#e0b72b; --cell:76px;
}
@keyframes fadeIn{from{opacity:0;} to{opacity:1;}}
@keyframes popIn{from{transform:scale(.8); opacity:0;} to{transform:scale(1); opacity:1;}}
</style>

<script>
// ========= PLAYER CLASS =========
class Player {
 	constructor(name,color,isRobot=false){this.name=name; this.color=color; this.time=300; this.coins=21; this.isRobot=!!isRobot;}
	setTime(s){this.time=s;}
	usesCoin(){if(this.coins>0){this.coins--; return true;} return false;}
	hasTimeLeft(){return this.time>0;}
	decrementTime(){if(this.time>0)this.time--;}
	reset(t){this.time=t; this.coins=21;}
}

// ========= GAME BOARD =========
class GameBoard{
	constructor(rows=6,cols=7){this.rows=rows; this.cols=cols; this.grid=[]; this.initialize();}
	initialize(){this.grid=Array.from({length:this.rows},()=>Array(this.cols).fill(null));}
	isValidColumn(col){return col>=0&&col<this.cols&&this.grid[0][col]===null;}
	getDropRow(col){if(!this.isValidColumn(col))return-1; for(let r=this.rows-1;r>=0;r--){if(this.grid[r][col]===null)return r;} return-1;}
	placePiece(row,col,color){if(row>=0&&row<this.rows&&col>=0&&col<this.cols){this.grid[row][col]=color; return true;} return false;}
	checkWin(row,col){
		const color=this.grid[row][col]; if(!color)return false;
		const dirs=[[1,0],[0,1],[1,1],[1,-1]];
		for(const [dr,dc] of dirs){
			let count=1; for(const dir of [-1,1]){
				let r=row+dr*dir,c=col+dc*dir;
				while(r>=0&&r<this.rows&&c>=0&&c<this.cols&&this.grid[r][c]===color){count++; r+=dr*dir; c+=dc*dir;}
			}
			if(count>=4)return true;
		}
		return false;
	}
	isFull(){return this.grid.every(r=>r.every(c=>c!==null));}
	reset(){this.initialize();}
}

// ========= TIMER =========
class GameTimer{
	constructor(){this.intervalId=null; this.onTick=null; this.onTimeUp=null;}
	start(tickCb,timeUpCb){this.onTick=tickCb; this.onTimeUp=timeUpCb; this.intervalId=setInterval(()=>{if(this.onTick)this.onTick();},1000);} 
	stop(){if(this.intervalId){clearInterval(this.intervalId); this.intervalId=null;}}
}

// ========= GAME UI =========
class GameUI{
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
	showStartScreen(){this.elements.start.classList.remove('hidden'); this.elements.game.classList.add('hidden');}
	showGameScreen(){this.elements.start.classList.add('hidden'); this.elements.game.classList.remove('hidden');}
	createBoard(rows,cols){this.elements.board.innerHTML=''; for(let r=0;r<rows;r++){for(let c=0;c<cols;c++){const hole=document.createElement('div'); hole.className='hole'; hole.dataset.col=c; this.elements.board.appendChild(hole);}}}
	updateBoard(board){const holes=[...this.elements.board.children]; holes.forEach((hole,i)=>{const row=Math.floor(i/board.cols),col=i%board.cols; const piece=board.grid[row][col]; hole.classList.remove('filled','red','yellow'); if(piece) hole.classList.add('filled',piece);});}
	updatePlayerInfo(red,yellow){this.elements.redTimer.textContent=this.formatTime(red.time); this.elements.yellowTimer.textContent=this.formatTime(yellow.time); this.elements.redCoins.textContent=red.coins; this.elements.yellowCoins.textContent=yellow.coins;}
	formatTime(s){const m=Math.floor(Math.max(0,s)/60); const sec=Math.max(0,s)%60; return `${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;}
	async animateFallingCoin(col,row,color){const cs=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--cell')); const gap=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--gap')); const pad=parseInt(getComputedStyle(document.documentElement).getPropertyValue('--boardPad')); const x=pad+col*(cs+gap), y=pad+row*(cs+gap); this.elements.fallCoin.className=`coin ${color}`; this.elements.fallCoin.style.left=x+'px'; this.elements.fallCoin.style.transform='translate(0,-100%)'; this.elements.fallCoin.classList.remove('hidden'); void this.elements.fallCoin.offsetWidth; const duration=Math.min(120+row*120,520); this.elements.fallCoin.style.transitionDuration=`${duration}ms`; this.elements.fallCoin.style.transform=`translate(0,${y}px)`; return new Promise(resolve=>{const f=()=>{this.elements.fallCoin.removeEventListener('transitionend',f); this.elements.fallCoin.classList.add('hidden'); resolve();}; this.elements.fallCoin.addEventListener('transitionend',f,{once:true});});}
	showWinMessage(msg){const winOverlay=document.createElement('div'); winOverlay.className='win-overlay'; winOverlay.innerHTML=`<div class="win-card"><h2 class="win-title">${msg}</h2><button class="btn win-btn" onclick="this.parentElement.parentElement.remove()">Continue</button></div>`; document.body.appendChild(winOverlay);}
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
		this.initEvents();
	}
	initEvents(){
		this.ui.elements.start.querySelectorAll('.btn').forEach(b=>b.addEventListener('click',()=>{this.startGame(parseInt(b.dataset.time));}));
		window.addEventListener('keydown',e=>{if(!this.ui.elements.game.classList.contains('hidden')||e.key!=='Enter')return; this.startGame(300);});
		this.ui.elements.board.addEventListener('click',e=>this.handleBoardClick(e));
		this.ui.elements.restartBtn.addEventListener('click',()=>this.restart());
	}
	startGame(t){
		// configure robot and theme from start screen
		const vsRobot = document.getElementById('vsRobot');
		const themeSelect = document.getElementById('themeSelect');
		const theme = themeSelect ? themeSelect.value : 'dark';
		document.documentElement.setAttribute('data-theme', theme);
		this.board.reset(); this.redPlayer.reset(t); this.yellowPlayer.reset(t); this.currentPlayer=this.redPlayer; this.isRunning=true; this.isAnimating=false;
		// enable robot for yellow if requested
		if (vsRobot && vsRobot.checked) this.yellowPlayer.isRobot = true; else this.yellowPlayer.isRobot = false;
		this.ui.showGameScreen(); this.ui.createBoard(this.board.rows,this.board.cols); this.ui.updateBoard(this.board); this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
		this.timer.start(()=>this.handleTimerTick());
		// if the starting player is a robot, let it move
		if (this.currentPlayer && this.currentPlayer.isRobot) setTimeout(()=>this.performAIMove(), 400);
	}
	async handleBoardClick(e){
		const hole=e.target.closest('.hole'); if(!hole||!this.isRunning||this.isAnimating)return;
		const col=parseInt(hole.dataset.col); const row=this.board.getDropRow(col); if(row<0)return; if(!this.currentPlayer.usesCoin())return;
		this.isAnimating=true; await this.ui.animateFallingCoin(col,row,this.currentPlayer.color);
		this.board.placePiece(row,col,this.currentPlayer.color);
		this.ui.updateBoard(this.board); this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
		this.isAnimating=false;
		if(this.board.checkWin(row,col)){this.endGame(`${this.currentPlayer.name} wins!`); return;}
		if(this.board.isFull()){this.endGame('Draw!'); return;}
		this.switchPlayer();
	}
	handleTimerTick(){if(!this.isRunning)return; this.currentPlayer.decrementTime(); if(!this.currentPlayer.hasTimeLeft()){const winner=this.currentPlayer===this.redPlayer?this.yellowPlayer:this.redPlayer; this.endGame(`${winner.name} wins on time!`); return;} this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);}
	switchPlayer(){
		this.currentPlayer=this.currentPlayer===this.redPlayer?this.yellowPlayer:this.redPlayer;
		if (this.currentPlayer && this.currentPlayer.isRobot) setTimeout(()=>this.performAIMove(), 400);
	}

	async performAIMove(){
		if (!this.isRunning || this.isAnimating) return;
		// choose a random valid column
		const validCols = [];
		for (let c=0;c<this.board.cols;c++){ if (this.board.isValidColumn(c)) validCols.push(c); }
		if (validCols.length===0) return;
		const col = validCols[Math.floor(Math.random()*validCols.length)];
		const row = this.board.getDropRow(col);
		if (row<0) return;
		this.isAnimating=true;
		await this.ui.animateFallingCoin(col,row,this.currentPlayer.color);
		this.board.placePiece(row,col,this.currentPlayer.color);
		this.ui.updateBoard(this.board); this.ui.updatePlayerInfo(this.redPlayer,this.yellowPlayer);
		this.isAnimating=false;
		if(this.board.checkWin(row,col)){ this.endGame(`${this.currentPlayer.name} wins!`); return; }
		if(this.board.isFull()){ this.endGame('Draw!'); return; }
		this.switchPlayer();
	}
	endGame(msg){this.isRunning=false; this.timer.stop(); this.ui.showWinMessage(msg);} 
	restart(){this.timer.stop(); this.ui.showStartScreen(); this.isRunning=false;}
}

window.addEventListener('DOMContentLoaded', () => { new Connect4Game(); });
</script>
