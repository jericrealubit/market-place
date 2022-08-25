<template>
  <v-app>
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Profiles</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded>Home</v-btn>      
      <v-btn text rounded>Login</v-btn>
    </v-app-bar>

    <v-main class="mt-10 mx-auto">
             
      <v-data-table
        :headers="headTitle"
        :items="profiles"
        :search="search"
        :items-per-page="10"        
        :loading="loading"
        loading-text="Loading... Please wait"        
        class="elevation-1"
      >
      <v-divider inset></v-divider>

        <template v-slot:top>
          <v-toolbar flat color="white">
            <v-spacer />
            <div class="d-flex w-100">              
              <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" dense outlined single-line hide-details></v-text-field>              
              <v-btn                
                color="primary"
                class="ml-2 white--text"
                @click="addNew">
                <v-icon dark>mdi-account-plus-outline</v-icon>Add
              </v-btn>
            </div>
          </v-toolbar>
        </template>

        <template v-slot:[`item.username`]="{ item }">
          <v-text-field v-model="editedItem.username" :hide-details="true" dense single-line :autofocus="true" v-if="item._id === editedItem._id"></v-text-field>
          <span v-else>{{item.username}}</span>
        </template>

        <template v-slot:[`item.email`]="{ item }">
          <v-text-field v-model="editedItem.email" :hide-details="true" dense single-line v-if="item._id === editedItem._id" ></v-text-field>
          <span v-else>{{item.email}}</span>
        </template>

        <template v-slot:[`item.imageUrl`]="{ item }">
          <v-text-field v-model="editedItem.imageUrl" :hide-details="true" dense single-line v-if="item._id === editedItem._id" ></v-text-field>
          <img v-else :src="item.imageUrl" style="width: 50px; height: 50px" />            
        </template>

        <template v-slot:[`item.multipleImageUrl`]="{ item }">
          <v-text-field v-model="editedItem.multipleImageUrl" :hide-details="true" dense single-line v-if="item._id === editedItem._id" ></v-text-field>
          <div v-else v-html="showImage(item.multipleImageUrl)"></div>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <div v-if="item._id === editedItem._id">
            <v-icon color="red" class="mr-3" @click="close">
              mdi-window-close
            </v-icon>
            <v-icon color="green"  @click="save">
              mdi-content-save
            </v-icon>
          </div>
          <div v-else>
            <v-icon color="green" class="mr-3" @click="editItem(item)">
              mdi-pencil
            </v-icon>
            <v-icon color="red" @click="deleteItem(item)">
              mdi-delete
            </v-icon>
          </div>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="getAll">Reset</v-btn>
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
    search: '',
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
      { text: "Actions", value: "actions" }
    ],    
    links: [
      'Home',
      'Login'
    ],   
    editedIndex: -1,
    editedItem: {      
      username: '',
      email: '',
      multipleImageUrl: '',
      imageUrl: ''
    },
    defaultItem: {      
      username: '',
      email: '',
      multipleImageUrl: '',
      imageUrl: ''
    }
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
      },
      editItem(item) {
        this.editedIndex = this.profiles.indexOf(item);
        this.editedItem = Object.assign({}, item);
      },

      deleteItem(item) {
        const index = this.desserts.indexOf(item);
        confirm('Are you sure you want to delete this item?') && this.desserts.splice(index, 1);
      },
      close() {
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem);
          this.editedIndex = -1;
        }, 300)
      },
      addNew() {
        const addObj = Object.assign({}, this.defaultItem);        
        this.profiles.unshift(addObj);
        this.editItem(addObj);
      },
      save() {
        if (this.editedIndex > -1) {
          Object.assign(this.profiles[this.editedIndex], this.editedItem)
        }
        this.close()
      },
    },
    mounted() {      
      this.getAll();
    }
};
</script>

<style>

  /* striped */
  tbody tr:nth-of-type(odd) {
    background-color: rgba(0, 0, 0, .05);
  }

  .v-btn:not(.v-btn--round).v-size--default {
    height: auto;
  }
  
  .w-100 {
    width: 100%
  }
</style>