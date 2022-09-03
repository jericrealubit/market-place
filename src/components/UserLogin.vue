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
                    <v-form ref="loginForm" v-model="valid" lazy-validation>
                    <v-row>
                        <v-col cols="12">
                        <v-text-field v-model="loginEmail" :rules="loginEmailRules" label="E-mail" required></v-text-field>
                        </v-col>
                        <v-col cols="12">
                        <v-text-field v-model="loginPassword" :append-icon="show1?'eye':'eye-off'" :rules="[rules.required, rules.min]" :type="show1 ? 'text' : 'password'" name="input-10-1" label="Password" hint="At least 8 characters" counter @click:append="show1 = !show1"></v-text-field>
                        </v-col>
                        <v-col class="d-flex" cols="12" sm="6" xsm="12">
                        </v-col>
                        <v-spacer></v-spacer>
                        <v-col class="d-flex" cols="12" sm="3" xsm="12" align-end>
                        <v-btn x-large block :disabled="!valid" color="success" @click="validate"> Login </v-btn>
                        </v-col>
                    </v-row>
                    </v-form>
                </v-card-text>
                </v-card>
            </v-tab-item>
            <v-tab-item>
                <v-card class="px-4">
                <v-card-text>
                    <v-form ref="registerForm" v-model="valid" lazy-validation>
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
                        <v-text-field v-model="formValues.password" :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'" :rules="[rules.required, rules.min]" :type="show1 ? 'text' : 'password'" name="input-10-1" label="Password" hint="At least 8 characters" counter @click:append="show1 = !show1"></v-text-field>
                        </v-col>
                        <v-col cols="12">
                        <v-text-field block v-model="verify" :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'" :rules="[rules.required, passwordMatch]" :type="show1 ? 'text' : 'password'" name="input-10-1" label="Confirm Password" counter @click:append="show1 = !show1"></v-text-field>
                        </v-col>
                        <v-spacer></v-spacer>
                        <v-col class="d-flex ml-auto" cols="12" sm="3" xsm="12">
                        <v-btn x-large block :disabled="!valid" color="success" @click="validate">Register</v-btn>
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
        links: [
            'Home',
            'Login'
        ],
        dialog: true,
        tab: 0,
        tabs: [
            {name:"Login", icon:"mdi-account"},
            {name:"Register", icon:"mdi-account-outline"}
        ],
        valid: true,
        formValues: {
            firstName: "",
            lastName: "",
            email: "",
            password: "",
        },      
        verify: "",
        loginPassword: "",
        loginEmail: "",
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
            register(formData) {
                console.log(formData)

                fetch(apiUsers, {
                    method: "POST",
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(formData)
                })
                .then((response) => response.text())
                .then((data) => {
                    console.log(data)
                })
                .catch((err) => {
                    if (err) throw err;
                })
            },
            validate() {
                if (this.$refs.registerForm.validate()) {
                    //console.log(this.formValues)
                    this.register(this.formValues);
                }
                if (this.$refs.loginForm.validate()) {
                    // submit form to server/API here...
                }
            },
            reset() {
                this.$refs.form.reset();
            },
            resetValidation() {
                this.$refs.form.resetValidation();
            }
        }
    }
</script>