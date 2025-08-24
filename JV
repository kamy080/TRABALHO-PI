const startBtn = document.getElementById("startBtn");
const startScreen = document.getElementById("startScreen");
const gameContainer = document.getElementById("gameContainer");
const gameOverScreen = document.getElementById("gameOverScreen");
const restartBtn = document.getElementById("restartBtn");

const canvas = document.getElementById('board');
const ctx = canvas.getContext('2d');
const pCanvas = document.getElementById('particles');
const pCtx = pCanvas.getContext('2d');
const player = document.getElementById('player');
const mineCountEl = document.getElementById('mineCount');
const timerEl = document.getElementById('timer');
const message = document.getElementById('message');

const gridSize = 10, cellSize = 40, totalMines = 10;
const emojis = ['🍎', '🍌', '🍇', '🍉', '🍒', '🥝'];
const totalEmojis = 20;

let board = [], flagged = 0, timer = 0, timerInterval, score = 0;
let particles = [];
let marcadorAtivo = false;

let currentClicks = []; // armazenar os emojis clicados com timestamp para 5s
const MAX_TIME = 5000; // 5 segundos

startBtn.addEventListener("click", () => {
  startScreen.classList.add("hidden");
  gameContainer.classList.remove("hidden");
  initBoard();
  drawBoard();
  drawParticles();
});

restartBtn.addEventListener("click", () => {
  gameOverScreen.classList.add("hidden");
  gameContainer.classList.remove("hidden");
  initBoard();
});

function initBoard() {
  board = Array.from({ length: gridSize }, () => Array.from({ length: gridSize }, () => ({
    hasMine: false,
    revealed: false,
    flagged: false,
    count: 0,
    emoji: null,
    emojiVisibleTemp: false // Para mostrar a fruta temporariamente
  })));

  let placed = 0;
  while (placed < totalMines) {
    const y = Math.floor(Math.random() * gridSize);
    const x = Math.floor(Math.random() * gridSize);
    if (!board[y][x].hasMine) {
      board[y][x].hasMine = true;
      placed++;
    }
  }

  board.forEach((row, y) => row.forEach((cell, x) => {
    if (!cell.hasMine) {
      let cnt = 0;
      for (let dy = -1; dy <= 1; dy++)
        for (let dx = -1; dx <= 1; dx++)
          if ((dy || dx) && board[y + dy]?.[x + dx]?.hasMine)
            cnt++;
      cell.count = cnt;
    }
  }));

  // Colocar emojis nas células sem minas
  let emojiPlaced = 0;
  while (emojiPlaced < totalEmojis) {
    const y = Math.floor(Math.random() * gridSize);
    const x = Math.floor(Math.random() * gridSize);
    const cell = board[y][x];
    if (!cell.hasMine && !cell.emoji) {
      cell.emoji = emojis[Math.floor(Math.random() * emojis.length)];
      emojiPlaced++;
    }
  }

  flagged = 0; 
  mineCountEl.textContent = totalMines - flagged;
  timer = score = 0; 
  timerEl.textContent = timer;
  marcadorAtivo = false;
  currentClicks = [];

  clearInterval(timerInterval);
  timerInterval = setInterval(() => timerEl.textContent = ++timer, 1000);
}

function showMessage(text, duration = 1000) {
  message.textContent = text;
  message.classList.add('show');
  message.classList.remove('hidden');
  setTimeout(() => message.classList.remove('show'), duration);
}

function emitParticles(x, y) {
  for (let i = 0; i < 20; i++) {
    particles.push({
      x, y,
      vx: (Math.random() - 0.5) * 4,
      vy: (Math.random() - 0.5) * 4,
      alpha: 1
    });
  }
}

function drawParticles() {
  pCtx.clearRect(0, 0, pCanvas.width, pCanvas.height);
  particles = particles.filter(p => p.alpha > 0);
  particles.forEach(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.alpha -= 0.02;
    pCtx.fillStyle = `rgba(255,192,203,${p.alpha})`;
    pCtx.fillRect(p.x, p.y, 4, 4);
  });
  requestAnimationFrame(drawParticles);
}

