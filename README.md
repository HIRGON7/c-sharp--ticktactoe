# Tic-Tac-Toe AI — Minimax in C#

**A console-based Tic-Tac-Toe game featuring a search-based AI opponent, randomized player assignment, and adjustable difficulty.**

This project implements the classic 3×3 game of Tic-Tac-Toe in C#, with an artificial intelligence opponent powered by the **Minimax algorithm**. Players compete against the computer directly in the terminal. The computer evaluates possible future moves, anticipates the human player's responses, and chooses a move according to the selected difficulty.

The project demonstrates how a deterministic game-playing algorithm can be combined with controlled randomness to create opponents that behave differently without requiring machine learning or a trained model.

## Featured Book & Research Publication

> **I am also the author of a companion written research work exploring the AI concept behind this game.**
>
> **Evaluating the Effects of Controlled Stochasticity on Minimax-Based Game-Playing Agents: A Tic-Tac-Toe Study**  
> **Author:** Hadi Jawhari · **Year:** 2026 · **Available on:** Academia.edu  
> **[Read my research work on Academia.edu](https://www.academia.edu/175468806/Evaluating_the_Effects_of_Controlled_Stochasticity_on_Minimax_Based_Game_Playing_Agents_A_Tic_Tac_Toe_Study)**

This C# project is more than a playable console game: it also serves as the implementation behind my written research on **how controlled stochasticity changes the behavior of Minimax-based game-playing agents**. The accompanying work connects an established adversarial search algorithm with a practical question in game-AI design: **can an otherwise optimal opponent be made more approachable and less predictable by selectively introducing random decisions?**

The work examines the difference between consistently following the strongest Minimax move and occasionally replacing that decision with a randomly selected legal move. Rather than changing the game's rules or training a new model, this approach modifies the *policy used to choose an action* after the game state has been evaluated.

The full publication discusses the reasoning, implementation, manually observed gameplay behavior, and implications for designing adjustable AI difficulty. A more detailed explanation of the connection between this repository and the study appears in the [Research Background & Findings](#research-background--findings) section below.

---

## Overview

Tic-Tac-Toe is a two-player, turn-based game played on a 3×3 grid. Each player places either an `X` or an `O` in an empty square. The first player to complete three matching symbols in a row, column, or diagonal wins. If the board fills without a winner, the game ends in a draw.

In this implementation, one player is human and the other is computer-controlled. Rather than selecting moves from a fixed script, the computer can **simulate future game states** and score them using Minimax.

### Highlights

- **Human vs. computer:** play a complete Tic-Tac-Toe match in the console.
- **Minimax-powered opponent:** the AI explores possible continuations of the current game.
- **Two difficulty options:** `hard` for optimal Minimax moves and `medium` for a mixture of optimal and random moves.
- **Randomized starting player:** each match randomly assigns the human `X` or `O`, determining who moves first.
- **Live board display:** the 3×3 grid is redrawn as the game progresses.
- **Win and draw detection:** rows, columns, and diagonals are checked after turns.
- **Replay support:** start another match without restarting the program.
- **Standard C# implementation:** no third-party packages are required.

## How to Play

The board begins with nine numbered positions:

```text
 1 | 2 | 3
---|---|---
 4 | 5 | 6
---|---|---
 7 | 8 | 9
```

Each number corresponds to a square. On your turn, enter the number of the square where you want to place your symbol.

1. Start the program.
2. Choose `medium` or `hard` when prompted.
3. The game randomly assigns you `X` or `O`. `X` plays first.
4. When it is your turn, enter a position from `1` to `9`.
5. The computer selects and plays its move.
6. Continue until someone wins or the board is full.
7. Type `yes` at the replay prompt to start a new game.

For example, if you are `X` and choose position `5`, the center square is marked:

```text
 1 | 2 | 3
---|---|---
 4 | X | 6
---|---|---
 7 | 8 | 9
```

The console uses a red background and positions the board using `Console.SetCursorPosition` to make each turn easier to follow.

## Difficulty Modes

The project provides two selectable difficulty levels, both using the same board and game rules.

| Mode | Move-selection behavior | Playing experience |
| --- | --- | --- |
| **Hard** | Always selects a highest-scoring move found by Minimax. | Optimal play: against valid play, the AI cannot be forced into a loss. |
| **Medium** | Selects an optimal Minimax move 70% of the time and a randomly chosen legal move 30% of the time. | Usually plays strategically, but occasionally makes less-than-optimal decisions. |

The random move in Medium mode is selected from all currently available squares. It can sometimes coincide with the optimal move; therefore, a random selection does not necessarily mean a mistake.

This approach is an example of **controlled stochasticity**: introducing randomness into a deterministic decision process to adjust how challenging the opponent feels.

## Understanding Minimax

Minimax is a **recursive adversarial search algorithm** commonly used in two-player, turn-based, zero-sum games with complete information.

It considers both sides of the game:

- The **maximizing player** (the AI) tries to obtain the highest possible score.
- The **minimizing player** (the human) is assumed to choose responses that lead to the lowest possible score for the AI.

For every available move, Minimax explores the game tree until it reaches a terminal state: a win, a loss, or a draw. It then propagates the resulting scores backward through the tree.

### Terminal-state scoring

| Outcome | Score from the AI's perspective |
| --- | ---: |
| AI wins | `+1` |
| Draw | `0` |
| Human wins | `-1` |

The implementation determines the winner from whose turn would come next when `checkwin()` finds a completed line. Because players alternate turns, this identifies whether the previous move belonged to the AI or the human.

### How a move is evaluated

Suppose several squares are available to the AI:

1. The computer temporarily places its symbol in a candidate square.
2. It recursively explores the human's possible responses.
3. It continues alternating simulated AI and human moves.
4. Completed games are scored as `+1`, `0`, or `-1`.
5. At AI turns, the algorithm retains the **maximum** child score; at human turns, it retains the **minimum**.
6. The candidate move is undone so the original board can be used to evaluate the next candidate.
7. The computer selects a move with the highest resulting score.

The search does not need to be trained on past games: it makes decisions from the current board state and the rules of Tic-Tac-Toe.

### Decision flow

```text
Current board
     |
     v
Find all empty squares
     |
     v
Try one candidate AI move
     |
     v
Explore human responses
     |
     v
Recursively explore future turns
     |
     v
Score win / draw / loss
     |
     v
Undo candidate move
     |
     v
Compare scores and select a move
     |
     v
Apply difficulty setting
     |
     v
Computer plays its turn
```

### Why Hard mode is difficult to beat

Tic-Tac-Toe has a small, finite state space, allowing the program to search through possible continuations of a match. Minimax assumes the human will choose the strongest possible responses, so the AI avoids moves that would allow a forced loss when a better outcome is available.

On Hard mode, the AI consistently follows this evaluation. Against another perfectly playing opponent, Tic-Tac-Toe ends in a draw.

## Implementation Details

The game is organized around a few core methods inside the C# `Program` class.

### `Main()` — Game loop and turn management

`Main()` initializes the console, creates a fresh 3×3 board, chooses who starts, reads the selected difficulty, and alternates between human and computer turns. It also tracks the number of remaining moves and handles the replay prompt.

Each new match begins with the numbered board:

```csharp
char[,] board = {
    { '1', '2', '3' },
    { '4', '5', '6' },
    { '7', '8', '9' }
};
```

The board is represented as a **two-dimensional character array**, with numbered characters for empty squares and `X` or `O` for occupied squares.

### `viewboard(char[,] board)` — Console rendering

This function prints the board as a three-row grid, separating squares with vertical and horizontal lines. `Console.SetCursorPosition` controls where the grid appears in the terminal.

### `addboard(char[,] board, int choice, char user)` — Making a move

The entered position is converted into array coordinates using:

```csharp
int row = (choice - 1) / 3;
int column = (choice - 1) % 3;
```

If the selected square is available, it is replaced with the player's symbol. If the square is already occupied, the player is prompted to choose a different one.

### `checkwin(char[,] board)` — Detecting a winner

The game checks all eight possible winning lines:

- Three horizontal rows.
- Three vertical columns.
- Two diagonals.

The method returns `true` when any line contains three matching symbols, otherwise `false`.

### `comp(char[,] board, char computer, char player, string difficulty)` — Computer move selection

This method collects all legal moves and, for difficulty modes that use Minimax, simulates each available choice. It remembers the move with the highest evaluation score.

The selected difficulty determines whether the computer plays the best-scoring move or introduces a random move.

### `minimax(char[,] board, bool isMaximizing, char aiPlayer, char humanPlayer)` — Recursive search

This is the core AI algorithm. It checks whether the current board is terminal, then recursively evaluates every remaining legal move.

At maximizing levels it selects the largest score; at minimizing levels it selects the smallest. Each simulated move is reversed after evaluation, so multiple game-tree branches can be examined using the same board array.

## Technical Concepts Demonstrated

| Concept | How it appears in the project |
| --- | --- |
| **Artificial intelligence** | The computer selects actions using adversarial game-tree search. |
| **Recursive algorithms** | `minimax()` calls itself to evaluate future turns. |
| **Search and decision-making** | Each available move is evaluated against possible responses. |
| **Game theory** | Optimal play assumes an opponent who also attempts to maximize their own outcome. |
| **Controlled randomness** | Medium mode combines optimal search with stochastic decisions. |
| **State representation** | The board is stored in a 3×3 `char[,]` array. |
| **Backtracking** | Simulated board changes are undone after each recursive branch. |
| **Console application design** | The game manages input, output, turns, and replay in a terminal. |

### Search complexity

Without pruning, Minimax explores a branching tree of possible moves. The branching factor decreases as the board fills: at most nine choices initially, then eight, then seven, and so on. A loose upper bound on the number of full move sequences is `9!` (362,880), with actual evaluation reduced by games ending earlier.

Because the game board is only 3×3, exhaustive search is practical for this application. The implementation uses full recursive Minimax rather than alpha-beta pruning.

## Research Background & Findings

### From a Minimax game to an authored study

After implementing a Tic-Tac-Toe opponent capable of evaluating legal moves with Minimax, I explored a separate question: **what happens to an agent's behavior when its optimal decision is occasionally replaced by a random legal action?**

That investigation led to my written research publication, **_Evaluating the Effects of Controlled Stochasticity on Minimax-Based Game-Playing Agents: A Tic-Tac-Toe Study_** (Hadi Jawhari, 2026), which is available on Academia.edu. The study uses the same general C# Tic-Tac-Toe/Minimax setting demonstrated by this repository.

The research focuses on several connected ideas:

1. **Deterministic decision-making:** In standard Minimax, the agent evaluates reachable terminal states and selects a move that maximizes its expected game-theoretic outcome against a minimizing opponent.
2. **Stochastic decision-making:** By introducing a probability of selecting a random legal move, the agent no longer follows the best-scoring move on every turn.
3. **Difficulty calibration:** The probability of random selection becomes a simple, adjustable parameter for changing the challenge offered to the human player.
4. **Behavioral observation:** Different randomness levels are compared through gameplay to consider how they affect predictability, strength, and the subjective feel of an opponent.

### Connecting the research to this implementation

This program demonstrates two versions of that decision policy:

| Agent configuration | Probability of Minimax selection | Probability of random legal selection |
| --- | ---: | ---: |
| Hard | 100% | 0% |
| Medium | 70% | 30% |

In `comp()`, the AI first identifies legal squares and evaluates them through Minimax. On **Hard**, it plays the best-scoring move. On **Medium**, it draws a pseudorandom integer and selects a random legal square when the roll is below `30`; otherwise it follows Minimax.

A random choice is not automatically a bad move: the randomly selected square may happen to be one of the optimal choices. Therefore, **30% random selection is not equivalent to a 30% mistake rate**. This distinction matters when interpreting the relationship between the probability setting and real gameplay difficulty.

The research explores the broader idea of varying stochasticity rather than restricting the discussion only to the two difficulty settings exposed in this version of the game.

### Research observations

The accompanying study reports observations from **manual gameplay comparisons**, rather than claiming results from a large automated benchmark:

- **Low randomness** tended to produce little visible difference from standard Minimax in a small game such as Tic-Tac-Toe.
- **Higher randomness** increased variation in the agent's decisions, making it less predictable but also more vulnerable.
- **Higher-middle randomness levels** were observed to provide a more balanced playing experience in the study's manual comparisons.
- **Fully random play** generally weakened the opponent substantially relative to Minimax-guided play.

These observations illustrate the central design trade-off: **optimality and player experience are not always the same objective**. A game AI that always chooses the mathematically strongest move may be desirable for competitive play, while a partially stochastic opponent may be better suited to more accessible gameplay.

### Why this work matters

This project and the accompanying written work connect multiple areas of computer science:

- **Artificial intelligence and adversarial search:** Using recursive Minimax to reason about competing actions.
- **Probability and stochastic processes:** Introducing an explicit random-choice probability into the agent's policy.
- **Game theory:** Understanding optimal outcomes in a finite, two-player, zero-sum game.
- **Experimental game design:** Observing how algorithmic settings influence perceived challenge and behavior.
- **Software engineering:** Implementing the search logic, game state, user interaction, and difficulty configuration in C#.

Together, the repository and the research publication show both the **engineering implementation** and the **analysis of the decision-making strategy** behind it.

### Read the complete publication

**Hadi Jawhari (2026).** *Evaluating the Effects of Controlled Stochasticity on Minimax-Based Game-Playing Agents: A Tic-Tac-Toe Study.* Academia.edu.

**[Open the full research publication on Academia.edu](https://www.academia.edu/175468806/Evaluating_the_Effects_of_Controlled_Stochasticity_on_Minimax_Based_Game_Playing_Agents_A_Tic_Tac_Toe_Study)**

---

## Technologies

- **C#** — game logic, board management, recursion, and computer opponent.
- **.NET Console** — terminal input, output, and formatting.
- **Minimax** — adversarial decision-making algorithm.
- **`System.Random`** — randomized opening assignment and Medium-mode move selection.

No external libraries or machine learning frameworks are required.

## Run Locally

### Prerequisites

Install a recent [.NET SDK](https://dotnet.microsoft.com/download) and confirm it is available:

```bash
dotnet --version
```

### If the repository contains a `.csproj` file

Clone the repository and run it from the project directory:

```bash
git clone https://github.com/HIRGON7/c-sharp--ticktactoe.git
cd c-sharp--ticktactoe
dotnet run
```

If the project file is inside a subfolder, run `dotnet run` from that subfolder instead.

### If you only have the `.cs` source file

Create a console project and replace its generated `Program.cs` with the game source:

```bash
dotnet new console -n TicTacToeMinimax
cd TicTacToeMinimax
# Replace the generated Program.cs with the game code.
dotnet run
```

You can also open the project in Visual Studio and run it as a console application.

## Learning Outcomes

This project is a practical introduction to **game-playing AI without machine learning**. It demonstrates that intelligent decisions can emerge from searching possible outcomes rather than training a statistical model.

The two difficulty levels also illustrate a useful design idea: an optimal algorithm can act as the foundation for multiple gameplay experiences by adjusting how consistently its recommendations are followed.

---

**Developed in C# with Minimax AI, and accompanied by an original research publication by Hadi Jawhari (2026).**
