<template>
  <div>
    <div class="gameStatus" :class="gameStatusColor">{{ gameStatusMessage }}</div>
    <table class="grid">
      <tr>
        <cell name="1"></cell>
        <cell name="2"></cell>
        <cell name="3"></cell>
      </tr>
      <tr>
        <cell name="4"></cell>
        <cell name="5"></cell>
        <cell name="6"></cell>
      </tr>
      <tr>
        <cell name="7"></cell>
        <cell name="8"></cell>
        <cell name="9"></cell>
      </tr>
    </table>
  </div>
</template>

<script>
import Cell from "./Cell.vue";
export default {
  components: { Cell },
  data() {
    return {
      // can be O or X
      activePlayer: "O",
      // mantains the status of the game: turn or win or draw
      gameStatus: "turn",
      gameStatusMessage: "O's turn",
      // status color is used as background color in the status bar
      // it can hold the name of either of the following CSS classes
      // statusTurn (default) is yellow for a turn
      // statusWin is green for a win
      // statusDraw is purple for a draw
      gameStatusColor: "statusTurn",
      // no. of moves played by both players in a single game (max = 9)
      moves: 0,
      // stores the placement of X and O in cells by their cell number
      cells: {
        1: "",
        2: "",
        3: "",
        4: "",
        5: "",
        6: "",
        7: "",
        8: "",
        9: ""
      },
      // contains all (8) possible winning conditions
      winConditions: [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9], // rows
        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9], // columns
        [1, 5, 9],
        [3, 5, 7] // diagonals
      ]
    };
  },
  computed: {
    // helper property to get the non-active player
    nonActivePlayer() {
      if (this.activePlayer === "O") {
        return "X";
      }
      return "O";
    }
  },
  created() {
    // listens for a strike made by the user on cell
    // it is called by the Cell component
    this.$eventHub.$on("strike", this.strike);
    // listens for a restart button press
    // the data of the component is reinitialized
    // it is called by the App component
    this.$eventHub.$on("gridReset", this.gridReset);
  },
  methods: {
    changePlayer() {
      this.activePlayer = this.nonActivePlayer;
    },
    changeGameStatus() {
      if (this.checkForWin()) {
        return this.gameIsWon();
      } else if (this.moves === 9) {
        return "draw";
      }
      return "turn";
    },
    checkForWin() {
      for (let i = 0; i < this.winConditions.length; i++) {
        let wc = this.winConditions[i];
        let cells = this.cells;
        if (this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])) {
          return true;
        }
      }
      return false;
    },
    areEqual() {
      var len = arguments.length;
      for (let i = 1; i < len; i++) {
        if (arguments[i] === "" || arguments[i] !== arguments[i - 1]) {
          return false;
        }
      }
      return true;
    },
    gameIsWon() {
      // fires win event for the App component to change the score
      this.$eventHub.$emit("win", this.activePlayer);
      // sets the game status message
      this.gameStatusMessage = `${this.activePlayer} Wins !`;
      // fires an event for the Cell to freeze
      this.$eventHub.$emit("freeze");
      // sets the status to win
      return "win";
    },
    strike(cellNumber, mark) {
      // sets either X or O in the clicked cell of the cells array
      this.cells[cellNumber] = this.activePlayer;
      // increments the number of moves
      this.moves++;
      // stores the game status by calling the changeGameStatus method
      this.gameStatus = this.changeGameStatus();
      this.changePlayer();
      if (mark === "X") {
        this.gameStatusMessage = `O's turn`;
      } else {
        this.gameStatusMessage = `X's turn`;
      }
    },
    gridReset() {
      Object.assign(this.$data, this.$options.data());
    }
  },
  watch: {
    // watches for change in the value of gameStatus and changes the status
    // message and color accordingly
    gameStatus: function() {
      if (this.gameStatus === "win") {
        this.gameStatusColor = "statusWin";
      } else if (this.gameStatus === "draw") {
        this.gameStatusColor = "statusDraw";
        this.gameStatusMessage = "Draw !";
      }
    }
  }
};
</script>

<style>
.grid {
  background-color: #34495e;
  color: white;
  width: 100%;
  border-collapse: collapse;
}
.gameStatus {
  margin: 0px;
  padding: 15px;
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  background-color: #f1c40f;
  color: white;
  font-size: 1.4em;
  font-weight: bold;
}
.statusTurn {
  background-color: #f1c40f;
}
.statusWin {
  background-color: #2ecc71;
}
.statusDraw {
  background-color: #9b59b6;
}
</style>