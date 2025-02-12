<template>
  <div class="container">
    <game-score :blacks="blacks" :whites="whites" />
    <game-deco :isBlack="isBlack" :isYourTurn="isYourTurn()"/>
    <div class="grid">
      <div class="cell" v-for="(n, i) in 64" :key="i" @click="put(i)">
        <div v-if="black(i)" class="b dot"></div>
        <div v-if="white(i)" class="w dot"></div>
        <div v-if="isValidMove(i)[0] && isYourTurn()" class="move dot"></div>
      </div>
    </div>
  </div>
</template>

<script>
import GameScore from "./GameScore.vue";
import Vue from "vue";
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
import GameDeco from "@/components/game/GameDeco";

export default {
  components: { GameDeco, GameScore },
  data: () => ({
    isBlack: true,
    blacks: 2,
    whites: 2,
    board: [
      "", "", "", "", "", "", "", "",
      "", "", "", "", "", "", "", "",
      "", "", "", "", "", "", "", "",
      "", "", "", "w", "b", "", "", "",
      "", "", "", "b", "w", "", "", "",
      "", "", "", "", "", "", "", "",
      "", "", "", "", "", "", "", "",
      "", "", "", "", "", "", "", ""
    ],
    possibleMoves: {},
    connected: false,
    turn: 0,
    color: "",
    score: 0
  }),
  async mounted() {
    let colorRes = await Vue.axios.post("/api/color", {
      "roomId": this.$store.state.roomId,
      "username": this.$store.state.username
    });
    this.color = colorRes.data.color;

    let response = await Vue.axios.post("/api/post-board", {
      "isBlack": this.isBlack,
      "board": this.board
    });

    this.possibleMoves = response.data.possibleMoves;
    console.log(this.possibleMoves);
    console.log(this.$store.state.username);
    this.connect();
  },
  methods: {
    connect() {
      this.socket = new SockJS("/api/board-socket");
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
        {},
        frame => {
          this.connected = true;
          console.log(frame);
          this.stompClient.subscribe("/topic/play/" + this.$store.state.roomId, tick => {
            // console.log(tick.body);
            this.board = JSON.parse(tick.body)["board"];
            this.possibleMoves = JSON.parse(tick.body)["possibleMoves"];
            this.blacks = JSON.parse(tick.body)["blacks"];
            this.whites = JSON.parse(tick.body)["whites"];
            this.isBlack = JSON.parse(tick.body)["isBlack"];
            this.turn = JSON.parse(tick.body)["turn"];
            this.gameOver();
          });
        },
        error => {
          console.log(error);
          this.connected = false;
        }
      );
    },
    send(toPlace) {
      if (this.stompClient && this.stompClient.connected) {
        const obj = {roomId: this.$store.state.roomId, board: this.board, isBlack: this.isBlack, turn: this.turn, toFlip: this.possibleMoves[toPlace] };
        this.stompClient.send("/app/board/" + this.$store.state.roomId, JSON.stringify(obj), {});
      }
    },
    async put(i) {
      let move = this.isValidMove(i);
      let toPlace = move[1];
      if (move[0] && this.board[i] === "" && this.isYourTurn()) {
        this.send(toPlace);
      }
    },
    black(i) {
      return this.board[i] === "b";
    },
    white(i) {
      return this.board[i] === "w";
    },
    isValidMove(i) {
      let ret = [false, ""];
      let moves = Object.keys(this.possibleMoves);
      moves.forEach((function(move) {
        // move is key of map(String), i is int
        if (i == move) {
          ret = [true, move];
        }
      }));
      return ret;
    },
    async gameOver() {
      if (!this.board.includes("") || Object.keys(this.possibleMoves).length === 0) {
        await this.sleep(40);
        if (this.blacks > this.whites) {
          alert("Black Won!!!");
          if (this.color === "b") {
            this.score = 1;
          }
        } else if (this.whites > this.blacks) {
          alert("White Won!!!");
          if (this.color === "w") {
            this.score = 1;
          }
        } else {
          alert("Draw!");
        }

        await this.recordLatestGame();
        await this.sleep(100);
        await this.$router.replace("/");
      }
    },
    async recordLatestGame() {
      await Vue.axios.post("/api/latest-game", {
        "roomId": this.$store.state.roomId,
        "username": this.$store.state.username,
        "score": this.score
      });
    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    isYourTurn() {
      let c = "";
      if (this.isBlack) {
        c = "b";
      } else {
        c = "w";
      }
      return c === this.color;
    }

  }
};
</script>

<style scoped>
.grid {
  display: grid;
  grid-template-columns: repeat(8, 55px);
  grid-template-rows: repeat(8, 55px);
  margin: auto;
  width: fit-content;
}

.cell {
  display: inline-block;
  background-color: rgb(216, 216, 216);
  border: solid 1px white;
  padding: 6px;
  overflow: hidden;
}

.dot {
  display: inline-block;
  border-radius: 50%;
  margin: auto;
  align-content: center;
  height: 40px;
  width: 40px;
  box-shadow: 1px 1px;
}

.move {
  border: #807d7d solid 1px;
  box-shadow: 0 0;
}

.b {
  background-color: black;
}

.w {
  background-color: white;
}
</style>
