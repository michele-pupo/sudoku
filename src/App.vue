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
  <div class="container py-5">
    <h1 class="pb-5 text-uppercase text-center display-1 fw-bold text-success">Sudoku</h1>
    <div class="sudoku-grid">
      <div class="sudoku-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
        <div
          class="sudoku-cell"
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          :class="{ 'focused-cell': focusedCell.row === rowIndex && focusedCell.col === colIndex }"
        >
        <input
          class="fw-bold"
          type="number"
          min="1"
          max="9"
          v-model.number="grid[rowIndex][colIndex]"
          @input="handleInput(rowIndex, colIndex, $event.target.value)"
          @keydown="handleKeyDown($event, rowIndex, colIndex)"
          :disabled="isSolving || isSolved"
          :class="{
            'computer-solved': animationStates[rowIndex][colIndex] === 'computer-solved',
            'user-input': initialNumbers[rowIndex][colIndex] !== null,
            'error': errorCells.some(err => err.row === rowIndex && err.col === colIndex)
          }"
          :style="{ color: initialNumbers[rowIndex][colIndex] !== null ? 'black' : 'red' }"
          placeholder=""
        />
        </div>
      </div>
    </div>
    <div class="d-flex justify-content-center pt-5">
      <!-- Mostra il pulsante "Solve Sudoku" solo se il Sudoku non è stato risolto -->
      <button 
        v-if="!isSolved && !hasError" 
        class="btn btn-success solve-btn" 
        @click="solve"
        :class="{ hide: isSolving }"
      >
        Solve Sudoku
      </button>

      <!-- Mostra il pulsante "Reset Sudoku" solo dopo che il Sudoku è stato risolto o se c'è un errore -->
      <button 
        v-if="isSolved || hasError"
        class="btn btn-danger reset-btn" 
        @click="resetGrid"
      >
        Reset Sudoku
    </button>
    </div>
    <div v-if="errorMessage">
      <h3 class="text-danger text-uppercase fw-bold mt-4 text-center">{{ errorMessage }}</h3>
    </div>
  </div>
</template>

<style>

body {
  font-family: 'Roboto Mono', monospace;
  background: linear-gradient(135deg, #a8edea, #fed6e3);
  animation: gradientAnimation 10s infinite alternate;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

@keyframes gradientAnimation {
  0% {
    background: linear-gradient(135deg, #a8edea, #fed6e3);
  }
  100% {
    background: linear-gradient(135deg, #fed6e3, #a8edea);
  }
}

h1 {
  font-family: 'Poppins', sans-serif;
  color: #28a745;
  background: linear-gradient(90deg, #28a745 40%, #1d72b8 60%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-size: 2.5rem;
  font-weight: bold;
  text-align: center;
  text-transform: uppercase;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
  animation: subtleGlow 2s infinite alternate;
  margin-bottom: 10px;
}

/* Animazione più discreta */
@keyframes subtleGlow {
  0% {
    text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3), 0 0 10px #28a745;
  }
  100% {
    text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3), 0 0 10px #1d72b8;
  }
}

.text-danger {
  font-family: 'Roboto', sans-serif;
  font-size: 1.2rem;
  background: #fdecea;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  position: fixed; /* Fissa il messaggio */
  bottom: 10px; /* Posiziona in basso */
  left: 50%; /* Centra orizzontalmente */
  transform: translateX(-50%); /* Centra correttamente */
  z-index: 100; /* Assicura che il messaggio sia visibile sopra gli altri elementi */
  width: auto;
  max-width: 90%; /* Limita la larghezza massima */
  text-align: center;
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
  border-radius: 15px; /* Angoli arrotondati */
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2), 0 4px 10px rgba(0, 0, 0, 0.1); /* Ombra */
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
  font-family: 'Roboto Mono', monospace;
  width: 100%;
  height: 100%;
  text-align: center;
  border: none;
  font-size: 1.8rem;
  font-weight: bold;
  color: #333; /* Colore scuro per i numeri */
  background: transparent;
}

.sudoku-cell input:hover {
  background-color: #e0f7fa; /* Colore blu chiaro per evidenziare la cella */
  transform: scale(1.05); /* Leggera espansione */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Ombra leggera */
  transition: all 0.2s ease-in-out;
  cursor: pointer;
}

.sudoku-cell input:focus {
  background: #fffae6; /* Cambia colore quando si interagisce */
  border-radius: 5px;
  outline: none;
  transition: 0.2s ease-in-out;
}

input[type=number]::-webkit-outer-spin-button,
input[type=number]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

button {
  padding: 12px 20px;
  font-size: 1.2rem;
  font-family: 'Roboto', sans-serif;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2), 0 3px 6px rgba(0, 0, 0, 0.15);
}

button:hover {
  background-color: #45a049;
  transform: scale(1.05);
  transform: translateY(-2px); /* Effetto sollevamento */
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
  transform: translateY(-2px); /* Effetto sollevamento */
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

/* Animazione per il pulsante "Solve Sudoku" */
@keyframes hideButton {
  0% {
    opacity: 1;
    transform: scale(1);
  }
  100% {
    opacity: 0;
    transform: scale(0);
  }
}

.solve-btn.hide {
  animation: hideButton 0.5s forwards;
}

/* Allinea il pulsante "Reset Sudoku" al centro dopo che "Solve" è stato premuto */
.reset-btn {
  animation: moveToCenter 0.5s ease-in-out;
  margin-top: 10px;
}

@keyframes moveToCenter {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(1.2);
  }
}

/* Animazione per il pulsante Solve Sudoku */
@keyframes showButton {
  0% {
    opacity: 0;
    transform: scale(0);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.solve-btn {
  animation: showButton 0.5s ease-in-out;
}

@keyframes blink {
  0% {
    background-color: #ffcccc; /* Colore rosso chiaro */
  }
  50% {
    background-color: #ff6666; /* Colore rosso scuro */
  }
  100% {
    background-color: #ffcccc;
  }
}

.sudoku-cell input.error {
  background: linear-gradient(135deg, #ff8a80, #ff5252);
  border: none;
  color: white;
  animation: shake 0.3s ease-in-out, glowError 1s infinite;
}

@keyframes shake {
  0%, 100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-5px);
  }
  50% {
    transform: translateX(5px);
  }
  75% {
    transform: translateX(-5px);
  }
}

@keyframes glowError {
  0% {
    box-shadow: 0 0 10px rgba(255, 82, 82, 0.5);
  }
  50% {
    box-shadow: 0 0 20px rgba(255, 82, 82, 1);
  }
  100% {
    box-shadow: 0 0 10px rgba(255, 82, 82, 0.5);
  }
}

.sudoku-cell input.correct {
  animation: glow 1s ease-out infinite;
}

.sudoku-cell input.user-input {
  background-color: #507687; /* Colore grigio chiaro */
  transition: background-color 0.3s ease; /* Effetto di transizione morbida */
}
</style>