<template>
  <v-app>
    <v-app-bar app color="black" dark>
      <div class="d-flex align-center">
        <v-img
          class="logo"
          contain
          src="@/assets/othello-logo.png"
          transition="scale-transition"
          width="120"
          @click="goHome"
        />

      </div>

      <v-spacer></v-spacer>

      <v-btn v-if="isLoggedIn()" tile color="black" @click="logout">
        <v-icon left>mdi-logout</v-icon>
        LOGOUT
      </v-btn>
      <v-btn v-else  tile color="black" @click="signin">
        <v-icon left>mdi-logout</v-icon>
        SIGN UP
      </v-btn>

<!--      <button v-if="isLoggedIn" class="btn btn-danger"-->
<!--              v-on:click="logout" >LOGOUT</button>-->
<!--      <button v-else class="btn btn-primary"-->
<!--              v-on:click="login" >SIGNIN</button>-->

    </v-app-bar>

    <v-main class="blue-grey lighten-4">
      <router-view />
    </v-main>
  </v-app>
</template>

<style>
.logo {
  cursor: pointer;
}
</style>

<script>
import Vue from "vue";
import store from "./store"

export default {
  name: "App",

  data: () => ({
    //
  }),
  methods: {
    async logout () {
      let response = await Vue.axios.get("/api/logout");
      if (response.data.success) {
        store.dispatch("clearUser", response.data);
        await this.$router.push({path: "/login"});
      }
    },
    signin(){
      this.$router.push({ path: "/signIn"});
    },
    isLoggedIn(){
      return store.state.isLoggedIn;
    },
    goHome(){
      this.$router.push({ path: "/"});
    }
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Kanit:ital,wght@1,200&display=swap');
div {
  font-family: 'Bebas Neue', cursive;
  font-family: 'Kanit', sans-serif;
}
</style>
