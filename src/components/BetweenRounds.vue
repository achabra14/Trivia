<template>
<div>
    <h2>Round 1 Over!</h2>
    <h2>Current Standings:</h2>
    <ScoreContainer 
        :players="players" 
        flexDirection="column"
    />
</div>
</template>

<script setup>
import ScoreContainer from './ScoreContainer.vue';

const props = defineProps({
    round: Number,
    players: Array
})

</script>