<template>
  <div class="flex flex-col items-center justify-center min-h-screen">
    <h1 class="my-4 mx-2 md:mx-0 lg:text-2xl text-lg text-center text-blue-600">HOME INSURANCE MARKETING SAS PLATFORM</h1>
    <div v-if="isSuccess">
      <div v-if="isReconmmend" class="bg-white p-6 rounded-lg shadow-lg text-center">
        <!-- Recommendation section -->
        <div class="mb-4 text-gray-600 text-sm">Channel: <span class="font-semibold">{{ responseData?.channel }}</span></div>
        <h2 class="text-2xl font-bold mb-4">Recommendation</h2>
        <ul class="text-left mb-6">
          <li class="mb-2"><span class="text-gray-800 font-semibold">Age:</span> {{ responseData?.recommendation?.age }}</li>
          <li class="mb-2"><span class="text-gray-800 font-semibold">Building:</span> {{ responseData?.recommendation?.building }}</li>
          <li class="mb-2"><span class="text-gray-800 font-semibold">Property Type:</span> {{ responseData?.recommendation?.property_type }}</li>
          <li class="mb-2"><span class="text-gray-800 font-semibold">Valuable:</span> {{ responseData?.recommendation?.valuable }}</li>
          <li class="mb-2"><span class="text-gray-800 font-semibold">Year of Construction:</span> {{ responseData?.recommendation?.year_of_construction }}</li>
        </ul>
        <button class="px-6 py-3 bg-blue-500 text-white rounded-full inline-block hover:bg-blue-600 transition-colors duration-300" @click="handleRestart">Restart</button>
      </div>
      <div v-else class="bg-white py-6 px-8 rounded-lg shadow-lg text-center">
        <!-- Message section -->
        <h1 class="text-md font-bold mb-4">{{ responseData?.message }}</h1>
        <button class="mt-4 px-4 py-2 bg-blue-500 text-white rounded inline-block" @click="handleRecommend">Recommend</button>
      </div>
    </div>
    <div v-else class="bg-white py-2 px-8 rounded-lg shadow-lg w-[80%] lg:w-[40rem] h-auto grid grid-cols-2 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <!-- Question section -->
      <template v-for="(question, index) in formFields" :key="question.label">
        <div class="bg-white py-6 px-4 rounded-lg shadow-lg">
          <h1 class="text-lg font-bold mb-4">{{ question.label }}</h1>
          <div v-for="option in question.options" :key="option" @click="selectAnswer(question, option)" class="flex items-center cursor-pointer mb-2">
            <div class="w-6 h-6 border border-gray-400 rounded-full mr-2 flex items-center justify-center" :class="{ 'bg-red-500': isSelected(question, option) }">
              <div v-if="isSelected(question, option)" class="w-4 h-4 bg-white rounded-full"></div>
            </div>
            <span>{{ option }}</span>
          </div>
        </div>
      </template>
      <button class="col-span-full mt-4 px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50" :disabled="isAnswered" @click="submitForm">{{ isLoading ? "Loading.." : "Submit" }}</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';

const formFields = ref([
  {
    label: 'Are you insuring your building?',
    options: ['Yes', 'No'],
    answer: ''
  },
  {
    label: 'Are you insuring valuable objects?',
    options: ['Yes', 'No'],
    answer: ''
  },
  {
    label: 'Insure building year of construction:',
    options: ['Under 10 years', '11-20 years', '21-30 years', '31-40 years', 'Over 40 years'],
    answer: ''
  },
  {
    label: 'Property type:',
    options: ['Business use', 'Personal use'],
    answer: ''
  },
  {
    label: 'Insure building susceptible to flooding:',
    options: ['Yes', 'No'],
    answer: ''
  },
  {
    label: 'Sex:',
    options: ['Female', 'Male'],
    answer: ''
  },
  {
    label: 'Number of bedrooms:',
    options: ['5', '4', '2', '1'],
    answer: ''
  },
  {
    label: 'Age:',
    options: ['Under 25', '26-50', '51-75', '76-100', 'Over 100'],
    answer: ''
  },
]);

const isLoading = ref(false);
const isSuccess = ref(false);
const responseData = ref({});
const isReconmmend = ref(false);

const selectedAnswer = computed(() => formFields.value.filter(question => question.answer));

const handleRecommend = () => {
  isReconmmend.value = true;
};

const handleRestart = () => {
  isSuccess.value = false;
  formFields.value.forEach(question => {
    question.answer = '';
  });
  isReconmmend.value = false;
};

const selectAnswer = (question, answer) => {
  question.answer = answer;
};

const isSelected = (question, option) => {
  return question.answer === option;
};

const isAnswered = computed(() => {
  return formFields.value.some(question => question.answer === '');
});

const submitForm = async() => {
  const formattedAnswers = {};

  formFields.value.forEach((question) => {
    let key = question.label.split(':')[0].trim().toLowerCase().replace(/\s+/g, '_');
    let value = question.answer;

    if (key === 'insure_building_year_of_construction') {
      switch (value) {
        case 'Under 10 years':
          value = 5;
          break;
        case '11-20 years':
          value = 4;
          break;
        case '21-30 years':
          value = 3;
          break;
        case '31-40 years':
          value = 2;
          break;
        case 'Over 40 years':
          value = 1;
          break;
      }
      key = 'year_of_construction';
    }

    if (key === 'age') {
      switch (value) {
        case 'Under 25':
          value = 1;
          break;
        case '26-50':
          value = 2;
          break;
        case '51-75':
          value = 3;
          break;
        case '76-100':
          value = 4;
          break;
        case 'Over 100':
          value = 5;
          break;
      }
    }

    if (key === 'number_of_bedrooms') {
      value = parseInt(value); // Convert to number
    }

    formattedAnswers[key] = value;
  });

  const formData = {
    age: formattedAnswers?.age,
    valuable: formattedAnswers?.are_you_ensuring_valuable_objects === "Yes" ? true : false,
    flooding: formattedAnswers?.insure_building_susceptible_to_flooding === "Yes" ? true : false,
    building: formattedAnswers?.are_you_insuring_your_building === "Yes" ? true : false,
    sex: formattedAnswers?.sex,
    property_type: formattedAnswers?.property_type,
    bedrooms: formattedAnswers?.number_of_bedrooms,
    year_of_construction: formattedAnswers?.year_of_construction
  };

  isLoading.value = true;

  try {
    const response = await axios.post('https://theinsuranceapp-1263194a4a6e.herokuapp.com/api/insure', formData);
    isLoading.value = false;
    isSuccess.value = true;
    responseData.value = response?.data;
    // console.log(response?.data);
  } catch (error) {
    // console.log(error, "errorror")
  }
};
</script>

<style scoped>
/* Add custom styles here if needed */
</style>
