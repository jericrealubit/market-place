<template>
  <div>
    <v-card class="mx-auto" max-width="400" tile>
      <v-card-title>
        <v-icon large left>
          mdi-message-bulleted
        </v-icon>
        <span class="text-h6 font-weight-light">Messages</span>
      </v-card-title>
      <v-list-item v-for="(msg, i) in msglist" :key="i">
        <v-list-item-content>
          <v-list-item-title>{{ usersNames[msg.user_id] }}</v-list-item-title>
          <v-list-item-subtitle>{{ msg.message }}</v-list-item-subtitle>
          <v-divider></v-divider>
        </v-list-item-content>
      </v-list-item>
    </v-card>
  </div>
</template>

<script>
export default {
  name: "MessageList",
  props: {
    msglist: {
      type: Array,
      required: true
    },
    usersNames: {
      type: Array,
      required: true
    }
  },
  data: () => ({}),
};
</script>
