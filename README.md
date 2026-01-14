# chess.com-in
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Classical Chess</title>

  <!-- Chessboard.js CSS (CDN) -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css">
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <div class="app">
    <header class="header">
      <h1>Classical Chess</h1>
      <div class="status">
        <span id="turn">White to move</span>
        <span id="state">Ready</span>
        <span id="fen" class="fen"></span>
      </div>
    </header>

    <main class="layout">
      <section class="board-wrap">
        <div class="coords top">
          <span>A</span><span>B</span><span>C</span><span>D</span><span>E</span><span>F</span><span>G</span><span>H</span>
        </div>
        <div class="board-frame">
          <div id="board" class="board"></div>
          <div id="overlay" class="overlay"></div>
        </div>
        <div class="coords bottom">
          <span>A</span><span>B</span><span>C</span><span>D</span><span>E</span><span>F</span><span>G</span><span>H</span>
        </div>
        <div class="ranks left">
          <span>8</span><span>7</span><span>6</span><span>5</span><span>4</span><span>3</span><span>2</span><span>1</span>
        </div>
        <div class="ranks right">
          <span>8</span><span>7</span><span>6</span><span>5</span><span>4</span><span>3</span><span>2</span><span>1</span>
        </div>
      </section>

      <aside class="panel">
        <div class="controls">
          <button id="newGame">New game</button>
          <button id="undo">Undo</button>
          <button id="redo">Redo</button>
          <button id="flip">Flip board</button>
          <button id="exportPGN">Export PGN</button>
        </div>

        <div class="moves">
          <h3>Move list</h3>
          <ol id="moveList"></ol>
        </div>

        <div class="info">
          <h3>Game status</h3>
          <ul>
            <li><strong>Check:</strong> <span id="checkFlag">No</span></li>
            <li><strong>Checkmate:</strong> <span id="mateFlag">No</span></li>
            <li><strong>Draw:</strong> <span id="drawFlag">No</span></li>
          </ul>
        </div>
      </aside>
    </main>

    <footer class="footer">
      <small>Staunton pieces • chess.js + chessboard.js • Classical wood theme</small>
    </footer>
  </div>

  <!-- Promotion modal -->
  <div id="promotionModal" class="modal hidden">
    <div class="modal-content">
      <h3>Choose promotion</h3>
      <div class="promo-choices">
        <button data-piece="q">Queen</button>
        <button data-piece="r">Rook</button>
        <button data-piece="b">Bishop</button>
        <button data-piece="n">Knight</button>
      </div>
      <button id="cancelPromotion" class="cancel">Cancel</button>
    </div>
  </div>

  <!-- Dependencies -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>

  <script src="script.js"></script>
</body>
</html>
// Game state
const game = new Chess();
let board = null;
let isFlipped = false;
const moveListEl = document.getElementById("moveList");
const statusEl = document.getElementById("status");
const fenEl = document.getElementById("fen");

// Helpers
function updateStatus() {
  let status = "";
  const moveColor = game.turn() === "w" ? "White" : "Black";

  if (game.in_checkmate()) {
    status = `Checkmate — ${moveColor === "White" ? "Black" : "White"} wins`;
  } else if (game.in_draw()) {
    status = "Draw";
  } else {
    status = `${moveColor} to move`;
    if (game.in_check()) status += " — Check";
  }

  statusEl.textContent = status;
  fenEl.textContent = game.fen();
}

function addMoveToList(move) {
  const li = document.createElement("li");
  li.textContent = `${move.san}`;
  moveListEl.appendChild(li);
  moveListEl.scrollTop = moveListEl.scrollHeight;
}

function resetMoves() {
  moveListEl.innerHTML = "";
}

// Board config
function onDragStart(source, piece, position, orientation) {
  if (game.game_over()) return false;
  // Only allow dragging current player's pieces
  if ((game.turn() === "w" && piece.search(/^b/) !== -1) ||
      (game.turn() === "b" && piece.search(/^w/) !== -1)) {
    return false;
  }
}

function onDrop(source, target) {
  const move = game.move({
    from: source,
    to: target,
    promotion: "q" // auto promote to queen for simplicity
  });

  if (move === null) return "snapback";

  addMoveToList(move);
  updateStatus();
}

function onSnapEnd() {
  board.position(game.fen());
}

// Initialize board
function initBoard() {
  board = Chessboard("board", {
    position: "start",
    draggable: true,
    onDragStart,
    onDrop,
    onSnapEnd
  });
  updateStatus();
}

// Controls
document.getElementById("newGame").addEventListener("click", () => {
  game.reset();
  board.start();
  resetMoves();
  updateStatus();
});

document.getElementById("undo").addEventListener("click", () => {
  const undone = game.undo();
  if (undone) {
    // Remove last move from list
    if (moveListEl.lastChild) moveListEl.removeChild(moveListEl.lastChild);
    board.position(game.fen());
    updateStatus();
  }
});

document.getElementById("flip").addEventListener("click", () => {
  isFlipped = !isFlipped;
  board.flip();
});

// Boot
window.addEventListener("load", initBoard);
window.addEventListener("load", () => {
  const game = new Chess();
  const board = Chessboard("board", {
    draggable: true,
    position: "start",
    onDrop: (source, target) => {
      const move = game.move({ from: source, to: target, promotion: "q" });
      if (move === null) return "snapback";
    }
  });

:root {
  --bg: #0f1115;
  --panel: #161a22;
  --text: #e6edf3;
  --muted: #9aa4b2;
  --accent: #66ccff;
}

* { box-sizing: border-box; }
body {
  margin: 0;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
  background: var(--bg);
  color: var(--text);
}

.app {
  max-width: 1100px;
  margin: 24px auto;
  padding: 0 16px;
}

header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 16px;
}

h1 {
  margin: 0;
  font-size: 24px;
}

.status {
  display: flex;
  gap: 16px;
  color: var(--muted);
}

main {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: 16px;
}

.board {
  max-width: 640px;
  width: 100%;
  margin: 0 auto;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0,0,0,0.35);
}

.panel {
  background: var(--panel);
  border: 1px solid #222836;
  border-radius: 8px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.controls {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

button {
  background: #22293a;
  color: var(--text);
  border: 1px solid #2b3347;
  border-radius: 6px;
  padding: 8px 12px;
  cursor: pointer;
}
button:hover { border-color: var(--accent); }

.moves h3 {
  margin: 0 0 8px 0;
  font-size: 16px;
  color: var(--muted);
}
#moveList {
  margin: 0;
  padding-left: 18px;
  max-height: 360px;
  overflow: auto;
}

footer {
  margin-top: 16px;
  color: var(--muted);
  text-align: center;
}

/* Chessboard colors override */
.white-1e1d7 { background: #f0d9b5; }
.black-3c85d { background: #b58863; }

/* Responsive */
@media (max-width: 900px) {
  main { grid-template-columns: 1fr; }
  .panel { order: -1; }
}
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chess.js@1.0.0/dist/chess.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chessboardjs@1.0.0/www/js/chessboard.js"></script>
<script src="script.js"></script>
