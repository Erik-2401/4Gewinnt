<script lang="ts">
  type Player = 1 | 2;
  type Cell = Player | null;

  const ROWS = 6;
  const COLS = 7;

  let board = $state<Cell[][]>(
    Array(ROWS).fill(null).map(() => Array(COLS).fill(null))
  );
  let currentPlayer = $state<Player>(1);
  let winner = $state<Player | 'Draw' | null>(null);
  let winningCells = $state<[number, number][]>([]);

  function makeMove(col: number) {
    if (winner) return;

    let targetRow = -1;
    for (let r = ROWS - 1; r >= 0; r--) {
      if (board[r][col] === null) {
        targetRow = r;
        break;
      }
    }

    if (targetRow === -1) return;

    board[targetRow][col] = currentPlayer;

    if (checkWin(targetRow, col)) {
      winner = currentPlayer;
      return;
    }

    if (board.every(row => row.every(cell => cell !== null))) {
      winner = 'Draw';
      return;
    }

    currentPlayer = currentPlayer === 1 ? 2 : 1;
  }

  function checkWin(r: number, c: number): boolean {
    const player = board[r][c];
    if (!player) return false;

    const directions: [number, number][][] = [
      [[0, 1], [0, -1]],   // Horizontal
      [[1, 0], [-1, 0]],   // Vertikal
      [[1, 1], [-1, -1]],  // Diagonal 1
      [[1, -1], [-1, 1]]   // Diagonal 2
    ];

    for (const dir of directions) {
      let count = 1;
      let winningCoords: [number, number][] = [[r, c]];

      for (const [dr, dc] of dir) {
        let nr = r + dr;
        let nc = c + dc;
        while (nr >= 0 && nr < ROWS && nc >= 0 && nc < COLS && board[nr][nc] === player) {
          count++;
          winningCoords.push([nr, nc]);
          nr += dr;
          nc += dc;
        }
      }

      if (count >= 4) {
        winningCells = winningCoords;
        return true;
      }
    }

    return false;
  }

  function resetGame() {
    board = Array(ROWS).fill(null).map(() => Array(COLS).fill(null));
    currentPlayer = 1;
    winner = null;
    winningCells = [];
  }

  function isWinningCell(r: number, c: number): boolean {
    return winningCells.some(([wr, wc]) => wr === r && wc === c);
  }
</script>

