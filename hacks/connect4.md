---
layout: post
title: Connect 4
permalink: /connect4/
hide: true
show_reading_time: false
---


# Connect 4


Welcome! This page hosts our **Connect 4** Python game. You can view and copy the code below to run locally in Python.


---


### Python Code


```python
import os


# Colors for themes
class Theme:
    RED = '\033[91m●\033[0m'
    YELLOW = '\033[93m●\033[0m'
    EMPTY = '○'
    RESET = '\033[0m'


ROWS = 6
COLS = 7


def create_board():
    return [[Theme.EMPTY for _ in range(COLS)] for _ in range(ROWS)]


def print_board(board):
    os.system('cls' if os.name == 'nt' else 'clear')
    for row in board:
        print(' '.join(row))
    print(' '.join([str(i+1) for i in range(COLS)]))  # Column numbers


def is_valid_move(board, col):
    return board[0][col] == Theme.EMPTY


def make_move(board, col, piece):
    for row in reversed(board):
        if row[col] == Theme.EMPTY:
            row[col] = piece
            return


def check_winner(board, piece):
    # Horizontal
    for r in range(ROWS):
        for c in range(COLS - 3):
            if all(board[r][c+i] == piece for i in range(4)):
                return True
    # Vertical
    for c in range(COLS):
        for r in range(ROWS - 3):
            if all(board[r+i][c] == piece for i in range(4)):
                return True
    # Diagonal /
    for r in range(3, ROWS):
        for c in range(COLS - 3):
            if all(board[r-i][c+i] == piece for i in range(4)):
                return True
    # Diagonal \
    for r in range(ROWS - 3):
        for c in range(COLS - 3):
            if all(board[r+i][c+i] == piece for i in range(4)):
                return True
    return False


def play_game():
    board = create_board()
    turn = 0
    print_board(board)
   
    while True:
        player = turn % 2
        piece = Theme.RED if player == 0 else Theme.YELLOW
        try:
            col = int(input(f"Player {player+1} ({piece}): Choose column (1-{COLS}): ")) - 1
        except ValueError:
            print("Enter a valid number!")
            continue
       
        if 0 <= col < COLS and is_valid_move(board, col):
            make_move(board, col, piece)
            print_board(board)
           
            if check_winner(board, piece):
                print(f"Player {player+1} wins! {piece}")
                break
            turn += 1
        else:
            print("Invalid move! Try again.")


if __name__ == "__main__":
    play_game()


