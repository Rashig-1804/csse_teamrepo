---
layout: post
title: Breakout Game!
permalink: /breakout
---

<!-- This is our Breakout Game! -->

<style>
  canvas {
    background: #000;
    display: block;
    margin: 0 auto;
    border: 1px solid #333;
  }
  
  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  button:hover:not(:disabled) {
    background: #f0f0f0;
  }

  .title {
    margin-top: 5px !important;
  }

  .back-button {
    margin-bottom: 5px !important;
  }
</style>

<canvas id="gameCanvas" width="600" height="400"></canvas>

<div class="controls" style="text-align: center; margin: 20px 0;">
    <button id="startBtn" style="margin: 5px; padding: 10px 16px; font-size: 16px; font-weight: 600; border: 1px solid #222; background: #fff; cursor: pointer; border-radius: 8px; color: #111;">Start Game</button>
    <button id="pauseBtn" disabled style="margin: 5px; padding: 10px 16px; font-size: 16px; font-weight: 600; border: 1px solid #222; background: #fff; cursor: pointer; border-radius: 8px; color: #111;">Pause</button>
    <button id="resetBtn" style="margin: 5px; padding: 10px 16px; font-size: 16px; font-weight: 600; border: 1px solid #222; background: #a52c2cff; cursor: pointer; border-radius: 8px; color: #111;">Reset</button>
    <button id="nextLevelBtn" style="display:none;margin:10px auto 0;padding:10px 16px;font-family:system-ui,Arial;font-size:16px;font-weight:600;border:1px solid #222;background:#fff;cursor:pointer;border-radius:8px;color:#111 !important;">Next Level ▶</button>
</div>

<!-- Color Pickers -->
<div class="controls" style="text-align: center; margin: 12px 0;">
    <label style="margin-right:10px; font-weight:600;">Ball Color: <input id="ballColorPicker" type="color" value="#e91aa4" style="margin-left:6px;"/></label>
    <label style="margin-right:10px; font-weight:600;">Brick Color: <input id="brickColorPicker" type="color" value="#0095DD" style="margin-left:6px;"/></label>
    <label style="font-weight:600;">Brick Gradient: <input id="brickGradientToggle" type="checkbox" checked style="margin-left:6px;"/></label>
</div>

<div id="information" style="max-width:600px;margin:8px auto;font-family:system-ui,Arial;">
    <h2>OOP Breakout Game</h2>
    <p><strong>How this game works!</strong></p>
    <ul style="margin:8px 0 12px 20px;">
        <li>The game loop: The whole game is a continuous loop that constantly updates object positions and redraws the screen</li>
        <li>The object-oriented code: Every major item like the ball and brick is a separate class in the code, keeping their unique behaviors separate and organized and easy to find! (like movement or score points) </li>
        <li>Movement: The ball updates its position based on its speed, while the paddle moves based on input such as the arrow keys we press</li>
        <li>Collisions & Bouncing: A core function checks for when the ball and brick overlap or touch, and that makes the ball's direction reverse </li>
        <li>Bricks & Score: Hitting a Brick changes its status to "destroyed," and adds points to the score </li>
    </ul>
</div>

