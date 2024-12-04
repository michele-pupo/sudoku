<script>
export default {
  data() {
    return {
      // Griglia corrente da visualizzare e modificare
      grid: [],
      // Griglia che contiene i numeri iniziali
      initialNumbers: Array(9).fill(null).map(() => Array(9).fill(null)),
      // Griglia per controllare l'animazione di ogni cella
      animationStates: Array(9).fill(null).map(() => Array(9).fill(false)),
      // Messaggio di errore
      errorMessage: '',
      // Flag per interrompere la risoluzione
      isSolving: false
    };
  },

  mounted() {
    this.resetGrid();
  },

  methods: {
    resetGrid() {
      this.isSolving = false; // Interrompe qualsiasi operazione in corso
      this.grid = Array(9).fill(null).map(() => Array(9).fill(null));
      this.initialNumbers = Array(9).fill(null).map(() => Array(9).fill(null));
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

    async solve() {
      if (!this.checkGridValidity()) {
        this.errorMessage = 'Invalid Sudoku grid! Please fix duplicate numbers.';
        return;
      }

      const gridCopy = JSON.parse(JSON.stringify(this.grid)); // Copia la griglia corrente
      this.errorMessage = ''; // Resetta eventuali messaggi di errore
      this.isSolving = true; // Disabilita gli input

      // Risolve il Sudoku senza animazione
      const solved = this.solveSudoku(gridCopy);
      if (!solved) {
        alert("No solution exists!");
        this.isSolving = false; // Riabilita gli input
        return;
      }

      // Mostra l'animazione progressiva
      await this.animateSpinningNumbers(gridCopy);
      this.isSolving = false; // Riabilita gli input dopo che la risoluzione è terminata
    },

    async animateSpinningNumbers(solvedGrid) {
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (!this.isSolving) return; // Interrompe l'animazione se il flag è falso
          if (this.initialNumbers[row][col] === null) {
            await this.spinToCorrectNumber(row, col, solvedGrid[row][col]);
          }
        }
      }
    },

    async spinToCorrectNumber(row, col, correctNumber) {
      let num = 1;

      while (num !== correctNumber) {
        if (!this.isSolving) return; // Interrompe l'animazione se il flag è falso
        this.grid[row][col] = num; // Mostra il numero animato
        await this.sleep(50); // Breve pausa per l'animazione
        num = (num % 9) + 1; // Passa al numero successivo
      }

      // Una volta trovato il numero corretto, lo evidenzia
       this.grid[row][col] = correctNumber;
    },

  sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
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
            :disabled="isSolving"
            :class="{
              'computer-solved': animationStates[rowIndex][colIndex] === 'computer-solved',
              'user-input': initialNumbers[rowIndex][colIndex] !== null
            }"
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

body {
  background: linear-gradient(135deg, #a8edea, #fed6e3);
  font-family: 'Poppins', sans-serif;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

h1 {
  font-family: 'Poppins', sans-serif;
  color: #28a745;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
  margin-bottom: 20px;
}

.text-danger {
  font-family: 'Roboto', sans-serif;
  font-size: 1.2rem;
  background: #fdecea;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.sudoku-grid {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  grid-template-rows: repeat(9, 1fr);
  gap: 2px;
  width: 600px;
  height: 600px;
  margin: auto;
  background: #333; /* Colore di sfondo per evidenziare i bordi */
  border: 5px solid #333;
  border-radius: 10px; /* Angoli arrotondati */
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5); /* Ombra */
}

.sudoku-row {
  display: contents; /* Consente di mantenere la struttura della griglia */
}

.sudoku-cell {
  background: #f5f5f5; /* Sfondo chiaro per le celle */
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Roboto', sans-serif; /* Font moderno */
  border: 1px solid #ddd;
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
  font-size: 1.8rem;
  font-weight: bold;
  color: #333; /* Colore scuro per i numeri */
  background: transparent;
}

.sudoku-cell input:focus {
  background: #fffae6; /* Cambia colore quando si interagisce */
  border-radius: 5px;
  outline: none;
  transition: 0.2s ease-in-out;
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

button {
  padding: 12px 20px;
  font-size: 1.2rem;
  font-family: 'Roboto', sans-serif;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

button:hover {
  background-color: #45a049;
  transform: scale(1.05);
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #c82333;
}

@keyframes spin {
  0% {
    transform: rotateX(0deg);
    opacity: 1;
  }
  50% {
    transform: rotateX(90deg);
    opacity: 0.5;
  }
  100% {
    transform: rotateX(180deg);
    opacity: 1;
  }
}

.sudoku-cell input.animate {
  animation: spin 0.1s linear infinite;
}

.sudoku-cell input.correct {
  animation: none;
  color: green;
  font-weight: bold;
}

</style>