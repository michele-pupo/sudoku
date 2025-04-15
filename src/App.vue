
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
      isDesktop: window.innerWidth >= 1024,
      // Tema scuro
      darkMode: false,
      // Mostra dialogo di conferma
      showConfirmDialog: false,
      // Difficoltà selezionata per conferma
      selectedDifficulty: null
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

  computed: {
    gameState() {
      if (this.isSolving) return 'solving';
      if (this.isSolved) return 'solved';
      if (this.hasError) return 'error';
      if (this.grid.every(row => row.every(cell => cell !== null))) return 'complete';
      return 'playing';
    },
    
    isGameComplete() {
      return this.grid.every(row => row.every(cell => cell !== null));
    },
    
    emptyCount() {
      return this.grid.flat().filter(cell => cell === null).length;
    }
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
      this.animationStates = Array(9).fill(null).map(() => Array(9).fill(false));

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

    toggleDarkMode() {
      this.darkMode = !this.darkMode;
      document.body.classList.toggle('dark-mode', this.darkMode);
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
      this.focusedCell = { row, col };
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
      this.checkGridValidity(); // Controlla immediatamente se ci sono errori
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

    isValidMove(grid, row, col, num) {
      // Controlla la riga
      for (let x = 0; x < 9; x++) {
        if (x !== col && grid[row][x] === num) return false;
      }
      
      // Controlla la colonna
      for (let x = 0; x < 9; x++) {
        if (x !== row && grid[x][col] === num) return false;
      }
      
      // Controlla il riquadro 3x3
      let boxRow = Math.floor(row / 3) * 3;
      let boxCol = Math.floor(col / 3) * 3;
      
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          if ((boxRow + i !== row || boxCol + j !== col) && 
              grid[boxRow + i][boxCol + j] === num) {
            return false;
          }
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

      this.hasError = this.errorCells.length > 0;
      return !this.hasError; // Restituisce false se ci sono errori
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

    confirmSolveWithDifficulty(difficulty) {
      this.selectedDifficulty = difficulty;
      this.showConfirmDialog = true;
    },

    cancelSolve() {
      this.showConfirmDialog = false;
      this.selectedDifficulty = null;
    },

    confirmSolve() {
      this.showConfirmDialog = false;
      const difficulty = this.selectedDifficulty;
      this.selectedDifficulty = null;
      this.solveWithDifficulty(difficulty);
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
      // Inizia con un'animazione più veloce e rallenta quando ci si avvicina al numero corretto
      let num = 1;
      let delay = 50; // Delay iniziale in ms
      
      this.animationStates[row][col] = true; // Attiva l'animazione
      
      while (num !== correctNumber) {
        if (!this.isSolving) return; // Interrompe l'animazione se il flag è falso
        this.grid[row][col] = num; // Mostra il numero animato
        await this.sleep(delay); // Breve pausa per l'animazione
        
        // Calcola quanto siamo vicini al numero corretto
        const distance = Math.min(
          Math.abs(num - correctNumber),
          Math.abs(num + 9 - correctNumber),
          Math.abs(num - 9 - correctNumber)
        );
        
        // Aumenta il delay quando ci avviciniamo al numero corretto
        delay = 50 + Math.max(0, (3 - distance) * 30);
        
        num = (num % 9) + 1; // Passa al numero successivo
      }
      
      // Una volta trovato il numero corretto, lo evidenzia
      this.grid[row][col] = correctNumber;
      
      // Aggiunge un effetto visivo speciale quando il numero corretto è trovato
      setTimeout(() => {
        this.animationStates[row][col] = false;
      }, 500);
    },

    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },

    getHint() {
      if (!this.hasSolution) {
        // Calcola prima la soluzione completa
        this.completeSolution = JSON.parse(JSON.stringify(this.grid));
        const solved = this.solveSudoku(this.completeSolution);
        
        if (!solved) {
          this.errorMessage = "Non esiste una soluzione per questo Sudoku!";
          this.hasError = true;
          return;
        }
        
        this.hasSolution = true;
      }
      
      // Trova celle vuote
      let emptyCells = [];
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (this.grid[row][col] === null) {
            emptyCells.push({ row, col });
          }
        }
      }
      
      if (emptyCells.length === 0) {
        this.errorMessage = "Non ci sono celle vuote!";
        return;
      }
      
      // Seleziona casualmente una cella vuota
      const randomIndex = Math.floor(Math.random() * emptyCells.length);
      const { row, col } = emptyCells[randomIndex];
      
      // Applica un'animazione più breve per il suggerimento
      this.currentSolveColor = '#E91E63'; // Colore rosa per i suggerimenti
      this.spinToCorrectNumber(row, col, this.completeSolution[row][col]);
    },

    verifySolution() {
      if (!this.isGameComplete) {
        this.errorMessage = "Completa il Sudoku prima di verificarlo!";
        return;
      }
      
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (!this.hasSolution) {
            // Calcola la soluzione se non esiste
            this.completeSolution = JSON.parse(JSON.stringify(this.initialNumbers));
            const solved = this.solveSudoku(this.completeSolution);
            if (!solved) {
              this.errorMessage = "Impossibile verificare: nessuna soluzione esistente!";
              this.hasError = true;
              return;
            }
            this.hasSolution = true;
          }
          
          // Verifica se la soluzione del giocatore corrisponde alla soluzione calcolata
          if (this.grid[row][col] !== this.completeSolution[row][col]) {
            this.errorMessage = "La tua soluzione contiene errori!";
            this.hasError = true;
            this.errorCells.push({ row, col });
            return;
          }
        }
      }
      
      // Se arriviamo qui, la soluzione è corretta
      this.isSolved = true;
      this.errorMessage = "";
      this.hasError = false;
    },

    // Funzione per ottenere lo stile di colore specifico per i numeri risolti dal computer
    getComputerSolvedStyle() {
      return { color: this.currentSolveColor };
    }
  }
}
</script>

