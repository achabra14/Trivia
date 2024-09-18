<template>
<div class="trivia-game">
  <WelcomeScreen 
    v-if="!username" 
    @start-game="(inputUsername, categories) => startGame(inputUsername, categories)" 
  />

  <!-- Trivia game begins once username is entered -->
  <div v-if="username && currentQuestion">
    <ScoreContainer :players="players" />
    <h3>{{ `\$${currentQuestionValue} -  ${decodeHTMLEntities(currentQuestion.category)}` }}</h3>
    
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
      <p>{{ feedbackText }}</p>
      <button @click="nextQuestion">Next Question</button>
    </div>
  </div>

  <!-- End of game -->
  <p v-if="username && !currentQuestion && !loading">Game over! You scored {{ heroScore }}</p>
  <button v-if="username && !currentQuestion && !loading" @click="resetGame">Restart</button>

  <!-- Loading state -->
  <p v-if="username && loading">Loading questions...</p>
</div>
</template>
  
<script setup>

import { ref, computed } from 'vue';
import axios from 'axios';
import Question from './Question.vue';
import WelcomeScreen from './WelcomeScreen.vue';
import ScoreContainer from './ScoreContainer.vue';

const username = ref('');
const selectedAnswer = ref(null);
const currentQuestionIndex = ref(0);
const questions = ref([]);
const loading = ref(true);
const selectedCategories = ref([]);
const currentQuestionValue = computed(() => (currentQuestionIndex.value + 1) * 100);
const feedbackText = ref('');

const heroScore = ref(0);
const ai1Score = ref(0);
const ai2Score = ref(0);

const players = ref([
{ name: username, score: heroScore, profilePic: 'images/evie_goofy.jpg' },
{ name: '', score: ai1Score, profilePic: '' },
{ name: '', score: ai2Score, profilePic: '' }
]);

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

async function fetchProfilePicAndName(playerIndex) {
  try {
    const response = await fetch('https://randomuser.me/api?format=json');
    const data = await response.json();
    const thumbnail = data.results[0].picture.thumbnail; // Get the thumbnail URL
    const username = data.results[0].login.username;

       // Update the player profilePic
    if (playerIndex != 0) {
      players.value[playerIndex].profilePic = thumbnail;
      players.value[playerIndex].name = username;
    }
  } catch (error) {
    console.error('Error fetching profile picture:', error);
  }
}

// Fetch profile pictures for all players
function fetchAllProfilePicsAndNames() {
  players.value.forEach((player, index) => {
    fetchProfilePicAndName(index);  // Fetch for each player
  });
};

async function fetchQuestions() {
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
}

function handleAnswer(answer) {
  selectedAnswer.value = answer;
  const correctAnswer = decodeHTMLEntities(currentQuestion.value.correct_answer);
  if (answer === correctAnswer) {
    feedbackText.value = 'Correct! 🎉';
    heroScore.value += currentQuestionValue.value;
  }
  else if (answer === 'pass') {
    feedbackText.value = 'You passed! 😅';
  }
  else {
    feedbackText.value = 'Incorrect!';
    heroScore.value -= currentQuestionValue.value;
  }

  const AIPASS = 0.2;
  const AICORRECT = 0.7;

  // simulate AI players
  let ai1Result = simAIResult(AIPASS, AICORRECT);
  if (ai1Result === 'correct') {
    ai1Score.value += currentQuestionValue.value;
  }
  else if (ai1Result === 'incorrect') {
    ai1Score.value -= currentQuestionValue.value;
  }

  let ai2Result = simAIResult(0.2, 0.7);
  if (ai2Result === 'correct') {
    ai2Score.value += currentQuestionValue.value;
  }
  else if (ai2Result === 'incorrect') {
    ai2Score.value -= currentQuestionValue.value;
  }
}

function simAIResult(probPass, probCorrect) {
  const passSim = Math.random();
  if (passSim < probPass)
    return 'pass';

  const correctSim = Math.random();

  return (correctSim < probCorrect) ? 'correct' : 'incorrect';
}

function nextQuestion() {
  selectedAnswer.value = null;
  currentQuestionIndex.value++;
}

function resetGame() {
  currentQuestionIndex.value = 0;
  heroScore.value = 0;
  ai1Score.value = 0;
  ai2Score.value = 0;
  fetchQuestions();
  fetchAllProfilePicsAndNames();
}

function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
};

function decodeHTMLEntities(text) {
  const textArea = document.createElement('textarea');
  textArea.innerHTML = text;
  return textArea.value;
};

function startGame(inputUsername, categories) {
  username.value = inputUsername.trim();
  if (username) {
    fetchQuestions();
    selectedCategories.value = categories;
  }
};

fetchAllProfilePicsAndNames();


  
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
  