<template>
  <div class="trivia-game">
    <div v-if="!username" class="username-screen">
      <h2>Welcome to the Trivia Game!</h2>
      <p>Please enter your username to start:</p>
      <input v-model="inputUsername" placeholder="Enter your username" />
      <button @click="startGame">Start</button>
    </div>

     <!-- Trivia game begins once username is entered -->
     <div v-if="username && currentQuestion">
      <h3>{{ decodeHTMLEntities(currentQuestion.category) }}</h3>
      <Question 
        v-if="currentQuestion" 
        :question="decodeHTMLEntities(currentQuestion.question)" 
        :answers="shuffledAnswers"
        :selected-answer="selectedAnswer"
        :correct-answer="decodeHTMLEntities(currentQuestion.correct_answer)"
        @answer-selected="handleAnswer"
      />

      <!-- Feedback and Next Question Button -->
      <div v-if="selectedAnswer" class="feedback">
        <p>{{ isCorrect ? 'Correct! 🎉' : 'Incorrect!' }}</p>
        <button @click="nextQuestion">Next Question</button>
      </div>

      <!-- Username and score displayed in the bottom right corner -->
      <div class="score-display">{{ username }} : {{ score }}</div>
    </div>

    <!-- End of game -->
    <p v-if="username && !currentQuestion && !loading">Game over! You scored {{ score }}/{{ questions.length }}</p>
    <button v-if="username && !currentQuestion && !loading" @click="resetGame">Restart</button>

    <!-- Loading state -->
    <p v-if="username && loading">Loading questions...</p>
  </div>
  </template>
  
  <script>
  import { ref, reactive, computed, onMounted } from 'vue';
  import axios from 'axios';
  import Question from './Question.vue';


  export default {
    components: { Question },

    setup() {
      const username = ref('');
      const inputUsername = ref('');
      const score = ref(0);
      const selectedAnswer = ref(null);
      const isCorrect = ref(false);
      const currentQuestionIndex = ref(0);
      const questions = ref([]);
      const loading = ref(true);

  
      const currentQuestion = computed(() => {
        return questions.value[currentQuestionIndex.value] || null;
      });
  
      const shuffledAnswers = computed(() => {
        if (!currentQuestion.value) return [];
        const answers = [
          ...currentQuestion.value.incorrect_answers.map(answer => decodeHTMLEntities(answer)),
          decodeHTMLEntities(currentQuestion.value.correct_answer)
        ];
        return shuffle(answers);
      });
  
      const fetchQuestions = async () => {
        try {
          const response = await axios.get('https://opentdb.com/api.php?amount=5&type=multiple');
          questions.value = response.data.results;
          loading.value = false;
        } catch (error) {
          console.error('Error fetching questions:', error);
          loading.value = false;
        }
      };

      const handleAnswer = (answer) => {
        selectedAnswer.value = answer;
        const correctAnswer = decodeHTMLEntities(currentQuestion.value.correct_answer);
        isCorrect.value = answer === correctAnswer;
        if (isCorrect.value) {
          score.value++;
        }
      };

      const nextQuestion = () => {
        selectedAnswer.value = null;
        isCorrect.value = false;
        currentQuestionIndex.value++;
      };

      const resetGame = () => {
        currentQuestionIndex.value = 0;
        score.value = 0;
        fetchQuestions();
      };

      const shuffle = (array) => {
        for (let i = array.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
      };
  
      const decodeHTMLEntities = (text) => {
        const textArea = document.createElement('textarea');
        textArea.innerHTML = text;
        return textArea.value;
      };

      const startGame = () => {
        if (inputUsername.value.trim()) {
          username.value = inputUsername.value.trim();
          fetchQuestions();
        }
      };
  
      onMounted( () => {
        //fetchQuestions();
      });
  
      return {
        username,
        inputUsername,
        score,
        loading,
        selectedAnswer,
        isCorrect,
        currentQuestionIndex,
        questions,
        currentQuestion,
        shuffledAnswers,
        handleAnswer,
        nextQuestion,
        resetGame,
        fetchQuestions,
        decodeHTMLEntities,
        startGame,
        shuffle
      };
    }
  };
  </script>
  
  <style scoped>
  /* Add any styles you need */
  .username-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 50px;
}

  .feedback {
    margin-top: 20px;
    text-align: center;
  }
  
  .score-display {
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
  