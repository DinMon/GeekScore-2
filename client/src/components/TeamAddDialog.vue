<template>
  <the-dialog
    activator-icon="plus"
    header="Add new team"
    button-text="New team"
    :submit-logic="onSubmit"
  >
    <v-text-field
      v-model.trim="name"
      clearable
      :rules="nameRules"
      prepend-icon="mdi-account-group"
      label="Name"
    />
    <template v-if="!isCoop">
      <v-select
        prepend-icon="mdi-account-multiple-plus"
        :rules="selectRules"
        :items="numberOfPlayers"
        label="Number of players"
        @change="setPlayers"
      />
      <v-text-field
        v-for="(player, i) in players"
        :key="i"
        v-model.trim="player.name"
        :readonly="isMe(player)"
        :clearable="!isMe(player)"
        prepend-icon="mdi-account"
        :rules="playerRules"
        :label="`Player #${i + 1}`"
        @input="isUniqueName"
      />
      <v-switch v-model="coop" label="Cooperative" color="secondary" hide-details />
    </template>
  </the-dialog>
</template>

<script>
import { requiredField, standardField } from "@/utils/validations";
import { mapActions, mapGetters } from "vuex";

export default {
  props: {
    gameId: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      name: "",
      coop: false,
      players: [{ name: "Me" }],
      nameRules: standardField,
      playerRules: standardField,
      selectRules: [requiredField],
      // Generates number of players from 1 to N
      numberOfPlayers: Array(8)
        .join(0)
        .split(0)
        .map((v, i) => i + 1)
    };
  },
  computed: {
    ...mapGetters("games", ["getGame"]),
    game() {
      return this.getGame(this.gameId);
    },
    isCoop() {
      return this.game.coop || this.coop;
    }
  },
  methods: {
    ...mapActions("teams", ["createTeam"]),
    setPlayers($ev) {
      this.players = [{ name: "Me" }];
      for (let i = 1; i < $ev; i++) {
        let player = { name: "" };
        this.players.push(player);
      }
      if ($ev === 1) this.coop = true;
    },
    isMe(player) {
      return player.name === "Me";
    },
    isUniqueName($ev) {
      let duplicatedPlayerName = this.players.filter(
        player => player.name === $ev
      );
      const isDuplicated =
        duplicatedPlayerName.length < 2 || "This field should be unique";
      this.playerRules = [...this.playerRules, isDuplicated];
    },
    onSubmit() {
      const team = {
        games: [this.gameId],
        gameName: this.game.name,
        name: this.name,
        coop: this.isCoop,
        players: this.players
      };
      this.createTeam(team);
    }
  }
};
</script>
