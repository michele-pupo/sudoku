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
      isSolving: false,
      //Flag per tracciare se il sudoku è stato risolto
      isSolved: false,
      //Array per tenere traccia delle celle con errori
      errorCells:[],
      hasError: false,
      //Cella attualmente selezionata
      focusedCell: { row: 0, col: 0 },
    };
  },

  mounted() {
    this.resetGrid();
    window.addEventListener('keydown', this.handleArrowKeys); // Assegna il listener
  },

  beforeUnmount() {
    window.removeEventListener('keydown', this.handleArrowKeys); // Rimuovi il listener
  },

  methods: {
    resetGrid() {
      // Reset della griglia
      this.initialNumbers = Array(9).fill(null).map(() => Array(9).fill(null));
      this.grid = JSON.parse(JSON.stringify(this.initialNumbers));

      // Resetta gli errori
      this.errorCells = [];
      this.hasError = false;
      this.errorMessage = ''; // Rimuovi eventuali messaggi di errore

      // Riabilita gli input e il pulsante "Solve Sudoku"
      this.isSolving = false;  // Permette di inserire numeri
      this.isSolved = false;   // Permette di risolvere di nuovo il Sudoku
      this.focusedCell = { row: 0, col: 0 }; // Resetta la cella selezionata
    },

    handleArrowKeys(event) {
      if (this.isSolving || this.isSolved) return; // Non consentire movimenti durante la risoluzione o se il Sudoku è risolto
      const { row, col } = this.focusedCell;
      switch (event.key) {
        case 'ArrowUp':
          if (row > 0) this.focusedCell.row--;
          break;
        case 'ArrowDown':
          if (row < 8) this.focusedCell.row++;
          break;
        case 'ArrowLeft':
          if (col > 0) this.focusedCell.col--;
          break;
        case 'ArrowRight':
          if (col < 8) this.focusedCell.col++;
          break;
        default:
          return; // Ignora altri tasti
      }
      // Sposta il focus sulla nuova cella
      this.focusCell(this.focusedCell.row, this.focusedCell.col);
    },

    handleKeyDown(event, rowIndex, colIndex) {
      const { key } = event;

      // Impedisce il comportamento predefinito per le frecce su e giù
      if (key === "ArrowUp" || key === "ArrowDown") {
        event.preventDefault();
      }

      switch (key) {
        case "ArrowRight":
          // Vai a destra, se possibile
          if (colIndex < 8) {
            this.focusCell(rowIndex, colIndex + 1);
          }
          break;
        case "ArrowLeft":
          // Vai a sinistra, se possibile
          if (colIndex > 0) {
            this.focusCell(rowIndex, colIndex - 1);
          }
          break;
        case "ArrowDown":
          // Vai giù, se possibile
          if (rowIndex < 8) {
            this.focusCell(rowIndex + 1, colIndex);
          }
          break;
        case "ArrowUp":
          // Vai su, se possibile
          if (rowIndex > 0) {
            this.focusCell(rowIndex - 1, colIndex);
          }
          break;
      }
    },

    focusCell(row, col) {
      // Trova l'input e mettilo a fuoco
      this.$nextTick(() => {
        const input = this.$el.querySelector(
          `.sudoku-row:nth-child(${row + 1}) .sudoku-cell:nth-child(${col + 1}) input`
        );
        if (input) {
          input.focus();
        }
      });
    },

    handleInput(rowIndex, colIndex, value) {
      const num = parseInt(value);
      if (isNaN(num) || num < 1 || num > 9) {
        this.grid[rowIndex][colIndex] = null;
      } else {
        this.grid[rowIndex][colIndex] = num;
      }
      this.initialNumbers[rowIndex][colIndex] = this.grid[rowIndex][colIndex];
      this.errorMessage = '';
      this.hasError = false;
      this.focusedCell = { row: rowIndex, col: colIndex }; // Aggiorna la cella selezionata
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
      this.errorCells = []; // Resetta la lista degli errori

      // Controlla le righe e colonne
      for (let i = 0; i < 9; i++) {
        if (isValid(this.grid[i])) { // Se ci sono numeri duplicati in una riga
          for (let col = 0; col < 9; col++) {
            if (this.grid[i][col] !== null) {
              this.errorCells.push({ row: i, col }); // Aggiungi la cella all'array degli errori
            }
          }
        }
        if (isValid(this.grid.map(row => row[i]))) { // Se ci sono numeri duplicati in una colonna
          for (let row = 0; row < 9; row++) {
            if (this.grid[row][i] !== null) {
              this.errorCells.push({ row, col: i }); // Aggiungi la cella all'array degli errori
            }
          }
        }
      }

      // Controlla i quadrati 3x3
      for (let boxRow = 0; boxRow < 9; boxRow += 3) {
        for (let boxCol = 0; boxCol < 9; boxCol += 3) {
          let box = [];
          for (let row = 0; row < 3; row++) {
            for (let col = 0; col < 3; col++) {
              box.push(this.grid[boxRow + row][boxCol + col]);
            }
          }
          if (isValid(box)) {
            for (let row = 0; row < 3; row++) {
              for (let col = 0; col < 3; col++) {
                if (this.grid[boxRow + row][boxCol + col] !== null) {
                  this.errorCells.push({ row: boxRow + row, col: boxCol + col }); // Aggiungi la cella all'array degli errori
                }
              }
            }
          }
        }
      }

      return this.errorCells.length === 0; // Restituisce false se ci sono errori
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
        this.hasError = true; // Imposta hasError a true
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
      this.isSolved = true; // Imposta lo stato a "risolto"
      this.hasError = false; // Imposta hasError a false se la risoluzione è corretta
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
        this.grid[rowIndex][colIndex] = null; // Resetto le celle se l'input non è valido
      } else {
        this.grid[rowIndex][colIndex] = num;
      }
      this.initialNumbers[rowIndex][colIndex] = this.grid[rowIndex][colIndex];
      this.errorMessage = ''; // Rimuovi il messaggio di errore
      this.hasError = false; // Ripristina il flag hasError
    }
  }
};