<template>
  <div class="sudoku-container" :class="{ 'desktop-layout': isDesktop, 'dark-mode': darkMode }">
    <div class="game-area">
      <div class="header-section">
        <h1 class="title">Sudoku Solver</h1>
        <div class="theme-toggle">
          <button @click="toggleDarkMode" class="theme-btn" :title="darkMode ? 'Passa a tema chiaro' : 'Passa a tema scuro'">
            <span v-if="!darkMode">🌙</span>
            <span v-else>☀️</span>
          </button>
        </div>
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
                  'odd-block': Math.floor(rowIndex/3) % 2 === Math.floor(colIndex/3) % 2,
                  'animated': animationStates[rowIndex][colIndex]
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
          
          <!-- Game Status -->
          <div class="game-status" v-if="!isSolving">
            <div v-if="gameState === 'playing'" class="status-playing">
              <p class="status-message">{{ emptyCount }} celle da riempire</p>
            </div>
            
            <div v-else-if="gameState === 'solved'" class="status-success">
              <p class="status-message">Sudoku risolto con successo!</p>
            </div>
            
            <div v-else-if="gameState === 'complete'" class="status-checking">
              <p class="status-message">Sudoku completato! Controlliamo la soluzione...</p>
              <!-- Aggiungi un pulsante per verificare -->
              <button class="btn verify-btn" @click="verifySolution">Verifica Soluzione</button>
            </div>
          </div>
        </div>
        
        <div class="controls-section">
          <div v-if="errorMessage" class="error-message">
            <span>{{ errorMessage }}</span>
          </div>
          
          <div class="difficulty-selector" v-if="!isSolved && !isSolving">
            <p class="difficulty-text">Risolvi con livello di difficoltà:</p>
            <div class="difficulty-buttons">
              <button class="difficulty-btn easy" @click="confirmSolveWithDifficulty('easy')">
                <span class="btn-text">Facile</span>
              </button>
              <button class="difficulty-btn medium" @click="confirmSolveWithDifficulty('medium')">
                <span class="btn-text">Medio</span>
              </button>
              <button class="difficulty-btn hard" @click="confirmSolveWithDifficulty('hard')">
                <span class="btn-text">Difficile</span>
              </button>
              <button class="difficulty-btn complete" @click="confirmSolveWithDifficulty('complete')">
                <span class="btn-text">Completo</span>
              </button>
            </div>
          </div>
          
          <div class="controls">
            <!-- Hint button -->
            <button 
              v-if="!isSolved && !isSolving && emptyCount > 0"
              class="btn hint-btn" 
              @click="getHint"
            >
              <span class="btn-text">Suggerimento</span>
            </button>
            
            <!-- Reset button - always visible -->
            <button 
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
              <li>Clicca "Suggerimento" per ricevere aiuto su una cella casuale</li>
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
    
    <!-- Dialog di conferma -->
    <div class="modal-overlay" v-if="showConfirmDialog">
      <div class="confirm-dialog">
        <h3>Conferma azione</h3>
        <p>Sei sicuro di voler risolvere il Sudoku con difficoltà {{ selectedDifficulty === 'easy' ? 'Facile' : 
            selectedDifficulty === 'medium' ? 'Media' : 
            selectedDifficulty === 'hard' ? 'Difficile' : 'Completa' }}?</p>
        <div class="dialog-buttons">
          <button @click="cancelSolve" class="btn cancel-btn">Annulla</button>
          <button @click="confirmSolve" class="btn confirm-btn">Conferma</button>
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
  transition: background 0.3s ease;
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
  display: flex;
  flex-direction: column;
}

