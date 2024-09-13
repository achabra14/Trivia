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
  
  <script>
  export default {
    props: {
      question: String,
      answers: Array,
      selectedAnswer: String,
      correctAnswer: String
    },
    methods: {
      selectAnswer(answer) {
        if (!this.selectedAnswer) {
          this.$emit('answer-selected', answer);
        }
      },
      buttonStyle(answer) {
        if (!this.selectedAnswer) return {};
  
        if (answer === this.correctAnswer) {
          return { backgroundColor: 'green', color: 'white' };
        } else if (answer === this.selectedAnswer) {
          return { backgroundColor: 'red', color: 'white' };
        } else {
          return {};
        }
      }
    }
  }
  </script>
  
  <style scoped>
  /* Adjust button styles */
  button {
    padding: 10px;
    margin: 5px;
    border: none;
    cursor: pointer;
    border-radius: 5px;
  }
  
  button:disabled {
    cursor: not-allowed;
  }
  
  .correct {
    background-color: green;
  }
  
  .incorrect {
    background-color: red;
  }
  
  ul {
    list-style-type: none;
    padding: 0;
  }
  </style>
  