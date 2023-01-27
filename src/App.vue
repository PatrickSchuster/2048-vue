<template>
  <main>
    <div class="game-stats">
      <div v-if="gameState.won">YOU WON 🏆</div>
      <div v-else-if="gameState.lost">YOU LOST 💀</div>
      <div v-else>⬅️ ⬆️ ➡️ ⬇️</div>
    </div>
    <GameBoard @won="onGameWon" @lost="onGameLost"/>
  </main>
</template>

<script setup lang="ts">
import { reactive } from "vue";
import GameBoard from "./components/GameBoard.vue";

let gameState = reactive({
  won: false,
  lost: false
});

function onGameWon() {
  gameState.won = true;
}

function onGameLost() {
  gameState.lost = true;
}

</script>

<style scoped>
main {
  background-color: darkgray;
}

.game-stats {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 5vmin;
  padding: 5vmin;
  color: black;
}
</style>