.game-area {
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 1200px;
  padding: 20px;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  overflow: hidden;
}

.header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.title {
  color: #333;
  font-size: 2rem;
  margin: 0;
  text-align: center;
  flex-grow: 1;
  transition: color 0.3s ease;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
  height: calc(100% - 60px);
  overflow: hidden;
}

.desktop-layout .content-wrapper {
  flex-direction: row;
}

.sudoku-grid-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 350px;
}

.sudoku-grid {
  display: flex;
  flex-direction: column;
  border: 2px solid #333;
  border-radius: 4px;
  overflow: hidden;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}

.sudoku-row {
  display: flex;
  width: 100%;
}

.sudoku-cell {
  position: relative;
  width: 11.11%;
  aspect-ratio: 1/1;
  border: 1px solid #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
  background-color: #fff;
  transition: all 0.3s ease;
}

.odd-block {
  background-color: #f5f7fa;
}

.sudoku-cell.right-border {
  border-right: 2px solid #333;
}

.sudoku-cell.bottom-border {
  border-bottom: 2px solid #333;
}

.sudoku-cell input {
  width: 100%;
  height: 100%;
  border: none;
  text-align: center;
  font-size: 1.2rem;
  background: transparent;
  color: #333;
  font-weight: bold;
  padding: 0;
  cursor: pointer;
  transition: all 0.3s ease;
}

.sudoku-cell input:focus {
  outline: none;
  background-color: rgba(33, 150, 243, 0.1);
}

.sudoku-cell input:disabled {
  cursor: not-allowed;
}

.focused-cell {
  background-color: rgba(33, 150, 243, 0.1) !important;
}

.user-input {
  color: #333 !important;
  font-weight: 700;
}

.computer-solved {
  /* Il colore sarà impostato dinamicamente in base alla difficoltà */
  font-weight: 500;
}

.error {
  background-color: rgba(244, 67, 54, 0.1) !important;
}

.error input {
  color: #f44336 !important;
}

.animated {
  animation: spin-number 0.5s ease-in-out;
}

