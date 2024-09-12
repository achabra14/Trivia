<template>
    <div>
      <h2>{{ question }}</h2>
      <ul>
        <li v-for="(answer, index) in answers" :key="index">
          <button @click="selectAnswer(answer)">{{ answer }}</button>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    props: ['question', 'answers'],
    methods: {
      selectAnswer(answer) {
        this.$emit('answer-selected', answer);  // Emit an event when an answer is selected
      }
    }
  }
  </script>
  