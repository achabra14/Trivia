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
import { ref, watch } from 'vue';
import axios from 'axios';

const props = defineProps({
  modelValue: String,
  modelValueSelectedCategories: Array,
  categories: Array
});

const emit = defineEmits(['update:modelValue', 'update:modelValueSelectedCategories', 'update:modelValueShowCategoryDialog', 'start-game']);

// what was chatGPT doing here?
//const localInputUsername = ref(props.modelValue);
const inputUsername = ref('');
//const localSelectedCategories = ref(props.modelValueSelectedCategories);
const categories = ref([]);
const selectedCategories = ref([]);
const showCategoryDialog = ref(false);

// watch(localInputUsername, (newValue) => {
//   emit('update:modelValue', newValue);
// });

// watch(localSelectedCategories, (newValue) => {
//   emit('update:modelValueSelectedCategories', newValue);
// });

watch(showCategoryDialog, (newValue) => {
  emit('update:modelValueShowCategoryDialog', newValue);
});

const isAllSelected = ref(false);

const toggleSelectAll = () => {
  if (isAllSelected.value) {
    localSelectedCategories.value = [];
  } else {
    localSelectedCategories.value = props.categories.map(category => category.name);
  }
  isAllSelected.value = !isAllSelected.value;
};

const confirmCategories = () => {
  showCategoryDialog.value = false;
};

const closeDialog = () => {
  showCategoryDialog.value = false;
};

const startGame = () => {
  emit('start-game', inputUsername.value);
};

const fetchCategories = async () => {
    try {
      const response = await axios.get('https://opentdb.com/api_category.php');
      categories.value = response.data.trivia_categories;
      selectedCategories.value = categories.value.map(category => category.name);
    } catch (error) {
      console.error('Error fetching categories:', error);
    }
  };

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
</style>
