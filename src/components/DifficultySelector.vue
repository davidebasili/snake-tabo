<script setup>
import { ref, watch} from "vue";

const props = defineProps({
    selectedDifficulty: String
});
const emit = defineEmits(["update:selectedDifficulty"]);

const difficulties = ["Facile", "Medio", "Difficile"];
const currentDifficulty = ref(difficulties.indexOf(props.selectedDifficulty) !== -1 ? difficulties.indexOf(props.selectedDifficulty) : 0);

watch(currentDifficulty, (newVal) => {
    emit("update:selectedDifficulty", difficulties[newVal]);
});

function nextDifficulty() {
    currentDifficulty.value = (currentDifficulty.value + 1) % difficulties.length;
}

function prevDifficulty() {
    currentDifficulty.value =
    (currentDifficulty.value - 1 + difficulties.length) % difficulties.length;
}
</script>
<template>
    <div class="Difficulty-selector">
        <button class="arrow" @click="prevDifficulty">◀</button>
        <div class="difficulty-preview">{{ difficulties[currentDifficulty] }}</div>
        <button class="arrow" @click="nextDifficulty">▶</button>
    </div>
</template>

<style>
.Difficulty-selector {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 30px;
}
.arrow {
    font-size: 35px;
    padding: 10px 20px;
    background: #222;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}


.difficulty-preview {
    text-align: center;
    padding-top: 25px;
    width: 120px;
    height: 40px;
    margin: 0 25px;
    border: 4px solid #444;
    border-radius: 10px;
}
</style>
