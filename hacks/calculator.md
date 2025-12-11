---
layout: opencs
title: Calculator
description: Use JavaScript to code a calculator
permalink: /calculator
---

<style>
  body {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    font-family: Arial, sans-serif;
    margin: 0;
  }

  .calculator-container {
    background: #2d3436;
    border-radius: 20px;
    padding: 20px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    width: 320px;
  }

  .display {
    background: #1e272e;
    color: #00ff00;
    font-size: 2.5em;
    padding: 20px;
    border-radius: 10px;
    text-align: right;
    margin-bottom: 20px;
    min-height: 60px;
    word-wrap: break-word;
    word-break: break-all;
    font-weight: bold;
    border: 2px solid #00ff00;
  }

  .buttons-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
  }

  button {
    padding: 20px;
    font-size: 1.3em;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    font-weight: bold;
    transition: all 0.2s;
  }

  button:hover {
    transform: scale(1.05);
  }

  button:active {
    transform: scale(0.95);
  }

  .btn-number {
    background: #00ff00;
    color: #000;
  }

  .btn-number:hover {
    background: #00dd00;
  }

  .btn-operation {
    background: #0099ff;
    color: white;
  }

  .btn-operation:hover {
    background: #0077dd;
  }

  .btn-equals {
    background: #00ffff;
    color: #000;
    grid-column: span 2;
  }

  .btn-equals:hover {
    background: #00dddd;
  }

  .btn-clear {
    background: #ff3333;
    color: white;
    grid-column: span 2;
  }

  .btn-clear:hover {
    background: #dd1111;
  }

  .btn-decimal {
    background: #ffaa00;
    color: #000;
  }

  .btn-decimal:hover {
    background: #ff8800;
  }
</style>

<div class="calculator-container">
  <div class="display" id="display">0</div>
  
  <div class="buttons-grid">
    <button class="btn-clear" onclick="calculator.clear()">C</button>
    <button class="btn-operation" onclick="calculator.divide()">/</button>
    <button class="btn-operation" onclick="calculator.multiply()">×</button>

    <button class="btn-number" onclick="calculator.addNumber('7')">7</button>
    <button class="btn-number" onclick="calculator.addNumber('8')">8</button>
    <button class="btn-number" onclick="calculator.addNumber('9')">9</button>
    <button class="btn-operation" onclick="calculator.subtract()">−</button>

    <button class="btn-number" onclick="calculator.addNumber('4')">4</button>
    <button class="btn-number" onclick="calculator.addNumber('5')">5</button>
    <button class="btn-number" onclick="calculator.addNumber('6')">6</button>
    <button class="btn-operation" onclick="calculator.add()">+</button>

    <button class="btn-number" onclick="calculator.addNumber('1')">1</button>
    <button class="btn-number" onclick="calculator.addNumber('2')">2</button>
    <button class="btn-number" onclick="calculator.addNumber('3')">3</button>
    <button class="btn-equals" onclick="calculator.calculate()">=</button>

    <button class="btn-number" onclick="calculator.addNumber('0')" style="grid-column: span 2;">0</button>
    <button class="btn-decimal" onclick="calculator.addDecimal()">.</button>
  </div>
</div>

<script>
  const calculator = {
    display: document.getElementById('display'),
    currentInput: '0',
    previousValue: null,
    operation: null,
    shouldResetDisplay: false,

    updateDisplay() {
      this.display.textContent = this.currentInput;
    },

    addNumber(num) {
      if (this.shouldResetDisplay) {
        this.currentInput = num;
        this.shouldResetDisplay = false;
      } else {
        if (this.currentInput === '0' && num !== '0') {
          this.currentInput = num;
        } else if (this.currentInput !== '0' || num === '0') {
          this.currentInput += num;
        }
      }
      this.updateDisplay();
    },

    addDecimal() {
      if (this.shouldResetDisplay) {
        this.currentInput = '0.';
        this.shouldResetDisplay = false;
      } else if (!this.currentInput.includes('.')) {
        this.currentInput += '.';
      }
      this.updateDisplay();
    },

    add() {
      this.handleOperation('+');
    },

    subtract() {
      this.handleOperation('−');
    },

    multiply() {
      this.handleOperation('×');
    },

    divide() {
      this.handleOperation('/');
    },

    handleOperation(op) {
      const currentValue = parseFloat(this.currentInput);

      if (this.previousValue === null) {
        this.previousValue = currentValue;
      } else if (!this.shouldResetDisplay) {
        const result = this.performCalculation(this.previousValue, currentValue, this.operation);
        this.currentInput = result.toString();
        this.previousValue = result;
        this.updateDisplay();
      }

      this.operation = op;
      this.shouldResetDisplay = true;
    },

    calculate() {
      if (this.operation !== null && this.previousValue !== null) {
        const currentValue = parseFloat(this.currentInput);
        const result = this.performCalculation(this.previousValue, currentValue, this.operation);
        this.currentInput = result.toString();
        this.previousValue = null;
        this.operation = null;
        this.shouldResetDisplay = true;
        this.updateDisplay();
      }
    },

    performCalculation(prev, current, op) {
      switch (op) {
        case '+':
          return prev + current;
        case '−':
          return prev - current;
        case '×':
          return prev * current;
        case '/':
          return current !== 0 ? prev / current : 0;
        default:
          return current;
      }
    },

    clear() {
      this.currentInput = '0';
      this.previousValue = null;
      this.operation = null;
      this.shouldResetDisplay = false;
      this.updateDisplay();
    }
  };

  calculator.updateDisplay();
</script>