<template>
    <div>
      <Question 
        v-if="currentQuestion" 
        :question="decodeHTMLEntities(currentQuestion.question)" 
        :answers="shuffledAnswers"
        @answer-selected="checkAnswer"
      />
      <p v-if="!currentQuestion && !loading">Game over! You scored {{ score }}/{{ questions.length }}</p>
      <button v-if="!currentQuestion && !loading" @click="resetGame">Restart</button>
      <p v-if="loading">Loading questions...</p>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import Question from './Question.vue';  // Import the Question component
  
  export default {
    components: { Question },
    data() {
      return {
        currentQuestionIndex: 0,
        score: 0,
        questions: [],
        loading: true,  // To track loading state
      };
    },
    computed: {
      currentQuestion() {
        return this.questions[this.currentQuestionIndex] || null;
      },
      shuffledAnswers() {
        if (!this.currentQuestion) return [];
        const answers = [
          ...this.currentQuestion.incorrect_answers.map(answer => this.decodeHTMLEntities(answer)),
          this.decodeHTMLEntities(this.currentQuestion.correct_answer)
        ];
        return this.shuffle(answers);
      }
    },
    methods: {
      async fetchQuestions() {
        try {
          const response = await axios.get('https://opentdb.com/api.php?amount=5&type=multiple'); // Fetch 5 questions
          console.log("Fetched questions:", response);
          this.questions = response.data.results;
          this.loading = false;
        } catch (error) {
          console.error("Failed to fetch questions:", error);
          this.loading = false;
        }
      },
      checkAnswer(selectedAnswer) {
        if (selectedAnswer === this.decodeHTMLEntities(this.currentQuestion.correct_answer)) {
          this.score++;
        }
        this.currentQuestionIndex++;
      },
      resetGame() {
        this.currentQuestionIndex = 0;
        this.score = 0;
        this.fetchQuestions();  // Fetch new set of questions
      },
      shuffle(array) {
        for (let i = array.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
      },
      decodeHTMLEntities(str) {
        const textArea = document.createElement('textarea');
        textArea.innerHTML = str;
        return textArea.value;
      }
    },
    mounted() {
      this.fetchQuestions();  // Fetch questions when the component is mounted
    }
  }
  </script>
  
  <style scoped>
  /* Add any styles you need */
  </style>
  