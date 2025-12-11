---
layout: opencs
title: Rock Paper Scissors Game
description: Playing rock paper scissors using javascript code
permalink: /rockpaper
---

<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Rock Paper Scissors</title>

  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin-top: 50px;
    }
    button {
      font-size: 18px;
      padding: 10px 20px;
      margin: 10px;
      cursor: pointer;
    }
    #result {
      font-size: 24px;
      margin-top: 20px;
      font-weight: bold;
    }
  </style>
</head>

<body>

  <h1>Rock Paper Scissors</h1>

  <!-- Rock + Paper buttons only -->
  <button id="rockBtn">Rock</button>
  <button id="paperBtn">Paper</button>

  <p id="player-choice"></p>
  <p id="computer-choice"></p>
  <p id="result"></p>

  <script>
    // Add event listeners
    document.getElementById("rockBtn").addEventListener("click", () => play("rock"));
    document.getElementById("paperBtn").addEventListener("click", () => play("paper"));

    function play(playerChoice) {
      const choices = ["rock", "paper", "scissors"];
      const computerChoice = choices[Math.floor(Math.random() * 3)];

      document.getElementById("player-choice").innerText = `You chose: ${playerChoice}`;
      document.getElementById("computer-choice").innerText = `Computer chose: ${computerChoice}`;

      let result = "";

      // Rock logic
      if (playerChoice === "rock" && computerChoice === "scissors") {
        result = "You win! Rock crushes Scissors.";
      } else if (playerChoice === "rock" && computerChoice === "paper") {
        result = "You lose! Paper covers Rock.";
      } else if (playerChoice === "rock" && computerChoice === "rock") {
        result = "It's a tie!";
      }

      // Paper logic
      if (playerChoice === "paper" && computerChoice === "rock") {
        result = "You win! Paper covers Rock.";
      } else if (playerChoice === "paper" && computerChoice === "scissors") {
        result = "You lose! Scissors cuts Paper.";
      } else if (playerChoice === "paper" && computerChoice === "paper") {
        result = "It's a tie!";
      }

      document.getElementById("result").innerText = result;
    }
  </script>

</body>
</html>

