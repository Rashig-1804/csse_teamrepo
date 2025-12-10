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

  <div>
    <button onclick="play('rock')">Rock</button>
    <button onclick="play('paper')">Paper</button>
    <button onclick="play('scissors')">Scissors</button>
  </div>

  <p id="player-choice"></p>
  <p id="computer-choice"></p>
  <p id="result"></p>

  <script>
    function play(playerChoice) {
      const choices = ['rock', 'paper', 'scissors'];
      const computerChoice = choices[Math.floor(Math.random() * 3)];

      document.getElementById('player-choice').innerText = `You chose: ${playerChoice}`;
      document.getElementById('computer-choice').innerText = `Computer chose: ${computerChoice}`;

      let result = '';
      if (playerChoice === computerChoice) {
        result = "It's a tie!";
      } else if (
        (playerChoice === 'rock' && computerChoice === 'scissors') ||
        (playerChoice === 'paper' && computerChoice === 'rock') ||
        (playerChoice === 'scissors' && computerChoice === 'paper')
      ) {
        result = 'You win!';
      } else {
        result = 'You lose!';
      }

      document.getElementById('result').innerText = result;
    }
  </script>
</body>
</html>
