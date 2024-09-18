<template>
<div>
    <p>{{ roundMessage }}</p>
    <p>Current Scores:</p>
    <ScoreContainer 
        :players="players" 
        flexDirection="column"
    />
    <button @click="nextRound">Next Round</button>
</div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import ScoreContainer from './ScoreContainer.vue';

const props = defineProps({
    round: Number,
    players: Array
})

const emit = defineEmits(['next-round']);

const roundMessage = ref('');

function nextRound() {
    emit('next-round');
}

onMounted(() => {
    roundMessage.value = `End of Round ${props.round}`;
})

</script>