function drawBoard() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  board.forEach((row, y) => row.forEach((cell, x) => {
    ctx.fillStyle = 'green';  // cor viva sem transparência
    ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
    ctx.strokeStyle = '#222';
    ctx.strokeRect(x * cellSize, y * cellSize, cellSize, cellSize);

    if (cell.revealed) {
      if (cell.hasMine) {
        
        if (cell.flagged) {
          ctx.fillStyle = 'yellow';
          ctx.font = '20px Press Start 2P';
          ctx.fillText('⚑', x * cellSize + cellSize / 3, y * cellSize + 2 * cellSize / 3);
        }
      } else {
        if (cell.count > 0) {
          ctx.fillStyle = '#000';
          ctx.font = '20px Press Start 2P';
          ctx.fillText(cell.count, x * cellSize + cellSize / 3, y * cellSize + 2 * cellSize / 3);
        }
      }
    } else {
      if (cell.flagged) {
        ctx.fillStyle = 'yellow';
        ctx.font = '20px Press Start 2P';
        ctx.fillText('⚑', x * cellSize + cellSize / 3, y * cellSize + 2 * cellSize / 3);
      }
    }

    
    if (cell.emoji && cell.emojiVisibleTemp) {
      ctx.font = '26px serif';
      ctx.fillText(cell.emoji, x * cellSize + cellSize / 3, y * cellSize + 2 * cellSize / 3);
    }
  }));

  requestAnimationFrame(drawBoard);
}

// Função para gerenciar clique no quadrado verde (revele emoji temporário)
function revealEmojiTemp(x, y) {
  const cell = board[y]?.[x];
  if (!cell || cell.revealed || cell.flagged || !cell.emoji) return;

  if (cell.emojiVisibleTemp) return; // Já está visível, ignora clique

  cell.emojiVisibleTemp = true;
  drawBoard();

  
  currentClicks.push({emoji: cell.emoji, timestamp: Date.now()});

  setTimeout(() => {
    cell.emojiVisibleTemp = false;
    drawBoard();
  }, MAX_TIME);

  checkTripleWithinTime();
}

function checkTripleWithinTime() {
  const now = Date.now();
  currentClicks = currentClicks.filter(c => now - c.timestamp <= MAX_TIME);
  const counts = {};
  for (const click of currentClicks) {
    counts[click.emoji] = (counts[click.emoji] || 0) + 1;
    if (counts[click.emoji] >= 3) {
      showMessage(`3 imagens "${click.emoji}" encontradas! Revelando bomba...`, 3000);
      revealOneBombTemporarily();
      
      currentClicks = [];
      break;
    }
  }
}

function revealOneBombTemporarily() {
  for (let y = 0; y < gridSize; y++) {
    for (let x = 0; x < gridSize; x++) {
      const cell = board[y][x];
      if (cell.hasMine && !cell.flagged && !cell.revealed) {
      
        cell.revealed = true;
        drawBoard();
        setTimeout(() => {
          cell.revealed = false;
          cell.flagged = true;
          flagged++;
          mineCountEl.textContent = totalMines - flagged;
          drawBoard();
        }, 2000);
        return;
      }
    }
  }
}

canvas.addEventListener('mousedown', e => {
  e.preventDefault();
  const rect = canvas.getBoundingClientRect();
  const x = Math.floor((e.clientX - rect.left) / cellSize);
  const y = Math.floor((e.clientY - rect.top) / cellSize);
  player.style.left = x * cellSize + 'px';
  player.style.top = y * cellSize + 'px';

  if (marcadorAtivo && e.button === 0) {
    const c = board[y][x];
    if (c && !c.revealed) {
      c.flagged = !c.flagged;
      flagged += c.flagged ? 1 : -1;
      mineCountEl.textContent = totalMines - flagged;
    }
  } else {
    if (e.button === 2) {
      const c = board[y][x];
      if (c && !c.revealed) {
        c.flagged = !c.flagged;
        flagged += c.flagged ? 1 : -1;
        mineCountEl.textContent = totalMines - flagged;
      }
    } else if (e.button === 0) {
    
      const cell = board[y]?.[x];
      if (cell && !cell.revealed && !cell.flagged && cell.emoji) {
        revealEmojiTemp(x, y);
      } else {
        
        revealCell(x, y);
      }
    }
  }
});

function revealCell(x, y) {
  const c = board[y]?.[x];
  if (!c || c.revealed || c.flagged) return;

  c.revealed = true;

  if (!c.hasMine) {
    score++;
    if (score % 3 === 0) showMessage('Você está indo bem!', 1000);
  }

  if (c.count === 0 && !c.hasMine) {
    for (let dy = -1; dy <= 1; dy++)
      for (let dx = -1; dx <= 1; dx++)
        if (dx || dy)
          revealCell(x + dx, y + dy);
  }

  if (c.hasMine) {
    clearInterval(timerInterval);
    gameContainer.classList.add("hidden");
    gameOverScreen.classList.remove("hidden");
  }
}

canvas.oncontextmenu = e => e.preventDefault();
window.addEventListener('keydown', (e) => {
  if (e.key.toLowerCase() === 'a') {
    marcadorAtivo = !marcadorAtivo;
    showMessage(marcadorAtivo ? 'Marcador de bombas ATIVADO' : 'Marcador de bombas DESATIVADO', 1500);
  }
});

