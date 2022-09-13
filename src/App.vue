<template>
  <v-app>
    <UserLogin @logged-user="setLoggedUser" v-if="loginform" />

    <!-- App bar -->
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Market Place</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded @click="setBuying">Buy</v-btn>
      <v-btn text rounded @click="setSelling">Sell</v-btn>
      <v-btn text rounded>{{ loggedUser }}</v-btn>
      <v-btn
        v-if="loggedUser != 'guest'"
        text
        rounded
        @click="logout"
        title="logout"
      >
        <v-icon small>mdi-logout</v-icon>
      </v-btn>
      <v-btn v-else text rounded @click="login" title="login">
        <v-icon small>mdi-login</v-icon>
      </v-btn>
    </v-app-bar>

    <!-- main -->
    <v-main>
      <v-container class="grey lighten-5" height="100%">
        <v-row no-gutters>
          <v-col>
            <!-- <ProfileList /> -->
            <PostList />
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <!-- footer -->
    <v-footer color="primary lighten-1" padless>
      <v-row justify="center" no-gutters>
        <v-card-text class="py-2 white--text text-center">
          {{ new Date().getFullYear() }} — <strong>Market Place</strong>
        </v-card-text>
      </v-row>
    </v-footer>
  </v-app>
</template>

<script>
import UserLogin from "./components/UserLogin.vue";
import PostList from "./components/PostList.vue";

import { provide } from "vue";
import store from "@/store";
export default {
  setup() {
    provide("store", store);
  },

  components: { PostList, UserLogin },
  name: "App",
  data: () => ({
    loggedUser: "guest",
    loginform: false,
  }),
  methods: {
    setSelling() {
      store.state.buying = false;
    },
    setBuying() {
      store.state.buying = true;
    },
    logout() {
      localStorage.removeItem("loggedUser");
      localStorage.removeItem("userId");
      document.location.reload(true); // force page reload
    },
    setLoggedUser(loggedInUser) {
      this.loggedUser = loggedInUser;
    },
    login() {
      this.loginform = true;
    },
  },
  mounted() {
    if (localStorage.loggedUser) {
      this.loggedUser = localStorage.loggedUser;
    }
  },
};
</script>

<style scoped></style>
