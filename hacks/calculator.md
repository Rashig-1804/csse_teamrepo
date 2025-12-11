---
layout: opencs
title: Calculator
description: Use JavaScript to code a calculator
permalink: /calculator
---

<style>
  .calculator-output {
    grid-column: span 4;
    grid-row: span 1;
    border-radius: 10px;
    padding: 0.25em;
    font-size: 20px;
    border: 5px solid black;
    display: flex;
    align-items: center;
  }

  canvas {
    filter: none;
  }
</style>

<!-- Calculator output display -->
<div id="output">0</div>

<!-- Calculator buttons -->
<button class="calculator-number">1</button>
<button class="calculator-number">2</button>
<button class="calculator-number">3</button>
<button class="calculator-operation">+</button>

<button class="calculator-number">4</button>
<button class="calculator-number">5</button>
<button class="calculator-number">6</button>
<button class="calculator-operation">-</button>

<button class="calculator-number">7</button>
<button class="calculator-number">8</button>
<button class="calculator-number">9</button>
<button class="calculator-operation">*</button>

<button class="calculator-clear">A/C</button>
<button class="calculator-number">0</button>
<button class="calculator-number">.</button>
<button class="calculator-equals">=</button>

<script>
  /* Initialize important variables to manage calculations */
  var firstNumber = null;
  var operator = null;
  var nextReady = true;

  /* Build objects containing key elements */
  const output = document.getElementById("output");
  const numbers = document.querySelectorAll(".calculator-number");
  const operations = document.querySelectorAll(".calculator-operation");
  const clear = document.querySelectorAll(".calculator-clear");
  const equals = document.querySelectorAll(".calculator-equals");

  /* Number buttons listener */
  numbers.forEach(button => {
    button.addEventListener("click", function() {
      number(button.textContent);
    });
  });

  /* Number action function - input numbers into the calculator */
  function number(value) {
    if (value != ".") {
      if (nextReady == true) {
        /* nextReady is used to tell the computer when the user is going to input a completely new number */
        output.innerHTML = value;
        if (value != "0") {
          /* If statement to ensure that there are no multiple leading zeroes */
          nextReady = false;
        }
      } else {
        /* Concatenation is used to add the numbers to the end of the input */
        output.innerHTML = output.innerHTML + value;
      }
    } else {
      /* Special case for adding a decimal; can't have two decimals */
      if (output.innerHTML.indexOf(".") == -1) {
        output.innerHTML = output.innerHTML + value;
        nextReady = false;
      }
    }
  }

  /* Operation buttons listener */
  operations.forEach(button => {
    button.addEventListener("click", function() {
      operation(button.textContent);
    });
  });

  /* Operator action function - input operations into the calculator */
  function operation(choice) {
    if (firstNumber == null) {
      /* Once the operation is chosen, the displayed number is stored into the variable firstNumber */
      firstNumber = parseInt(output.innerHTML);
      nextReady = true;
      operator = choice;
      return;
    }
    /* Occurs if there is already a number stored in the calculator */
    firstNumber = calculate(firstNumber, parseFloat(output.innerHTML));
    operator = choice;
    output.innerHTML = firstNumber.toString();
    nextReady = true;
  }

  /* Calculator function - calculate the result of the equation */
  function calculate(first, second) {
    let result = 0;
    switch (operator) {
      case "+":
        result = first + second;
        break;
      case "-":
        result = first - second;
        break;
      case "*":
        result = first * second;
        break;
      case "/":
        result = first / second;
        break;
      default:
        break;
    }
    return result;
  }

  /* Equals button listener */
  equals.forEach(button => {
    button.addEventListener("click", function() {
      equal();
    });
  });

  /* Equal action function - calculates equation and displays it when equals button is clicked */
  function equal() {
    firstNumber = calculate(firstNumber, parseFloat(output.innerHTML));
    output.innerHTML = firstNumber.toString();
    nextReady = true;
  }

  /* Clear button listener */
  clear.forEach(button => {
    button.addEventListener("click", function() {
      clearCalc();
    });
  });

  /* A/C action function - clears calculator */
  function clearCalc() {
    firstNumber = null;
    output.innerHTML = "0";
    nextReady = true;
  }
</script>

<!-- Load Vanta animation libraries -->
<script src="{{site.baseurl}}/assets/js/three.r119.min.js"></script>
<script src="{{site.baseurl}}/assets/js/vanta.halo.min.js"></script>
<script src="{{site.baseurl}}/assets/js/vanta.birds.min.js"></script>
<script src="{{site.baseurl}}/assets/js/vanta.net.min.js"></script>
<script src="{{site.baseurl}}/assets/js/vanta.rings.min.js"></script>

<!-- Setup and initialize Vanta animation -->
<script>
  /* Setup vanta scripts as functions */
  var vantaInstances = {
    halo: VANTA.HALO,
    birds: VANTA.BIRDS,
    net: VANTA.NET,
    rings: VANTA.RINGS
  };

  /* Obtain a random vanta function */
  var vantaInstance = vantaInstances[Object.keys(vantaInstances)[Math.floor(Math.random() * Object.keys(vantaInstances).length)]];

  /* Run the animation */
  vantaInstance({
    el: "#animation",
    mouseControls: true,
    touchControls: true,
    gyroControls: false
  });
</script>