@keyframes spin-number {
  0% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.game-status {
  margin-top: 15px;
  text-align: center;
  font-size: 1rem;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.status-message {
  margin-bottom: 10px;
  font-weight: 500;
  transition: color 0.3s ease;
}

.status-success .status-message {
  color: #27ae60;
}

.status-checking .status-message {
  color: #2196f3;
}

.controls-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
  background-color: #f9fafc;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  overflow-y: auto;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

.desktop-layout .controls-section {
  min-width: 300px;
  max-width: 400px;
}

.error-message {
  padding: 10px;
  background-color: rgba(244, 67, 54, 0.1);
  color: #f44336;
  border-radius: 4px;
  text-align: center;
  font-weight: 500;
  margin-bottom: 10px;
  transition: all 0.3s ease;
}

.difficulty-selector {
  margin-bottom: 15px;
}

.difficulty-text {
  margin-bottom: 10px;
  font-weight: 500;
  text-align: center;
  transition: color 0.3s ease;
}

.difficulty-buttons {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.btn {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.2s ease;
  width: 100%;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.difficulty-btn {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  cursor: pointer;
  color: white;
  transition: all 0.2s ease;
}

.difficulty-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.difficulty-btn.easy {
  background-color: #4CAF50;
}

.difficulty-btn.medium {
  background-color: #2196F3;
}

.difficulty-btn.hard {
  background-color: #FF9800;
}

.difficulty-btn.complete {
  background-color: #9C27B0;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 15px;
}

.hint-btn {
  background-color: #E91E63;
}

.reset-btn {
  background-color: #607D8B;
}

.verify-btn {
  background-color: #2196F3;
  margin-top: 10px;
}

.instructions {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: background-color 0.3s ease;
}

.instructions h3 {
  margin-bottom: 10px;
  color: #333;
  font-size: 1.1rem;
  transition: color 0.3s ease;
}

.instructions ul {
  padding-left: 20px;
  margin-bottom: 15px;
}

.instructions li {
  margin-bottom: 5px;
  font-size: 0.9rem;
  color: #555;
  transition: color 0.3s ease;
}

.difficulty-legend {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  color: #555;
  transition: color 0.3s ease;
}

.color-sample {
  width: 15px;
  height: 15px;
  border-radius: 3px;
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

.theme-toggle {
  margin-left: 10px;
}

.theme-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.theme-btn:hover {
  transform: rotate(30deg);
}

/* Modal di conferma */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.confirm-dialog {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
}

.confirm-dialog h3 {
  margin-bottom: 15px;
  color: #333;
}

.dialog-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.cancel-btn {
  background-color: #9e9e9e;
}

.confirm-btn {
  background-color: #2196F3;
}

/* Dark Mode */
.dark-mode {
  background: linear-gradient(135deg, #2d3748 0%, #1a202c 100%);
}

body.dark-mode {
  background: #121212;
}

.dark-mode .game-area {
  background-color: #1e1e1e;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.dark-mode .title {
  color: #e0e0e0;
}

.dark-mode .sudoku-grid {
  border-color: #555;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.dark-mode .sudoku-cell {
  border-color: #444;
  background-color: #2d2d2d;
}

.dark-mode .odd-block {
  background-color: #252525;
}

.dark-mode .sudoku-cell.right-border {
  border-right-color: #555;
}

.dark-mode .sudoku-cell.bottom-border {
  border-bottom-color: #555;
}

.dark-mode .sudoku-cell input {
  color: #e0e0e0;
}

.dark-mode .focused-cell {
  background-color: rgba(33, 150, 243, 0.2) !important;
}

.dark-mode .user-input {
  color: #e0e0e0 !important;
}

.dark-mode .controls-section {
  background-color: #2d2d2d;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.dark-mode .error-message {
  background-color: rgba(244, 67, 54, 0.15);
}

.dark-mode .difficulty-text {
  color: #e0e0e0;
}

.dark-mode .instructions {
  background-color: rgba(45, 45, 45, 0.9);
}

.dark-mode .instructions h3 {
  color: #e0e0e0;
}

.dark-mode .instructions li {
  color: #bbb;
}

.dark-mode .legend-item {
  color: #bbb;
}

.dark-mode .confirm-dialog {
  background-color: #2d2d2d;
}

.dark-mode .confirm-dialog h3 {
  color: #e0e0e0;
}

.dark-mode .status-message {
  color: #bbb;
}

/* Responsive design per schermi piccoli */
@media (max-width: 768px) {
  .title {
    font-size: 1.5rem;
  }
  
  .sudoku-cell input {
    font-size: 1rem;
  }
  
  .controls-section {
    padding: 15px;
  }
  
  .difficulty-buttons {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .sudoku-container {
    padding: 10px;
  }
  
  .game-area {
    padding: 10px;
  }
  
  .sudoku-cell input {
    font-size: 0.9rem;
  }
  
  .btn {
    padding: 8px 12px;
    font-size: 0.9rem;
  }
}

</style>