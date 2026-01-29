<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';

const initGrid = () => {
  return Array.from({ length: 9 }, () => Array(9).fill(0));
};

const grid = ref(initGrid());
const initialGrid = ref(Array.from({ length: 9 }, () => Array(9).fill(false)));
const difficulty = ref(30); // Número de celdas que se vacían
const gameStarted = ref(false); // Control si el juego ha comenzado

// --- VALIDACIÓN DE ERRORES ---
const hasConflict = (r, c) => {
  const val = grid.value[r][c];
  if (!val) return false;

  // Revisar fila
  for (let i = 0; i < 9; i++) {
    if (i !== c && grid.value[r][i] === val) return true;
  }
  // Revisar columna
  for (let i = 0; i < 9; i++) {
    if (i !== r && grid.value[i][c] === val) return true;
  }
  // Revisar bloque 3x3
  let startRow = r - (r % 3);
  let startCol = c - (c % 3);
  for (let i = startRow; i < startRow + 3; i++) {
    for (let j = startCol; j < startCol + 3; j++) {
      if ((i !== r || j !== c) && grid.value[i][j] === val) return true;
    }
  }
  return false;
};

// --- CONDICIÓN DE VICTORIA ---
const gameWon = computed(() => {
  for (let r = 0; r < 9; r++) {
    for (let c = 0; c < 9; c++) {
      // Si hay un 0 o hay un conflicto, no has ganado aún
      if (!grid.value[r][c] || hasConflict(r, c)) return false;
    }
  }
  return true;
});

// --- GENERACIÓN Y RESOLUCIÓN ---
const isSafe = (board, row, col, num) => {
  for (let x = 0; x < 9; x++) {
    if (board[row][x] === num || board[x][col] === num) return false;
  }
  let startRow = row - (row % 3);
  let startCol = col - (col % 3);
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      if (board[i + startRow][j + startCol] === num) return false;
    }
  }
  return true;
};

const fillGrid = (board) => {
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (board[row][col] === 0) {
        let nums = [1, 2, 3, 4, 5, 6, 7, 8, 9].sort(() => Math.random() - 0.5);
        for (let num of nums) {
          if (isSafe(board, row, col, num)) {
            board[row][col] = num;
            if (fillGrid(board)) return true;
            board[row][col] = 0;
          }
        }
        return false;
      }
    }
  }
  return true;
};

const generateSudoku = () => {
  gameStarted.value = true;
  grid.value = initGrid();
  initialGrid.value = Array.from({ length: 9 }, () => Array(9).fill(false));
  const tempBoard = Array.from({ length: 9 }, () => Array(9).fill(0));
  fillGrid(tempBoard);
  grid.value = JSON.parse(JSON.stringify(tempBoard));
  
  // Hacer agujeros según dificultad
  let count = difficulty.value;
  while (count > 0) {
    let r = Math.floor(Math.random() * 9);
    let c = Math.floor(Math.random() * 9);
    if (grid.value[r][c] !== 0) {
      grid.value[r][c] = 0;
      count--;
    }
  }

  for (let r = 0; r < 9; r++) {
    for (let c = 0; c < 9; c++) {
      if (grid.value[r][c] !== 0) initialGrid.value[r][c] = true;
    }
  }
};

const solve = () => {
  // Limpiamos los errores del usuario antes de resolver
  for (let r = 0; r < 9; r++) {
    for (let c = 0; c < 9; c++) {
      if (!initialGrid.value[r][c]) grid.value[r][c] = 0;
    }
  }
  fillGrid(grid.value);
};

const selectedCell = ref({ r: null, c: null });

// Función para seleccionar una celda
const selectCell = (r, c) => {
  if (!initialGrid.value[r][c]) { // Solo seleccionar si no es fija
    selectedCell.value = { r, c };
  }
};

// Función para poner un número desde el teclado externo
const setNumber = (num) => {
  const { r, c } = selectedCell.value;
  if (r !== null && c !== null) {
    grid.value[r][c] = num;
  }
};

// Función para borrar la celda actual
const eraseCell = () => {
  const { r, c } = selectedCell.value;
  if (r !== null && c !== null) {
    grid.value[r][c] = 0;
  }
};

// Manejar la presión de teclas
const handleKeyPress = (event) => {
  const key = event.key;
  if (key >= '1' && key <= '9') {
      setNumber(parseInt(key));
  } else if (key === 'Backspace' || key === 'Delete') {
      eraseCell();
  }
};

onMounted(() => {
  // Escuchar teclas del ordenador solo cuando el juego ha comenzado
  window.addEventListener('keydown', handleKeyPress);
});

// Limpiar el listener al destruir el componente
onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyPress);
});
</script>

