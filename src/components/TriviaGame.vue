<template>
    <div>
      <Question 
        v-if="currentQuestion" 
        :question="currentQuestion.text" 
        :answers="currentQuestion.options"
        @answer-selected="checkAnswer"
      />
      <p v-if="!currentQuestion">Game over! You scored {{ score }}/{{ questions.length }}</p>
      <button v-if="!currentQuestion" @click="resetGame">Restart</button>
    </div>
  </template>
  
  <script>
  import Question from './Question.vue';  // Import the Question component
  
  export default {
    components: { Question },
    data() {
      return {
        currentQuestionIndex: 0,
        score: 0,
        questions: [
          {
            text: "What is the capital of France?",
            options: ["Paris", "Berlin", "London", "Rome"],
            correct: "Paris"
          },
          {
            text: "Who wrote 'To Kill a Mockingbird'?",
            options: ["Harper Lee", "George Orwell", "J.K. Rowling", "Ernest Hemingway"],
            correct: "Harper Lee"
          }
          // Add more questions here
        ]
      };
    },
    computed: {
      currentQuestion() {
        return this.questions[this.currentQuestionIndex] || null;
      }
    },
    methods: {
      checkAnswer(selectedAnswer) {
        if (selectedAnswer === this.currentQuestion.correct) {
          this.score++;
        }
        this.currentQuestionIndex++;
      },
      resetGame() {
        this.currentQuestionIndex = 0;
        this.score = 0;
      }
    }
  }
  </script>
  