<template>
<div>
    <p>{{ roundMessage1 }}</p>
    <p>{{ roundMessage2 }}</p>
    <ScoreContainer 
        :players="players" 
        flexDirection="column"
    />
    <button @click="nextRound">{{ buttonMessage }}</button>
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

const roundMessage1 = ref('');
const roundMessage2 = ref('');
const buttonMessage = ref('');

function nextRound() {
    emit('next-round');
}

onMounted(() => {
    roundMessage1.value = `End of Round ${props.round}`;
    roundMessage2.value = props.round == 2 ? 'Final Scores:' : 'Current Scores:';
    buttonMessage.value = props.round == 2 ? 'Restart' : 'Next Round';
})

</script>