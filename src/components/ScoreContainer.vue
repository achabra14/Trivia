<template>
<div class="score-container">
    <div class="player-score" v-for="(player, index) in players" :key="index">
        <img :src="player.profilePic" alt="Profile Picture" class="profile-pic" />
        <div class="player-info">
            <span class="player-name">{{ player.name }}</span>
            <span class="player-score-value">Score: {{ player.score }}</span>
        </div>
    </div>
</div>
</template>
  
<script setup>

const props = defineProps({
    players: Array
});

</script>
  
<style scoped>
.score-container {
    display: flex;
    justify-content: space-around;
    width: 100%;
    padding: 10px;
    background-color: #f1f1f1;
    color: #fff;
}

.player-score {
    display: flex;
    align-items: center;
    background-color: #3a3f47;
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    flex: 1;
    max-width: 33%;
    margin: 0 10px;
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
}

.player-score-value {
    font-size: 14px;
    color: #d1d1d1;
}
</style>
  