<template>
  <div class="container">
    <h1>Sudoku Master</h1>

    <!-- SELECTOR DE DIFICULTAD (SIEMPRE VISIBLE) -->
    <div class="settings">
      <label>Dificultad: </label>
      <select v-model="difficulty">
        <option :value="20">Fácil</option>
        <option :value="40">Medio</option>
        <option :value="60">Difícil</option>
      </select>
    </div>

    <!-- BOTÓN JUGAR (SOLO AL INICIO) -->
    <button v-if="!gameStarted" @click="generateSudoku" class="btn-play">Jugar</button>

    <!-- PANTALLA DEL JUEGO -->
    <div v-if="gameStarted" class="game-screen">
      <div v-if="gameWon" class="win-message">
        🎉 ¡Felicidades! Has resuelto el Sudoku 🎉
      </div>

      <div class="game-container">
        <!-- TABLERO -->
        <div class="sudoku-board">
          <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
            <div 
              v-for="(cell, colIndex) in row" 
              :key="colIndex"
              :class="[
                'cell', 
                { 'is-fixed': initialGrid[rowIndex][colIndex] },
                { 'is-invalid': !initialGrid[rowIndex][colIndex] && hasConflict(rowIndex, colIndex) },
                { 'is-selected': selectedCell.r === rowIndex && selectedCell.c === colIndex }
              ]"
              @click="selectCell(rowIndex, colIndex)"
            >
              <!-- Si el valor es 0, no mostramos nada -->
              {{ cell !== 0 ? cell : '' }}
            </div>
          </div>
        </div>

        <!-- TECLADO NUMÉRICO -->
        <div class="keypad">
          <button v-for="n in 9" :key="n" @click="setNumber(n)">
            {{ n }}
          </button>
          <button @click="eraseCell" class="btn-erase">Borrar</button>
        </div>
      </div>

      <div class="controls">
        <button @click="generateSudoku">Nuevo Juego</button>
        <button @click="solve" class="btn-solve">Resolver</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 30px;
}

h1 {
  margin-bottom: 30px;
}

.settings {
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 1.2rem;
  margin-bottom: 20px;
}

.settings select {
  padding: 10px 15px;
  font-size: 1rem;
  border: 2px solid #42b983;
  border-radius: 4px;
  cursor: pointer;
}

.btn-play {
  padding: 15px 40px;
  font-size: 1.2rem;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
  margin-bottom: 30px;
}

.btn-play:hover {
  background: #359970;
}

.game-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.sudoku-board {
  border: 3px solid #000;
  margin-bottom: 20px;
}

.row {
  display: flex;
}

.row:nth-child(3n) {
  border-bottom: 3px solid #000;
}

.cell {
  width: 45px;
  height: 45px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
  border: 1px solid #ccc;
  cursor: pointer;
  user-select: none;
  background: white;
  transition: background 0.2s;
}

.cell:nth-child(3n) {
  border-right: 3px solid #000;
}

.cell:last-child {
  border-right: none;
}

/* Estilo para los números que pone la máquina */
.is-fixed {
  background-color: #f0f0f0;
  font-weight: bold;
  color: #2c3e50;
}

/* Estilo para los números que pone el usuario */
.cell:not(.is-fixed) {
  color: #3498db;
}

/* Resaltar celda seleccionada */
.is-selected {
  background-color: #e3f2fd !important;
  outline: 2px solid #2196f3;
  z-index: 1;
}

/* Celda con error */
.is-invalid {
  background-color: #ffcccc !important;
  color: #cc0000 !important;
}

.win-message {
  background-color: #42b983;
  color: white;
  padding: 10px 20px;
  border-radius: 8px;
  margin-bottom: 15px;
  font-weight: bold;
  animation: bounce 0.5s infinite alternate;
}

@keyframes bounce {
  from { transform: scale(1); }
  to { transform: scale(1.05); }
}

/* Contenedor para tablero y teclado */
.game-container {
  display: flex;
  gap: 30px;
  align-items: flex-start;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 30px;
}

/* Teclado */
.keypad {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.keypad button {
  width: 50px;
  height: 50px;
  font-size: 1.2rem;
  background: #f8f9fa;
  border: 1px solid #ddd;
  border-radius: 8px;
  color: #333;
  cursor: pointer;
}

.keypad button:hover {
  background: #e9ecef;
}

.keypad .btn-erase {
  grid-column: span 3;
  width: auto;
  background: #ffebee;
  color: #c62828;
  font-size: 1rem;
}

.controls {
  display: flex;
  gap: 10px;
}

button {
  padding: 10px 20px;
  cursor: pointer;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  transition: background 0.3s;
}

button:hover {
  background: #359970;
}

.btn-solve {
  background: #34495e;
}

.btn-solve:hover {
  background: #2c3e50;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
</style>