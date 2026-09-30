const size = 4;
const totalCells = size * size;
const solvedBoard = Array.from({ length: totalCells }, (_, index) => (index + 1) % totalCells);

const boardEl = document.getElementById('board');
const movesEl = document.getElementById('moves');
const timerEl = document.getElementById('timer');
const messageEl = document.getElementById('message');
const shuffleBtn = document.getElementById('shuffleBtn');
const resetBtn = document.getElementById('resetBtn');

let board = [...solvedBoard];
let moves = 0;
let seconds = 0;
let timerId = null;
let started = false;

function formatTime(totalSeconds) {
  const minutes = Math.floor(totalSeconds / 60)
    .toString()
    .padStart(2, '0');
  const secs = (totalSeconds % 60).toString().padStart(2, '0');
  return `${minutes}:${secs}`;
}

function setMessage(text, success = false) {
  messageEl.textContent = text;
  messageEl.classList.toggle('success', success);
}

function updateStats() {
  movesEl.textContent = String(moves);
  timerEl.textContent = formatTime(seconds);
}

function startTimer() {
  if (timerId) return;

  timerId = setInterval(() => {
    seconds += 1;
    updateStats();
  }, 1000);
}

function stopTimer() {
  if (timerId) {
    clearInterval(timerId);
    timerId = null;
  }
}

function isSolved(currentBoard) {
  return currentBoard.every((value, index) => value === solvedBoard[index]);
}

function getEmptyIndex(currentBoard) {
  return currentBoard.indexOf(0);
}

function isAdjacent(indexA, indexB) {
  const rowA = Math.floor(indexA / size);
  const colA = indexA % size;
  const rowB = Math.floor(indexB / size);
  const colB = indexB % size;

  return Math.abs(rowA - rowB) + Math.abs(colA - colB) === 1;
}

function moveTile(index) {
  const emptyIndex = getEmptyIndex(board);

  if (!isAdjacent(index, emptyIndex)) {
    return;
  }

  [board[index], board[emptyIndex]] = [board[emptyIndex], board[index]];
  moves += 1;
  updateStats();

  if (!started) {
    started = true;
    startTimer();
  }

  renderBoard();

  if (isSolved(board)) {
    stopTimer();
    setMessage(`Solved in ${moves} moves and ${formatTime(seconds)}!`, true);
  } else {
    setMessage('Keep going!');
  }
}

function renderBoard() {
  boardEl.innerHTML = '';

  board.forEach((value, index) => {
    const tile = document.createElement('button');
    tile.type = 'button';
    tile.className = value === 0 ? 'tile empty' : 'tile';
    tile.setAttribute('aria-label', value === 0 ? 'Empty tile' : `Tile ${value}`);

    if (value !== 0) {
      tile.textContent = String(value);
      tile.addEventListener('click', () => moveTile(index));
    }

    boardEl.appendChild(tile);
  });
}

function shuffleBoard() {
  let shuffled = [...solvedBoard];
  let emptyIndex = getEmptyIndex(shuffled);

  for (let step = 0; step < 200; step += 1) {
    const possibleMoves = [];

    for (let i = 0; i < totalCells; i += 1) {
      if (isAdjacent(i, emptyIndex)) {
        possibleMoves.push(i);
      }
    }

    const nextIndex = possibleMoves[Math.floor(Math.random() * possibleMoves.length)];
    [shuffled[nextIndex], shuffled[emptyIndex]] = [shuffled[emptyIndex], shuffled[nextIndex]];
    emptyIndex = nextIndex;
  }

  board = shuffled;
  moves = 0;
  seconds = 0;
  started = false;

  stopTimer();
  updateStats();
  renderBoard();

  if (isSolved(board)) {
    shuffleBoard();
    return;
  }

  setMessage('Puzzle shuffled. Start moving the tiles!');
}

function resetBoard() {
  board = [...solvedBoard];
  moves = 0;
  seconds = 0;
  started = false;
  stopTimer();
  updateStats();
  renderBoard();
  setMessage('Arrange the tiles in order.');
}

shuffleBtn.addEventListener('click', shuffleBoard);
resetBtn.addEventListener('click', resetBoard);

resetBoard();
