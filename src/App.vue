<template>
  <v-app>
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Profiles</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded>Home</v-btn>      
      <v-btn text rounded>Login</v-btn>
    </v-app-bar>

    <v-main class="mt-10 mx-auto">
      <!-- Login Module -->
      <!-- <v-card width="400" class="mx-auto mt-5">
        <v-card-title>
          <h1 class="display-1">Login</h1>
        </v-card-title>
        <v-card-text>
          <v-form>
            <v-text-field label="Username" prepend-icon="mdi-account-circle" />
            <v-text-field label="Password" 
              :type="showPassword ? 'text' : 'password'" 
              prepend-icon="mdi-lock" 
              :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
              @click:append="showPassword = !showPassword"
            />
          </v-form>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
        <v-btn color="success">Register</v-btn>
        <v-spacer></v-spacer>
        <v-btn color="info">Login</v-btn>
      </v-card-actions>
      </v-card>     -->

      <v-data-table
        :headers="headTitle"
        :items="profiles"
        :items-per-page="5"
        class="elevation-1"
        :loading="loading"
        loading-text="Loading... Please wait"
      >
        <template v-slot:[`item.imageUrl`]="{ item }">
            <img :src="item.imageUrl" style="width: 50px; height: 50px" />            
        </template>
      
        <template v-slot:[`item.multipleImageUrl`]="{ item }">            
            <div v-html="showImage(item.multipleImageUrl)"></div>
        </template>

        <template v-slot:[`item._id`]="{ item }">
          <v-icon small @click="editDoc(item._id)" title="Edit">mdi-pencil</v-icon>
          <v-icon small @click="deleteDoc(item._id)" title="Delete">mdi-trash-can-outline</v-icon>
        </template>

      </v-data-table>   
    </v-main>
    
    <v-footer
      color="primary lighten-1"
      padless       
    >
      <v-row
        justify="center"
        no-gutters
      >
        <!-- <v-btn
          v-for="link in links"
          :key="link"
          color="white"
          text
          rounded
          class="my-2"
        >
          {{ link }}
        </v-btn> -->
    
      <v-card-text class="py-2 white--text text-center">
        {{ new Date().getFullYear() }} — <strong>Profiles</strong>
      </v-card-text>

      </v-row>
    </v-footer>
    
  </v-app>
</template>

<script>
const api = "https://rest-api-express-mongodb.netlify.app/.netlify/functions/api/"
export default {
  name: 'App',

  data: () => ({
    loading: true,
    showPassword: false,
    profiles: [],
    id: "", // id to update PUT:id, was set by GET:id
    formValues: {
      username: '',
      email: '',
      imageUrl: '',
      multipleImageUrl: ''
    },
    headTitle: [
      { text: "Username", value: "username" },
      { text: "Email", value: "email" },
      { text: "Images", value: "imageUrl" },
      { text: "Multiple Images", value: "multipleImageUrl" },
      { text: "Actions", value: "_id" }
    ],    
    links: [
      'Home',
      'Login'
    ],   
  }),
  methods: {      
      showImage(imageUrls) {
        let urls = '';
        if (imageUrls) {          
          const imgArr = imageUrls.split(',')  
          // return imgArr.length
          imgArr.forEach(element => {
            urls += `<img src="${element}" width="50">`
          });          
        }         
        return urls;
      },
      insertDoc() { // done
        fetch(api, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.formValues)            
          })
          .then((response) => response.text())
          .then((data) => {
            console.log(data)
            this.getAll();
          })
          .catch((err) => {
            if (err) throw err;
          })
      },
      updateDoc() {
        fetch(api + this.id, {
            method: 'PUT',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.formValues)
          })
          .then((response) => response.text())
          .then((data) => {
            console.log(data)
            this.getAll();
          })
          .catch((err) => {
            if (err) throw err;
          })        
      },
      getDoc(id) { // done      
        this.id = id // PUT:id needs this to update the selected document
        fetch(api + id, {
            method: 'GET'
          })
          .then((response) => response.json())
          .then((data) => {
            console.log(data)
            this.formValues.username = data.username
            this.formValues.email = data.email
            this.formValues.imageUrl = data.imageUrl
            this.formValues.multipleImageUrl = data.multipleImageUrl            
          })
          .catch((err) => {
            if (err) throw err;
          })
      },
      deleteDoc(id) { // done
        fetch(api + id, {
            method: 'DELETE'           
          })
          .then((response) => response.text())
          .then((data) => {
            console.log(data)
            this.getAll();
          })
          .catch((err) => {
            if (err) throw err;
          })
        
      },
      getAll() {
        fetch(api)
        .then((response) => response.json())
        .then((data) => {
          this.profiles = data
          this.loading = false
        })
        .catch((err) => {
          if (err) throw err;
        })
      }
    },
    mounted() {      
      this.getAll();
    }
};
</script>
