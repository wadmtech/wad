# JavaScript Lab Exam Prep Kit: Bingo and DOM Games

*Practice papers, compact model solutions, a cheat sheet and viva questions for ICS1511-style JavaScript lab exercises.*

## 1. How these lab exams work

Exercises like the Bingo game follow one pattern: **a small turn-based game on a web page.** The same skeleton solves almost all of them (Bingo, Tic-Tac-Toe, Memory Match, Number Guessing):

1. **State:** variables that hold the game (arrays, whose turn, game over?).
2. **Render:** a function that builds the board in the DOM from the state.
3. **Handlers:** `onclick` for Start, Reset and Mark that update the state, then re-render.
4. **Win check:** one function that tests the state for a winning condition.
5. **Messages:** one `<p id='msg'>` that tells the user what happened.

If you can write those five pieces from memory, you can write any variant.

**Suggested time plan for a full game (about 35 min):** HTML skeleton 3, CSS 7, state and render 8, handlers 8, win check 5, testing and bonus 4.

## 2. Practice papers

### Drill 1: Click counter (5 min, 10 marks)

A button labelled *Click me* and a text showing the count. Each click adds 1. At 10 the count turns red. A *Reset* button sets it back to 0. **Skills:** `getElementById`, `onclick`, `textContent`, `style`/`classList`.

### Drill 2: Random 5x5 card (8 min, 15 marks)

A *Start* button builds a 5x5 grid containing the numbers 1 to 25 once each in random order (Fisher–Yates shuffle). Clicking a cell marks it green. A *Reset* button clears the grid. **Skills:** loops, `createElement`, CSS Grid, shuffle.

### Drill 3: Win checker (8 min, 15 marks)

Given a 25-item array of true/false (marked or not), write `hasBingo(marked)` that returns `true` if any row, column or diagonal is fully true. Test it in the console with at least one row, one column and one diagonal case. **Skills:** nested loops or `some`/`every`, index arithmetic (`row*5+col`).

### Drill 4: Input validation (5 min, 10 marks)

An input and a *Check* button. Show a message for each case: empty, not a number, outside 1–25, valid. Use `alert` first, then replace it with an on-page message. **Skills:** `parseInt`/`Number`, `isNaN`, `if/else`.

### Paper A: Full two-player Bingo (35 min, 50 marks)

Build the Bingo game described in the lab sheet.

| # | Requirement | Marks |
| --- | --- | --- |
| 1 | Two 5x5 cards, random numbers 1–25, Start button generates them | 10 |
| 2 | Reset clears everything and allows a fresh start | 5 |
| 3 | Players take turns; input is validated (1–25, not already used) | 8 |
| 4 | Mark button highlights the number on both cards if present | 7 |
| 5 | Win check for row, column and diagonal; winner message; game stops | 10 |
| 6 | Clean UI: aligned buttons and inputs, distinct marked and winning styles | 5 |
| 7 | Bonus: AI opponent, sound effects or animation | +5 |

### Variant papers (practice as 20-minute builds)

Examiners often change the game but keep the pattern. Practice these:

- **Paper B: Tic-Tac-Toe.** 3x3 grid, X and O alternate by clicking cells, announce the winner or a draw, Reset button. *Win lines:* 3 rows, 3 columns, 2 diagonals (8 lines). Draw = all 9 cells filled with no winner.
- **Paper C: Number guessing.** The computer picks 1–100. After each guess say *Too high*, *Too low* or *Correct*. Limit to 7 attempts and show the attempts left. Reset starts a new game.
- **Paper D: Memory match.** 4x4 grid with 8 pairs of emoji, face down. Click two cards: matching pairs stay open, others flip back after 800 ms (`setTimeout`). Show the move count and a win message.
- **Paper E: Dice race.** Two players press *Roll* in turns (random 1–6); first to 30 points wins. Show both scores and whose turn it is.

## 3. Model solutions

### 3.1 Paper A: compact exam-speed Bingo (single file)

