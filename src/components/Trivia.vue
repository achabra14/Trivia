<template>
    <div>
      <div v-if="!quizFinished">
        <p>Question {{ currentQuestionIndex + 1 }}: {{ questions[currentQuestionIndex].question }}</p>
        <div v-for="(answer, index) in questions[currentQuestionIndex].answers" :key="index">
          <button @click="selectAnswer(index)">{{ answer }}</button>
        </div>
      </div>
  
      <div v-else>
        <h2>Quiz Finished!</h2>
        <p>Your score: {{ score }}/{{ questions.length }}</p>
        <button @click="restartQuiz">Restart Quiz</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        questions: [
          {
            question: "What is the capital of France?",
            answers: ["Paris", "London", "Berlin", "Madrid"],
            correctAnswer: 0
          },
          {
            question: "Which is the largest planet in the solar system?",
            answers: ["Earth", "Mars", "Jupiter", "Saturn"],
            correctAnswer: 2
          },
          // Add more questions here
        ],
        currentQuestionIndex: 0,
        score: 0,
        quizFinished: false
      };
    },
    methods: {
      selectAnswer(index) {
        if (index === this.questions[this.currentQuestionIndex].correctAnswer) {
          this.score++;
        }
        this.nextQuestion();
      },
      nextQuestion() {
        if (this.currentQuestionIndex < this.questions.length - 1) {
          this.currentQuestionIndex++;
        } else {
          this.quizFinished = true;
        }
      },
      restartQuiz() {
        this.currentQuestionIndex = 0;
        this.score = 0;
        this.quizFinished = false;
      }
    }
  };
  </script>
  
  <style scoped>
  button {
    margin: 10px;
    padding: 10px 20px;
    background-color: #42b983;
    color: white;
    border: none;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #35a076;
  }
  </style>
  