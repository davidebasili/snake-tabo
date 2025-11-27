<script setup>
import { ref } from "vue";

const emit = defineEmits(["update:selectedSong"]);

// Lista delle canzoni
const songs = [
    { name: "come fa1 - thasup", file: new URL('@/assets/music/song1.mp3', import.meta.url).href },
    { name: "Se SipPo La LeaN - yungest Moonstar", file: new URL('@/assets/music/song2.mp3', import.meta.url).href },
    { name: "m12ano - thasup ft.Mara Sattei", file: new URL('@/assets/music/song3.mp3', import.meta.url).href },
    { name: "MOLECOLE SPRITE - Sfera Ebbasta, Shiva", file: new URL('@/assets/music/song4.mp3', import.meta.url).href },

];


const currentSongIndex = ref(0);
const audio = ref(null);
const isPlaying = ref(false);

function setupAudioEvents() {
    if (!audio.value) return;
    audio.value.addEventListener('play', () => {
        isPlaying.value = true;
    });
    audio.value.addEventListener('pause', () => {
        isPlaying.value = false;
    });
    audio.value.addEventListener('ended', () => {
        isPlaying.value = false;
    });
}

function startAudio() {
    audio.value = new Audio(songs[currentSongIndex.value].file);
    audio.value.loop = true;
    audio.value.volume = 0.3;
    setupAudioEvents();
    audio.value.play().catch(() => console.log("Click necessario per riprodurre audio"));
}

function changeSong(index) {
    if (index < 0 || index >= songs.length) return;
    currentSongIndex.value = index;
    if (audio.value) {
        audio.value.pause();
        audio.value = new Audio(songs[index].file);
        audio.value.loop = true;
        audio.value.volume = 0.3;
        setupAudioEvents();
        audio.value.play().catch(() => console.log("Click necessario per riprodurre audio"));
    }
}

function togglePlay() {
    if (!audio.value) {
        startAudio();
    } else if (audio.value.paused) {
        audio.value.play();
    } else {
        audio.value.pause();        
    }
}

function nextSong() {
    currentSongIndex.value = (currentSongIndex.value + 1) % songs.length;
    changeSong(currentSongIndex.value);
    emit("update:selectedSong", songs[currentSongIndex.value]);
}

function prevSong() {
    currentSongIndex.value = (currentSongIndex.value - 1 + songs.length) % songs.length;
    changeSong(currentSongIndex.value);
    emit("update:selectedSong", songs[currentSongIndex.value]);
}
</script>

<template>
    <div class="music-player">
        <p>🎵 Radio</p>
        <p class="current-song-name">{{ songs[currentSongIndex].name }}</p>
        <div class="controls">
            <p>{{ songs.name }}</p>
            <button class="arrowMusic" @click="prevSong">◀</button>
            <button 
                @click="togglePlay" 
            >
                {{ !isPlaying ? "▶" : "⏸" }}
            </button>
            <button class="arrowMusic" @click="nextSong">▶</button>
        </div>
    </div>
</template>

<style>
.music-player {
    width: 90%;
    max-width: 400px;
    background-color: #222;
    border-radius: 12px;
    padding: 20px;
    margin: 50px auto 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: white;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.current-song-name {
    font-size: 18px;
    font-weight: bold;
    margin: 10px 0;
}

.controls {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 15px;
    width: 100%;
}

.arrowMusic {
    font-size: 20px;
    padding: 10px 16px;
    background: #f0c040;
    color: white;
    border: none;
    border-radius: 50%;
    cursor: pointer;
    transition: background-color 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    user-select: none;
}

.arrowMusic:hover:not(.disabled) {
    background-color: #5e5e5e;
}

button {
    font-size: 16px;
    padding: 10px 20px;
    border-radius: 20px;
    border: none;
    cursor: pointer;
    background-color: #282828;
    color: white;
    transition: background-color 0.3s ease;
    user-select: none;
}

button:hover:not(.disabled) {
    background-color: #3a3a3a;
}

button.disabled,
.song-list button.disabled {
    background-color: #2a2a2a !important;
    color: #555 !important;
    cursor: not-allowed;
}

.song-list {
    display: flex;
    gap: 10px;
    overflow-x: auto;
    width: 100%;
    padding-bottom: 5px;
    -webkit-overflow-scrolling: touch;
}

.song-list button {
    flex: 0 0 auto;
    padding: 8px 16px;
    border-radius: 20px;
    background-color: #282828;
    color: white;
    white-space: nowrap;
}

.song-list button.active,
.song-list button:hover:not(.disabled) {
    background-color: #1db954;
    color: white;
}

.song-list::-webkit-scrollbar {
    height: 6px;
}

.song-list::-webkit-scrollbar-thumb {
    background-color: #837f82;
    border-radius: 3px;
}

.song-list::-webkit-scrollbar-track {
    background: transparent;
}
</style>