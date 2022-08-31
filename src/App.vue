<template>
  <v-app>
    <v-app-bar app color="primary" dark dense>
      <v-toolbar-title>Profiles</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text rounded>Home</v-btn>      
      <v-btn text rounded>Login</v-btn>
    </v-app-bar>

    <v-main>

      <v-container class="grey lighten-5">
        <v-row no-gutters>
          <v-col>
            <v-card>
              <v-card-text class="pl-10 pb-0">
                <div>* {{ msg }}</div>
              </v-card-text>
            </v-card>
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
                      title="Add New Profile"
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
                  <v-icon color="red" class="mr-3" @click="close" title="Cancel">
                    mdi-window-close
                  </v-icon>
                  <v-icon color="green"  @click="save" title="Save">
                    mdi-content-save
                  </v-icon>
                </div>
                <div v-else>
                  <v-icon color="green" class="mr-3" @click="editItem(item)" title="Edit">
                    mdi-pencil
                  </v-icon>
                  <v-icon color="red" @click="deleteItem(item)" title="Delete">
                    mdi-delete
                  </v-icon>
                </div>
              </template>
              <template v-slot:no-data>
                <v-btn color="primary" @click="getAll">Reset</v-btn>
              </template>

            </v-data-table>   
          </v-col>          
          <v-col>
            <v-card 
              class="pa-10"
              elevation="2"
            >
          
              <v-file-input
                accept="image/*"
                label="File input"
                show-size
                @change="doUpload"
              ></v-file-input>
          
            </v-card>
          </v-col>
        </v-row>
      </v-container>
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
    msg: '',
    add: true,
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
      { text: "Actions", value: "actions" },
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
    doUpload(event) {
    
        //var $files = $(this).get(0).files;
        console.log(event)
        console.log(event.lastModified)

        // if ($files.length) {
        //   // Reject big files
        //   if ($files[0].size > $(this).data('max-size') * 1024) {
        //     console.log('Please select a smaller file');
        //     return false;
        //   }

        //   // Begin file upload
        //   console.log('Uploading file to Imgur..');

        //   // Replace ctrlq with your own API key
        //   var apiUrl = 'https://api.imgur.com/3/image';
        //   var apiKey = 'ctrlq';

        //   var settings = {
        //     async: false,
        //     crossDomain: true,
        //     processData: false,
        //     contentType: false,
        //     type: 'POST',
        //     url: apiUrl,
        //     headers: {
        //       Authorization: 'Client-ID ' + apiKey,
        //       Accept: 'application/json',
        //     },
        //     mimeType: 'multipart/form-data',
        //   };

        //   var formData = new FormData();
        //   formData.append('image', $files[0]);
        //   settings.data = formData;

        //   // Response contains stringified JSON
        //   // Image URL available at response.data.link
        //   $.ajax(settings).done(function (response) {
        //     console.log(response);
        //   });
        // }
      
    },
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
      this.id = item._id
      this.add = false
    },
    deleteItem(item) {
      if (confirm('Are you sure you want to delete this item?')) {
        const index = this.profiles.indexOf(item);
        this.profiles.splice(index, 1);
        fetch(api + item._id, {
            method: 'DELETE'           
          })
          .then((response) => response.text())
          .then((data) => {
            this.msg = data
            this.getAll();
          })
          .catch((err) => {
            if (err) throw err;
          })
        console.log("confirm delete")
      } else {
        console.log("cancel delete")
      }
    },
    close() {
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
        if (this.add) {
          this.profiles.shift(this.editedItem);
          this.editItem(this.editedItem);
        }
      }, 300)
    },
    addNew() {
      const addObj = Object.assign({}, this.defaultItem);        
      this.profiles.unshift(addObj);
      this.editItem(addObj);
      this.add = true
    },
    save() {
      if (this.editedIndex > -1) {
        this.formValues.username = this.editedItem.username
        this.formValues.email = this.editedItem.email
        this.formValues.imageUrl = this.editedItem.imageUrl
        this.formValues.multipleImageUrl = this.editedItem.multipleImageUrl 
        console.log(this.formValues)
        Object.assign(this.profiles[this.editedIndex], this.editedItem)
        let fetchApi = (this.id) ? (api + this.id) : api;
        let fetchMethod = (this.id) ? "PUT" : "POST";
        fetch(fetchApi, {
            method: fetchMethod,
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.formValues)
          })
          .then((response) => response.text())
          .then((data) => {
            this.msg = data
            this.getAll();
          })
          .catch((err) => {
            if (err) throw err;
          })        
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