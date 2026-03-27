# Monopoly (C++)

A console-based Monopoly game written in C++. Players take turns rolling dice, buying properties, paying rent, and trying to bankrupt their opponents. The last player with money wins.

---

## Table of Contents

1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Requirements & Build](#requirements--build)
4. [How to Run](#how-to-run)
5. [Game Rules](#game-rules)
   - [Starting the Game](#starting-the-game)
   - [Taking a Turn](#taking-a-turn)
   - [Board Slot Types](#board-slot-types)
   - [Properties (Assets)](#properties-assets)
   - [Mortgage System](#mortgage-system)
   - [Jail](#jail)
   - [Ticket (Chance) Cards](#ticket-chance-cards)
   - [Losing & Winning](#losing--winning)
6. [Board Layout](#board-layout)
7. [Configuration (`board.txt`)](#configuration-boardtxt)

---

## Overview

This is a simplified Monopoly implementation for 2 or more players, played entirely in a Windows console. Each player has a name and a color, starts with **350 NIS**, and moves around an 18-slot board by rolling a single six-sided die. Players can buy unowned properties, collect rent from opponents, and must mortgage their assets if they run out of money. A player is eliminated when they cannot pay a debt even after mortgaging everything. The last player standing wins.

---

## Project Structure

```
Monopoly---Cplus/
├── README.md               # This file
├── DOCS.md                 # Full technical/API documentation
├── CITY.png                # City image asset
├── boardGame.sln           # Visual Studio solution file
└── boardGame/
    ├── Main.cpp            # Entry point
    ├── GameEngine.h/.cpp   # Core game loop, board management, turn logic
    ├── Player.h/.cpp       # Player state, money, assets, payment logic
    ├── Asset.h/.cpp        # Purchasable property (inherits Slot)
    ├── Instruction.h/.cpp  # Special slots: Start, Jail, Ticket (inherits Slot)
    ├── Slot.h/.cpp         # Abstract base class for all board slots
    └── board.txt           # Board configuration (slots, cities, prices)
```

---

## Requirements & Build

**Requirements:**
- Windows (the game uses the Windows Console API for colored text output)
- Visual Studio 2019 or later (v142 toolset or newer)
- C++14 or later

**Build with Visual Studio:**
1. Open `boardGame.sln` in Visual Studio.
2. Select **Debug** or **Release** and your target platform (x64 recommended).
3. Press **Ctrl+Shift+B** (or go to **Build → Build Solution**).
4. The executable is placed in `boardGame/Debug/` or `boardGame/Release/`.

**Build manually (MSVC command line from the `boardGame/` folder):**
```
cl /EHsc /std:c++14 Main.cpp GameEngine.cpp Player.cpp Asset.cpp Slot.cpp Instruction.cpp /Fe:Monopoly.exe
```

---

## How to Run

1. Build the project (see above).
2. Run `boardGame.exe` (or `Monopoly.exe` if built manually) from the `boardGame/` directory so it can find `board.txt`.
3. Follow the on-screen prompts:
   - Enter the number of players (must be **2 or more**).
   - Enter a unique name for each player.
   - Each player's text will appear in a different console color.

---

## Game Rules

### Starting the Game

- Each player starts with **350 NIS**.
- All players begin at **slot 1 (Start)**.
- Turn order follows the order in which players entered their names.

---

### Taking a Turn

At the start of every turn, a player is shown their current balance and asked:

```
0 - quit.  1 - keep playing
```

- **0 (Quit):** The player immediately leaves the game and loses all assets.
- **1 (Play):** A die is rolled (1–6). The player advances that many slots forward.

After moving, the game automatically handles whatever slot the player lands on (see [Board Slot Types](#board-slot-types) below).

At the end of the turn the player's updated balance is shown, and after the last player finishes their turn the current ownership of all properties is printed.

---

### Board Slot Types

| Slot Type  | What Happens |
|------------|--------------|
| **Start**  | Player receives **+350 NIS**. Any mortgaged properties they can now afford are automatically released. Mortgage years are also incremented for any still-mortgaged assets. |
| **Jail**   | Player is jailed and **skips their next turn**. No money changes hands. |
| **Ticket** | Player draws the top card from a 20-card deck. The value is between **−350** and **+350 NIS**. Positive = gain money, Negative = lose money (may trigger mortgage or elimination). The card is returned to the bottom of the deck. |
| **Property** | See [Properties (Assets)](#properties-assets). |

> **Passing Start:** If a player's move carries them past slot 1 (Start) without landing on it, they still receive **+350 NIS** automatically.

---

### Properties (Assets)

Properties have a **buy price** and a **rent price**.

**Landing on an unowned property:**
- If you have enough money you are offered the chance to buy it (enter `1`) or decline (enter `0`).
- If you cannot afford it, the property stays unowned.

**Landing on a property you own:**
- Nothing happens; you see a message confirming it is yours.

**Landing on a property owned by another player:**
- You must pay the **rent** amount to the owner.
- If the property is currently **mortgaged**, you still pay the rent amount but it goes to the **bank** (not the owner).

---

### Mortgage System

When a player cannot afford a payment, the game automatically **mortgages their properties** one by one (in the order they were purchased) until the debt can be paid or all assets are exhausted.

- **Mortgaging** a property adds its buy price back to your balance and marks it as mortgaged.
- A mortgaged property still counts as yours (opponents must pay rent on it, but the money goes to the bank).
- **Each time a player lands on Start**, the mortgage years counter on all their mortgaged properties is incremented.
- **Releasing a mortgage:** When you receive money (e.g., from Start or a Ticket card) and you can afford the payoff, the game automatically releases the mortgage.

**Mortgage payoff formula:**

```
Payoff = BuyPrice + BuyPrice × MortgageYears × 0.10
```

*Example:* A property bought for 300 NIS, mortgaged for 2 years → payoff = 300 + (300 × 2 × 0.10) = **360 NIS**.

---

### Jail

- Landing on the **Jail** slot sets the player's jail flag.
- On their **next turn**, the player is notified they are in jail, released from the flag, and their turn is **skipped**.
- There is no bail-out option; the player simply loses one turn.

---

### Ticket (Chance) Cards

- The deck contains **20 cards**, each with a random integer value in the range **[−350, +350]**.
- Cards are drawn from the top and returned to the bottom (the deck cycles, never runs out).
- A positive value adds money; a negative value subtracts money (which may trigger automatic mortgaging).

---

### Losing & Winning

**A player is eliminated when:**
- They cannot pay a debt (rent, ticket penalty) even after mortgaging every property they own, **or**
- They choose to **quit** at the start of their turn.

When eliminated:
- All remaining money is transferred to the creditor (if the debt was owed to another player).
- All properties revert to the bank (unowned, mortgages cleared).

**The last player remaining is declared the winner.**

---

## Board Layout

The board contains **18 slots** (positions 1–18). After slot 18, the board wraps back to slot 1 (Start).

| Slot | Type       | Name / Details                                   |
|------|------------|--------------------------------------------------|
| 1    | Start      | Start — land or pass to receive **+350 NIS**    |
| 2    | Property   | City1 · property1 — buy: 150 NIS, rent: 20 NIS  |
| 3    | Property   | City1 · property2 — buy: 180 NIS, rent: 35 NIS  |
| 4    | Property   | City1 · property3 — buy: 140 NIS, rent: 15 NIS  |
| 5    | Jail       | Jail — skip your next turn                      |
| 6    | Property   | City2 · property1 — buy: 300 NIS, rent: 50 NIS  |
| 7    | Ticket     | Get Ticket — draw a chance card                 |
| 8    | Property   | City2 · property2 — buy: 450 NIS, rent: 90 NIS  |
| 9    | Property   | City2 · property3 — buy: 375 NIS, rent: 65 NIS  |
| 10   | Ticket     | Get Ticket — draw a chance card                 |
| 11   | Property   | City3 · property1 — buy: 250 NIS, rent: 45 NIS  |
| 12   | Ticket     | Get Ticket — draw a chance card                 |
| 13   | Property   | City3 · property2 — buy: 220 NIS, rent: 35 NIS  |
| 14   | Property   | City3 · property3 — buy: 280 NIS, rent: 60 NIS  |
| 15   | Property   | City4 · property1 — buy: 100 NIS, rent: 5 NIS   |
| 16   | Property   | City4 · property2 — buy: 150 NIS, rent: 12 NIS  |
| 17   | Property   | City4 · property3 — buy: 180 NIS, rent: 25 NIS  |
| 18   | Property   | City4 · property4 — buy: 220 NIS, rent: 35 NIS  |

---

## Configuration (`board.txt`)

The board is defined in `boardGame/board.txt`. Each line describes one slot:

```
I,<type_code>,<name>          # Instruction slot
P,<city>,<street>,<buy>,<rent> # Property slot
```

**Instruction type codes:**

| Code | Meaning |
|------|---------|
| `S`  | Start   |
| `J`  | Jail    |
| `T`  | Ticket  |

**Example lines:**

```
I,S,Start : get 350 Nis       → Start slot
I,J,Jail                      → Jail slot
I,T,Get Ticket                → Ticket slot
P,City1,property1,150,20      → Property in City1, costs 150, rent 20
```

You can edit `board.txt` to change property names, cities, prices, or the number/order of slots. The game reads the file fresh every time it starts, so no recompilation is needed.

> **Note:** There must be exactly **one Start slot** (`I,S,...`). The game will throw an error if a second Start is detected.

---

*For detailed technical documentation (class hierarchy, function signatures, data structures, and game-flow diagrams), see [DOCS.md](DOCS.md).*