```html
<!DOCTYPE html>
<html lang='en'>
<head>
<meta charset='UTF-8'>
<title>Bingo Game</title>
<style>
body{font-family:Arial,sans-serif;text-align:center;background:#d8f3dc;color:#1b4332}
.boards{display:flex;flex-wrap:wrap;justify-content:center;gap:30px}
.card{display:grid;grid-template-columns:repeat(5,50px);gap:5px;padding:8px;background:#fff;border-radius:8px}
.cell{height:50px;line-height:50px;border:1px solid #2d9b78;border-radius:50%;font-weight:bold}
.marked{background:#2d9b78;color:#fff}
.win{background:gold;color:#000}
button{padding:8px 18px;border:0;border-radius:5px;background:#2d9b78;color:#fff;font-weight:bold;cursor:pointer}
</style>
</head>
<body>
<h1>Bingo Game</h1>
<div class='boards'>
  <div><h2>Player 1's Card</h2><div class='card' id='c0'></div></div>
  <div><h2>Player 2's Card</h2><div class='card' id='c1'></div></div>
</div>
<p><button id='start'>Start</button> <button id='reset'>Reset</button></p>
<p>Enter a number (1-25): <input type='number' id='num' min='1' max='25'> <button id='mark'>Mark</button></p>
<h3 id='msg'></h3>

<script>
let cards = [[], []], called = new Set(), turn = 0, over = false;
const $ = id => document.getElementById(id);

// 12 winning lines as index lists
const LINES = [];
for (let i = 0; i < 5; i++) {
  LINES.push([0,1,2,3,4].map(c => i*5 + c));   // row i
  LINES.push([0,1,2,3,4].map(r => r*5 + i));   // column i
}
LINES.push([0,6,12,18,24], [4,8,12,16,20]);    // diagonals

function shuffle() {                           // Fisher-Yates
  const a = Array.from({length: 25}, (_, i) => i + 1);
  for (let i = 24; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

function draw(p) {
  $('c' + p).innerHTML = '';
  cards[p].forEach(n => {
    const d = document.createElement('div');
    d.className = 'cell';
    d.id = `c${p}_${n}`;
    d.textContent = n;
    $('c' + p).appendChild(d);
  });
}

function winLine(p) {
  return LINES.find(l => l.every(i => called.has(cards[p][i])));
}

$('start').onclick = () => {
  cards = [shuffle(), shuffle()];
  called.clear(); turn = 0; over = false;
  draw(0); draw(1);
  $('msg').textContent = "Player 1's turn";
};

$('reset').onclick = () => {
  cards = [[], []]; called.clear(); over = false;
  $('c0').innerHTML = ''; $('c1').innerHTML = '';
  $('msg').textContent = ''; $('num').value = '';
};

$('mark').onclick = () => {
  if (over || cards[0].length === 0) return;      // not started or finished
  const n = Number($('num').value);
  if (!Number.isInteger(n) || n < 1 || n > 25 || called.has(n)) {
    $('msg').textContent = 'Enter an unused number from 1 to 25';
    return;
  }
  called.add(n);
  [0, 1].forEach(p => $(`c${p}_${n}`).classList.add('marked'));
  $('num').value = '';

  const winners = [0, 1].filter(p => winLine(p));
  if (winners.length) {
    over = true;
    winners.forEach(p => winLine(p).forEach(i => $(`c${p}_${cards[p][i]}`).classList.add('win')));
    $('msg').textContent = winners.length === 2 ? 'Tie! Both players have Bingo' : `Bingo! Player ${winners[0] + 1} wins`;
    return;
  }
  turn = 1 - turn;
  $('msg').textContent = `Player ${turn + 1}'s turn`;
};
</script>
</body>
</html>
```

**Bonus add-ons (2 minutes each):**

```js
// Sound: no audio file needed
function beep(freq = 660, dur = 0.12) {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const o = ctx.createOscillator(), g = ctx.createGain();
  o.frequency.value = freq; o.connect(g); g.connect(ctx.destination);
  g.gain.setValueAtTime(0.15, ctx.currentTime);
  g.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + dur);
  o.start(); o.stop(ctx.currentTime + dur);
}

