<template>
<div>
  <div class="question">
    <h4>{{ question }}</h4>
    <button @click="passSelected" class="pass-button" :disabled="!!selectedAnswer">Pass</button>
  </div>
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

function passSelected() {
  selectedAnswer.value = 'pass';
  emit('answer-selected', 'pass');
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

.question {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.pass-button {
  font-size: 14px;
  padding: 5px 10px;
  background-color: #d3d3d3;
  color: black;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  width: auto;
}

.pass-button:hover {
  background-color: #b0b0b0;
}

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
  