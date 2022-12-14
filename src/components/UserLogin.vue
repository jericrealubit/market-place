<template>
  <v-container>
    <v-dialog v-model="dialog" persistent max-width="600px" min-width="360px">
      <div>
        <v-tabs
          v-model="tab"
          show-arrows
          background-color="blue accent-4"
          icons-and-text
          dark
          grow
        >
          <v-tabs-slider color="blue darken-4"></v-tabs-slider>
          <v-tab v-for="(tab, i) in tabs" :key="i">
            <v-icon large>{{ tab.icon }}</v-icon>
            <div class="caption py-1">{{ tab.name }}</div>
          </v-tab>
          <v-tab-item>
            <v-card class="px-4">
              <v-card-text>
                <v-form ref="loginForm" v-model="isFormValid" lazy-validation>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        autofocus
                        v-model="loginFormValue.loginEmail"
                        :rules="loginEmailRules"
                        label="E-mail"
                        required
                        @focus="hideLoginErrorMsg"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="loginFormValue.loginPassword"
                        :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                        :rules="[rules.required, rules.min]"
                        :type="show1 ? 'text' : 'password'"
                        name="input-10-1"
                        label="Password"
                        hint="At least 8 characters"
                        counter
                        @click:append="show1 = !show1"
                        @keyup.enter="login"
                        @focus="hideLoginErrorMsg"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      class="d-flex"
                      cols="12"
                      sm="6"
                      xsm="12"
                      style="color: red"
                      ><div v-if="isLoginError">{{ loginErrorMsg }}</div></v-col
                    >
                    <v-spacer></v-spacer>
                    <v-col class="d-flex" cols="12" sm="3" xsm="12" align-end>
                      <v-btn
                        x-large
                        block
                        :disabled="!isFormValid"
                        color="success"
                        @click="login"
                      >
                        Login
                      </v-btn>
                    </v-col>
                  </v-row>
                </v-form>
              </v-card-text>
            </v-card>
          </v-tab-item>
          <v-tab-item>
            <v-card class="px-4">
              <v-card-text>
                <v-form
                  ref="registerForm"
                  v-model="isFormValid"
                  lazy-validation
                >
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        autofocus
                        v-model="formValues.firstname"
                        :rules="[rules.required]"
                        label="First Name"
                        maxlength="20"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="formValues.lastname"
                        :rules="[rules.required]"
                        label="Last Name"
                        maxlength="20"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        :value="formValues.email"
                        @input="textToLower"
                        :rules="emailRules"
                        label="E-mail"
                        required
                        @focus="hideEmailErrorMsg"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="formValues.password"
                        :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                        :rules="[rules.required, rules.min]"
                        :type="show1 ? 'text' : 'password'"
                        name="input-10-1"
                        label="Password"
                        hint="At least 8 characters"
                        counter
                        @click:append="show1 = !show1"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        block
                        v-model="verify"
                        :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                        :rules="[rules.required, passwordMatch]"
                        :type="show1 ? 'text' : 'password'"
                        name="input-10-1"
                        label="Confirm Password"
                        counter
                        @click:append="show1 = !show1"
                        @keyup.enter="register"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      class="d-flex"
                      cols="12"
                      sm="6"
                      xsm="12"
                      style="color: red"
                      ><div v-if="isEmailError">{{ emailErrorMsg }}</div></v-col
                    >
                    <v-spacer></v-spacer>
                    <v-col class="d-flex ml-auto" cols="12" sm="3" xsm="12">
                      <v-btn
                        x-large
                        block
                        :disabled="!isFormValid"
                        color="success"
                        @click="register"
                        >Register</v-btn
                      >
                    </v-col>
                  </v-row>
                </v-form>
              </v-card-text>
            </v-card>
          </v-tab-item>
        </v-tabs>
      </div>
    </v-dialog>
  </v-container>
</template>

<script>
const apiUsers = "https://api-users-jeric.netlify.app/.netlify/functions/api";

import { inject } from "vue";
export default {
  setup() {
    const store = inject("store");

    return {
      store,
    };
  },

  name: "UserLogin",
  data: () => ({
    isEmailError: false,
    emailErrorMsg: "Email is already taken",
    isLoginError: false,
    loginErrorMsg: "Email or Password does not match",
    loggedUser: "",
    users: [],
    dialog: true,
    tab: 0,
    tabs: [
      { name: "Login", icon: "mdi-account" },
      { name: "Register", icon: "mdi-account-outline" },
    ],
    isFormValid: true,
    formValues: {
      firstname: "",
      lastname: "",
      email: "",
      password: "",
    },
    loginFormValue: {
      loginEmail: "",
      loginPassword: "",
    },
    verify: "",
    loginEmailRules: [
      (v) => !!v || "Required",
      (v) => /.+@.+\..+/.test(v) || "E-mail must be valid",
    ],
    emailRules: [
      (v) => !!v || "Required",
      (v) => /.+@.+\..+/.test(v) || "E-mail must be valid",
    ],

    show1: false,
    rules: {
      required: (value) => !!value || "Required.",
      min: (v) => (v && v.length >= 8) || "Min 8 characters",
    },
  }),
  computed: {
    passwordMatch() {
      return () =>
        this.formValues.password === this.verify || "Password must match";
    },
  },
  methods: {
    textToLower(val) {
      this.formValues.email = val.toLowerCase();
    },
    register() {
      let validform = this.$refs.registerForm.validate();
      if (validform) {
        // check email from database
        this.users.forEach((element) => {
          if (element.email == this.formValues.email.toLocaleLowerCase()) {
            this.isEmailError = true;
          }
        });

        if (!this.isEmailError) {
          fetch(apiUsers, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(this.formValues),
          })
            .then((response) => response.text())
            .then((data) => {
              console.log(data);
              this.dialog = false;
              this.loggedUser =
                this.formValues.firstname + " " + this.formValues.lastname;
              // localStorage
              localStorage.userId = data; // inserted document id
              localStorage.loggedUser = this.loggedUser;
              this.$emit("logged-user", this.loggedUser);
              document.location.reload(true); // force page reload to show admin table
            })
            .catch((err) => {
              if (err) throw err;
            });
        }
      }
    },
    login() {
      // let validform = this.$refs.loginForm.validate();
      // if (validform) {
      // verify login details
      this.users.forEach((users) => {
        if (
          users.email == this.loginFormValue.loginEmail.toLowerCase() &&
          users.password == this.loginFormValue.loginPassword
        ) {
          this.loggedUser = users.firstname + " " + users.lastname;
          // localStorage
          localStorage.userId = users._id;
          localStorage.loggedUser = this.loggedUser;
        }
      });
      if (this.loggedUser) {
        console.log("login successful");
        this.dialog = false;
        this.$emit("logged-user", this.loggedUser);
        document.location.reload(true); // force page reload to show admin table
      } else {
        console.log("login failed");
        this.isLoginError = true;
      }
      // }
    },
    hideLoginErrorMsg() {
      this.isLoginError = false;
    },
    hideEmailErrorMsg() {
      this.isEmailError = false;
    },
  },
  mounted() {
    // check if user is logged-in, do not show login form
    if (localStorage.loggedUser) {
      this.dialog = false;
    }

    // get all users
    fetch(apiUsers)
      .then((response) => response.json())
      .then((data) => {
        this.users = data;
      })
      .catch((err) => {
        if (err) throw err;
      });
  },
};
</script>
