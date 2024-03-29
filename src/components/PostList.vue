<template>
  <v-container>
    <!-- details -->
    <v-dialog v-model="detailsDialog" max-width="700px">
      <v-card>
        <v-layout wrap class="px-2">
          <v-flex xs10>
            <v-card-title>Message {{ sellerName }}</v-card-title>
          </v-flex>
          <v-flex xs2>
            <v-btn
              icon
              class="float-right mt-4"
              title="close"
              @click="detailClose"
            >
              <v-icon large>mdi-close-circle-outline</v-icon>
            </v-btn>
          </v-flex>
        </v-layout>

        <v-divider></v-divider>
        <v-layout wrap>
          <v-flex xs5 sm3>
            <div class="px-5 pt-5 pb-2 float-right">
              <v-img
                class="white--text rounded-lg pa-5"
                width="100px"
                height="100px"
                :src="details.productimage"
              >
              </v-img>
            </div>
          </v-flex>
          <v-flex xs7 sm9 class="pr-5">
            <v-card-title
              >$ {{ details.price }} - {{ details.title }}</v-card-title
            >
            <v-card-text class="text--primary">
              <div>{{ details.description }}</div>
            </v-card-text>
          </v-flex>
        </v-layout>

        <v-layout wrap class="pb-5">
          <v-flex>
            <MessageList
              :msglist="msglist"
              :usersNames="usersNames"
              :key="reloadMsg"
            />
          </v-flex>
        </v-layout>

        <v-layout wrap>
          <v-flex>
            <v-btn
              rounded
              class="float-right px-2 py-1 ma-1"
              @click="appendMessageToSeller(` I'm interested in this item. `)"
              >I'm interested in this item.</v-btn
            >
          </v-flex>
          <v-flex>
            <v-btn
              rounded
              class="float-left px-2 py-1 ma-1"
              @click="appendMessageToSeller(` Is this item still available? `)"
              >Is this item still available?</v-btn
            >
          </v-flex>
        </v-layout>
        <v-layout wrap>
          <v-flex>
            <v-btn
              rounded
              class="float-right px-2 py-1 ma-1"
              @click="
                appendMessageToSeller(` What condition is this item in? `)
              "
              >What condition is this item in?</v-btn
            >
          </v-flex>
          <v-flex>
            <v-btn
              rounded
              class="float-left px-2 py-1 ma-1"
              @click="appendMessageToSeller(` Do you deliver? `)"
              >Do you deliver?</v-btn
            >
          </v-flex>
        </v-layout>

        <v-textarea
          rows="1"
          auto-grow
          v-model="msgFormValues.message"
          id="messageToSeller"
          outlined
          label="Please type your message to the seller"
          class="pa-5"
        >
        </v-textarea>

        <v-divider></v-divider>
        <v-layout>
          <v-flex>
            <v-btn
              class="px-5 py-2 ma-3 ml-5 float-left"
              @click="detailClose"
              title="cancel"
            >
              <v-icon class="pr-2">mdi-message-off</v-icon>
              Cancel
            </v-btn>
          </v-flex>
          <v-flex>
            <v-btn
              class="px-5 py-2 ma-3 float-right mr-5"
              title="send message"
              :disabled="msgFormValues.message.length ? false : true"
              @click="sendMsgToSeller(details.post_id)"
            >
              <v-icon class="pr-2">mdi-message-text-outline</v-icon>
              Send Message
            </v-btn>
          </v-flex>
        </v-layout>
      </v-card>
    </v-dialog>

    <v-row v-if="store.state.buying">
      <v-progress-linear
        v-if="postsLoading"
        indeterminate
        color="cyan"
      ></v-progress-linear>

      <v-col
        v-for="(post, n) in posts"
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
          id="card"
          title="View Details"
          @click="showDetails(post._id)"
        >
          <template slot="progress">
            <v-progress-linear
              color="deep-purple"
              height="10"
              indeterminate
            ></v-progress-linear>
          </template>
          <!--
          <v-card-text>
            <ul>
              <li v-for="(postmsg, i) in postMessages[post._id]" :key="i">
                {{ postmsg.user_id ? usersNames[postmsg.user_id] : "Guest" }}:
                {{ postmsg.message }}
              </li>
            </ul>
          </v-card-text> -->

          <v-img
            :src="post.productimage"
            :lazy-src="`https://picsum.photos/10/6?image=${n * 5 + 10}`"
            aspect-ratio="1"
            class="grey lighten-2"
          >
            <template v-slot:placeholder>
              <v-row class="fill-height ma-0" align="center" justify="center">
                <v-progress-circular
                  indeterminate
                  color="grey lighten-5"
                ></v-progress-circular>
              </v-row>
            </template>
          </v-img>

          <v-card-title>${{ post.price }}</v-card-title>
          <v-card-text>
            <div class="text-subtitle-2">
              {{ post.title }}
            </div>

            <!-- <div>{{ post.description }}</div> -->
            <div>{{ post.location }}</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-row v-if="!store.state.buying">
      <!-- message -->
      <v-card>
        <v-card-text class="pl-10 pb-0">
          <div>* {{ msg }}</div>
        </v-card-text>

        <!-- datatable -->
        <v-data-table
          :headers="headTitle"
          :items="userPosts"
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
                <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="Search"
                  dense
                  outlined
                  single-line
                  hide-details
                ></v-text-field>
                <v-btn
                  title="Add New Profile"
                  color="primary"
                  class="ml-2 white--text"
                  @click="addNew"
                >
                  <v-icon dark>mdi-storefront-plus</v-icon>Add
                </v-btn>
              </div>
            </v-toolbar>
          </template>

          <template v-slot:[`item.productimage`]="{ item }">
            <v-file-input
              v-if="item._id === editedItem._id && !item.productimage"
              :hide-details="true"
              dense
              single-line
              :autofocus="true"
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
              >
              </v-img>
            </span>
          </template>

          <template v-slot:[`item.price`]="{ item }">
            <v-text-field
              v-model="editedItem.price"
              :hide-details="true"
              dense
              single-line
              v-if="item._id === editedItem._id"
            ></v-text-field>
            <span v-else>{{ item.price }}</span>
          </template>

          <template v-slot:[`item.title`]="{ item }">
            <v-text-field
              v-model="editedItem.title"
              :hide-details="true"
              dense
              single-line
              v-if="item._id === editedItem._id"
            ></v-text-field>
            <span v-else>{{ item.title }}</span>
          </template>

          <template v-slot:[`item.description`]="{ item }">
            <v-text-field
              v-model="editedItem.description"
              :hide-details="true"
              dense
              single-line
              v-if="item._id === editedItem._id"
            ></v-text-field>
            <span v-else>{{ item.description }}</span>
          </template>

          <template v-slot:[`item.location`]="{ item }">
            <v-text-field
              v-model="editedItem.location"
              :hide-details="true"
              dense
              single-line
              v-if="item._id === editedItem._id"
            ></v-text-field>
            <span v-else>{{ item.location }}</span>
          </template>

          <template v-slot:[`item.actions`]="{ item }">
            <div v-if="item._id === editedItem._id">
              <v-icon color="red" class="mr-3" @click="close" title="Cancel">
                mdi-window-close
              </v-icon>
              <v-icon color="green" @click="save" title="Save">
                mdi-content-save
              </v-icon>
            </div>
            <div v-else>
              <v-icon
                color="green"
                class="mr-3"
                @click="editItem(item)"
                title="Edit"
              >
                mdi-pencil
              </v-icon>
              <v-icon color="red" @click="deleteItem(item)" title="Delete">
                mdi-delete
              </v-icon>
            </div>
          </template>
          <template v-slot:no-data>
            <v-btn color="primary" @click="getAllPosts">Reset</v-btn>
          </template>
        </v-data-table>
      </v-card>
    </v-row>
  </v-container>
