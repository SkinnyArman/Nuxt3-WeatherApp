<template>
  <div
    class="d-flex flex-column p-14 bg-gradient-to-r from-purple-300 via-purple-400 to-pink-300 h-screen"
  >
    <div class="flex justify-between w-full">
      <div v-if="!pending && !error">
        <div
          class="flex items-center justify-center bg-purple-500 rounded-full py-2 px-4"
        >
          <span class="text-3xl text-white font-bold">{{ data.name }}</span>
        </div>
      </div>
      <div class="bg-gray-200 rounded-lg py-2 px-4">
        <div>
          <div v-if="pending" class="inline-flex items-center justify-center">
            <svg
              class="animate-spin text-purple-500 h-4 w-4 mr-2"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 4.411 3.589 8 8 8v-2.009zm16-5.282A7.963 7.963 0 0120 12h4c0-4.411-3.589-8-8-8v2.009z"
              ></path>
            </svg>
            <span class="text-purple-500 font-medium">Loading..</span>
          </div>
          <p
            v-else-if="error"
            class="text-red-700 px-4 py-3 rounded relative"
            role="alert"
          >
            <strong class="font-bold">No Such City Exists.</strong>
          </p>
          <span v-else>
            <span class="text-4xl text-purple-500 font-bold mr-2"
              >{{ data.main.temp }}&deg;</span
            >
            <span class="text-lg text-gray-500 font-medium">{{
              isCelsius ? "Celsius" : "Fahrenheit"
            }}</span>
          </span>
        </div>
      </div>
    </div>
    <div class="flex justify-end mt-5">
      <input type="checkbox" id="toggle" class="sr-only" v-model="isCelsius" />
      <div>
        <label for="toggle" class="flex items-center cursor-pointer">
          <div class="relative">
            <div class="block bg-purple-500 w-14 h-8 rounded-full"></div>
            <div
              class="dot absolute left-1 top-1 bg-white w-6 h-6 rounded-full transition"
              :class="{ 'translate-x-full': isCelsius }"
            ></div>
          </div>
          <div class="ml-3 text-white font-medium">
            <span v-if="isCelsius">
              <span class="hidden sm:inline-block">Show in Fahrenheit</span>
              <span class="sm:hidden">°F</span>
            </span>
            <span v-else>
              <span class="hidden sm:inline-block">Show in Celsius</span>
              <span class="sm:hidden">°C</span>
            </span>
          </div>
        </label>
      </div>
    </div>
    <form class="flex relative rounded-md mt-5" @submit.prevent="submitSearch">
      <input
        v-model="input"
        type="text"
        name="City"
        placeholder="City Name"
        class="bg-gray-200 rounded-md py-2 px-4 w-full block appearance-none focus:outline-none focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-transparent mr-5"
      />
      <button
        class="bg-purple-500 hover:bg-purple-600 focus:bg-purple-400 disabled:bg-gray-400 disabled:opacity-50 rounded-md px-4 py-2 text-white font-medium"
        :disabled="!input.trim()"
        type="submit"
        @click="submitSearch"
      >
        Search!
      </button>
    </form>
  </div>
</template>

<script setup>
const config = useRuntimeConfig();
const input = ref("");
const isCelsius = ref(true);
const searchValue = ref("");
const cityName = useCookie("city");

if (!cityName.value) {
  cityName.value = "Tehran";
  searchValue.value = "Tehran";
} else {
  searchValue.value = cityName.value;
}

const submitSearch = () => {
  if (input.value.trim()) {
    searchValue.value = input.value.split(" ").join("+");
  }
};

const { data, pending, error } = useFetch(
  () =>
    `https://api.openweathermap.org/data/2.5/weather?q=${
      searchValue.value
    }&units=${isCelsius.value ? "metric" : "imperial"}&appid=${
      config.public.apiSecret
    }`,
  {
    onResponse({ request, response, options }) {
      if (response.ok) cityName.value = searchValue.value;
    },
    onResponseError({ request, response, options }) {
      // Handle the response errors
    },
  }
);
</script>
