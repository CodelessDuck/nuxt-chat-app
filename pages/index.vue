<template>
  <v-layout column justify-center align-center class="grey darken-3">
    <v-flex xs12 sm8>
      <v-card raised min-width="300" color="grey darken-2">
        <v-snackbar v-model="snackbar" :timeout="3000" top>
          {{ message }}
          <v-btn dark text @click="snackbar = false">Schließen</v-btn>
        </v-snackbar>

        <v-card-title>
          <h1>Spiel erstellen</h1>
        </v-card-title>
        <v-card-text>
          <v-form
            ref="form"
            v-model="valid"
            lazy-validation
            @submit.prevent="submit"
          >
            <v-text-field
              v-model="name"
              :counter="16"
              :rules="nameRules"
              label="Name"
              required
            ></v-text-field>
            <v-text-field
              v-model="room"
              :rules="roomRules"
              label="Spielzimmer"
              required
            ></v-text-field>
            <v-btn :disabled="!valid" color="primary" class="mr-4" type="submit"
              >Zimmer betreten</v-btn
            >
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";

export default {
  name: "index",
  layout: "login",
  head: {
    title: "Startseite"
  },
  data: () => ({
    valid: true,
    name: "",
    message: "",
    id: null,
    nameRules: [
      v => !!v || "Bitte tragen einen Namen ein",
      v => (v && v.length <= 16) || "Bitte höchstens 16 Zeichen"
    ],
    room: "",
    roomRules: [v => !!v || "Dein Spielzimmer braucht einen Namen"],
    snackbar: false
  }),
  mounted() {
    const { message } = this.$route.query;
    if (message === "noUser") {
      this.message = "Gib Namen und Spielzimmer an";
    } else if (message === "leftChat") {
      this.message = "Spiel verlassen";
    }
    this.snackbar = !!this.message;
  },

  methods: {
    ...mapMutations(["setUser"]),
    submit() {
      if (this.$refs.form.validate()) {
        const user = {
          name: this.name,
          room: this.room,
          id: 0,
          score: 0,
          hasTurn: false,
          finishedWaiting: false
        };
        this.$socket.emit("checkRoom", user, data => {
          console.log(data + " users in room");
          if (data <= 3) {
            this.$socket.emit("createUser", user, data => {
              user.id = data.id;
              this.setUser(user);
              // this.$router.push("/memory");
              this.$router.push({
                name: "room",
                params: { room: this.room }
              });
            });
          } else {
            this.message = "Der Raum ist schon voll";
            this.snackbar = true;
          }
        });
      }
    }
  }
};
</script>
