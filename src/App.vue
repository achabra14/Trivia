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

<script>
import TriviaGame from './components/TriviaGame.vue';

export default {
  name: 'App',
  components: {
    TriviaGame
  }
}
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
</style>

