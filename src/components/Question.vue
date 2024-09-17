<template>
    <div class="question">
      <h4>{{ question }}</h4>
      <ul>
        <li 
          v-for="answer in answers" 
          :key="answer" 
          :class="{'correct': selectedAnswer === answer && answer === correctAnswer, 'incorrect': selectedAnswer === answer && answer !== correctAnswer }"
          @click="selectAnswer(answer)"
        >
          <button :disabled="!!selectedAnswer" :style="buttonStyle(answer)">{{ answer }}</button>
        </li>
      </ul>
    </div>
  </template>
  
<script setup>
import { ref, watch, } from 'vue';

const props = defineProps({
  question: String,
  answers: Array,
  selectedAnswer: String,
  correctAnswer: String
});

const emit = defineEmits(['answer-selected']);

const selectedAnswer = ref(null);

function selectAnswer(answer) {
  selectedAnswer.value = answer;
  emit('answer-selected', answer);
};

function buttonStyle(answer) {
  if (!selectedAnswer.value) return {};

  if (answer === props.correctAnswer) {
    return { backgroundColor: 'green', color: 'white' };
  } else if (answer === selectedAnswer.value) {
    return { backgroundColor: 'red', color: 'white' };
  } else {
    return {};
  }
};

// Reset selected answer if a new question is loaded
watch(() => props.question, () => {
  selectedAnswer.value = null;
});
</script>

  
<style scoped>
/* Adjust button styles */
button {
  padding: 10px;
  margin: 5px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  width: 150px;
}

button:disabled {
  cursor: not-allowed;
}

ul {
  list-style-type: none;
  padding: 0;
}
</style>
  