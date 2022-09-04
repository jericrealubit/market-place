<template>
  <v-container>
    <v-dialog v-model="dialog" persistent max-width="600px" min-width="360px">
    <div>
      <v-tabs v-model="tab" show-arrows background-color="blue accent-4" icons-and-text dark grow>
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
              <v-text-field v-model="loginFormValue.loginEmail" :rules="loginEmailRules" label="E-mail" required></v-text-field>
              </v-col>
              <v-col cols="12">
              <v-text-field
                v-model="loginFormValue.loginPassword"
                :append-icon="show1?'eye':'eye-off'"
                :rules="[rules.required, rules.min]"
                :type="show1 ? 'text' : 'password'"
                name="input-10-1" label="Password"
                hint="At least 8 characters"
                counter
                @click:append="show1 = !show1"
              ></v-text-field>
              </v-col>
              <v-col class="d-flex" cols="12" sm="6" xsm="12">
              </v-col>
              <v-spacer></v-spacer>
              <v-col class="d-flex" cols="12" sm="3" xsm="12" align-end>
              <v-btn x-large block :disabled="!isFormValid" color="success" @click="login"> Login </v-btn>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>
        </v-card>
      </v-tab-item>
      <v-tab-item>
        <v-card class="px-4">
        <v-card-text>
          <v-form ref="registerForm" v-model="isFormValid" lazy-validation>
            <v-row>
              <v-col cols="12" sm="6" md="6">
              <v-text-field v-model="formValues.firstName" :rules="[rules.required]" label="First Name" maxlength="20" required></v-text-field>
              </v-col>
              <v-col cols="12" sm="6" md="6">
              <v-text-field v-model="formValues.lastName" :rules="[rules.required]" label="Last Name" maxlength="20" required></v-text-field>
              </v-col>
              <v-col cols="12">
              <v-text-field v-model="formValues.email" :rules="emailRules" label="E-mail" required></v-text-field>
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
              ></v-text-field>
              </v-col>
              <v-spacer></v-spacer>
              <v-col class="d-flex ml-auto" cols="12" sm="3" xsm="12">
              <v-btn x-large block :disabled="!isFormValid" color="success" @click="register">Register</v-btn>
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
  export default {
    name: 'UserLogin',
    data: () => ({
      loggedUser: "",
      users: [],
      dialog: true,
      tab: 0,
      tabs: [
        {name:"Login", icon:"mdi-account"},
        {name:"Register", icon:"mdi-account-outline"}
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
        v => !!v || "Required",
        v => /.+@.+\..+/.test(v) || "E-mail must be valid"
      ],
      emailRules: [
        v => !!v || "Required",
        v => /.+@.+\..+/.test(v) || "E-mail must be valid"
      ],

      show1: false,
      rules: {
        required: value => !!value || "Required.",
        min: v => (v && v.length >= 8) || "Min 8 characters"
      }
    }),
    computed: {
      passwordMatch() {
        return () => this.formValues.password === this.verify || "Password must match";
      }
    },
    methods: {
      register() {
        let validform = this.$refs.registerForm.validate();
        if (validform) {
            fetch(apiUsers, {
                method: "POST",
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(this.formValues)
            })
            .then((response) => response.text())
            .then((data) => {
                console.log(data)
                //this.loggedInUser = data.firstName // emet this
                this.dialog = false;
            })
            .catch((err) => {
                if (err) throw err;
            })
        }
      },
      login() {
        let validform = this.$refs.loginForm.validate();
        if (validform) {
            // verify login details
            this.users.forEach(element => {
            if (element.email == this.loginFormValue.loginEmail
                && element.password == this.loginFormValue.loginPassword
            ) {
                this.loggedUser = element.firstname + " " + element.lastname;
                console.log(this.loggedUser)
            }
            });
            if (this.loggedUser) {
                console.log("login successful")
                this.dialog = false;
            } else {
                console.log("login failed")
            }
        }
      }
    },
    mounted() {
      // get all users
      fetch(apiUsers)
        .then((response) => response.json())
        .then((data) => {
          this.users = data
        })
        .catch((err) => {
          if (err) throw err;
        })
    }
  }
</script>