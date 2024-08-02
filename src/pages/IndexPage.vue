<template>
  <q-page class="q-pa-xs">
    <q-card class="" flat>
      <q-card-section class="text-center">
        <div class="text-h6">Jogo da Velha - Julian e Angelina</div>
      </q-card-section>

      <q-card-section>
        <div class="row justify-center relative-positon">
          <!-- div class="absolute fixed-center bg-black fit" style="max-width: 250px;max-height: 250px"></div-->
          <q-btn
            v-for="(cell, index) in board"
            :key="index"
            :label="cell"
            text-color="primary"
            class="col-4 q-ma-xs"
            color="white"
            @click="makeMove(index)"
            :disable="cell !== '' || gameOver"
          />
        </div>
      </q-card-section>

      <q-card-section v-if="winner || gameOver" class="text-center col-12 text-primary">
        <div v-if="winner" class="q-mb-md handwritten text-h6"><a class="text-bold">{{ winner }}</a> venceu!</div>
        <div v-else-if="gameOver" class="handwrittenmb-md text-h6">Empate!</div>
        <q-btn label="Reiniciar" color="primary" class="fit handwritten" @click="resetGame"/>
      </q-card-section>

      <q-card-section class="text-center">
        <q-btn-toggle
          v-model="mode"
          push
          glossy
          toggle-color="primary"
          :options="[{label: 'Dois Jogadores', value: 'two-player'}, {label: 'Contra Máquina', value: 'vs-computer'}]"
          class="q-ma-sm"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref } from 'vue';

const board = ref(Array(9).fill(''));
const currentPlayer = ref('X');
const winner = ref(null);
const gameOver = ref(false);
const mode = ref('vs-computer');

const winningCombinations = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6]
];

const makeMove = (index) => {
  if (!board.value[index] && !gameOver.value) {
    board.value[index] = currentPlayer.value;
    if (checkWinner()) {
      winner.value = currentPlayer.value;
      gameOver.value = true;
    } else if (board.value.every(cell => cell !== '')) {
      gameOver.value = true;
    } else {
      currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X';
      if (mode.value === 'vs-computer' && currentPlayer.value === 'O' && !gameOver.value) {
        setTimeout(computerMove, 500);
      }
    }
  }
};

const computerMove = () => {
  let availableIndices = board.value.map((cell, index) => cell === '' ? index : null).filter(val => val !== null);

  const findWinningMove = (player) => {
    for (let combination of winningCombinations) {
      const [a, b, c] = combination;
      const cells = [board.value[a], board.value[b], board.value[c]];
      if (cells.filter(cell => cell === player).length === 2 && cells.includes('')) {
        return combination[cells.indexOf('')];
      }
    }
    return null;
  };

  // Check if computer can win
  let winningMove = findWinningMove('O');
  if (winningMove !== null) {
    makeMove(winningMove);
    return;
  }

  // Check if player needs to be blocked
  let blockingMove = findWinningMove('X');
  if (blockingMove !== null) {
    makeMove(blockingMove);
    return;
  }

  // Otherwise, make a random move
  let randomIndex = availableIndices[Math.floor(Math.random() * availableIndices.length)];
  makeMove(randomIndex);
};

const checkWinner = () => {
  return winningCombinations.some(combination =>
    combination.every(index => board.value[index] === currentPlayer.value)
  );
};

const resetGame = () => {
  board.value = Array(9).fill('');
  currentPlayer.value = 'X';
  winner.value = null;
  gameOver.value = false;
};
</script>

<style scoped>
.q-page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.q-card {
  max-width: 400px;
  width: 100%;
}
.q-btn {
  width: 90px;
  height: 90px;
  font-size: 2em;
}
.handwritten {
  font-family: 'Patrick Hand', cursive;
  color: #1a1a1a;
}

</style>
