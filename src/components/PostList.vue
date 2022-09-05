<template>

  <v-container>
    <v-row>

      <v-col
        v-for="(profile, n) in profiles"
        :key="n"
        class="d-flex child-flex pa-4"
        cols="12"
        xs="6"
        sm="4"
        md="3"
        xl="2"
      >

        <v-card
            :loading="loading"
            class="mx-auto my-1"
        >
          <template slot="progress">
            <v-progress-linear
              color="deep-purple"
              height="10"
              indeterminate
            ></v-progress-linear>
          </template>

          <v-img
            :src="profile.productimage"
            :lazy-src="`https://picsum.photos/10/6?image=${n * 5 + 10}`"
            aspect-ratio="1"
            class="grey lighten-2"
          >
            <template v-slot:placeholder>
              <v-row
                class="fill-height ma-0"
                align="center"
                justify="center"
              >
                <v-progress-circular
                  indeterminate
                  color="grey lighten-5"
                ></v-progress-circular>
              </v-row>
            </template>
          </v-img>

          <v-card-title>${{profile.price}}</v-card-title>
          <v-card-text>
            <div class="text-subtitle-2">
              {{profile.title}}
            </div>

            <div>{{ profile.description }}</div>
            <div>{{ profile.location }}</div>
          </v-card-text>
        </v-card>

      </v-col>
    </v-row>

    <!-- message -->
    <v-card>
      <v-card-text class="pl-10 pb-0">
        <div>* {{ msg }}</div>
      </v-card-text>
    </v-card>

    <!-- datatable -->
    <v-data-table
      :headers="headTitle"
      :items="profiles"
      :search="search"
      :items-per-page="10"
      :loading="loading"
      loading-text="Loading... Please wait"
      class="elevation-1"
    >
      <!-- search bar -->
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
              <v-icon dark>mdi-storefront-plus</v-icon>Add
            </v-btn>
          </div>
        </v-toolbar>
      </template>

      <template v-slot:[`item.productimage`]="{ item }">
        <v-file-input v-if="item._id === editedItem._id"
          v-model="editedItem.productimage"
          :hide-details="true"
          dense
          single-line
          :autofocus="true"
          accept="image/*"
          show-size
          prepend-icon="mdi-camera"
          @change="uploadImage"
        ></v-file-input>
        <span v-else>
          <v-img
            lazy-src="https://picsum.photos/id/11/10/6"
            max-height="50"
            max-width="50"
            :src="item.productimage"
          ></v-img>
        </span>
      </template>

      <template v-slot:[`item.price`]="{ item }">
        <v-text-field v-model="editedItem.price" :hide-details="true" dense single-line v-if="item._id === editedItem._id"></v-text-field>
        <span v-else>{{item.price}}</span>
      </template>

      <template v-slot:[`item.title`]="{ item }">
        <v-text-field v-model="editedItem.title" :hide-details="true" dense single-line v-if="item._id === editedItem._id"></v-text-field>
        <span v-else>{{item.title}}</span>
      </template>

      <template v-slot:[`item.description`]="{ item }">
        <v-text-field v-model="editedItem.description" :hide-details="true" dense single-line v-if="item._id === editedItem._id"></v-text-field>
        <span v-else>{{item.description}}</span>
      </template>

      <template v-slot:[`item.location`]="{ item }">
        <v-text-field v-model="editedItem.location" :hide-details="true" dense single-line v-if="item._id === editedItem._id"></v-text-field>
        <span v-else>{{item.location}}</span>
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

  </v-container>
</template>

<script>
  const axios = require("axios");
  const formData = require("form-data");
  const api = "https://api-posts-jeric.netlify.app/.netlify/functions/api/"
  export default {
    name: 'PostsList',

    data: () => ({
      msg: '',
      add: true,
      search: '',
      loading: true,
      showPassword: false,
      profiles: [],
      id: "", // id to update PUT:id, was set by GET:id
      headTitle: [
        { text: "Item", value: "productimage" },
        { text: "Price", value: "price" },
        { text: "Title", value: "title" },
        { text: "Description", value: "description" },
        { text: "Location", value: "location" },
        { text: "Actions", value: "actions" },
      ],
      formValues: {
        productimage: '',
        price: '',
        title: '',
        description: '',
        location: '',
      },
      editedIndex: -1,
      editedItem: {
        productimage: '',
        price: '',
        title: '',
        description: '',
        location: '',
      },
      defaultItem: {
        productimage: '',
        price: '',
        title: '',
        description: '',
        location: '',
      }
    }),
    methods: {
      showImage(imageUrls) {
        let urls = '';
        if (imageUrls) {
          const imgArr = imageUrls.split(',')
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
          this.formValues.price = this.editedItem.price
          this.formValues.title = this.editedItem.title
          this.formValues.description = this.editedItem.description
          this.formValues.location = this.editedItem.location
          Object.assign(this.profiles[this.editedIndex], this.editedItem)
          let fetchApi = (this.id) ? (api + this.id) : api;
          let fetchMethod = (this.id) ? "PUT" : "POST";
          fetch(fetchApi, {
            method: fetchMethod,
            headers: { 'Content-Type': 'application/json' },
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
      uploadImage(event) {
        try {
        console.log("ping !");
        const bodyFormData = new formData();
        bodyFormData.append("image", event); // event = is our image object

        const apiUrl = "https://api.imgbb.com/1/upload";
        const apiKey = "bdcadd576cf26e4fe1f31de4594ca4fd";
        axios({
          method: "POST",
          url: apiUrl + "?key=" + apiKey,
          data: bodyFormData,
          header: { "Content-Type": "multipart/form-data" }
          })
          .then((response) => {
            console.log("API response ↓");
            console.log(response);
            console.log(response.data.data.url) // image url
            this.uploadedImage = response.data.data.url; // assign to data property
            this.formValues.productimage = response.data.data.url;
          })
          .catch((err) => {
          console.log("API error ↓");
          console.log(err);

          if (err.response.data.error) {
            console.log(err.response.data.error);
            //When trouble shooting, simple informations about the error can be found in err.response.data.error so it's good to display it
          }
          });
        } catch (error) {
          console.log(error);
        }
      },
    },
    mounted() {
      this.getAll();
    },
  }
</script>

<style scoped>
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