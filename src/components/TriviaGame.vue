<template>
    <div class="trivia-game">
      <h3 v-if="currentQuestion">{{ decodeHTMLEntities(currentQuestion.category) }}</h3>
      <Question 
        v-if="currentQuestion" 
        :question="decodeHTMLEntities(currentQuestion.question)" 
        :answers="shuffledAnswers"
        :selected-answer="selectedAnswer"
        :correct-answer="decodeHTMLEntities(currentQuestion.correct_answer)"
        @answer-selected="handleAnswer"
      />
      
      <div v-if="selectedAnswer" class="feedback">
        <p>{{ isCorrect ? 'Correct! 🎉' : 'Incorrect!' }}</p>
        <button @click="nextQuestion">Next Question</button>
      </div>
  
      <p v-if="!currentQuestion && !loading">Game over! You scored {{ score }}/{{ questions.length }}</p>
      <button v-if="!currentQuestion && !loading" @click="resetGame">Restart</button>
      <p v-if="loading">Loading questions...</p>
  
      <!-- Score displayed in the bottom right corner -->
      <div class="score-display">Score: {{ score }}</div>
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
        loading: true,
        selectedAnswer: null,  // Tracks the selected answer
        isCorrect: false,      // Tracks if the selected answer is correct
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
          this.questions = response.data.results;
          this.loading = false;
        } catch (error) {
          console.error("Failed to fetch questions:", error);
          this.loading = false;
        }
      },
      handleAnswer(selectedAnswer) {
        this.selectedAnswer = selectedAnswer;
        const correctAnswer = this.decodeHTMLEntities(this.currentQuestion.correct_answer);
        this.isCorrect = selectedAnswer === correctAnswer;
        if (this.isCorrect) {
          this.score++;
        }
      },
      nextQuestion() {
        this.selectedAnswer = null; // Reset the selected answer
        this.isCorrect = false;
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
  .feedback {
    margin-top: 20px;
    text-align: center;
  }
  
  .score-display {
    position: fixed;
    bottom: 10px;
    right: 10px;
    font-weight: bold;
    font-size: 18px;
    background: #f4f4f4;
    padding: 10px;
    border-radius: 8px;
  }
  
  button {
    margin-top: 10px;
  }

/* Media query for mobile devices */
@media (max-width: 600px) {
    .score-display {
        font-size: 16px;
        padding: 8px;
        bottom: 5px;
        right: 5px;
    }

    button {
        width: 100%;
        margin-top: 5px;
    }
}


  </style>
  