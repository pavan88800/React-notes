# Tic-Tac-Toe Implementation - Mental Models & Notes

## 📚 Core Concepts

### 1. Grid Representation

**Mental Model:** Flatten 2D grid into 1D array

```
Visual 3x3 Grid:        Flattened Array:
[0] [1] [2]            [0, 1, 2, 3, 4, 5, 6, 7, 8]
[3] [4] [5]
[6] [7] [8]
```

**Formula:** `index = row * cols + col`

**Example:** For a 3x3 grid, position (1,2) → `1 * 3 + 2 = 5`

---

### 2. History & Time Travel Pattern

**Mental Model:** Array of game states (snapshots)

```javascript
history = [
  [" ", " ", " ", ...],  // Initial state (move 0)
  ["X", " ", " ", ...],  // After first move (move 1)
  ["X", "O", " ", ...],  // After second move (move 2)
  // ... more states
]
```

**Key Insight:**

- `move` is a pointer into the history array
- Undo: `move--` (go back in time)
- Redo: `move++` (go forward in time)
- New move: Slice history up to current move, then add new state

```javascript
// When making a new move from the past:
const newHistory = history.slice(0, move + 1); // Discard future
setHistory([...newHistory, newGrid]); // Add new present
```

---

### 3. Winner Detection Logic

#### 3.1 Generating Win Combinations

**Mental Model:** Every way to win = a set of indices

```javascript
// For 3x3 grid:
Rows: [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8]
];
Columns: [
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8]
];
Diagonals: [
  [0, 4, 8],
  [2, 4, 6]
];
```

**Pattern Recognition:**

- **Row i:** `[i*cols, i*cols+1, i*cols+2, ...]`
- **Column j:** `[j, j+cols, j+2*cols, ...]`
- **Main Diagonal:** `[0, cols+1, 2*cols+2, ...]` → Pattern: `i*cols + i`
- **Anti Diagonal:** `[cols-1, 2*cols-2, ...]` → Pattern: `i*cols + (cols-1-i)`

#### 3.2 Checking for Winner - The Key Insight

**Mental Model:** "Pick any cell as reference, check if all others match"

```javascript
const firstCell = grid[combo[0]];
if (firstCell && combo.every((index) => grid[index] === firstCell)) {
  winner = firstCell;
}
```

**🔑 Critical Realization:**
You can use **ANY index** from the combo, not just `combo[0]`!

```javascript
// ALL THESE ARE EQUIVALENT:
const firstCell = grid[combo[0]]; // ✅ Convention
const firstCell = grid[combo[1]]; // ✅ Also works
const firstCell = grid[combo[2]]; // ✅ Also works
const firstCell = grid[combo[combo.length - 1]]; // ✅ Also works
```

**Why?**
If all cells in a combo are the same (winning condition), it doesn't matter which one you pick as the reference point!

**Example:**

```javascript
combo = [0, 4, 8]  // Main diagonal
grid = ['X', '', '', '', 'X', '', '', '', 'X']

// Using combo[0]:
firstCell = grid[0] = 'X'
grid[0] === 'X' ✅
grid[4] === 'X' ✅
grid[8] === 'X' ✅
Winner: X

// Using combo[1]:
firstCell = grid[4] = 'X'
grid[0] === 'X' ✅
grid[4] === 'X' ✅
grid[8] === 'X' ✅
Winner: X  // Same result!
```

**Edge Cases to Handle:**

1. Empty cell check: `firstCell` must be truthy (not "")
2. All cells must match: `combo.every(...)`

---

### 4. Current Player Calculation

**Mental Model:** Alternate players based on move number

```javascript
currentPlayer = move % 2 === 0 ? "X" : "O";
```

**Pattern:**

- Move 0 → 0 % 2 = 0 → X
- Move 1 → 1 % 2 = 1 → O
- Move 2 → 2 % 2 = 0 → X
- Move 3 → 3 % 2 = 1 → O

**Why not track in state?**

- Derived data (can be calculated from move)
- Follows single source of truth principle
- Automatically correct during undo/redo

---

### 5. Draw Detection

**Mental Model:** "No winner + all cells filled = draw"

```javascript
const isDraw = !winner && grid.every((cell) => cell !== "");
```

**Order matters:**

1. Check winner first
2. If no winner AND grid is full → draw

---

## 🧠 Key Mental Models Summary

