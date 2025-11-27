<script setup>
import { ref, watch , onMounted } from "vue";

const props = defineProps({
    selectedFace: String
});
const emit = defineEmits(["update:selectedFace"]);

const faces = [
    new URL("@/assets/face1.png", import.meta.url).href,
    new URL("@/assets/face2.png", import.meta.url).href,
    new URL("@/assets/face3.png", import.meta.url).href,
];

const currentIndex = ref(0);

function nextFace() {
    currentIndex.value = (currentIndex.value + 1) % faces.length;
    emit("update:selectedFace", faces[currentIndex.value]);
}

function prevFace() {
    currentIndex.value = (currentIndex.value - 1 + faces.length) % faces.length;
    emit("update:selectedFace", faces[currentIndex.value]);
}

watch(() => props.selectedFace, (val) => {
    currentIndex.value = faces.indexOf(val);
});

onMounted(() => {
    emit("update:selectedFace", faces[currentIndex.value]);
});
</script>
<template>
    <div class="face-selector"> 
        <button class="arrow" @click="prevFace">◀</button>
        <img class="face-preview" :src="props.selectedFace || faces[0]" alt="face"/>
        <button class="arrow" @click="nextFace">▶</button>
    </div>
</template>

<style >
.face-selector{
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 30px;
}

.arrow{
    font-size: 35px;
    padding: 10px 20px;
    background: #222;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}
.face-preview {
    width: 120px;
    height: 120px;
    margin: 0 25px;
    border: 4px solid #444;
    border-radius: 10px;
}

</style>