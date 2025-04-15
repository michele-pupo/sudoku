<script>
export default {
  data() {
    return {
      // Griglia corrente da visualizzare e modificare
      grid: [],
      // Griglia che contiene i numeri iniziali
      initialNumbers: Array(9).fill(null).map(() => Array(9).fill(null)),
      // Griglia con la soluzione completa
      completeSolution: Array(9).fill(null).map(() => Array(9).fill(null)),
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
      // Livello di difficoltà corrente
      difficultyLevel: 'complete',
      // Indica se la soluzione completa è stata calcolata
      hasSolution: false,
      // Per tenere traccia del colore attuale dei numeri risolti automaticamente
      currentSolveColor: '#27ae60',
      // Dizionario dei colori per i diversi livelli di difficoltà
      difficultyColors: {
        easy: '#4CAF50',
        medium: '#2196F3',
        hard: '#FF9800',
        complete: '#9C27B0'
      },
      // Per verificare se siamo in modalità desktop
      isDesktop: window.innerWidth >= 1024
    };
  },

  mounted() {
    this.resetGrid();
    window.addEventListener('keydown', this.handleArrowKeys); // Assegna il listener
    window.addEventListener('resize', this.checkViewportSize); // Per controllare la dimensione della finestra
    this.checkViewportSize(); // Controlla la dimensione iniziale
  },

  beforeUnmount() {
    window.removeEventListener('keydown', this.handleArrowKeys); // Rimuovi il listener
    window.removeEventListener('resize', this.checkViewportSize); // Rimuovi il listener resize
  },

  methods: {
    checkViewportSize() {
      this.isDesktop = window.innerWidth >= 1024;
    },

    resetGrid() {
      // Reset della griglia
      this.initialNumbers = Array(9).fill(null).map(() => Array(9).fill(null));
      this.grid = JSON.parse(JSON.stringify(this.initialNumbers));
      this.completeSolution = Array(9).fill(null).map(() => Array(9).fill(null));

      // Resetta gli errori
      this.errorCells = [];
      this.hasError = false;
      this.errorMessage = ''; // Rimuovi eventuali messaggi di errore

      // Riabilita gli input e il pulsante "Solve Sudoku"
      this.isSolving = false;  // Permette di inserire numeri
      this.isSolved = false;   // Permette di risolvere di nuovo il Sudoku
      this.focusedCell = { row: 0, col: 0 }; // Resetta la cella selezionata
      this.hasSolution = false; // Resetta lo stato della soluzione
      this.currentSolveColor = '#27ae60'; // Resetta il colore di default
    },

    handleArrowKeys(event) {
      if (this.isSolving) return; // Non consentire movimenti durante la risoluzione
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
      this.hasSolution = false; // Resetta la soluzione quando l'utente modifica la griglia
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

    // Ottiene le celle da risolvere in base alla difficoltà
    getCellsToSolveByDifficulty(difficulty) {
      if (!this.hasSolution) return [];
      
      let emptyCells = [];
      
      // Trova tutte le celle vuote
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (this.initialNumbers[row][col] === null) {
            emptyCells.push({ row, col });
          }
        }
      }
      
      // Randomizza l'ordine delle celle per una soluzione più imprevedibile
      for (let i = emptyCells.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [emptyCells[i], emptyCells[j]] = [emptyCells[j], emptyCells[i]];
      }
      
      // Determina quante celle risolvere in base al livello di difficoltà
      let totalEmptyCells = emptyCells.length;
      let cellsToSolve = [];
      
      switch (difficulty) {
        case 'easy':
          // Risolvi il 75% delle celle (lasciando il 25% da risolvere all'utente)
          cellsToSolve = emptyCells.slice(0, Math.ceil(totalEmptyCells * 0.75));
          break;
        case 'medium':
          // Risolvi il 50% delle celle (lasciando il 50% da risolvere all'utente)
          cellsToSolve = emptyCells.slice(0, Math.ceil(totalEmptyCells * 0.5));
          break;
        case 'hard':
          // Risolvi il 25% delle celle (lasciando il 75% da risolvere all'utente)
          cellsToSolve = emptyCells.slice(0, Math.ceil(totalEmptyCells * 0.25));
          break;
        case 'complete':
          // Risolvi tutte le celle
          cellsToSolve = emptyCells;
          break;
      }
      
      return cellsToSolve;
    },

    // Risolvi il sudoku con un livello di difficoltà specifico
    async solveWithDifficulty(difficulty) {
      if (!this.checkGridValidity()) {
        this.errorMessage = 'Griglia Sudoku non valida! Correggi i numeri duplicati.';
        this.hasError = true;
        return;
      }

      // Imposta il colore per la difficoltà selezionata
      this.currentSolveColor = this.difficultyColors[difficulty];

      this.difficultyLevel = difficulty;
      this.errorMessage = '';
      this.isSolving = true;

      // Se non abbiamo già calcolato la soluzione completa, calcolala ora
      if (!this.hasSolution) {
        // Crea una copia della griglia per calcolare la soluzione completa
        this.completeSolution = JSON.parse(JSON.stringify(this.grid));
        const solved = this.solveSudoku(this.completeSolution);
        
        if (!solved) {
          this.errorMessage = "Non esiste una soluzione!";
          this.hasError = true;
          this.isSolving = false;
          return;
        }
        
        this.hasSolution = true;
      }

      // Ottieni le celle da risolvere in base al livello di difficoltà
      const cellsToSolve = this.getCellsToSolveByDifficulty(difficulty);
      
      // Applica l'animazione solo alle celle che verranno risolte
      await this.animateCells(cellsToSolve);
      
      this.isSolving = false;
      
      // Se abbiamo risolto tutte le celle, imposta lo stato a "risolto"
      if (difficulty === 'complete') {
        this.isSolved = true;
      }
      
      this.hasError = false;
    },

    async animateCells(cellsToSolve) {
      for (const cell of cellsToSolve) {
        if (!this.isSolving) return; // Interrompe l'animazione se il flag è falso
        await this.spinToCorrectNumber(cell.row, cell.col, this.completeSolution[cell.row][cell.col]);
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

    // Funzione per ottenere lo stile di colore specifico per i numeri risolti dal computer
    getComputerSolvedStyle() {
      return { color: this.currentSolveColor };
    }
  }
}
</script>

<template>
  <div class="sudoku-container" :class="{ 'desktop-layout': isDesktop }">
    <div class="game-area">
      <div class="header-section">
        <h1 class="title">Sudoku Solver</h1>
      </div>
      
      <div class="content-wrapper">
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
                  'right-border': (colIndex + 1) % 3 === 0 && colIndex < 8,
                  'bottom-border': (rowIndex + 1) % 3 === 0 && rowIndex < 8,
                  'user-cell': initialNumbers[rowIndex][colIndex] !== null,
                  'odd-block': Math.floor(rowIndex/3) % 2 === Math.floor(colIndex/3) % 2
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
                  :style="initialNumbers[rowIndex][colIndex] === null && grid[rowIndex][colIndex] !== null ? getComputerSolvedStyle() : {}"
                  placeholder=""
                />
              </div>
            </div>
          </div>
        </div>
        
        <div class="controls-section">
          <div v-if="errorMessage" class="error-message">
            <span>{{ errorMessage }}</span>
          </div>
          
          <div class="difficulty-selector" v-if="!isSolved && !hasError && !isSolving">
            <p class="difficulty-text">Risolvi con livello di difficoltà:</p>
            <div class="difficulty-buttons">
              <button class="difficulty-btn easy" @click="solveWithDifficulty('easy')">
                <span class="btn-text">Facile</span>
              </button>
              <button class="difficulty-btn medium" @click="solveWithDifficulty('medium')">
                <span class="btn-text">Medio</span>
              </button>
              <button class="difficulty-btn hard" @click="solveWithDifficulty('hard')">
                <span class="btn-text">Difficile</span>
              </button>
              <button class="difficulty-btn complete" @click="solveWithDifficulty('complete')">
                <span class="btn-text">Completo</span>
              </button>
            </div>
          </div>
          
          <div class="controls">
            <button 
              v-if="isSolved || hasError"
              class="btn reset-btn" 
              @click="resetGrid"
            >
              <span class="btn-text">Reset Sudoku</span>
            </button>
          </div>
          
          <div class="instructions">
            <h3>Istruzioni</h3>
            <ul>
              <li>Inserisci numeri da 1 a 9 nelle celle vuote</li>
              <li>Ogni riga, colonna e riquadro 3×3 deve contenere i numeri da 1 a 9 senza ripetizioni</li>
              <li>Usa i tasti freccia per navigare nella griglia</li>
              <li>Scegli un livello di difficoltà per risolvere il sudoku</li>
            </ul>
            <div class="difficulty-legend">
              <div class="legend-item">
                <span class="color-sample easy"></span>
                <span>Facile: risolve quasi tutto</span>
              </div>
              <div class="legend-item">
                <span class="color-sample medium"></span>
                <span>Medio: risolve metà delle celle</span>
              </div>
              <div class="legend-item">
                <span class="color-sample hard"></span>
                <span>Difficile: risolve poche celle</span>
              </div>
              <div class="legend-item">
                <span class="color-sample complete"></span>
                <span>Completo: risolve tutto</span>
              </div>
            </div>
          </div>
        </div>
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
  display: flex;
  align-items: center;
  justify-content: center;
}

