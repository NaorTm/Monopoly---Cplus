# Technical Documentation — Monopoly (C++)

This document is aimed at developers and describes every class, member variable, function, constant, and data structure in the project.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Class Hierarchy](#class-hierarchy)
3. [Constants & Macros](#constants--macros)
4. [Class: `Slot`](#class-slot)
5. [Class: `Asset`](#class-asset)
6. [Class: `Instruction`](#class-instruction)
7. [Class: `Player`](#class-player)
8. [Class: `GameEngine`](#class-gameengine)
9. [Entry Point: `main()`](#entry-point-main)
10. [Data Structures](#data-structures)
11. [Game Flow](#game-flow)
12. [Board Configuration Format](#board-configuration-format)

---

## Architecture Overview

The game is built around five classes. `Slot` is the abstract base class for every position on the board. `Asset` and `Instruction` extend `Slot` to represent purchasable properties and special action squares respectively. `Player` owns a collection of `Asset` pointers and tracks money and position. `GameEngine` orchestrates everything: it builds the board, creates players, drives the turn loop, and handles every in-game event.

```
main()
  └─ GameEngine
       ├─ builds board  →  list<Slot*>   (contains Asset* and Instruction* objects)
       ├─ creates players → deque<Player*>
       ├─ creates deck  →  queue<int>
       └─ runs startGame() loop
```

---

## Class Hierarchy

```
Slot  (base)
├── Asset        (purchasable property)
└── Instruction  (Start / Jail / Ticket action square)

Player           (standalone; owns vector<Asset*>)
GameEngine       (standalone; owns deque<Player*>, list<Slot*>, queue<int>)
```

---

## Constants & Macros

Defined across the header files:

> **Note on spelling:** Several identifiers in the source code contain typos (`INTREST_RATE`, `setMorgage`, `getMorgagePrice`). These are documented exactly as written in the source so that the names match what you see in the code.

| Symbol | File | Value / Expansion | Purpose |
|--------|------|-------------------|---------|
| `START_SUM` | `Player.h` | `350` | Starting money; also the bonus for landing on / passing Start |
| `START_POS` | `Player.h` | `1` | Board position every player starts on |
| `CARDS` | `GameEngine.h` | `20` | Number of cards in the Ticket deck |
| `INTREST_RATE` | `Asset.h` | `0.1` | Annual mortgage interest rate (10 %) |
| `BOARD` | `GameEngine.h` | `"board.txt"` | Path to the board configuration file |
| `ASST` | `GameEngine.h` | `dynamic_cast<Asset*>` | Shorthand for downcasting `Slot*` to `Asset*` |
| `INST` | `GameEngine.h` | `dynamic_cast<Instruction*>` | Shorthand for downcasting `Slot*` to `Instruction*` |
| `WHITE_PRINT` | `GameEngine.h` | `SetConsoleTextAttribute(hConsole, 15)` | Resets console text to white |
| `BAD_INPUT` | `Slot.h` | `throw "Bad input! exiting...\n"` | Throws on invalid input |
| `YOU_LOSE` | `Player.h` | `cout << "You don't have enough money, you lose!\n"` | Prints losing message |
| `COPY_CTRT` | `Slot.h` | `throw "Copy Constructor Not Used!\n"` | Guards against accidental copy construction |
| `ASSIGN_OPRT` | `Slot.h` | `throw "Assign Operator Not Used!\n"` | Guards against accidental assignment |

---

## Class: `Slot`

**File:** `Slot.h` / `Slot.cpp`  
**Role:** Abstract base class for every square on the board. Provides a name, an auto-assigned sequential position number, and a virtual `printSlot()`.

### Member Variables

| Variable | Type | Access | Description |
|----------|------|--------|-------------|
| `slotName` | `string` | private | Display name of the slot |
| `slotNum` | `int` | private | 1-based position on the board |
| `slotCounter` | `static int` | protected | Class-wide counter; auto-increments each time a `Slot` is constructed (starts at 1) |

### Member Functions

---

#### `Slot(const string name)` — Constructor

Calls `setSlotName(name)`, assigns the current value of `slotCounter` as this slot's number via `setSlotNum()`, then increments `slotCounter`. The result is that every slot created receives a unique, sequential position number matching the order in which board lines are read from `board.txt`.

---

#### `virtual ~Slot()` — Destructor

Calls `slotName.clear()` to release the string memory.

---

#### `void setSlotNum(const int num)`

Sets `slotNum` to `num`. Called only from the constructor; external code should not need to renumber slots.

---

#### `const int getSlotNum()`

Returns the slot's board position (1-based integer).

---

#### `const string getSlotName()`

Returns the slot's display name.

---

#### `void setSlotName(const string name)`

Sets `slotName`. Throws `BAD_INPUT` (via try/catch, printing the error) if `name` is an empty string.

---

#### `virtual const void printSlot()`

Prints `"(slot N) "` where N is the slot number. Overridden in `Asset` and `Instruction` to append additional details. Called as the first line inside both child overrides via `Slot::printSlot()`.

---

## Class: `Asset`

**File:** `Asset.h` / `Asset.cpp`  
**Role:** Represents a purchasable property. Inherits `Slot`. Tracks ownership, buy price, rent, and mortgage state.

### Member Variables

| Variable | Type | Access | Description |
|----------|------|--------|-------------|
| `cityName` | `string` | private | Group/city this property belongs to |
| `housePrice` | `int` | private | Cost to purchase the property |
| `rent` | `int` | private | Rent paid by an opponent who lands here |
| `owner` | `Player*` | private | Pointer to the owning `Player`; `nullptr` if unowned |
| `years` | `int` | private | Number of turns this property has been mortgaged |
| `isMortgage` | `bool` | private | `true` if the property is currently mortgaged |

### Member Functions

---

#### `Asset(string streetName, string city, int hPrice, int fee, Player* p = nullptr, int years = 0)` — Primary Constructor

Calls the `Slot(streetName)` base constructor, then `setCityName(city)`, `setHousePrice(hPrice)`, `setRent(fee)`, and `setOwner(p)`. The `years` parameter is declared in the signature for interface consistency with the copy constructor but is not assigned here; the `years` field therefore starts at 0 (its default-initialised value).

---

#### `Asset(const string Sname, const string Cname, int hPrice, int fee, Player* p, int years, bool mort)` — Copy Constructor

Direct field-by-field copy. Used when an exact duplicate of an existing asset is needed, preserving the mortgage flag.

---

#### `~Asset()` — Destructor

Prints `"Asset Destructor\n"`. The `Slot` base destructor then clears `slotName`.

---

#### `void setCityName(const string name)`

Sets `cityName`. Throws `BAD_INPUT` (caught internally) if `name` is empty.

---

#### `string getCityName()`

Returns `cityName`.

---

#### `const int getHousePrice()`

Returns the buy price of the property.

---

#### `void setHousePrice(int price)`

Sets the buy price.

---

#### `const int getRent()`

Returns the rent amount charged when an opponent lands here.

---

#### `void setRent(int rentPrice)`

Sets the rent amount.

---

#### `void setOwner(Player* p = nullptr)`

Sets `owner` to `p`. Passing no argument (or `nullptr`) marks the property as unowned — used when a player is eliminated.

---

#### `Player* getOwner()`

Returns the `owner` pointer (`nullptr` if unowned).

---

#### `bool isMortg()`

Returns `true` if the property is currently mortgaged.

---

#### `const int getMortgageYears()`

Returns the number of turns this property has been under mortgage.

---

#### `void increaseMortgageYears()`

Increments `years` by 1. Called by `Player::findMortgage()` each time the owner lands on Start.

---

#### `void setMorgage()`

Marks the property as mortgaged: sets `isMortgage = true` and resets `years = 1`.

---

#### `void freeMortgage()`

Clears the mortgage: sets `isMortgage = false` and `years = 0`.

---

#### `const int getMorgagePrice()`

Calculates and returns the total amount needed to release the mortgage:

```
payoff = housePrice + housePrice × years × INTREST_RATE
       = housePrice × (1 + 0.10 × years)
```

The result is truncated to `int` (no fractional NIS).

---

#### `virtual const void printSlot()`

Calls `Slot::printSlot()` then prints:

```
City Name: <cityName>
Street name: <slotName>
Costs <housePrice> to buy and <rent> to rent
Mortgaged for <years> years   OR   Not Mortgaged
Owned by: <ownerName>         OR   nobody
=================
```

---

## Class: `Instruction`

**File:** `Instruction.h` / `Instruction.cpp`  
**Role:** Represents a non-property action square (Start, Jail, or Ticket). Inherits `Slot`.

### Enum: `slotType`

```cpp
typedef enum slotType { Start, Ticket, Jail } slotType;
```

Used both inside `Instruction` and by `GameEngine` to dispatch the correct action.

### Member Variables

| Variable | Type | Access | Description |
|----------|------|--------|-------------|
| `type` | `slotType` | private | Which kind of instruction square this is |

### Member Functions

---

#### `Instruction(const string name, slotType t)` — Constructor

Calls `Slot(name)` then `setSlotType(t)`.

---

#### `~Instruction()` — Destructor

Prints `"Instruction Destructor\n"`. The `Slot` base destructor clears `slotName`.

---

#### `const slotType getSlotType()`

Returns the `type` field.

---

#### `void setSlotType(const slotType& t)`

Sets `type` to `t`.

---

#### `virtual const void printSlot()`

Calls `Slot::printSlot()` then prints the slot name and a separator line:

```
(slot N) <slotName>
=================
```

---

## Class: `Player`

**File:** `Player.h` / `Player.cpp`  
**Role:** Manages all state for a single player: name, money balance, board position, owned assets, jail status, and console color.

### Member Variables

| Variable | Type | Access | Description |
|----------|------|--------|-------------|
| `playerName` | `string` | private | Player's display name |
| `myMoney` | `int` | private | Current NIS balance (starts at `START_SUM` = 350) |
| `PlayerPosition` | `int` | private | 1-based index of the player's current board slot |
| `houses` | `vector<Asset*>` | private | All properties currently owned by this player |
| `isJail` | `bool` | private | `true` when this player must skip their next turn |
| `color` | `int` | public | Windows console color attribute used to print this player's text in a distinct color |

### Member Functions

---

#### `Player(const string name, const int colorIndx)` — Primary Constructor

Calls `setPlayerName(name)`, `setMoney(START_SUM)` (balance = 350), `setPosition(START_POS)` (position = 1), and sets `isJail = false`. `colorIndx` is stored in `color`.

---

#### `~Player()` — Destructor

Prints `"kill player <name>"`. If the player still owns assets, calls `removeAssets()` then `houses.clear()`. Otherwise prints `"player had no assets!"`. Finally calls `playerName.clear()`.

---

#### `Player(const string name, vector<Asset*> homes, int money, int pos, bool j, int col)` — Copy Constructor

Direct field assignment; used when an exact snapshot of a player is required.

---

#### `Player* operator=(Player* p)`

Copies all fields from `p` into `this` and returns `this`.

---

#### `const bool getJail()`

Returns `isJail`.

---

#### `void setJail(bool j)`

Sets `isJail` to `j`. `GameEngine` sets this to `true` when the player lands on Jail and back to `false` at the start of the skipped turn.

---

#### `void setPlayerName(string name)`

Sets `playerName`. Throws `BAD_INPUT` (caught internally, printing the error) if `name` is empty.

---

#### `const string getPlayerName()`

Returns the player's name.

---

#### `void setMoney(int const cash)`

Adds `cash` to `myMoney`. Pass a negative value to subtract (e.g., `setMoney(-50)` deducts 50 NIS). This is the raw mutator — callers are responsible for ensuring the balance stays valid; `payment()` wraps this with safety checks.

---

#### `const int getMoney()`

Returns the current NIS balance.

---

#### `void setPosition(int pos)`

Sets `PlayerPosition` to `pos`.

---

#### `const int getPosition()`

Returns the current board position.

---

#### `const bool payment(int toPay, Player* hOwner = nullptr)`

**The central financial transaction function.** Returns `true` if the player survives, `false` if they are eliminated.

**When `toPay > 0` (player receives money):**
1. Calls `setMoney(toPay)` to add funds.
2. Iterates over `houses`; for each mortgaged asset whose payoff (`getMorgagePrice()`) the player can now afford, calls `releaseMortgage(asset, -payoff)` to automatically release it.

**When `toPay < 0` (player pays money):**
1. Checks whether `myMoney + toPay < 0` (insufficient funds).
   - If the player **has assets**: enters a mortgage loop — mortgages assets one by one (via `setMorgage()`, recovering `housePrice`) until the balance is sufficient or all assets are exhausted.
   - If all assets are mortgaged and the debt still cannot be paid, or if the player has no assets at all, calls `playerLost(hOwner)` and returns `false`.
2. If the player **can** pay: calls `setMoney(toPay)` to deduct the amount. If `hOwner` is non-null, calls `hOwner->setMoney(-toPay)` to credit the recipient.

---

#### `void releaseMortgage(Asset* h, const int mortPrice)`

Calls `setMoney(mortPrice)` (mortPrice is negative, so this deducts the payoff cost), then `h->freeMortgage()`. Prints a confirmation message showing the property and the amount paid.

---

#### `void buyAsset(Asset* a)`

Calls `setMoney(-a->getHousePrice())` to deduct the cost, pushes `a` onto `houses`, and calls `a->setOwner(this)` to register ownership.

---

#### `void removeAssets()`

Called when a player is eliminated. Iterates over `houses`, calls `freeMortgage()` and `setOwner()` (no argument → `nullptr`) on each asset, returning all properties to the bank. Prints each released asset.

---

#### `void findMortgage()`

Iterates over `houses`; for each mortgaged asset calls `increaseMortgageYears()`. Called by `GameEngine::instOptions()` when a player lands on Start, so mortgage interest accumulates once per Start visit.

---

#### `void getHouses()`

Prints all owned assets by calling `printSlot()` on each, or prints `"nothing!"` if the player owns nothing.

---

#### `bool playerLost(Player* hOwner)`

Prints the `YOU_LOSE` message. If `hOwner` is non-null, transfers all remaining money to `hOwner` via `hOwner->setMoney(this->myMoney)`; otherwise sets `myMoney = 0`. Returns `false` to signal elimination to `payment()`.

---

## Class: `GameEngine`

**File:** `GameEngine.h` / `GameEngine.cpp`  
**Role:** Owns the game board, the player queue, and the card deck. Builds everything at startup and drives the main game loop.

### Member Variables

| Variable | Type | Access | Description |
|----------|------|--------|-------------|
| `players` | `deque<Player*>` | private | All active players in turn order |
| `deck` | `queue<int>` | private | Circular 20-card deck; values in [−350, +350] |
| `board` | `list<Slot*>` | private | Ordered list of all board slots; polymorphic (`Asset*` or `Instruction*`) |
| `slotTotal` | `int` | private | Total number of slots (determined by `buildBoard()`) |
| `hConsole` | `HANDLE` | private | Windows console handle used to change text color per player |

### Member Functions

---

#### `GameEngine(const int playersNum)` — Constructor

The constructor drives the entire game setup in order:

1. `buildBoard()` — reads `board.txt` and populates `board`.
2. `createPlayers(playersNum)` — prompts for names and creates `Player` objects.
3. `createDeck()` — generates the random card deck.
4. Initialises `hConsole` via `GetStdHandle(STD_OUTPUT_HANDLE)`.
5. `startGame()` — runs the main game loop until a winner is determined.

---

#### `~GameEngine()` — Destructor

Calls `players.clear()`, drains the `deck` queue, and calls `board.clear()`.

---

#### `void buildBoard()`

Opens `board.txt` and reads it line by line using comma-separated parsing:

- Lines starting with `'I'` create an `Instruction` object:  
  - Reads the type character (`S`/`J`/`T`), converts it via `getSlotType()`, reads the slot name, and pushes a `new Instruction` to `board`.
  - Throws an error if a second Start slot is encountered.
- Lines starting with `'P'` create an `Asset` object:  
  - Reads city name, street name, buy price (`atoi()`), and rent price (`atoi()`), and pushes a `new Asset` to `board`.
- Any other non-null character throws `"File not valid!"`.
- A null/empty line signals end-of-file.

Each successfully parsed slot increments `slotTotal`.

---

#### `const slotType getSlotType(const char c)`

Converts a single character from `board.txt` to a `slotType` enum value:

| Character | Returns |
|-----------|---------|
| `'S'` | `Start` |
| `'J'` | `Jail` |
| anything else | `Ticket` |

---

#### `void createPlayers(const int total)`

Loops `total` times. For each iteration:
1. Prints `"Player N name: "` and reads a name.
2. Calls `nameCheck(name)`; if the name is already taken, prompts again.
3. Creates `new Player(name, colorIndx)` and pushes it to `players`.

Colors cycle through Windows console color indices 9–15 (blue through white), wrapping back to 9 after 15.

---

#### `void createDeck()`

Seeds `srand` with the current time, then pushes `CARDS` (20) random integers in the range [−350, +350] onto `deck` using `rand() % 700 - 350`. Calls `drawFromDeck()` once to discard the first card (cycle the deck).

---

#### `void startGame()`

**The main game loop.**

```
while (players.size() > 1):
    print current player name and balance in their color
    if player is in jail:
        release jail flag, skip turn, advance to next player
    else:
        result = turn(currentPlayer)
        if result == false:
            delete and erase player from deque
        else:
            print end-of-turn summary
            if last player in round, print all ownership
    advance iterator (wrap to begin after last player)

print winner name in white
```

---

#### `const bool turn(Player* p)`

Handles a single player's full turn:

1. Calls `quitOrPlay()`; if player enters `0`, prints a quit message and returns `false`.
2. Calls `rollDice()` to get a value 1–6.
3. Calls `movePlayer(p, diceValue)` to advance the player and get the landed `Slot*`.
4. Calls `temp->printSlot()` to display the new slot.
5. Downcasts the slot:
   - If `ASST(temp)` succeeds → calls `assetOptions(p, asset)`.
   - Otherwise → calls `instOptions(p, instruction)`.
6. Returns the result of whichever options function was called.

---

#### `const bool quitOrPlay()`

Prints the quit/play prompt and delegates to `choose_0_or_1()`. Returns `false` for quit, `true` for play.

---

#### `const int rollDice()`

Seeds `srand` with the current time and returns `(rand() % 6) + 1` — a value in [1, 6]. Prints the rolled value.

---

#### `Slot* GameEngine::movePlayer(Player* p, const int dice)`

Advances the player `dice` steps along the board:

1. Gets an iterator to the player's current slot via `findSlot(p->getPosition())`.
2. Loops `dice` times:
   - If the iterator equals the last slot's iterator, it wraps to `board.begin()` (slot 1).
   - If there are still remaining steps after wrapping (i.e., the player passes Start without landing on it), calls `instOptions(p, Start_slot)` to award +350 NIS.
   - Otherwise advances the iterator.
3. Updates the player's position via `p->setPosition((*iter)->getSlotNum())`.
4. Returns the pointer to the new slot.

---

#### `const bool assetOptions(Player* p, Asset* house)`

Handles landing on a property slot:

| Condition | Action |
|-----------|--------|
| Property unowned, player **cannot** afford it | Print message; no transaction |
| Property unowned, player **can** afford it | Prompt buy (1) or skip (0); if yes, call `p->buyAsset(house)` |
| Property owned by **this player** | Print "This is your asset" |
| Property owned by **another player**, not mortgaged | Print rent info; call `p->payment(-rent, owner)` |
| Property owned by **another player**, mortgaged | Print bank message; call `p->payment(-rent)` (no recipient) |

Returns `false` only if `p->payment()` returns `false` (player eliminated).

---

#### `const bool instOptions(Player* p, Instruction* inst)`

Dispatches on `inst->getSlotType()`:

| Type | Actions |
|------|---------|
| `Start` | `p->payment(START_SUM)` (+350 NIS, auto-releases mortgages); prints new balance; `p->findMortgage()` (increments mortgage years) |
| `Ticket` | `drawFromDeck()` → print drawn value → `p->payment(value)` |
| `Jail` | Print jail message; `p->setJail(true)` |

Returns the result of `p->payment()` for Ticket; `true` for Start and Jail.

---

#### `const int drawFromDeck()`

Reads the front value of `deck`, pops it, then pushes it back to the rear (circular recycling). Returns the drawn value.

---

#### `const bool choose_0_or_1(int option1 = 0, int option2 = 1)`

Reads an integer from `cin`. Re-prompts in a loop if:
- `cin` fails (non-integer input), or
- The value is neither `option1` nor `option2`.

Returns the chosen value as a `bool` (0 → `false`, 1 → `true`).

---

#### `void boardPrint()`

Debug utility. Prints every slot's number and name to `cout`. Not called during normal gameplay (the call in `buildBoard()` is commented out).

---

#### `list<Slot*>::iterator findSlot(const int indx)`

Returns an iterator to the slot at 1-based position `indx` by stepping forward `indx - 1` times from `board.begin()`.

---

#### `void owners()`

Resets the console color to white, then prints each player's name followed by their owned assets (via `getHouses()`). Called at the end of every full round (after the last player's turn).

---

#### `const bool nameCheck(const string str)`

Iterates over `players` and returns `false` (printing a warning) if any existing player already has the name `str`; returns `true` if the name is unique.

---

## Entry Point: `main()`

**File:** `Main.cpp`

```
1. Print welcome message.
2. Read playersNum from cin.
3. Throw BAD_INPUT if cin fails.
4. Throw "Not Enough Players!" if playersNum < 2.
5. Allocate new GameEngine(playersNum).
   → This triggers buildBoard(), createPlayers(), createDeck(), startGame().
6. Explicitly call temp->~GameEngine() to clean up.
7. Catch and print any const char* exceptions.
8. system("pause") — wait for user to press a key before the window closes.
```

---

## Data Structures

| Container | Type | Where Used | Why |
|-----------|------|-----------|-----|
| `list<Slot*>` | Doubly-linked list | `GameEngine::board` | Efficient sequential traversal; supports wrapping via iterators |
| `deque<Player*>` | Double-ended queue | `GameEngine::players` | Efficient deletion from the middle when a player is eliminated; iterator stays valid after erase |
| `queue<int>` | FIFO queue | `GameEngine::deck` | Natural circular-deck behavior: pop front, push back |
| `vector<Asset*>` | Dynamic array | `Player::houses` | Random access for mortgage iteration; grows as properties are purchased |

---

## Game Flow

```
main()
 └─ new GameEngine(N)
      │
      ├─ buildBoard()
      │    reads board.txt line by line
      │    creates Instruction or Asset objects → pushes to board list
      │
      ├─ createPlayers(N)
      │    for each player: read name, check duplicate, new Player → deque
      │
      ├─ createDeck()
      │    push 20 random ints in [-350, 350] → deck queue
      │
      └─ startGame()
           │
           while (players.size() > 1)
           │
           ├─ [player in jail?]
           │     yes → clear jail flag, skip turn, advance iterator
           │
           └─ turn(player)
                 │
                 ├─ quitOrPlay()  ──── 0 → return false (player eliminated)
                 │
                 ├─ rollDice()  →  1-6
                 │
                 ├─ movePlayer()
                 │    advance iterator by dice steps
                 │    if wrap past Start → +350 NIS (instOptions Start)
                 │    return new Slot*
                 │
                 └─ landed on Asset?
                       yes → assetOptions()
                              unowned + can afford → offer to buy
                              owned by other → pay rent (or to bank if mortgaged)
                              own slot → message only
                       no  → instOptions()
                              Start  → +350 NIS, auto-release mortgages, increment mortgage years
                              Ticket → draw card, payment(±value)
                              Jail   → set jail flag

           player.payment() < 0 and can't cover → playerLost()
             transfer money to creditor, clear all assets → return false
             GameEngine erases player from deque

           last player remaining → print winner
```

---

## Board Configuration Format

The board is defined in `boardGame/board.txt`. The file is read top-to-bottom; each line becomes one slot numbered sequentially from 1.

**Instruction slot syntax:**
```
I,<code>,<name>
```
- `<code>` is one of `S` (Start), `J` (Jail), `T` (Ticket).
- `<name>` is the display name (read until end of line).
- Exactly one `S` slot must exist; a second `S` throws an error.

**Property slot syntax:**
```
P,<city>,<street>,<buyPrice>,<rentPrice>
```
- `<city>` and `<street>` are strings.
- `<buyPrice>` and `<rentPrice>` are integers (parsed with `atoi`).

**Example:**
```
I,S,Start : get 350 Nis
P,City1,property1,150,20
I,J,Jail
I,T,Get Ticket
P,City2,property2,450,90
```

After the last slot, the board wraps back to slot 1 when a player's movement would carry them past the end.
