<template>
  <v-container>
    <v-banner app sticky color="grey darken-3">
      <v-badge
        v-for="(user, index) in users"
        :key="`user-chips-${index}`"
        bottom
        overlap
        class="mx-2"
        color="orange"
        :content="user.score"
      >
        <v-chip :color="user.hasTurn ? 'green' : 'grey'">{{
          user.name
        }}</v-chip></v-badge
      >
    </v-banner>
    <v-snackbar v-model="snackbar" :timeout="3000" top>
      {{ message }}
      <v-btn dark text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
    <v-row justify="center">
      <v-col cols="12">
        <v-card>
          <v-btn light color="primary" block v-on:click="createGame()"
            >Spiel starten!</v-btn
          >

          <v-card-subtitle v-if="hasTurn">Du bist dran!</v-card-subtitle>
        </v-card>
        <!--
        <Message
          v-for="(message, index) in messages"
          :key="`message-${index}`"
          :name="message.name"
          :text="message.text"
          :time="message.time"
          :owner="message.id === user.id"
        />-->
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-card color="grey darken-3">
          <v-container fluid>
            <v-row>
              <v-col
                v-for="(card, index) in cards"
                :key="`card-${index}`"
                class="d-flex child-flex"
                cols="3"
              >
                <v-card flat tile class="d-flex">
                  <v-img
                    :src="card.img"
                    aspect-ratio="1"
                    class="grey lighten-2"
                    v-on:click="flipCard(card)"
                    :class="card.found ? 'found' : ''"
                  >
                  </v-img>
                </v-card>
              </v-col>
            </v-row>
          </v-container>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { mapState, mapMutations } from "vuex";
import Message from "@/components/Message";
// import { Memory } from "@/utils/memory.js";

//const Memory = require("@/utils/memory");

export default {
  name: "memory",
  layout: "default",
  head: {
    title: "Nuxt-chat-memory"
  },
  components: {
    Message
  },
  data() {
    return {
      snackbar: false,
      message: ""
    };
  },
  computed: {
    ...mapState([
      "user",
      "users",
      "hasTurnUserId",
      "cards",
      "cardsChosen",
      "messages"
    ]),
    hasTurn() {
      return this.user.hasTurn;
    }
  },
  mounted() {},

  methods: {
    ...mapMutations(["nextUser"]),
    createGame() {
      console.log("los geklickt");
      /*
      this.Memory = new Memory(grid, result);
      this.Memory.createBoard();
      */
      this.$socket.emit("createGame", this.user, 12);
      //createBoard(grid);
    },
    flipCard(card) {
      if (!this.hasTurn) {
        this.message = "Du bist nicht dran";
        this.snackbar = true;
        return;
      }
      console.log(card);
      if (this.cardsChosen.length < 2) {
        this.$socket.emit("flipCard", this.user, card, data => {
          console.log("callback flipcard");
          if (this.cardsChosen.length === 2) {
            console.log("time to check for a match");
            setTimeout(() => {
              this.$socket.emit("checkForMatch", this.user);
            }, 1000);
          }
        });
      }
    }
  }
};
</script>

<style>
.grid {
  display: flex;
  flex-wrap: wrap;
  height: 300px;
  width: 400px;
}

.grid img {
  height: 100px;
  width: 100px;
}

.found {
  filter: brightness(50%);
}

.v-banner__content {
  padding-bottom: 8px;
}
</style>
