<template>
  <div id="app">
    <header>
      <h1>Trivia</h1>
    </header>
    <main>
      <TriviaGame />
    </main>
  </div>
</template>

<script setup>
import TriviaGame from './components/TriviaGame.vue';

</script>

<style>

#app {
  text-align: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
}

header {
  padding: 20px;
}

main {
  margin: 0 auto;
  max-width: 600px;
  min-height: 650px;
}

button {
  margin: 10px;
  padding: 10px 20px;
  /* pick a light blue for background-color */
  background-color: #3498db;
  color: white;
  border: none;
  cursor: pointer;
}

button:focus {
  outline: none;
}

button:hover {
  background-color: #2980b9;
}

.logoImage {
  width: 100px;
  height: 1000px;
  margin-right: 10px;
}

</style>

