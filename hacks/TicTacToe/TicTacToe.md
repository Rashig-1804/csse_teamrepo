---
title: Tic-Tac-Toe — OOP Lesson
layout: post
permalink: /hacks/TicTacToe/TicTacToe
author: Aadi Saini
---


## 1. Understanding Object-Oriented Programming

Our Tic-Tac-Toe game is built using Object-Oriented Programming (OOP), which organizes code into classes and objects. 

### Why Use OOP for Tic-Tac-Toe?
- Organization: Each part of the game has its own responsibility
- Reusability: We can create multiple players or boards easily
- Maintainability: Changes to one class don’t break others
- Real world modeling: Code mirrors how we think about the game

### Game Structure Flow

Player Class (Represents each player) + 📋 Board Class (Manages the game grid) = 🎮 TicTacToe Class (Controls the entire game)

```
Player
+name
+symbol
Board
+grid
+display()
+make_move()
+check_winner()
TicTacToe
+board
+players
+current_player
+switch_player()
```

### Player (example)
```python
class Player:
    def __init__(self, name, symbol):
        self.name = name
        self.symbol = symbol

# Let's test it by creating some players
player1 = Player("Alice", "X")
player2 = Player("Bob", "O")

print(f"Player 1: {player1.name} uses symbol '{player1.symbol}'")
print(f"Player 2: {player2.name} uses symbol '{player2.symbol}'")
```

Output:

```
Player 1: Alice uses symbol 'X'
Player 2: Bob uses symbol 'O'
```

### Board (overview and display)
```python
class Board:
    def __init__(self):
        self.grid = [" "] * 9  # Creates 9 empty spaces

    def display(self):
        print("\n")
        print(" " + self.grid[0] + " | " + self.grid[1] + " | " + self.grid[2])
        print("---+---+---")
        print(" " + self.grid[3] + " | " + self.grid[4] + " | " + self.grid[5])
        print("---+---+---")
        print(" " + self.grid[6] + " | " + self.grid[7] + " | " + self.grid[8])
        print("\n")

    def display_reference(self):
        reference = ["1","2","3","4","5","6","7","8","9"]
        print("Board positions:\n")
        print(" " + reference[0] + " | " + reference[1] + " | " + reference[2])
        print("---+---+---")
        print(" " + reference[3] + " | " + reference[4] + " | " + reference[5])
        print("---+---+---")
        print(" " + reference[6] + " | " + reference[7] + " | " + reference[8])
        print("\n")

# Test creating a board
test_board = Board()
```

Output (example):

```
New board created!
Grid contents: [' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ']
```

### Make move / Validation
```python
def make_move(self, position, symbol):
    index = position - 1  # Convert 1-9 to 0-8
    if index < 0 or index > 8:
        print("Invalid position. Choose a number between 1 and 9.")
        return False
    if self.grid[index] != " ":
        print("That spot is already taken. Try again.")
        return False
    self.grid[index] = symbol
    return True
```

### Winner detection
```python
def check_winner(self, symbol):
    win_combinations = [
        [0,1,2], [3,4,5], [6,7,8],
        [0,3,6], [1,4,7], [2,5,8],
        [0,4,8], [2,4,6]
    ]
    for combo in win_combinations:
        if (self.grid[combo[0]] == symbol and
            self.grid[combo[1]] == symbol and
            self.grid[combo[2]] == symbol):
            return True
    return False
```

## TicTacToe class (orchestration)
```python
class TicTacToe:
    def __init__(self, player1, player2):
        self.board = Board()
        self.players = [player1, player2]
        self.current_player = player1

    def switch_player(self):
        self.current_player = (
            self.players[1] if self.current_player == self.players[0] else self.players[0]
        )
        print(f"Now it's {self.current_player.name}'s turn")
```

## Simulated game demo (example)
```python
player1 = Player("Alice","X")
player2 = Player("Bob","O")
game = TicTacToe(player1, player2)

# Simulate moves
game.board.make_move(1, game.current_player.symbol)
game.switch_player()
game.board.make_move(5, game.current_player.symbol)
game.switch_player()
# ... continue
```

---

Try modifying the classes above: add score tracking, different board sizes, AI players, or a reset method.

