<script setup>
import { ref, nextTick, onMounted, watch, onBeforeUnmount } from "vue";
import FaceSelector from "./components/FaceSelector.vue";
import DifficultySelector from "./components/DifficultySelector.vue";
import SnakeGame from "./components/SnakeGame.vue";
import MusicSelector from "./components/MusicSelector.vue";

const selectedFace = ref(null);
const selectedDifficulty = ref("Facile");
const gameStarted = ref(false);
const snakeGameRef = ref(null);
const bestScore = ref(0);

function updateBestScore() {
  const score = parseInt(localStorage.getItem("bestScore") || "0", 10);
  bestScore.value = isNaN(score) ? 0 : score;
}

function startGame(){
  if(!selectedFace.value){
    alert("Seleziona una faccia");
    return;
  }
  gameStarted.value = true;
  nextTick(() => {
    if (snakeGameRef.value && snakeGameRef.value.initGame) {
      snakeGameRef.value.initGame();
    }
  });
}

function goHome(){
  gameStarted.value = false;
  updateBestScore();
}

function preventArrowKeys(e) {
  const arrowKeys = ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'];
  if (arrowKeys.includes(e.key)) {
    e.preventDefault();
  }
}

onMounted(() => {
  updateBestScore();
  // Prevent arrow keys from scrolling the page globally
  window.addEventListener('keydown', preventArrowKeys, { passive: false });
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', preventArrowKeys);
});

watch(gameStarted, (newVal, oldVal) => {
  if (!newVal && oldVal) {
    updateBestScore();
  }
});
</script>

<template>
  <div class="container">
    <h1>Snake Tabo🐵</h1>
    <div v-if="!gameStarted">
      <FaceSelector v-model:selectedFace="selectedFace"/>
      <DifficultySelector v-model:selectedDifficulty="selectedDifficulty"/>
      <p class="best-score">Best score: {{ bestScore }} </p>
      <button class="btn" @click="startGame" :disabled="!selectedFace">Start Game</button>
    </div>

    <SnakeGame
      ref="snakeGameRef"
      v-if="gameStarted"
      :face="selectedFace"
      :difficulty="selectedDifficulty"
      @goHome="goHome"
    />

    <MusicSelector 
      v-show="!gameStarted"
    />
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  background: #111;
  font-family: Arial, sans-serif;
  height: 100vh;
  width: 100vw;
  box-sizing: border-box;
  overflow: hidden !important;
  overscroll-behavior: none;
  touch-action: none !important;
  -ms-touch-action: none !important;
  -webkit-overflow-scrolling: none !important;
  -moz-user-select: none !important;
  -webkit-user-select: none !important;
  user-select: none !important;
  position: fixed !important;
  inset: 0;
}
.container {
  text-align: center;
  color: white;
  max-width: 95vw;
  margin: auto;
  padding: 10px 5vw;
  overflow: hidden !important;
  touch-action: none !important;
  -ms-touch-action: none !important;
  -webkit-user-select: none !important;
  -moz-user-select: none !important;
  user-select: none !important;
  height: 100vh;
  width: 100vw;
  position: relative;
}
.btn {
  height: 8vh;       
  width: 20vw;
  min-width: 120px;
  max-width: 200px;
  font-size: calc(1em + 0.5vw);
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease, color 0.3s ease;
}
.btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}
.btn:hover:not(:disabled),
.btn:focus:not(:disabled) {
  background-color: #f0c040;
  color: #111;
  outline: none;
}
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
  margin-bottom: 20px;
}

.best-score {
  color: #f0c040;
  font-size: clamp(1rem, 3vw, 2rem);
  margin-top: 15px;
  font-weight: bold;
}
</style>
