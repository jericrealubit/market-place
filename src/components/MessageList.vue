<template>
  <div>
    <div>Message List Here</div>

    <v-card class="mx-auto" max-width="400" tile>
      <v-list-item v-for="(msg, i) in msglist" :key="i">
        <v-list-item-content>
          <v-list-item-title>aa{{ msg.user_id }}</v-list-item-title>
          <v-list-item-subtitle>bb{{ msg.message }}</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-card>
  </div>
</template>

<script>
  const apiMessages =
    "https://api-messages-jeric.netlify.app/.netlify/functions/api";

  export default {
    name: "MessageList",

    data: () => ({
      msglist: [],
    }),
    methods: {
      getAll() {
        fetch(apiMessages)
          .then((response) => response.json())
          .then((data) => {
            this.msglist = data;
            console.log("get all messages");
          })
          .catch((err) => {
            if (err) throw err;
          });
      },
    },
  };
</script>