.sudoku-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 20px;
  transition: all 0.3s ease;
}

/* Layout per desktop che entra tutto in una finestra senza scrollare */
.desktop-layout {
  height: 100vh;
  max-height: 100vh;
  overflow: hidden;
}

.desktop-layout .game-area {
  height: calc(100vh - 40px);
  max-height: calc(100vh - 40px);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.game-area {
  width: 100%;
  max-width: 1400px;
  background-color: white;
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1), 0 5px 15px rgba(0, 0, 0, 0.07);
  padding: 30px;
  margin: 20px 0;
  transition: all 0.3s ease;
}

.header-section {
  margin-bottom: 20px;
  text-align: center;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #2c3e50;
  position: relative;
  display: inline-block;
  margin: 0;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -8px;
  left: 0;
  width: 100%;
  height: 4px;
  background: linear-gradient(90deg, #3498db, #2ecc71);
  border-radius: 2px;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  height: calc(100% - 70px);
}

@media (min-width: 1024px) {
  .content-wrapper {
    flex-direction: row;
    gap: 30px;
  }
  
  .sudoku-grid-container {
    flex: 1;
    max-width: 55%;
  }
  
  .controls-section {
    flex: 1;
    max-width: 45%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding-left: 20px;
  }
}

.sudoku-grid-container {
  margin-bottom: 30px;
  width: 100%;
}

.sudoku-grid {
  display: grid;
  grid-template-rows: repeat(9, 1fr);
  background-color: #34495e;
  border: 3px solid #34495e;
  border-radius: 10px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
  aspect-ratio: 1 / 1;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

.sudoku-row {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
}

.sudoku-cell {
  position: relative;
  transition: all 0.3s ease;
  border: 1px solid #c4c4c4;
  background-color: #f9f9f9;
}

/* Bordi più marcati per i quadratoni 3x3 */
.sudoku-cell.right-border {
  border-right: 3px solid #34495e;
}

.sudoku-cell.bottom-border {
  border-bottom: 3px solid #34495e;
}

/* Stile per differenziare i blocchi 3x3 con colori alternati */
.sudoku-cell.odd-block {
  background-color: #f2f7fc;
}

.sudoku-cell.focused-cell {
  background-color: #e3f2fd;
  box-shadow: inset 0 0 0 2px #3498db;
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
  padding: 0;
}

@media (min-width: 768px) {
  .sudoku-cell input {
    font-size: 1.8rem;
  }
}

.sudoku-cell input.user-input {
  color: #2980b9;
  font-weight: 700;
}

.sudoku-cell input.computer-solved {
  animation: solveFadeIn 0.5s ease forwards;
  font-weight: 600;
}

.sudoku-cell input:disabled {
  cursor: default;
}

.sudoku-cell input::-webkit-outer-spin-button,
.sudoku-cell input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.controls-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.controls {
  display: flex;
  justify-content: center;
  width: 100%;
}

.btn {
  position: relative;
  padding: 14px 30px;
  font-size: 1.1rem;
  font-weight: 600;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  z-index: 1;
  min-width: 200px;
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
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
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

.error-message {
  padding: 15px 20px;
  background-color: #ffecec;
  color: #e74c3c;
  border-radius: 10px;
  font-weight: 500;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-left: 5px solid #e74c3c;
  width: 100%;
}

.instructions {
  background-color: #f8f9fa;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  height: 100%;
  overflow: auto;
}

.instructions h3 {
  color: #2c3e50;
  margin-bottom: 15px;
  font-weight: 600;
  position: relative;
  padding-bottom: 10px;
}

.instructions h3::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 50px;
  height: 3px;
  background: linear-gradient(90deg, #3498db, #2ecc71);
  border-radius: 2px;
}

.instructions ul {
  padding-left: 20px;
  margin-bottom: 20px;
}

.instructions li {
  color: #555;
  margin-bottom: 10px;
  font-size: 1rem;
  position: relative;
}

.instructions li::before {
  content: '•';
  color: #3498db;
  font-weight: bold;
  display: inline-block;
  width: 1em;
  margin-left: -1em;
}

.difficulty-selector {
  margin-bottom: 20px;
  width: 100%;
}

.difficulty-text {
  color: #2c3e50;
  font-weight: 500;
  margin-bottom: 15px;
  text-align: center;
}

.difficulty-buttons {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

@media (min-width: 768px) {
  .difficulty-buttons {
    grid-template-columns: repeat(4, 1fr);
  }
}

.difficulty-btn {
  padding: 12px 15px;
  font-size: 0.9rem;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  color: white;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.difficulty-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.difficulty-btn:active {
  transform: translateY(1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
}

.difficulty-btn.easy {
  background: linear-gradient(135deg, #4CAF50, #2E7D32);
}

.difficulty-btn.medium {
  background: linear-gradient(135deg, #2196F3, #1565C0);
}

.difficulty-btn.hard {
  background: linear-gradient(135deg, #FF9800, #EF6C00);
}

.difficulty-btn.complete {
  background: linear-gradient(135deg, #9C27B0, #6A1B9A);
}

.difficulty-legend {
  margin-top: 20px;
  background-color: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.legend-item:last-child {
  margin-bottom: 0;
}

.color-sample {
  display: inline-block;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  margin-right: 10px;
}

.color-sample.easy {
  background-color: #4CAF50;
}

.color-sample.medium {
  background-color: #2196F3;
}

.color-sample.hard {
  background-color: #FF9800;
}

.color-sample.complete {
  background-color: #9C27B0;
}

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

</style>