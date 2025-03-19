
<script setup>
import { ref } from 'vue';
const emit = defineEmits(['start-quiz']);
const props = defineProps(['errorMessage']);
const topic = ref('');

</script>

<template>

<section class="start-screen container">
    <h2>Here are 5 Questions to test your knowledge on Todays topic</h2>
    
    <button @click="emit('start-quiz', topic)"  >Start Quiz</button>
    <small class="error" v-if="errorMessage">There was an error, please try again later.</small>
</section>

</template>