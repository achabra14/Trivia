<template>
<div class="score-container" ref="scoreContainerRef">
    <div class="player-score" v-for="(player, index) in sortedPlayers" :key="index">
        <img :src="player.profilePic" alt="Profile Picture" class="profile-pic" />
        <div class="player-info">
            <span class="player-name">{{ player.name }}</span>
            <span class="player-score-value">Score: {{ player.score }}</span>
        </div>
    </div>
</div>
</template>
  
<script setup>
import { computed, ref, onMounted } from 'vue';

const props = defineProps({
    players: Array,
    flexDirection: {
        type: String,
        default: 'row'
    }
});

const sortedPlayers = computed(() => {
    return [...props.players].sort((a, b) => b.score - a.score);
});

const scoreContainerRef = ref(null);

onMounted(() => {
    if (props.flexDirection === 'column') {
        scoreContainerRef.value.classList.add('flex-column');
    }
});


</script>
  
<style scoped>
.score-container {
    display: flex;
    justify-content: space-between;
    width: 100%;
    padding: 10px;
    color: #fff;
    box-sizing: border-box;
}

.flex-column {
  flex-direction: column;
  gap: 10px; /* Add gap for spacing between items */
}

.player-score {
    display: flex;
    align-items: center;
    background-color: #3a3f47;
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    flex: 1;
    flex-basis: 28%;
    flex-shrink: 0;
    flex-grow: 0;
    margin: 0 5px;
    white-space: nowrap; 
}

.profile-pic {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    margin-right: 10px;
    border: 2px solid #fff;
}

.player-info {
    display: flex;
    flex-direction: column;
}

.player-name {
    font-size: 16px;
    font-weight: bold;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 100%;
}

.player-score-value {
    font-size: 14px;
    color: #d1d1d1;
}
</style>
  