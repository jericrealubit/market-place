<template>
  <v-app>

    <UserLogin @logged-user="setLoggedUser" />

    <!-- App bar -->
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Profiles</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded>Home</v-btn>
      <v-btn text rounded>{{ loggedUser }}</v-btn>
      <v-btn v-if="loggedUser != 'guest'" text rounded @click="logout">
        <v-icon small>mdi-logout</v-icon>
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
    <v-footer
      color="primary lighten-1"
      padless
    >
      <v-row
        justify="center"
        no-gutters
      >
        <v-card-text class="py-2 white--text text-center">
          {{ new Date().getFullYear() }} — <strong>Profiles</strong>
        </v-card-text>
      </v-row>
    </v-footer>

  </v-app>
</template>

<script>
  // import ProfileList from "./components/ProfileList.vue"
  // import UploadImage from "./components/UploadImage.vue"
  import UserLogin from "./components/UserLogin.vue"
  import PostList from "./components/PostList.vue"
  export default {
    components: { PostList, UserLogin },
    name: 'App',

    data: () => ({
      loggedUser: "guest"
    }),
    methods: {
      logout() {
        localStorage.removeItem("loggedUser");
        document.location.reload(true); // force page reload
      },
      setLoggedUser(loggedInUser) {
        this.loggedUser =  "guest";
        this.loggedUser = loggedInUser;
      }
    },
    mounted() {
      if (localStorage.loggedUser) {
        this.loggedUser = localStorage.loggedUser
      }
    }
  };
</script>

<style scoped>

</style>