</script>

<template>
  <div class="sudoku-container">
    <div class="game-area">
      <h1 class="title">Sudoku Solver</h1>
      
      <div class="sudoku-grid-container">
        <div class="sudoku-grid">
          <div class="sudoku-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
            <div
              class="sudoku-cell"
              v-for="(cell, colIndex) in row"
              :key="colIndex"
              :class="{
                'focused-cell': focusedCell.row === rowIndex && focusedCell.col === colIndex,
                'error': errorCells.some(err => err.row === rowIndex && err.col === colIndex),
                'border-right': (colIndex + 1) % 3 === 0 && colIndex !== 8,
                'border-bottom': (rowIndex + 1) % 3 === 0 && rowIndex !== 8,
                'user-cell': initialNumbers[rowIndex][colIndex] !== null
              }"
            >
              <input
                type="number"
                min="1"
                max="9"
                v-model.number="grid[rowIndex][colIndex]"
                @input="handleInput(rowIndex, colIndex, $event.target.value)"
                @keydown="handleKeyDown($event, rowIndex, colIndex)"
                :disabled="isSolving || isSolved"
                :class="{
                  'computer-solved': initialNumbers[rowIndex][colIndex] === null && grid[rowIndex][colIndex] !== null,
                  'user-input': initialNumbers[rowIndex][colIndex] !== null
                }"
                placeholder=""
              />
            </div>
          </div>
        </div>
      </div>
      
      <div class="controls">
        <button 
          v-if="!isSolved && !hasError" 
          class="btn solve-btn" 
          @click="solve"
          :class="{ 'hide': isSolving }"
        >
          <span class="btn-text">Solve Sudoku</span>
        </button>

        <button 
          v-if="isSolved || hasError"
          class="btn reset-btn" 
          @click="resetGrid"
        >
          <span class="btn-text">Reset Sudoku</span>
        </button>
      </div>
      
      <div v-if="errorMessage" class="error-message">
        <span>{{ errorMessage }}</span>
      </div>
      
      <div class="instructions">
        <h3>How to Play</h3>
        <ul>
          <li>Fill in numbers 1-9 in empty cells</li>
          <li>Each row, column, and 3×3 box must contain numbers 1-9 without repetition</li>
          <li>Use arrow keys to navigate the grid</li>
          <li>Click "Solve Sudoku" to find the solution</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Poppins', sans-serif;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  margin: 0;
  min-height: 100vh;
}

