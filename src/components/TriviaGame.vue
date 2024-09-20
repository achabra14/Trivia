<template>
<div class="trivia-game">
  <WelcomeScreen 
    v-if="state === states.WELCOME" 
    @start-game="(inputUsername, categories) => startGame(inputUsername, categories)" 
  />

  <div v-if="state === states.QUESTION || state === states.FEEDBACK || state === states.WAGER">
    <ScoreContainer :players="players" />
    <h3>{{ `${valueMessage} -  ${decodeHTMLEntities(currentQuestion.category)}` }}</h3>
  </div>

  <Question 
    v-if="state === states.QUESTION || state === states.FEEDBACK"
    :question="decodeHTMLEntities(currentQuestion.question)" 
    :answers="shuffledAnswers"
    :selected-answer="selectedAnswer"
    :correct-answer="decodeHTMLEntities(currentQuestion.correct_answer)"
    @answer-selected="handleAnswer"
  />

  <!-- Feedback and Next Question Button -->
  <div v-show="state === states.FEEDBACK" class="feedback">
    <p>{{ feedbackText }}</p>
    <button @click="nextQuestion">Next Question</button>
  </div>
  

  <BetweenRounds 
    v-if="state === states.BETWEEN_ROUNDS"
    :round="currentRound"
    :players="players"
    @next-round="advanceRound"
  />

  <FinalRound
    v-if="state === states.WAGER"
    :final-round-category="decodeHTMLEntities(currentQuestion.category)"
    :user-score="heroScore"
    @submit-wager="wager => onFinalRoundWagerSubmit(wager)"
  />
  <!-- Loading state -->
  <p v-if="state === states.LOADING">Loading questions...</p>
</div>
</template>
  
<script setup>

import { ref, computed } from 'vue';
import axios from 'axios';
import Question from './Question.vue';
import WelcomeScreen from './WelcomeScreen.vue';
import ScoreContainer from './ScoreContainer.vue';
import BetweenRounds from './BetweenRounds.vue';
import FinalRound from './FinalRoundWager.vue';

const states = {
  WELCOME: 'welcome',
  LOADING: 'loading',
  QUESTION: 'question',
  FEEDBACK: 'feedback',
  BETWEEN_ROUNDS: 'between_rounds',
  WAGER: 'wager'
};

const state = ref(states.WELCOME);

const username = ref('');
const selectedAnswer = ref(null);
const currentQuestionIndex = ref(0);
const currentRound = ref(1);

const questions = ref([]);
const loading = ref(true);
const selectedCategories = ref([]);
const currentQuestionValue = computed(() => ((currentQuestionIndex.value % 5 + 1)) * (100 * currentRound.value));
const feedbackText = ref('');
const valueMessage = computed(() => currentQuestionIndex.value == 10 ? 'Final Question' : `\$${currentQuestionValue.value}`);

const heroScore = ref(0);
const ai1Score = ref(0);
const ai2Score = ref(0);

const finalRoundWager = ref(0);

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
    loading.value = false;
  } catch (error) {
    console.error('Error fetching questions:', error);
    loading.value = false;
  }
}

function isFinalRound() {
  return currentQuestionIndex.value == 10;
}

function handleAnswer(answer) {
  let questionValue = isFinalRound() ? finalRoundWager.value : currentQuestionValue.value;

  selectedAnswer.value = answer;
  const correctAnswer = decodeHTMLEntities(currentQuestion.value.correct_answer);
  if (answer === correctAnswer) {
    feedbackText.value = '🎉';
    heroScore.value += questionValue;
  }
  else if (answer === 'pass') {
    feedbackText.value = '¯\\_(ツ)_/¯';
  }
  else {
    feedbackText.value = '😢';
    heroScore.value -= questionValue;
  }

  const AIPASS = 0.2;
  const AICORRECT = 0.7;

  questionValue = isFinalRound() ? getAIWagerRandom(ai1Score.value) : currentQuestionValue.value;

  // simulate AI players
  let ai1Result = simAIResult(AIPASS, AICORRECT);
  if (ai1Result === 'correct') {
    ai1Score.value += questionValue;
  }
  else if (ai1Result === 'incorrect') {
    ai1Score.value -= questionValue;
  }  

  questionValue = isFinalRound() ? getAIWagerRandom(ai2Score.value) : currentQuestionValue.value;

  let ai2Result = simAIResult(0.2, 0.7);
  if (ai2Result === 'correct') {
    ai2Score.value += questionValue;
  }
  else if (ai2Result === 'incorrect') {
    ai2Score.value -= questionValue;
  }

  state.value = states.FEEDBACK;
  // setting timeout causes content jumping
  // setTimeout(() => {
  //   feedbackText.value = '';
  // }, 3000);
}

// Get a random wager for AI players, must be divisible by 100
function getAIWagerRandom(aiScore) {
  const maxWager = Math.max(aiScore, 1000);
  return (Math.round(Math.random() * (maxWager / 100))) * 100;
}

function simAIResult(probPass, probCorrect) {
  if (Math.random() < probPass)
    return 'pass';

  return (Math.random() < probCorrect) ? 'correct' : 'incorrect';
}

function nextQuestion() {
  selectedAnswer.value = null;
  currentQuestionIndex.value++;
  if (currentQuestionIndex.value == 5 || currentQuestionIndex.value == 10 || currentQuestionIndex.value == 11) {
    state.value = states.BETWEEN_ROUNDS;
  }
  else {
    state.value = states.QUESTION;
  }
}

function advanceRound() {
  if (currentRound.value == 3) {
    resetGame();
    return;
  }
  
  currentRound.value++;
  
  if (currentRound.value == 3) {
    state.value = states.WAGER;
    return;
  }
  state.value = states.QUESTION;
}

function onFinalRoundWagerSubmit(wager) {
  finalRoundWager.value = Number(wager);
  state.value = states.QUESTION;
}

async function resetGame() {
  state.value = states.LOADING;

  currentQuestionIndex.value = 0;
  heroScore.value = 0;
  ai1Score.value = 0;
  ai2Score.value = 0;
  currentRound.value = 1;
  loading.value = true;
  await fetchQuestions();
  await fetchAllProfilePicsAndNames();
  state.value = states.QUESTION;
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

async function startGame(inputUsername, categories) {
  username.value = inputUsername.trim();
  
  if (username) {
    state.value = states.LOADING;

    selectedCategories.value = categories;
    await fetchQuestions();
    state.value = states.QUESTION;
    
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
/* @media (max-width: 600px) {
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
  } */
  </style>
  