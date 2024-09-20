<template>
<div>
    <p>{{ roundMessage1 }}</p>
    <p>{{ roundMessage2 }}</p>
    <ScoreContainer 
        :players="players" 
        flexDirection="column"
    />
    <p>{{ roundMessage3 }}</p>
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
const roundMessage3 = ref('');
const buttonMessage = ref('');

function nextRound() {
    emit('next-round');
}

function getUserPlace(players, user) {
    return [...props.players].sort((a, b) => b.score - a.score).findIndex(player => player.name === user.name) + 1;
}

function getPlaceString(place) {
    if (place === 1) {
        return '1st';
    } else if (place === 2) {
        return '2nd';
    } else if (place === 3) {
        return '3rd';
    } else {
        return `${place}th`;
    }
}

onMounted(() => {
    const userPlaceString = getPlaceString(getUserPlace(props.players, props.players[0]));
    roundMessage1.value = `End of Round ${props.round}`;
    roundMessage2.value = props.round == 3 ? 'Final Scores:' : 'Current Scores:';
    roundMessage3.value = props.round == 3 ? `You finished in ${userPlaceString} place`: `You are currently in ${userPlaceString} place`;
    buttonMessage.value = props.round == 3 ? 'Restart' : 'Next Round';
})

</script>