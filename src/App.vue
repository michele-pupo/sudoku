<script>
export default {
  data() {
    return {
      // Griglia iniziale del sudoku, con celle vuote
      initialGrid: Array(9).fill(null).map(() => Array(9).fill(null)),
      // Griglia corrente da visualizzare e modificare
      grid: [],
      // Griglia che contiene i numeri iniziali
      initialNumbers: Array(9).fill(null).map(() => Array(9).fill(null)),
      // Messaggio di errore
      errorMessage: ''
    };
  },

  mounted() {
    this.resetGrid();
  },

  methods: {
    resetGrid() {
      // Genera una griglia iniziale vuota
      this.initialGrid = Array(9).fill(null).map(() => Array(9).fill(null));
      this.grid = JSON.parse(JSON.stringify(this.initialGrid));
      this.initialNumbers = JSON.parse(JSON.stringify(this.initialGrid));
      this.errorMessage = ''; // Resetta il messaggio di errore
    },

    isValid(grid, row, col, num) {
      for (let x = 0; x < 9; x++) {
        if (grid[row][x] === num || grid[x][col] === num ||
            grid[3 * Math.floor(row / 3) + Math.floor(x / 3)][3 * Math.floor(col / 3) + x % 3] === num) {
          return false;
        }
      }
      return true;
    },

    checkGridValidity() {
      const isValid = (arr) => arr.filter(num => num !== null).length !== new Set(arr.filter(num => num !== null)).size;
      
      for (let i = 0; i < 9; i++) {
        if (isValid(this.grid[i])) return false; // Controlla righe
        if (isValid(this.grid.map(row => row[i]))) return false; // Controlla colonne
      }
      
      for (let boxRow = 0; boxRow < 9; boxRow += 3) {
        for (let boxCol = 0; boxCol < 9; boxCol += 3) {
          let box = [];
          for (let row = 0; row < 3; row++) {
            for (let col = 0; col < 3; col++) {
              box.push(this.grid[boxRow + row][boxCol + col]);
            }
          }
          if (isValid(box)) return false; // Controlla quadrati 3x3
        }
      }

      return true;
    },

    solveSudoku(grid) {
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (grid[row][col] === null) {
            for (let num = 1; num <= 9; num++) {
              if (this.isValid(grid, row, col, num)) {
                grid[row][col] = num;
                if (this.solveSudoku(grid)) {
                  return true;
                } else {
                  grid[row][col] = null;
                }
              }
            }
            return false;
          }
        }
      }
      return true;
    },

    solve() {
      if (!this.checkGridValidity()) {
        this.errorMessage = 'Invalid Sudoku grid! Please fix duplicate numbers.';
        return;
      }

      const gridCopy = JSON.parse(JSON.stringify(this.grid)); // Crea una copia della griglia
      if (this.solveSudoku(gridCopy)) {
        this.grid = gridCopy; // Aggiorna la griglia solo se è stata trovata una soluzione
        this.errorMessage = ''; // Rimuovi il messaggio di errore
      } else {
        alert("No solution exists!");
      }
    },

    handleInput(rowIndex, colIndex, value) {
      const num = parseInt(value);
      if (isNaN(num) || num < 1 || num > 9) {
        this.grid[rowIndex][colIndex] = null; // Resetto le celle se l'imput non è valido
      } else {
        this.grid[rowIndex][colIndex] = num;
      }
      this.initialNumbers[rowIndex][colIndex] = this.grid[rowIndex][colIndex];
      this.errorMessage = ''; // Rimuovi il messaggio di errore
    }
  }
};
</script>

<template>
  <div class="container py-5">
    <h1 class="pb-5 text-uppercase text-center display-1 fw-bold text-success">Sudoku</h1>
    <div class="sudoku-grid">
      <div class="sudoku-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
        <div class="sudoku-cell" v-for="(cell, colIndex) in row" :key="colIndex">
          <input
            class="fw-bold"
            type="number"
            min="1"
            max="9"
            v-model.number="grid[rowIndex][colIndex]"
            @input="handleInput(rowIndex, colIndex, $event.target.value)"
            :style="{ color: initialNumbers[rowIndex][colIndex] !== null ? 'black' : 'red' }"
            placeholder=""
          />
        </div>
      </div>
    </div>
    <div class="d-flex justify-content-center pt-5">
      <button class="btn btn-success" @click="solve">Solve Sudoku</button>
      <button class="btn btn-danger" @click="resetGrid">Reset Sudoku</button>
    </div>
    <div v-if="errorMessage">
      <h3 class="text-danger text-uppercase fw-bold mt-4 text-center">{{ errorMessage }}</h3>
    </div>
  </div>
</template>

<style>
.sudoku-grid {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  grid-template-rows: repeat(9, 1fr);
  gap: 1px; /* Spazio tra le celle */
  width: 500px; /* Larghezza totale della griglia */
  height: 500px; /* Altezza totale della griglia */
  margin: auto;
}

.sudoku-row {
  display: contents; /* Consente di mantenere la struttura della griglia */
}

.sudoku-cell {
  border: 1px solid black;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  background-image: url('/sudoku-fallback.jpg'); /* Percorso dell'immagine */
  background-size: cover;
  background-position: center; /* Centra l'immagine */
  background-color: rgba(255, 255, 255, 0.5); /* Colore di sfondo bianco con trasparenza del 50% */
}

.sudoku-cell:nth-child(3n) {
  border-right: 2px solid black;
}

.sudoku-cell:nth-child(3n + 1) {
  border-left: 2px solid black;
}

.sudoku-row:nth-child(3n) .sudoku-cell {
  border-bottom: 2px solid black;
}

.sudoku-row:nth-child(3n + 1) .sudoku-cell {
  border-top: 2px solid black;
}

.sudoku-cell input {
  width: 100%;
  height: 100%;
  text-align: center;
  border: none;
  outline: none;
  padding: 0;
  font-size: 1.5rem; /* Dimensione del testo */
  box-sizing: border-box;
  background: transparent; /* Rende lo sfondo dell'input trasparente per vedere l'immagine */
}

/* Rimuove le freccette dagli input numerici */
input[type=number]::-webkit-outer-spin-button,
input[type=number]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}

button {
  margin: 10px;
}

</style>