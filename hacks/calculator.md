---
layout: opencs
title: Calculator
description: Use JavaScript to code a calculator
permalink: /calculator
---

<style>
  body {
    font-family: Arial, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    margin: 0;
  }

  .calculator {
    background: white;
    border-radius: 10px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
    padding: 20px;
    width: 300px;
  }

  .display {
    background: #333;
    color: #fff;
    font-size: 2em;
    padding: 20px;
    border-radius: 5px;
    text-align: right;
    margin-bottom: 20px;
    min-height: 60px;
    word-wrap: break-word;
    word-break: break-all;
  }

  .buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
  }

  button {
    padding: 20px;
    font-size: 1.2em;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    background: #f0f0f0;
    transition: background 0.2s;
  }

  button:hover {
    background: #e0e0e0;
  }

  button.operator {
    background: #667eea;
    color: white;
  }

  button.operator:hover {
    background: #5568d3;
  }

  button.equals {
    background: #48bb78;
    color: white;
    grid-column: span 2;
  }

  button.equals:hover {
    background: #38a169;
  }

  button.clear {
    background: #f56565;
    color: white;
    grid-column: span 2;
  }

  button.clear:hover {
    background: #e53e3e;
  }
</style>

<div class="calculator">
  <div class="display" id="display">0</div>
  <div class="buttons">
    <button class="clear" onclick="calculator.clear()">C</button>
    <button class="operator" onclick="calculator.operate('/')">/</button>
    <button class="operator" onclick="calculator.operate('*')">*</button>

    <button onclick="calculator.appendNumber('7')">7</button>
    <button onclick="calculator.appendNumber('8')">8</button>
    <button onclick="calculator.appendNumber('9')">9</button>
    <button class="operator" onclick="calculator.operate('-')">-</button>

    <button onclick="calculator.appendNumber('4')">4</button>
    <button onclick="calculator.appendNumber('5')">5</button>
    <button onclick="calculator.appendNumber('6')">6</button>
    <button class="operator" onclick="calculator.operate('+')">+</button>

    <button onclick="calculator.appendNumber('1')">1</button>
    <button onclick="calculator.appendNumber('2')">2</button>
    <button onclick="calculator.appendNumber('3')">3</button>
    <button class="operator" onclick="calculator.operate('=')">=</button>

    <button onclick="calculator.appendNumber('0')" style="grid-column: span 2;">0</button>
    <button onclick="calculator.appendNumber('.')">.</button>
  </div>
</div>

<script>
  const calculator = {
    display: document.getElementById('display'),
    currentInput: '0',
    previousInput: '',
    operator: null,
    shouldResetDisplay: false,

    updateDisplay() {
      this.display.textContent = this.currentInput;
    },

    appendNumber(num) {
      // If display shows result or this is first input, reset
      if (this.shouldResetDisplay) {
        this.currentInput = num;
        this.shouldResetDisplay = false;
      } else {
        // Avoid multiple leading zeros
        if (this.currentInput === '0' && num !== '.') {
          this.currentInput = num;
        } else if (num === '.' && this.currentInput.includes('.')) {
          // Prevent multiple decimal points
          return;
        } else {
          this.currentInput += num;
        }
      }
      this.updateDisplay();
    },

    operate(op) {
      // If user presses = or another operator
      if (op === '=') {
        if (this.operator && this.previousInput !== '') {
          this.calculate();
        }
        this.shouldResetDisplay = true;
      } else {
        // Store current input and operator for next calculation
        if (this.operator && this.previousInput !== '' && !this.shouldResetDisplay) {
          this.calculate();
        }
        this.previousInput = this.currentInput;
        this.operator = op;
        this.shouldResetDisplay = true;
      }
    },

    calculate() {
      let result;
      const prev = parseFloat(this.previousInput);
      const current = parseFloat(this.currentInput);

      switch (this.operator) {
        case '+':
          result = prev + current;
          break;
        case '-':
          result = prev - current;
          break;
        case '*':
          result = prev * current;
          break;
        case '/':
          result = current !== 0 ? prev / current : 'Error';
          break;
        default:
          return;
      }

      this.currentInput = result.toString();
      this.operator = null;
      this.previousInput = '';
      this.updateDisplay();
    },

    clear() {
      this.currentInput = '0';
      this.previousInput = '';
      this.operator = null;
      this.shouldResetDisplay = false;
      this.updateDisplay();
    }
  };

  // Initialize display
  calculator.updateDisplay();
</script>
