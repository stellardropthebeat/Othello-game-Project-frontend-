<template>
  <v-container>

    <v-img
        alt="Vuetify Logo"
        class="homeLogo mb-10 mt-5"
        src="@/assets/othello-logo.png"
        contain
    />

    <v-row align="center" justify="space-around">
      <v-col sm="6" >
          <v-btn large block @click="gotoAbout" class="font-weight-black">
            HOW TO PLAY OTHELLO
          </v-btn>

          <v-btn large block @click="gotoLobby" class="font-weight-black">
            JOIN ROOM
          </v-btn>

          <v-btn large block  @click="gotoReplay" class="font-weight-black">
            REPLAY LATEST GAME
          </v-btn>

          <v-btn large block  @click="gotoLeaderboard" class="font-weight-black">
            LEADERBOARD
          </v-btn>

          <v-btn large block @click="gotoCredit" class="font-weight-black">
            CREDIT
          </v-btn>

      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Vue from "vue";
export default {
  name: "HomeView",

  components: {},

  methods: {
    gotoAbout() {
      this.$router.push({path: "/about"})
    },
    async gotoLobby() {
      let response = await Vue.axios.post("/api/join-room", {
        "username": this.$store.state.username}
      );
      this.$store.state.roomId = response.data.availableRoom;
      console.log(this.$store.state.roomId);
      await this.$router.push({path: "/waiting"})
    },
    gotoLeaderboard() {
      this.$router.push({path: "/leaderboard"})
    },
    gotoCredit() {
      this.$router.push({path: "/credit"})
    },
    gotoReplay() {
      this.$router.push({path: "/replay"})
    }
  },
};
</script>

<style scoped>
.homeLogo {
  margin: auto;
  width: 500px;

}

button {
  margin-bottom: 20px;
  border-radius: 15px;
  border-bottom: 2px darkgreen solid;
  box-shadow: 2px 3px black;
}

</style>

