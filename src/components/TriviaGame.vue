<template>
  <div class="trivia-game">
    <WelcomeScreen v-if="!username" @start-game="(inputUsername) => startGame(inputUsername)" />
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
  
<script setup>
  import { ref, computed, onMounted } from 'vue';
  import axios from 'axios';
  import Question from './Question.vue';
  import WelcomeScreen from './WelcomeScreen.vue';

  const username = ref('');
  const inputUsername = ref('');
  const score = ref(0);
  const selectedAnswer = ref(null);
  const isCorrect = ref(false);
  const currentQuestionIndex = ref(0);
  const questions = ref([]);
  const loading = ref(true);
  const showCategoryDialog = ref(false);
  const categories = ref([]);
  const selectedCategories = ref([]);
  const isAllSelected = computed(() => selectedCategories.value.length === categories.value.length);

  // const toggleSelectAll = () => {
  //     if (isAllSelected.value) {
  //       selectedCategories.value = [];
  //     } else {
  //       selectedCategories.value = categories.value.map(category => category.name);
  //     }
  //   };

  const currentQuestion = computed(() => {
    return questions.value[currentQuestionIndex.value] || null;
  });

  // const fetchCategories = async () => {
  //   try {
  //     const response = await axios.get('https://opentdb.com/api_category.php');
  //     categories.value = response.data.trivia_categories;
  //     selectedCategories.value = categories.value.map(category => category.name);
  //   } catch (error) {
  //     console.error('Error fetching categories:', error);
  //   }
  // };

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
      const response = await axios.get('https://opentdb.com/api.php?amount=50&type=multiple');
      questions.value = response.data.results;
      questions.value = questions.value.filter(question => selectedCategories.value.includes(question.category));
      questions.value = questions.value.slice(0, 5);
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

  const startGame = (inputUsername) => {

    username.value = inputUsername.trim();
    if (username) {
      fetchQuestions();
      //console.log(selectedCategories.value);
    }
  };

  // const startGame = () => {
  //   if (inputUsername.value.trim()) {
  //     username.value = inputUsername.value.trim();
  //     fetchQuestions();
  //     console.log(selectedCategories.value);
  //   }
  // };

  // const confirmCategories = () => {
  //   showCategoryDialog.value = false;
  // };

  // const closeDialog = () => {
  //   showCategoryDialog.value = false;
  // };

  onMounted( () => {
    //fetchCategories();
  });
  
</script>
  
<style scoped>
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

.category-container {
  display: flex;
  justify-content: flex-start;
}

.categories {
  list-style-type: none;
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* Two-column layout */
  column-gap: 20px; /* Space between columns */
}

.categories ul {
  text-align: left; /* Left align each category */
  margin-bottom: 10px; /* Space between category checkboxes */
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
  