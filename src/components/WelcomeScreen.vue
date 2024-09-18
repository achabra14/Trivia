<template>
<div class="welcome-screen">
  <h2>Welcome to Trivia!</h2>
  <p>Please enter your username to start:</p>
  <input v-model="inputUsername" placeholder="Enter your username" />
  
  <button @click="showCategoryDialog = true">Choose Categories</button>
  <dialog v-if="showCategoryDialog" @close="showCategoryDialog = false" open>
    <h3>Select Categories</h3>
    <div>
      <input type="checkbox" id="select-all" @change="toggleSelectAll" :checked="isAllSelected" />
      <label for="select-all">{{ isAllSelected ? 'Select None' : 'Select All' }}</label>
    </div>

    <div class="category-container">
      <ul class="categories">
        <ul v-for="category in categories" :key="category.id">
          <input type="checkbox" :id="category.id" :value="category.name" v-model="selectedCategories" />
          <label :for="category.id">{{ category.name }}</label>
        </ul>
      </ul>
    </div>
    
    <button @click="confirmCategories">Confirm</button>
    <button @click="closeDialog">Cancel</button>
  </dialog>
  <button @click="startGame">Start</button>
</div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';

const emit = defineEmits(['start-game']);

const inputUsername = ref('');
const categories = ref([]);
const selectedCategories = ref([]);
const showCategoryDialog = ref(false);
const isAllSelected = computed(() => selectedCategories.value.length === categories.value.length);


function toggleSelectAll() {
  if (isAllSelected.value) {
    selectedCategories.value = [];
  } else {
    selectedCategories.value = categories.value.map(category => category.name);
  }
}

function confirmCategories() {
  showCategoryDialog.value = false;
}

function closeDialog() {
  showCategoryDialog.value = false;
}

function startGame() {
  emit('start-game', inputUsername.value, selectedCategories.value);
}

async function fetchCategories() {
  try {
    const response = await axios.get('https://opentdb.com/api_category.php');
    categories.value = response.data.trivia_categories;
    selectedCategories.value = categories.value.map(category => category.name);
    selectedCategories.value = selectedCategories.value.filter(category => 
      category != 'Entertainment: Video Games' &&
      category != 'Entertainment: Japanese Anime & Manga'
    )
  } 
  catch (error) {
    console.error('Error fetching categories:', error);
  }
}

  fetchCategories();

</script>

<style scoped>
.welcome-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 50px;
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
</style>