// AI opponent: Player 2 picks a random unused number after a delay
function aiMove() {
  const free = [];
  for (let n = 1; n <= 25; n++) if (!called.has(n)) free.push(n);
  $('num').value = free[Math.floor(Math.random() * free.length)];
  $('mark').click();
}
// after switching turn:  if (turn === 1) setTimeout(aiMove, 900);
```

```css
.marked{animation:pop .35s}
@keyframes pop{0%{transform:scale(.4)}60%{transform:scale(1.25)}100%{transform:scale(1)}}
```

### 3.2 Paper B: Tic-Tac-Toe core logic

```js
let board = Array(9).fill(''), player = 'X', over = false;
const WINS = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];

function clickCell(i) {
  if (over || board[i]) return;
  board[i] = player;
  document.getElementById('cell' + i).textContent = player;
  if (WINS.some(l => l.every(k => board[k] === player))) {
    over = true; msg.textContent = player + ' wins!'; return;
  }
  if (board.every(c => c)) { over = true; msg.textContent = 'Draw'; return; }
  player = player === 'X' ? 'O' : 'X';
}
```

### 3.3 Paper C: Number guessing core logic

```js
let secret = Math.floor(Math.random() * 100) + 1, left = 7;
function guess() {
  const g = Number(document.getElementById('g').value);
  if (!Number.isInteger(g) || g < 1 || g > 100) { msg.textContent = 'Enter 1-100'; return; }
  left--;
  if (g === secret) msg.textContent = 'Correct!';
  else if (left === 0) msg.textContent = 'Out of attempts. It was ' + secret;
  else msg.textContent = (g > secret ? 'Too high' : 'Too low') + ', ' + left + ' left';
}
```

## 4. Cheat sheet

### 4.1 Wiring JS to HTML

```html
<script src='game.js'></script>   <!-- external, placed just before </body> -->
<script> ...code... </script>     <!-- internal -->
```

Place scripts **after** the HTML they use, or the elements will not exist yet (`null` errors).

### 4.2 DOM essentials

| Task | Code |
| --- | --- |
| Select by id | `document.getElementById('msg')` |
| Select by CSS | `document.querySelector('.cell')`, `document.querySelectorAll('.cell')` |
| Change text | `el.textContent = 'Hi'` |
| Change HTML | `el.innerHTML = ''` (also used to clear a board) |
| Read input | `Number(input.value)` or `parseInt(input.value, 10)` |
| Set input | `input.value = ''` |
| Add or remove class | `el.classList.add('marked')`, `.remove()`, `.toggle()`, `.contains()` |
| Inline style | `el.style.backgroundColor = 'green'` |
| Create and attach | `const d = document.createElement('div'); d.textContent = 5; parent.appendChild(d);` |
| Remove | `el.remove()` |
| Enable or disable | `button.disabled = true` |
| Attributes | `el.setAttribute('id', 'x')`, `el.id`, `el.dataset.index` (from `data-index='3'`) |

### 4.3 Events

```js
btn.onclick = () => { ... };                         // simple
btn.addEventListener('click', handler);              // preferred
input.addEventListener('keydown', e => { if (e.key === 'Enter') markNumber(); });
```

`onclick='fn()'` in HTML also works, but the function must be global.

### 4.4 Core JS you will use

| Need | Code |
| --- | --- |
| Variables | `let` (changes), `const` (fixed). Avoid `var` |
| Random integer 1–N | `Math.floor(Math.random() * N) + 1` |
| Array of 1..25 | `Array.from({length: 25}, (_, i) => i + 1)` |
| Fisher–Yates swap | `[a[i], a[j]] = [a[j], a[i]]` |
| Any or all test | `arr.some(fn)`, `arr.every(fn)` |
| Search | `arr.includes(x)`, `arr.indexOf(x)`, `arr.find(fn)` |
| Loop | `for (let i = 0; i < n; i++)`, `arr.forEach(fn)`, `arr.map(fn)`, `arr.filter(fn)` |
| Unique store | `const s = new Set(); s.add(3); s.has(3); s.clear()` |
| Row and column to index | `index = row * 5 + col`; reverse: `row = Math.floor(index / 5)`, `col = index % 5` |
| Toggle player | `turn = 1 - turn` (0/1) or `player = player === 'X' ? 'O' : 'X'` |
| Delay | `setTimeout(fn, 800)`; repeat: `setInterval(fn, 1000)` with `clearInterval(id)` |

### 4.5 Bingo geometry (memorise)

```
 0  1  2  3  4        rows:      [i*5 .. i*5+4]
 5  6  7  8  9        columns:   [i, i+5, i+10, i+15, i+20]
