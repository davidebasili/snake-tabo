<script setup>
import { ref, onMounted, watch, nextTick, onBeforeUnmount } from "vue";

const props = defineProps({
    face: String,
    difficulty: {
        type: String,
        default: "Facile"
    }
});

const emit = defineEmits(['goHome']);

const container = ref(null);
const canvas = ref(null);
const ctx = ref(null);

const gridCount = 20;

const canvasSize = ref(400);
const cellSize = ref(canvasSize.value / gridCount);

//snake
let snake = [{ x: 8, y: 8}];
let direction = "RIGHT";
let inputQueue = [];
let food = { x: 5, y: 5};
const score = ref(0);
let interval = null;

const speedMap = { Facile: 200, Medio: 120, Difficile: 80};

let faceImage = null;

const gameOverState = ref(false);

function updateCanvasSize() {
    if (!container.value) return;
    const containerWidth = container.value.clientWidth;
    const containerHeight = window.innerHeight - container.value.getBoundingClientRect().top - 20; // 20px padding
    const size = Math.min(containerWidth, containerHeight);
    canvasSize.value = size;
    cellSize.value = canvasSize.value / gridCount;
    if (canvas.value) {
        canvas.value.width = canvasSize.value;
        canvas.value.height = canvasSize.value;
    }
    draw();
}

function initGame() {
    snake = [{ x: 8, y: 8 }];
    direction = "RIGHT";
    inputQueue = [];
    score.value = 0;
    placeFood();
    gameOverState.value = false;

    clearInterval(interval);
    interval = setInterval(gameLoop, speedMap[props.difficulty]);

    if(props.face){
        faceImage = new Image();
        faceImage.src = props.face;
        faceImage.onload = () => {
            draw();
        };
        faceImage.onerror = () => {
            faceImage = null;
            draw();
        };
    } else {
        faceImage = null;
        draw();
    }
}

defineExpose({ initGame });

function placeFood() {
    food = {
        x: Math.floor(Math.random() * gridCount),
        y: Math.floor(Math.random() * gridCount),
    };   
}

function changeDirection(e) {
    let newDir = null;
    switch (e.key){
        case "ArrowUp":
            newDir = "UP";
            break;
        case "ArrowDown":
            newDir = "DOWN";
            break;
        case "ArrowRight":
            newDir = "RIGHT";
            break;
        case "ArrowLeft":
            newDir = "LEFT";
            break;
    }
    if(newDir){
        // Prevent reversing direction
        const lastDir = inputQueue.length > 0 ? inputQueue[inputQueue.length - 1] : direction;
        if(
            (newDir === "UP" && lastDir !== "DOWN") ||
            (newDir === "DOWN" && lastDir !== "UP") ||
            (newDir === "LEFT" && lastDir !== "RIGHT") ||
            (newDir === "RIGHT" && lastDir !== "LEFT")
        ){
            inputQueue.push(newDir);
        }
    }
}

function gameOver() {
    clearInterval(interval);
    gameOverState.value = true;   
    
    //salva il best score
    const best = parseInt(localStorage.getItem("bestScore") || "0");
    if(score.value > best){
        localStorage.setItem("bestScore", score.value);
    }
}

function gameLoop() {
    if(inputQueue.length > 0){
        direction = inputQueue.shift();
    }

    //calcola nuova testa
    const head = {...snake[0]};
    if(direction === "UP") head.y--;
    if(direction === "DOWN") head.y++;
    if(direction === "LEFT") head.x--;
    if(direction === "RIGHT") head.x++;

    //collisioni con muri 
    if(head.x < 0 || head.x >= gridCount|| head.y <0 || head.y >= gridCount){
        gameOver();
        return;
    }

    //collisioni con se stesso 
    if(snake.some(seg => seg.x === head.x && seg.y === head.y)) {
        gameOver();
        return;
    }

    snake.unshift(head);

    //mangia cibo
    if(head.x === food.x && head.y === food.y){
        score.value++;
        placeFood();
    }else{
        snake.pop();
    }
    draw();
}

function draw() {
    if (!ctx.value) return;
    ctx.value.clearRect(0, 0, canvasSize.value, canvasSize.value);

    //cibo
    ctx.value.fillStyle = "red";
    ctx.value.fillRect(food.x * cellSize.value, food.y * cellSize.value, cellSize.value, cellSize.value);

    //snake body
    ctx.value.fillStyle = "lime";
    for (let i = 1; i < snake.length; i++) {
        const seg = snake[i];
        ctx.value.fillRect(seg.x * cellSize.value, seg.y * cellSize.value, cellSize.value, cellSize.value);
    }

    //snake head
    const head = snake[0];
    if(faceImage){
        ctx.value.drawImage(faceImage, head.x * cellSize.value, head.y * cellSize.value, cellSize.value, cellSize.value);
    } else {
        ctx.value.fillStyle = "lime";
        ctx.value.fillRect(head.x * cellSize.value, head.y * cellSize.value, cellSize.value, cellSize.value);
    }
}

function handleResize() {
    updateCanvasSize();
}

onMounted(() => {
    ctx.value = canvas.value.getContext("2d");
    window.addEventListener("keydown", changeDirection);
    window.addEventListener("resize", handleResize);
    nextTick(() => {
        updateCanvasSize();
    });
});

onBeforeUnmount(() => {
    clearInterval(interval);
    window.removeEventListener("keydown", changeDirection);
    window.removeEventListener("resize", handleResize);
});
</script>
<template>
    <div class="game-container" ref="container">
        <p>Punteggio: {{ score }}</p>
        <canvas ref="canvas" :width="canvasSize" :height="canvasSize"></canvas>
        
        <div v-if="gameOverState" class="overlay">
            <div class="popup">
                <p>Game Over!</p>
                <button @click="$emit('goHome')">Rigioca</button>
            </div>
        </div>
    </div>
</template>
<style>
@media (max-width: 480px) {
    .game-container{
        max-width: 95vw;
    }
    canvas{
        border-width: 3px;
    }
    .btn {
        width: 40vw;
        height: 7vh;
        font-size: 1rem;
    }
    .best-score {
        font-size: 1.2rem;
    }
}
.game-container {
    display: flex;
    flex-direction: column;
    align-items: center;   /* centra orizzontalmente */
    justify-content: center; /* centra verticalmente se vuoi */
    margin: 0 auto;
    width: 100%;
    max-width: 800px;      /* evita che diventi troppo grande su desktop */
    box-sizing: border-box;
}
canvas {
    background-color: #111;
    box-sizing: border-box;
    border: 4px solid #444;
    display: block;
    width: 100%;
    max-width: 100%;
    height: auto;
}
p {
    color: white;
    font-size: 18px;
}
.overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(30, 30, 30, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}
.popup {
    background-color: #222;
    padding: 30px 40px;
    border-radius: 10px;
    text-align: center;
    color: white;
    box-shadow: 0 0 15px rgba(0,0,0,0.7);
}
.popup button {
    margin-top: 15px;
    padding: 8px 16px;
    font-size: 16px;
    cursor: pointer;
    border: none;
    border-radius: 5px;
    background-color: #4caf50;
    color: white;
    transition: background-color 0.3s ease;
}
.popup button:hover {
    background-color: #45a049;
}
</style>