<main class="game-container">
  <div class="header">
    <h1 class="title">4 GEWINNT</h1>
    
    <div class="status-box">
      {#if winner === 'Draw'}
        <span class="status draw">Unentschieden! 🤝</span>
      {:else if winner}
        <span class="status win player-{winner}">
          Spieler {winner} ({winner === 1 ? 'Rot' : 'Gelb'}) hat gewonnen! 🎉
        </span>
      {:else}
        <span class="status turn">
          Am Zug: <span class="badge player-{currentPlayer}">Spieler {currentPlayer} ({currentPlayer === 1 ? 'Rot' : 'Gelb'})</span>
        </span>
      {/if}
    </div>
  </div>

  <div class="board-wrapper">
    <div class="board">
      {#each board as row, r}
        <div class="row">
          {#each row as cell, c}
            <button 
              class="cell-btn" 
              onclick={() => makeMove(c)}
              aria-label="Spalte {c + 1} wählen"
            >
              <div class="slot">
                {#if cell === 1}
                  <div class="disc player-1 {isWinningCell(r, c) ? 'winning' : ''}"></div>
                {:else if cell === 2}
                  <div class="disc player-2 {isWinningCell(r, c) ? 'winning' : ''}"></div>
                {:else}
                  <div class="disc empty"></div>
                {/if}
              </div>
            </button>
          {/each}
        </div>
      {/each}
    </div>
  </div>

  {#if winner}
    <button class="reset-btn" onclick={resetGame}>
      Neues Spiel starten
    </button>
  {/if}
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;600;800&display=swap');

  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #090d16;
    font-family: 'Outfit', sans-serif;
    color: #ffffff;
  }

  .game-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    padding: 20px;
    box-sizing: border-box;
    background: radial-gradient(circle at center, #111c33 0%, #090d16 100%);
  }

  .header {
    text-align: center;
    margin-bottom: 24px;
  }

  .title {
    font-size: 2.8rem;
    font-weight: 800;
    letter-spacing: 2px;
    margin: 0 0 12px 0;
    background: linear-gradient(135deg, #38bdf8 0%, #facc15 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 10px 30px rgba(56, 189, 248, 0.2);
  }

  .status-box {
    font-size: 1.2rem;
    font-weight: 600;
    background: rgba(255, 255, 255, 0.05);
    padding: 10px 24px;
    border-radius: 50px;
    border: 1px solid rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  }

  .badge.player-1, .status.player-1 {
    color: #ef4444;
  }

  .badge.player-2, .status.player-2 {
    color: #facc15;
  }

  .status.draw {
    color: #38bdf8;
  }

  .board-wrapper {
    background: linear-gradient(135deg, #1d4ed8 0%, #1e40af 100%);
    padding: 18px;
    border-radius: 24px;
    box-shadow: 0 25px 50px -12px rgba(29, 78, 216, 0.4),
                inset 0 4px 6px rgba(255, 255, 255, 0.2),
                inset 0 -4px 6px rgba(0, 0, 0, 0.4);
    border: 4px solid #3b82f6;
  }

  .board {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .row {
    display: flex;
    gap: 10px;
  }

  .cell-btn {
    background: transparent;
    border: none;
    padding: 0;
    cursor: pointer;
    outline: none;
  }

  .slot {
    width: 52px;
    height: 52px;
    background-color: #0b1329;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: inset 0 6px 12px rgba(0, 0, 0, 0.8),
                0 2px 4px rgba(255, 255, 255, 0.05);
    transition: transform 0.2s ease, background-color 0.2s ease;
  }

  .cell-btn:hover .slot {
    background-color: #131f42;
    transform: scale(1.05);
  }

  .disc {
    width: 42px;
    height: 42px;
    border-radius: 50%;
    transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .disc.empty {
    background: rgba(255, 255, 255, 0.03);
  }

  .disc.player-1 {
    background: radial-gradient(circle at 30% 30%, #f87171 0%, #dc2626 70%, #991b1b 100%);
    box-shadow: 0 4px 12px rgba(220, 38, 38, 0.5), inset 0 2px 4px rgba(255, 255, 255, 0.4);
  }

  .disc.player-2 {
    background: radial-gradient(circle at 30% 30%, #fde047 0%, #eab308 70%, #a16207 100%);
    box-shadow: 0 4px 12px rgba(234, 179, 8, 0.5), inset 0 2px 4px rgba(255, 255, 255, 0.4);
  }

  .disc.winning {
    animation: pulse-glow 1.2s infinite alternate;
    transform: scale(1.1);
  }

  @keyframes pulse-glow {
    0% {
      box-shadow: 0 0 10px #fff, 0 0 20px currentColor;
    }
    100% {
      box-shadow: 0 0 20px #fff, 0 0 40px currentColor;
    }
  }

  .reset-btn {
    margin-top: 28px;
    background: linear-gradient(135deg, #38bdf8 0%, #2563eb 100%);
    color: white;
    border: none;
    padding: 14px 32px;
    font-size: 1.1rem;
    font-weight: 700;
    border-radius: 14px;
    cursor: pointer;
    box-shadow: 0 10px 25px rgba(37, 99, 235, 0.4);
    transition: all 0.2s ease;
  }

  .reset-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 15px 30px rgba(37, 99, 235, 0.6);
  }

  .reset-btn:active {
    transform: translateY(1px);
  }

  @media (max-width: 500px) {
    .slot {
      width: 42px;
      height: 42px;
    }
    .disc {
      width: 34px;
      height: 34px;
    }
    .title {
      font-size: 2.2rem;
    }
  }
</style>