10 11 12 13 14        diagonal:  0, 6, 12, 18, 24
15 16 17 18 19        anti-diag: 4, 8, 12, 16, 20
20 21 22 23 24
```

Rule of thumb: diagonal step = 6, anti-diagonal step = 4.

### 4.6 Styling a game board

```css
.board{display:grid;grid-template-columns:repeat(5,1fr);gap:5px}   /* grid for the cells */
.row{display:flex;justify-content:center;gap:10px}                 /* flex for buttons/inputs */
.cell{aspect-ratio:1;display:flex;align-items:center;justify-content:center}
.cell:hover{background:#eee;cursor:pointer}
.marked{background:green;color:#fff}      /* state class added from JS */
button:disabled{background:#ccc;cursor:not-allowed}
```

### 4.7 Debugging

- Press **F12**, open the **Console**: red errors give the file and line.
- `console.log(cards, called)` to inspect state.
- `Cannot set properties of null` means the id is wrong or the script runs before the element exists.
- Nothing happens on click: check the handler name and the spelling of the id.
- `NaN`: you read a string; wrap with `Number()`.

### 4.8 Common mistakes that lose marks

1. Using `==` between a string and a number. Convert the input first and use `===`.
2. Forgetting to stop the game after a win (`over` flag), so players keep marking.
3. Not validating: empty input, 0, 26, repeated numbers.
4. Reset that clears the screen but not the state (`called`, `turn`, `over`).
5. Generating duplicate numbers on a card (use shuffle, not repeated `Math.random()`).
6. Checking only the current player when the lab says to mark **both** cards. If both complete a line on the same number, declare a tie.
7. Duplicate `id` values (ids must be unique, so include the player number in the id).
8. Win check only for rows. Test row, column and both diagonals.

## 5. Viva questions with short answers

1. **Why shuffle instead of calling random 25 times?** Random calls can repeat numbers. Shuffling a 1–25 array guarantees each number appears exactly once.
2. **How many winning lines are there on a 5x5 card?** 12: 5 rows, 5 columns, 2 diagonals.
3. **What is the DOM?** The browser's tree of objects built from the HTML. JavaScript reads and changes it.
4. **`textContent` vs `innerHTML`?** `textContent` treats the value as plain text. `innerHTML` parses HTML, which is useful for clearing but risky with user input.
5. **`let` vs `const` vs `var`?** `let` can be reassigned and is block scoped. `const` cannot be reassigned. `var` is function scoped and hoisted, so avoid it.
6. **Why is the input always a string?** `input.value` returns text. Convert with `Number()` or `parseInt`.
7. **What does `every` do?** Returns true only if the callback is true for all items. `some` returns true if it is true for at least one.
8. **How do you stop the game after someone wins?** A boolean flag (`over`) checked at the start of the Mark handler, and the input and Mark button are disabled.
9. **How is the AI opponent implemented?** On Player 2's turn it picks a random number from those not yet called, fills the input and triggers Mark, after a `setTimeout` delay.
10. **`==` vs `===`?** `==` converts types before comparing, `===` does not. Prefer `===`.
11. **What is event handling?** Running a function in response to a user action such as click or keydown.
12. **How does CSS Grid help here?** `grid-template-columns:repeat(5,1fr)` turns a flat list of 25 divs into a 5x5 board automatically.

## 6. Last-minute checklist

1. Start, Reset, Mark and the input all exist and work.
2. Cards are 5x5 with unique numbers from 1 to 25.
3. Input validation covers empty, out of range and repeated numbers.
4. A marked number highlights on **both** cards.
5. Row, column and diagonal wins all trigger. Test each one by hand.
6. A winner message appears and the game stops. Reset starts a fresh game.
7. Marked and winning cells look different. Layout is aligned.
8. Console shows no red errors. Then try the bonus if time remains.
