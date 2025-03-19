<script setup>
import { GoogleGenerativeAI, SchemaType } from "@google/generative-ai";
import StartScreen from './components/StartScreen.vue';
import Quiz from "./components/Quiz.vue";
import Loader from "./components/Loader.vue";
import Result from "./components/Result.vue";

import { ref } from 'vue';

const questions = ref('');
const status = ref('start');
const userAnswers = ref([]);
const errorMessage = ref('');

const restartQuiz = ()=>{
  questions.value = '';
  status.value = 'start';
  userAnswers.value = [];
  errorMessage.value = '';
}

const storeAnswer = (answer) => {
  userAnswers.value.push(answer);
}

const startQuiz = async (topic)=>{

status.value = 'loading';

const genAI = new GoogleGenerativeAI("AIzaSyBpE6TivQjtgcJrIfdn0eTnfdyg6qfC-QM");

const schema = {
  description: "List of quiz questions",
  type: SchemaType.OBJECT,
  properties: {
    response_code: {
      type: SchemaType.NUMBER,
      description: "Response code indicating the status of the request",
      nullable: false,
    },
    results: {
      type: SchemaType.ARRAY,
      description: "Array of quiz questions",
      items: {
        type: SchemaType.OBJECT,
        properties: {
          type: {
            type: SchemaType.STRING,
            description: "Type of question (e.g., multiple choice, boolean)",
            nullable: false,
          },
          difficulty: {
            type: SchemaType.STRING,
            description: "Difficulty level of the question (easy, medium, hard)",
            nullable: false,
          },
          category: {
            type: SchemaType.STRING,
            description: "Category of the question",
            nullable: false,
          },
          question: {
            type: SchemaType.STRING,
            description: "The quiz question",
            nullable: false,
          },
          correct_answer: {
            type: SchemaType.STRING,
            description: "The correct answer to the question",
            nullable: false,
          },
          incorrect_answers: {
            type: SchemaType.ARRAY,
            description: "Array of incorrect answers",
            items: {
              type: SchemaType.STRING,
            },
            nullable: false,
          },
        },
        required: [
          "type",
          "difficulty",
          "category",
          "question",
          "correct_answer",
          "incorrect_answers",
        ],
      },
      nullable: false,
    },
  },
  required: ["response_code", "results"],
};

const model = genAI.getGenerativeModel({
  model: "gemini-1.5-pro",
  generationConfig: {
    responseMimeType: "application/json",
    responseSchema: schema,
  },
});

try {
  
  const result = await model.generateContent(
    `
      Create 5 quiz questions about operating system
      Difficulty: Medium
      Type: Multiple Choice
    `,
  );

  questions.value = JSON.parse(result.response.text());
  status.value = 'ready';

} catch (error) {
  
  errorMessage.value = error;
  status.value = 'start';

}

 
}

</script>

<template>
  <div id="app">

    <header>
      <div class="container">
        <!-- <img src="./assets/logo.png" alt="App Logo" class="logo"> -->
        <h1>E-Engage Quiz Assesment</h1>
      </div>
    </header>

    <StartScreen v-if="status == 'start'" :errorMessage="errorMessage" @start-quiz="startQuiz" />
    <Loader v-if="status == 'loading'" />
    <Quiz v-if="status == 'ready'" @end-quiz="status = 'finished'" @store-answer="storeAnswer" :questions="questions.results" />
    <Result v-if="status == 'finished'" @restart-quiz="restartQuiz" :userAnswers="userAnswers" />


  </div>
</template>