### 1. **State Machine Model**

```
Game States:
┌─────────┐
│ Playing │ ──[move]──> Continue
│         │ ──[win]───> Winner State
│         │ ──[full]──> Draw State
└─────────┘
```

### 2. **Time Travel Model**

```
History Array = Timeline
     ↓
[past, past, PRESENT, future, future]
              ↑
           move pointer
```

### 3. **Reference Cell Pattern**

```
When checking equality:
[A, A, A, A] → Pick any A, check if all match
               ↑ This one? ✅
                  ↑ Or this? ✅
                     ↑ Or this? ✅
Result is the same!
```

---

## 🎯 Design Decisions & Tradeoffs

### Why flatten the grid?

- **Pro:** Simpler indexing, easier to iterate
- **Pro:** Natural fit with React's single array state
- **Con:** Less intuitive than 2D array
- **Verdict:** Performance + simplicity wins

### Why store full history vs just moves?

- **Alternative:** Store only moves `[{index: 0, player: 'X'}, ...]`
- **Pro of full history:** O(1) undo/redo (just change pointer)
- **Con:** More memory (each state = full grid copy)
- **Verdict:** Better UX (instant undo/redo) worth the memory

### Why `useMemo` for winner calculation?

- Prevents recalculating on every render
- Only recalculates when grid actually changes
- Optimization matters as grid size increases

---

## 🔧 Common Pitfalls & Fixes

### ❌ Pitfall 1: Buttons inside grid container

```javascript
// Wrong:
<div className="grid">
  {cells}
  <button>Undo</button> // Renders as grid cell!
</div>
```

### ✅ Fix:

```javascript
<div className="grid">{cells}</div>
<div className="controls">
  <button>Undo</button>
</div>
```

### ❌ Pitfall 2: Not slicing history on new move

```javascript
// Wrong:
setHistory([...history, newGrid]); // Keeps divergent timelines!
```

### ✅ Fix:

```javascript
const newHistory = history.slice(0, move + 1); // Cut off future
setHistory([...newHistory, newGrid]);
```

### ❌ Pitfall 3: Diagonals on non-square grids

```javascript
// Wrong: Always checking diagonals
// For 3x4 grid, diagonal doesn't make sense!
```

### ✅ Fix:

```javascript
if (rows === cols) {
  // Only add diagonal combinations for square grids
}
```

---

## 📐 Scalability Considerations

### Current Implementation

- ✅ Works for any NxN or MxN grid
- ✅ Dynamic win detection
- ⚠️ Performance degrades with large grids (10x10+)

### For Production/Large Grids:

1. **Optimize win check:** Don't check ALL combos every time
   - Only check combos involving the last move
2. **Limit history:** Cap at last N moves
3. **Virtual scrolling:** For very large grids
4. **Web Workers:** Move win detection off main thread

---

## 🎓 Learning Takeaways

1. **Derived State > Stored State:** Calculate from existing data when possible
2. **Immutability:** Always create new arrays/objects, never mutate
3. **Single Source of Truth:** `move` pointer controls everything
4. **Algorithm Independence:** Reference cell can be any combo element
5. **Time Complexity:** Winner check = O(combos × combo length)

---

## 🚀 Extension Ideas

### Easy:

- [ ] Add score tracking across games
- [ ] Highlight winning combination
- [ ] Add player names input
- [ ] Sound effects on moves

### Medium:

- [ ] AI opponent (minimax algorithm)
- [ ] Different grid sizes selector
- [ ] Connect-N (win with N in a row, not just full row)
- [ ] Online multiplayer

### Hard:

- [ ] Tournament mode
- [ ] Replay with animation
- [ ] Custom win conditions
- [ ] Variable grid shapes (hexagonal, etc.)

---

## 📖 References & Further Reading

- **Time Travel Pattern:** [React Docs - Tutorial](https://react.dev/learn/tutorial-tic-tac-toe)
- **Immutability:** Why we copy arrays instead of mutating
- **Game State Machines:** Modeling game logic as state transitions
- **Minimax Algorithm:** For AI opponent implementation

---

---

- [Old Version (React.js)](https://codesandbox.io/p/sandbox/jnr53v?file=%2Fsrc%2FApp.js%3A129%2C6)

---

- [New Version (React.js)](https://codesandbox.io/p/sandbox/jnr53v?file=%2Fsrc%2FApp.js%3A14%2C40)