</template>

<script>
import MessageList from "./MessageList.vue";
const axios = require("axios");
const formData = require("form-data");
const apiPosts = "https://api-posts-jeric.netlify.app/.netlify/functions/api/";
const apiUsers = "https://api-users-jeric.netlify.app/.netlify/functions/api/";
const apiMessages =
  "https://api-messages-jeric.netlify.app/.netlify/functions/api";

import { inject } from "vue";
export default {
  setup() {
    const store = inject("store");

    return {
      store,
    };
  },

  components: { MessageList },
  name: "PostList",
  data: () => ({
    postMessages: [],
    reloadMsg: 0,
    allMessages: [],
    msglist: [],
    msgFormValues: {
      user_id: "",
      post_id: "",
      message: "",
      messageimage: "",
    },
    buySell: "",
    postsData: [],
    sellerName: "",
    usersNames: [],
    details: {
      post_id: "",
      productimage: "",
      price: "",
      title: "",
      description: "",
      location: "",
      user_id: "",
    },
    detailsDialog: false,
    postsLoading: true,
    msg: "",
    add: true,
    search: "",
    loading: true,
    showPassword: false,
    posts: [],
    userPosts: [],
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
      productimage: "",
      price: "",
      title: "",
      description: "",
      location: "",
      user_id: "",
    },
    editedIndex: -1,
    editedItem: {
      productimage: "",
      price: "",
      title: "",
      description: "",
      location: "",
      user_id: "",
    },
    defaultItem: {
      productimage: "",
      price: "",
      title: "",
      description: "",
      location: "",
      user_id: "",
    },
  }),
  watch: {
    detailsDialog(visible) {
      if (visible) {
        var delayInMilliseconds = 100;

        setTimeout(function () {
          //your code to be executed after 1 second
          document.querySelector(
            "div.v-text-field__details > div > div"
          ).innerHTML =
            "Don't share your email, phone or financial information.";
        }, delayInMilliseconds);
      }
    },
  },
  methods: {
    sendMsgToSeller(post_id) {
      // close the form
      //this.detailsDialog = false;

      // set data
      this.msgFormValues.post_id = post_id;
      this.msgFormValues.user_id = this.formValues.user_id;

      // push the new message
      //this.msglist.push(this.msgFormValues);

      //save to message database
      fetch(apiMessages, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.msgFormValues),
      })
        .then((response) => response.text())
        .then((data) => {
          console.log(data);
          this.getAllMessages(); // refresh all message list
          this.reloadMsg++; // changing the key to force component reload
          this.detailsDialog = false;
        })
        .catch((err) => {
          if (err) throw err;
        });
    },
    appendMessageToSeller(msg) {
      this.msgFormValues.message += msg;
      document.getElementById("messageToSeller").focus();
    },
    showImage(imageUrls) {
      let urls = "";
      if (imageUrls) {
        const imgArr = imageUrls.split(",");
        imgArr.forEach((element) => {
          urls += `<img src="${element}" width="50">`;
        });
      }
      return urls;
    },
    getAllPosts() {
      fetch(apiPosts)
        .then((response) => response.json())
        .then((data) => {
          // set posts data
          this.posts = data;

          // get user posts
          if (localStorage.userId) {
            let postData = [];
            this.posts.forEach((element) => {
              if (localStorage.userId == element.user_id) {
                postData.push(element);
              }
            });
            this.userPosts = postData;
          }

          // group post by post_id
          data.forEach((post) => {
            this.postsData[post._id] = post;
          });

          this.postsLoading = false;
          this.loading = false;
        })
        .catch((err) => {
          if (err) throw err;
        });
    },
    editItem(item) {
      this.editedIndex = this.userPosts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.id = item._id;
      this.add = false;
    },
    deleteItem(item) {
      if (confirm("Are you sure you want to delete this item?")) {
        const index = this.userPosts.indexOf(item);
        this.userPosts.splice(index, 1);
        fetch(apiPosts + item._id, {
          method: "DELETE",
        })
          .then((response) => response.text())
          .then((data) => {
            this.msg = data;
            this.getAllPosts();
          })
          .catch((err) => {
            if (err) throw err;
          });
        console.log("confirm delete");
      } else {
        console.log("cancel delete");
      }
    },
    close() {
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
        if (this.add) {
          this.userPosts.shift(this.editedItem);
          this.editItem(this.editedItem);
        }
      }, 300);
    },
    addNew() {
      this.store.state.showLogin = true;
      const addObj = Object.assign({}, this.defaultItem);
      this.userPosts.unshift(addObj);
      this.editItem(addObj);
      this.add = true;
    },
    save() {
      if (this.editedIndex > -1) {
        this.formValues.price = this.editedItem.price;
        this.formValues.title = this.editedItem.title;
        this.formValues.description = this.editedItem.description;
        this.formValues.location = this.editedItem.location;

        let fetchApi = this.id ? apiPosts + this.id : apiPosts;
        let fetchMethod = this.id ? "PUT" : "POST";
        fetch(fetchApi, {
          method: fetchMethod,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.formValues),
        })
          .then((response) => response.text())
          .then((data) => {
            this.msg = data;
            this.getAllPosts();
          })
          .catch((err) => {
            if (err) throw err;
          });
      }
      this.close();
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
          header: { "Content-Type": "multipart/form-data" },
        })
          .then((response) => {
            console.log("API response ↓");
            console.log(response);
            console.log(response.data.data.url); // image url
            this.editedItem.productimage = response.data.data.url; // assign to data property
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

    getMessages(post_id) {
      this.msglist = [];
      if (post_id) {
        let singlePost = [];
        this.allMessages.forEach((msg) => {
          if (msg.post_id == post_id) {
            singlePost.push(msg);
          }
        });
        this.msglist = singlePost;
      }
    },
    showDetails(post_id) {
      // clear message to seller
      this.msgFormValues.message = "";

      this.getMessages(post_id);

      let data = this.postsData[post_id];
      this.details.post_id = post_id;
      this.details.productimage = data.productimage;
      this.details.price = data.price;
      this.details.title = data.title;
      this.details.description = data.description;
      this.details.location = data.location;
      this.details.user_id = data.user_id;

      this.sellerName = this.usersNames[data.user_id];
      this.detailsDialog = true;
    },
    detailClose() {
      this.detailsDialog = false;
    },
    getAllMessages() {
      fetch(apiMessages)
        .then((response) => response.json())
        .then((data) => {
          // all messages
          this.allMessages = data;

          // grouping message by post_id
          this.postMessages = this.allMessages.reduce((results, msg) => {
            results[msg.post_id] = results[msg.post_id] || [];
            results[msg.post_id].push(msg);
            return results;
          }, {});
        })
        .catch((err) => {
          if (err) throw err;
        });
    },
    getAllUsers() {
      fetch(apiUsers, {
        method: "GET",
      })
        .then((response) => response.json())
        .then((data) => {
          data.forEach((user) => {
            this.usersNames[user._id] =
              user.firstname.charAt(0).toUpperCase() +
              user.firstname.slice(1) +
              " " +
              user.lastname.charAt(0).toUpperCase() +
              user.lastname.slice(1);
          });
        })
        .catch((err) => {
          if (err) throw err;
        });
    },
  },
  mounted() {
    // get all users
    this.getAllUsers();

    // call get all post
    this.getAllPosts();

    // get all messages
    this.getAllMessages();

    // set user_id
    if (localStorage.userId) {
      this.formValues.user_id = localStorage.userId;
    }
  },
};
</script>

<style scoped>
#card {
  cursor: pointer;
}

/* striped */
tbody tr:nth-of-type(odd) {
  background-color: rgba(0, 0, 0, 0.05);
}

.v-btn:not(.v-btn--round).v-size--default {
  height: auto;
}

.w-100 {
  width: 100%;
}
</style>
