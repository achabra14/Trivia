<template>
  <div class="final-round">
    <h2>Final Round<br> Category: {{ finalRoundCategory }} <br> </h2>
      <div class="wager-input">
        <label for="wager">Enter your wager </label>
        <input
          type="number"
          id="wager"
          v-model="wager"
          min="0"
          :max="maxWager"
          step="100"
          placeholder="Enter amount"
        />
      </div>
      <div class="wager-slider">
        <input
          type="range"
          id="wagerSlider"
          v-model="wager"
          :min="0"
          :max="maxWager"
          step="100"
        />
      </div>
        
      <button @click="submitWager" :disabled="!wager || wager > maxWager || wager < 0">
          Submit Wager
      </button>
        
      <div v-if="errorMessage" class="error">
          {{ errorMessage }}
      </div>
  </div>
</template>
  

<script setup>
import { ref } from 'vue';

const wager = ref(0);             // Model to store user's wager
const errorMessage = ref('');

const props = defineProps({
  finalRoundCategory: String,
  userScore: Number
});

const maxWager = Math.max(props.userScore, 1000);

const emit = defineEmits(['submit-wager']);

function submitWager() {
  if (wager.value > maxWager.value || wager.value < 0) {
    errorMessage.value = "Invalid wager amount!";
  } else {
    errorMessage.value = "";
    emit('submit-wager', wager.value);
  }
};
</script>


<style scoped>
  .final-round {
    text-align: center;
    margin-top: 20px;
  }
  
  .wager-input {
    margin: 10px 0;
  }
  
  input[type="number"] {
    padding: 5px;
    font-size: 16px;
  }
  
  button {
    background-color: #3a3f47;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  button:disabled {
    background-color: gray;
  }
  
  .error {
    color: red;
    margin-top: 10px;
  }

  .wager-slider {
    margin: 20px 0;
  }

  input[type="range"] {
    width: 100%;
    margin-top: 10px;
  }
</style>
  