<script>
  // Helper color utility functions
  function hexToRgb(hex) {
      hex = hex.replace('#', '');
      if (hex.length === 3) {
          hex = hex.split('').map((c) => c + c).join('');
      }
      const bigint = parseInt(hex, 16);
      return {
          r: (bigint >> 16) & 255,
          g: (bigint >> 8) & 255,
          b: bigint & 255,
      };
  }

  function rgbToHex(r, g, b) {
      return (
          '#' +
          [r, g, b]
              .map((n) => {
                  const hex = n.toString(16);
                  return hex.length === 1 ? '0' + hex : hex;
              })
              .join('')
      );
  }

  function darkenColor(color, percent) {
      // Accepts hex or rgb string; normalize to hex
      const ctxForParse = document.createElement('canvas').getContext('2d');
      ctxForParse.fillStyle = color;
      const computed = ctxForParse.fillStyle; // normalized hex
      let hex = computed;
      if (hex.startsWith('rgb')) {
          // parse rgb(r,g,b)
          const nums = hex.match(/\d+/g).map(Number);
          return rgbToHex(
              Math.max(0, Math.floor(nums[0] * (1 - percent / 100))),
              Math.max(0, Math.floor(nums[1] * (1 - percent / 100))),
              Math.max(0, Math.floor(nums[2] * (1 - percent / 100)))
          );
      }
      hex = hex.replace('#', '');
      const { r, g, b } = hexToRgb('#' + hex);
      return rgbToHex(
          Math.max(0, Math.floor(r * (1 - percent / 100))),
          Math.max(0, Math.floor(g * (1 - percent / 100))),
          Math.max(0, Math.floor(b * (1 - percent / 100)))
      );
  }
  // Base GameObject class - provides common functionality
  class GameObject {
      constructor(x, y) {
          this.x = x;
          this.y = y;
      }
      
      draw(ctx) {
          // Base draw method - to be overridden
      }
      
      update() {
          // Base update method - to be overridden
      }
  }

  // Ball class - handles ball physics and movement
  class Ball extends GameObject {
      constructor(x, y, radius = 8) {
          super(x, y);
          this.radius = radius;
          this.dx = 2;
          this.dy = -2;
          this.color = "#e91aa4";
      }
      
      draw(ctx) {
          ctx.beginPath();
          ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
          ctx.fillStyle = this.color;
          ctx.fill();
          ctx.closePath();
      }
      
      update(canvasWidth, canvasHeight) {
          // Wall collision
          if (this.x + this.dx > canvasWidth - this.radius || this.x + this.dx < this.radius) {
              this.dx = -this.dx;
          }
          if (this.y + this.dy < this.radius) {
              this.dy = -this.dy;
          }
          
          this.x += this.dx;
          this.y += this.dy;
      }
      
      reset(canvasWidth, canvasHeight) {
          this.x = canvasWidth / 2;
          this.y = canvasHeight - 30;
          const speed = Math.hypot(this.dx, this.dy);
          const angle = (Math.PI / 6) + Math.random() * (Math.PI / 3);
          const sign = Math.random() < 0.5 ? -1 : 1;
          this.dx = sign * speed * Math.cos(angle);
          this.dy = -Math.abs(speed * Math.sin(angle));
      }
      
      speedUp(multiplier = 1.12) {
          const currentSpeed = Math.hypot(this.dx, this.dy) * multiplier;
          const theta = Math.atan2(this.dy, this.dx);
          this.dx = currentSpeed * Math.cos(theta);
          this.dy = currentSpeed * Math.sin(theta);
      }

      setColor(newColor) {
          this.color = newColor;
      }
      
      collidesWith(obj) {
          return (
              this.x > obj.x &&
              this.x < obj.x + obj.width &&
              this.y > obj.y &&
              this.y < obj.y + obj.height
          );
      }
      
      collidesWithPaddle(paddle) {
          return (
              this.y + this.dy > paddle.canvasHeight - paddle.height &&
              this.x > paddle.x &&
              this.x < paddle.x + paddle.width
          );
      }
  }

  // Paddle class - handles paddle movement and controls
  class Paddle extends GameObject {
      constructor(x, y, canvasWidth, canvasHeight) {
          super(x, y);
          this.canvasWidth = canvasWidth;
          this.canvasHeight = canvasHeight;
          this.baseWidth = 75;
          this.width = this.baseWidth;
          this.height = 10;
          this.color = "#0095DD";
          this.speed = 7;
          this.leftPressed = false;
          this.rightPressed = false;
      }
      
      draw(ctx) {
          ctx.beginPath();
          ctx.rect(this.x, this.canvasHeight - this.height, this.width, this.height);
          ctx.fillStyle = this.color;
          ctx.fill();
          ctx.closePath();
      }
      
      update() {
          if (this.rightPressed && this.x < this.canvasWidth - this.width) {
              this.x += this.speed;
          } else if (this.leftPressed && this.x > 0) {
              this.x -= this.speed;
          }
      }
      
      setPosition(x) {
          if (x > 0 && x < this.canvasWidth) {
              this.x = x - this.width / 2;
          }
      }
      
      reset() {
          this.x = (this.canvasWidth - this.width) / 2;
          this.width = this.baseWidth;
      }
      
      applyPowerUp(type) {
          if (type === "wide") {
              this.width = this.baseWidth + 40;
          }
      }
      
      resetPowerUp() {
          this.width = this.baseWidth;
      }

      setColor(newColor) {
          this.color = newColor;
      }
  }

  // Brick class - individual brick with power-up capability
  class Brick extends GameObject {
      constructor(x, y, width = 75, height = 20) {
          super(x, y);
          this.width = width;
          this.height = height;
          this.status = 1; // 1 = active, 0 = destroyed
          this.hasPowerUp = Math.random() < 0.3; // 30% chance
          this.baseColor = this.hasPowerUp ? "gold" : "#0095DD";
          this.color = this.baseColor;
          this.useGradient = false;
      }
      
      draw(ctx) {
          if (this.status === 1) {
              ctx.beginPath();
              ctx.rect(this.x, this.y, this.width, this.height);
              let fillStyle = this.color;
              if (this.useGradient) {
                  const grad = ctx.createLinearGradient(this.x, this.y, this.x, this.y + this.height);
                  grad.addColorStop(0, this.color);
                  grad.addColorStop(1, darkenColor(this.color, 20));
                  fillStyle = grad;
              }

              if (this.hasPowerUp) {
                  ctx.fillStyle = fillStyle;
                  ctx.shadowColor = "orange";
                  ctx.shadowBlur = 10;
              } else {
                  ctx.fillStyle = fillStyle;
                  ctx.shadowBlur = 0;
              }
              
              ctx.fill();
              ctx.closePath();
          }
      }
      
      destroy() {
          this.status = 0;
      }
      
      isActive() {
          return this.status === 1;
      }


      getPoints() {
          return 1;
      }

      setColor(newColor) {
          this.color = newColor;
          this.baseColor = newColor;
      }

      setGradient(enabled) {
          this.useGradient = enabled;
      }
  }

  // ExplosiveBrick class - destroys nearby bricks when destroyed
  class ExplosiveBrick extends Brick {
      constructor(x, y, width = 75, height = 20) {
          super(x, y, width, height);
          this.color = "#ff3b30"; // distinctive explosive color
          this.baseColor = this.color;
          this.hasPowerUp = false; // explosive bricks don't include power-ups
          this.explosionRadius = 90; // pixels
      }

      getPoints() {
          return 3;
      }

      // Explode and destroy nearby bricks (accepts game instance)
      explodeNearby(game) {
          const cx = this.x + this.width / 2;
          const cy = this.y + this.height / 2;
          for (let other of game.bricks) {
              if (!other.isActive()) continue;
              if (other === this) continue; // skip self
              const ocx = other.x + other.width / 2;
              const ocy = other.y + other.height / 2;
              const d = Math.hypot(ocx - cx, ocy - cy);
              if (d <= this.explosionRadius) {
                  other.destroy();
                  game.score++;
                  if (other.hasPowerUp) {
                      game.powerUps.push(new PowerUp(other.x + other.width / 2, other.y));
                  }
                  // chain reaction for other explosive bricks
                  if (other instanceof ExplosiveBrick) {
                      other.explodeNearby(game);
                  }
              }
          }
      }
  }

  // PowerUp class - falling power-ups with effects
  class PowerUp extends GameObject {
      constructor(x, y) {
          super(x, y);
          this.size = 20;
          this.fallSpeed = 1.5;
          this.active = true;
          this.type = "wide"; // could be expanded for different types
      }
      
      draw(ctx) {
          if (this.active) {
              // Create gradient effect
              const gradient = ctx.createRadialGradient(
                  this.x, this.y, 5, this.x, this.y, this.size
              );
              gradient.addColorStop(0, "yellow");
              gradient.addColorStop(1, "red");
              
              ctx.beginPath();
              ctx.arc(this.x, this.y, this.size / 2, 0, Math.PI * 2);
              ctx.fillStyle = gradient;
              ctx.fill();
              ctx.closePath();
              
              // Draw "P" text
              ctx.fillStyle = "black";
              ctx.font = "bold 14px Arial";
              ctx.textAlign = "center";
              ctx.textBaseline = "middle";
              ctx.fillText("P", this.x, this.y);
          }
      }
      
      update(canvasHeight) {
          if (this.active) {
              this.y += this.fallSpeed;
              if (this.y > canvasHeight) {
                  this.active = false;
              }
          }
      }
      
      collidesWithPaddle(paddle) {
          return (
              this.active &&
              this.y + this.size / 2 >= paddle.canvasHeight - paddle.height &&
              this.x > paddle.x &&
              this.x < paddle.x + paddle.width
          );
      }
      
      collect() {
          this.active = false;
      }
  }

  // Main Game class - controls game state and orchestrates everything
  class Game {
      constructor(canvasId) {
          this.canvas = document.getElementById(canvasId);
          this.ctx = this.canvas.getContext("2d");
          this.width = this.canvas.width;
          this.height = this.canvas.height;
          
          // Game state
          this.score = 0;
          this.lives = 3;
          this.level = 1;
          this.paused = false;
          this.gameRunning = false;
          
          // Game objects
          this.ball = new Ball(this.width / 2, this.height - 30);
          this.paddle = new Paddle((this.width - 75) / 2, this.height - 10, this.width, this.height);
          this.bricks = [];
          this.powerUps = [];
          
          // Power-up state
          this.activePowerUp = null;
          this.powerUpTimer = 0;
          this.powerUpDuration = 5000; // 5 seconds

          // Default colors and gradient settings
          this.defaultBallColor = "#e91aa4";
          this.defaultBrickColor = "#0095DD";
          this.defaultPaddleColor = "#0095DD";
          this.brickGradient = true;
          
          // Brick configuration
          this.brickRows = 4;
          this.brickCols = 6;
          this.brickPadding = 10;
          this.brickOffsetTop = 30;
          this.brickOffsetLeft = 50;
          
          this.setupEventListeners();
          this.initBricks();

          // Apply default colors for existing objects
          this.ball.setColor(this.defaultBallColor);
          this.paddle.setColor(this.defaultPaddleColor);
      }
      
      setupEventListeners() {
          // Keyboard controls
          document.addEventListener("keydown", (e) => {
              if (e.key === "Right" || e.key === "ArrowRight") {
                  this.paddle.rightPressed = true;
              } else if (e.key === "Left" || e.key === "ArrowLeft") {
                  this.paddle.leftPressed = true;
              }
          });
          
          document.addEventListener("keyup", (e) => {
              if (e.key === "Right" || e.key === "ArrowRight") {
                  this.paddle.rightPressed = false;
              } else if (e.key === "Left" || e.key === "ArrowLeft") {
                  this.paddle.leftPressed = false;
              }
          });
          
          // Mouse controls
          this.canvas.addEventListener("mousemove", (e) => {
              const relativeX = e.clientX - this.canvas.offsetLeft;
              this.paddle.setPosition(relativeX);
          });
          
          // Button controls
          document.getElementById("startBtn").addEventListener("click", () => this.start());
          document.getElementById("pauseBtn").addEventListener("click", () => this.togglePause());
          document.getElementById("resetBtn").addEventListener("click", () => this.reset());
          document.getElementById("nextLevelBtn").addEventListener("click", () => this.nextLevel());

          // Color pickers
          const ballPicker = document.getElementById('ballColorPicker');
          const brickPicker = document.getElementById('brickColorPicker');
          const brickGradToggle = document.getElementById('brickGradientToggle');

          ballPicker.addEventListener('input', (e) => {
              this.ball.setColor(e.target.value);
          });

          brickPicker.addEventListener('input', (e) => {
              this.defaultBrickColor = e.target.value;
              for (let b of this.bricks) {
                  b.setColor(this.defaultBrickColor);
              }
          });

          brickGradToggle.addEventListener('change', (e) => {
              this.brickGradient = e.target.checked;
              for (let b of this.bricks) {
                  b.setGradient(this.brickGradient);
              }
          });
      }
      
      initBricks() {
          this.bricks = [];
          for (let c = 0; c < this.brickCols; c++) {
              for (let r = 0; r < this.brickRows; r++) {
                  const x = c * (75 + this.brickPadding) + this.brickOffsetLeft;
                  const y = r * (20 + this.brickPadding) + this.brickOffsetTop;
                  // Occasionally create an ExplosiveBrick instead of a regular Brick
                  const makeExplosive = Math.random() < 0.12; // ~12% chance
                  let brick;
                  if (makeExplosive) {
                      brick = new ExplosiveBrick(x, y);
                  } else {
                      brick = new Brick(x, y);
                  }
                  // Apply user-selected color/gradient but keep special bricks' colors intact
                  if (!(brick instanceof ExplosiveBrick) && !brick.hasPowerUp) {
                      brick.setColor(this.defaultBrickColor);
                  }
                  brick.setGradient(this.brickGradient);
                  this.bricks.push(brick);
              }
          }
      }
      
      start() {
          this.gameRunning = true;
          this.paused = false;
          document.getElementById("startBtn").disabled = true;
          document.getElementById("pauseBtn").disabled = false;
          this.gameLoop();
      }
      
      togglePause() {
          this.paused = !this.paused;
          document.getElementById("pauseBtn").textContent = this.paused ? "Resume" : "Pause";
          if (!this.paused) {
              this.gameLoop();
          }
      }
      
      reset() {
          this.score = 0;
          this.lives = 3;
          this.level = 1;
          this.brickRows = 4;
          this.paused = false;
          this.gameRunning = false;
          
          this.ball.reset(this.width, this.height);
          this.paddle.reset();
          this.powerUps = [];
          this.activePowerUp = null;
          
          this.initBricks();
          
          document.getElementById("startBtn").disabled = false;
          document.getElementById("pauseBtn").disabled = true;
          document.getElementById("pauseBtn").textContent = "Pause";
          document.getElementById("nextLevelBtn").style.display = "none";
          // Re-apply colors and gradients
          this.ball.setColor(this.defaultBallColor);
          this.paddle.setColor(this.defaultPaddleColor);
          for (let b of this.bricks) {
              if (!(b instanceof ExplosiveBrick) && !b.hasPowerUp) {
                  b.setColor(this.defaultBrickColor);
              }
              b.setGradient(this.brickGradient);
          }
          this.draw();
      }
      
      nextLevel() {
          this.level++;
          this.ball.speedUp(1.12);
          
          if (this.brickRows < 8) {
              this.brickRows++;
          }
          
          this.initBricks();
          this.ball.reset(this.width, this.height);
          this.paddle.reset();
          this.powerUps = [];
          this.activePowerUp = null;
          
          this.paused = false;
          document.getElementById("nextLevelBtn").style.display = "none";
          this.gameLoop();
      }
      
      collisionDetection() {
          for (let brick of this.bricks) {
              if (brick.isActive() && this.ball.collidesWith(brick)) {
                  this.ball.dy = -this.ball.dy;
                  brick.destroy();
                  game.score += brick.getPoints();

                  
                  if (brick.hasPowerUp) {
                      this.powerUps.push(new PowerUp(brick.x + brick.width / 2, brick.y));
                  }
                  // Trigger explosive brick behavior
                  if (brick instanceof ExplosiveBrick) {
                      brick.explodeNearby(this);
                  }
              }
          }
      }
      
      updatePowerUps() {
          for (let powerUp of this.powerUps) {
              powerUp.update(this.height);
              
              if (powerUp.collidesWithPaddle(this.paddle)) {
                  powerUp.collect();
                  this.paddle.applyPowerUp(powerUp.type);
                  this.activePowerUp = powerUp.type;
                  this.powerUpTimer = Date.now();
              }
          }
          
          // Check power-up timer
          if (this.activePowerUp) {
              const elapsed = Date.now() - this.powerUpTimer;
              if (elapsed > this.powerUpDuration) {
                  this.activePowerUp = null;
                  this.paddle.resetPowerUp();
              }
          }
          
          // Remove inactive power-ups
          this.powerUps = this.powerUps.filter(p => p.active);
      }
      
      checkWinCondition() {
          const activeBricks = this.bricks.filter(brick => brick.isActive()).length;
          if (activeBricks === 0) {
              this.paused = true;
              document.getElementById("nextLevelBtn").style.display = "block";
              return true;
          }
          return false;
      }
      

      checkBallCollision() {
          // Ball hits bottom
          if (this.ball.y + this.ball.dy > this.height - this.ball.radius) {
              if (this.ball.collidesWithPaddle(this.paddle)) {
                  this.ball.dy = -this.ball.dy;
              } else {
                  this.lives--;
                  if (this.lives === 0) {
                      this.gameOver();
                  } else {
                      this.ball.reset(this.width, this.height);
                      this.paddle.reset();
                  }
              }
          }
      }
      // This function says the final score, displays game over, and resets the game.
      gameOver() {
          this.gameRunning = false;
          this.paused = true;
          alert(`GAME OVER! Final Score: ${this.score}`);
          this.reset();
      }
      
      drawUI() {
          // Score
          this.ctx.font = "16px Arial";
          this.ctx.fillStyle = "#0095DD";
          this.ctx.textAlign = "left";
          this.ctx.fillText("Score: " + this.score, 8, 20);
          
          // Lives  
          this.ctx.textAlign = "right";
          this.ctx.fillText("Lives: " + this.lives, this.width - 8, 20);
          
          // Level
          this.ctx.textAlign = "center";
          this.ctx.fillText("Level: " + this.level, this.width / 2, 20);
          
          // Power-up timer
          if (this.activePowerUp) {
              const elapsed = Date.now() - this.powerUpTimer;
              const remaining = Math.max(0, this.powerUpDuration - elapsed);
              const barHeight = 100;
              const barWidth = 10;
              const fillHeight = (remaining / this.powerUpDuration) * barHeight;
              
              this.ctx.fillStyle = "gray";
              this.ctx.fillRect(this.width - 20, 30, barWidth, barHeight);
              
              this.ctx.fillStyle = "lime";
              this.ctx.fillRect(
                  this.width - 20,
                  30 + (barHeight - fillHeight),
                  barWidth,
                  fillHeight
              );
              
              this.ctx.strokeStyle = "black";
              this.ctx.strokeRect(this.width - 20, 30, barWidth, barHeight);
          }
      }
      
      update() {
          if (this.paused || !this.gameRunning) return;
          
          this.ball.update(this.width, this.height);
          this.paddle.update();
          this.updatePowerUps();
          this.collisionDetection();
          this.checkBallCollision();
          
          if (this.checkWinCondition()) return;
      }
      
      draw() {
          // Clear canvas
          this.ctx.clearRect(0, 0, this.width, this.height);
          
          // Draw all game objects
          for (let brick of this.bricks) {
              brick.draw(this.ctx);
          }
          
          this.ball.draw(this.ctx);
          this.paddle.draw(this.ctx);
          
          for (let powerUp of this.powerUps) {
              powerUp.draw(this.ctx);
          }
          
          this.drawUI();
      }
      
      gameLoop() {
          if (this.paused || !this.gameRunning) return;
          
          this.update();
          this.draw();
          
          requestAnimationFrame(() => this.gameLoop());
      }
  }

  // Initialize the game
  const game = new Game("gameCanvas");
  
  // Initial draw
  game.draw();
</script>