.sudoku-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.game-area {
  width: 100%;
  max-width: 800px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: white;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 30px;
  margin: 20px 0;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #2c3e50;
  margin-bottom: 30px;
  text-align: center;
  position: relative;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 4px;
  background: linear-gradient(90deg, #3498db, #2ecc71);
  border-radius: 2px;
}

.sudoku-grid-container {
  position: relative;
  width: 100%;
  max-width: 550px;
  margin: 0 auto;
}

.sudoku-grid {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  grid-template-rows: repeat(9, 1fr);
  gap: 1px;
  background-color: #34495e;
  border: 2px solid #34495e;
  border-radius: 5px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  aspect-ratio: 1 / 1;
  width: 100%;
  max-width: 550px;
}

.sudoku-row {
  display: contents;
}

.sudoku-cell {
  background-color: #f9f9f9;
  position: relative;
  transition: all 0.2s ease;
}

.sudoku-cell.border-right {
  border-right: 2px solid #34495e;
}

.sudoku-cell.border-bottom {
  border-bottom: 2px solid #34495e;
}

.sudoku-cell.focused-cell {
  background-color: #e3f2fd;
}

.sudoku-cell.error {
  background-color: rgba(255, 99, 71, 0.2);
}

.sudoku-cell.user-cell {
  background-color: #f0f8ff;
}

.sudoku-cell input {
  width: 100%;
  height: 100%;
  border: none;
  background: transparent;
  text-align: center;
  font-size: calc(1vw + 1vh + 0.5vmin);
  font-weight: 600;
  color: #2c3e50;
  cursor: pointer;
  appearance: none;
  -moz-appearance: textfield;
  outline: none;
  transition: all 0.3s ease;
}

@media (min-width: 768px) {
  .sudoku-cell input {
    font-size: 1.6rem;
  }
}

.sudoku-cell input.user-input {
  color: #2980b9;
}

.sudoku-cell input.computer-solved {
  color: #27ae60;
}

.sudoku-cell input:disabled {
  cursor: default;
}

.sudoku-cell input::-webkit-outer-spin-button,
.sudoku-cell input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.controls {
  display: flex;
  justify-content: center;
  margin-top: 30px;
  width: 100%;
}

.btn {
  position: relative;
  padding: 12px 30px;
  font-size: 1rem;
  font-weight: 600;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  z-index: 1;
  min-width: 180px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, rgba(255,255,255,0.1), rgba(255,255,255,0.3));
  transform: translateX(-100%);
  transition: all 0.3s ease;
  z-index: -1;
}

.btn:hover::before {
  transform: translateX(0);
}

.btn:active {
  transform: translateY(2px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.solve-btn {
  background: linear-gradient(90deg, #3498db, #2ecc71);
  color: white;
}

.reset-btn {
  background: linear-gradient(90deg, #e74c3c, #c0392b);
  color: white;
}

.btn-text {
  position: relative;
  z-index: 2;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.solve-btn:not(.hide) {
  animation: pulse 2s infinite;
}

.hide {
  opacity: 0;
  transform: scale(0);
  transition: all 0.5s ease;
}

.error-message {
  margin-top: 20px;
  padding: 12px 20px;
  background-color: #ffecec;
  color: #e74c3c;
  border-radius: 5px;
  font-weight: 500;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 550px;
}

.instructions {
  margin-top: 30px;
  width: 100%;
  max-width: 550px;
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.instructions h3 {
  color: #2c3e50;
  margin-bottom: 10px;
  font-weight: 600;
  border-bottom: 2px solid #eaeaea;
  padding-bottom: 8px;
}

.instructions ul {
  padding-left: 20px;
}

.instructions li {
  color: #555;
  margin-bottom: 8px;
  font-size: 0.95rem;
}

/* Responsive Design */
@media (max-width: 767px) {
  .game-area {
    padding: 20px 15px;
  }
  
  .title {
    font-size: 1.8rem;
    margin-bottom: 20px;
  }
  
  .sudoku-grid-container {
    max-width: 100%;
  }
  
  .btn {
    padding: 10px 20px;
    font-size: 0.9rem;
    min-width: 150px;
  }
  
  .instructions {
    padding: 15px;
  }
  
  .instructions h3 {
    font-size: 1.1rem;
  }
  
  .instructions li {
    font-size: 0.85rem;
  }
}

@media (max-width: 480px) {
  .game-area {
    padding: 15px 10px;
  }
  
  .title {
    font-size: 1.5rem;
  }
  
  .btn {
    padding: 8px 16px;
    font-size: 0.8rem;
    min-width: 130px;
  }
}

/* Animation for solved cells */
@keyframes solveFadeIn {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.computer-solved {
  animation: solveFadeIn 0.3s ease forwards;
}

/* Error animation */
@keyframes shake {
  0%, 100% {
    transform: translateX(0);
  }
  10%, 30%, 50%, 70%, 90% {
    transform: translateX(-2px);
  }
  20%, 40%, 60%, 80% {
    transform: translateX(2px);
  }
}

.error {
  animation: shake 0.5s ease;
}
</style>