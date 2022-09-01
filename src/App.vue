<template>
  <v-app>
    <!-- App bar -->
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Profiles</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded>Home</v-btn>
      <v-btn text rounded>Login</v-btn>
    </v-app-bar>

    <!-- main -->
    <v-main>
      <v-container class="grey lighten-5" height="100%">
        <v-row no-gutters>
          <v-col>

            <ProfileList />

          </v-col>
          <v-col>

            <UploadImage />

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
  import ProfileList from "./components/ProfileList.vue"
  import UploadImage from "./components/UploadImage.vue"
  export default {
    components: { ProfileList, UploadImage },
    name: 'App',

    data: () => ({
      links: [
        'Home',
        'Login'
      ],
    }),
    methods: {

    }
  };
</script>

<style scoped>

</style>