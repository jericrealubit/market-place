<template>
    <v-container>

        <v-card
            class="pa-10"
            elevation="2"
        >
            <!-- Image Input -->
            <v-file-input
                accept="image/*"
                label="File input"
                show-size
                prepend-icon="mdi-camera"
                @change="uploadImage"
            ></v-file-input>
            <h4>{{uploadedImage}}</h4>
            <v-img
                lazy-src="https://picsum.photos/id/11/10/6"
                max-height="150"
                max-width="250"
                :src="uploadedImage"
            ></v-img>

        </v-card>

    </v-container>
</template>

<script>
    const axios = require("axios");
    const formData = require("form-data");
    export default {
      name: 'UploadImage',

      data: () => ({
        uploadedImage: '',
      }),
      methods: {
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
      }
    }
</script>
