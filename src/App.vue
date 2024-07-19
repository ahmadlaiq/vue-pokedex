<template>
  <div class="container p-4 mx-auto">
    <h1 class="mb-8 text-4xl font-bold text-center">POKEDEX</h1>
    <div class="mb-5">
      <div class="flex justify-center mb-3">
        <input type="text"
          class="w-3/4 px-3 py-2 leading-tight text-gray-700 border border-gray-400 rounded shadow appearance-none focus:outline-none focus:shadow-outline"
          placeholder="Search Pokémon..." v-model="searchTerm" @input="debouncedSearch" />
      </div>
    </div>
    <div class="grid grid-cols-2 gap-4 mt-3 md:grid-cols-4 lg:grid-cols-6">
      <div v-for="pokemon in pokemons" :key="pokemon.name"
        class="p-4 bg-white rounded-lg shadow-md hover:bg-gray-600 hover:text-white">
        <img
          :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokemon.url.split('/')[6]}.png`"
          :alt="pokemon.name" class="mx-auto transition-transform duration-500 ease-in-out hover:scale-110" />
        <h3 class="text-lg font-semibold text-start">
          {{ pokemon.url.split('/')[6] }}
        </h3>
        <h2 class="mb-4 text-lg text-center capitalize">{{ pokemon.name }}</h2>
        <button @click="showPokemonDetails(pokemon.url)"
          class="text-white inline-flex w-full justify-center bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
          Detail
        </button>
      </div>
    </div>
    <div class="flex justify-center mt-4">
      <button v-if="page > 1" @click="changePage(page - 1)"
        class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700">
        Previous
      </button>
      <span class="p-2 mx-2">{{ page }} / {{ totalPages }}</span>
      <button v-if="page < totalPages" @click="changePage(page + 1)"
        class="px-4 py-2 font-bold text-white bg-gray-500 rounded hover:bg-gray-700">
        Next
      </button>
    </div>
  </div>

  <div v-if="showModal && selectedPokemon"
    class="fixed inset-0 z-50 flex items-center justify-center overflow-auto bg-black bg-opacity-50">
    <div class="relative w-3/4 max-w-md p-8 bg-white rounded-lg">
      <button class="absolute text-gray-600 top-2 right-2 hover:text-gray-800" @click="closeModal">
        <div
          class="inline-flex items-center justify-center w-8 h-8 text-sm text-gray-400 bg-transparent rounded-lg hover:bg-gray-200 hover:text-gray-900 ms-auto dark:hover:bg-gray-600 dark:hover:text-white"
          data-modal-toggle="select-modal">
          <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6" />
          </svg>
          <span class="sr-only">Close modal</span>
        </div>
      </button>
      <h2 class="mb-4 text-2xl font-bold text-center capitalize">{{ selectedPokemon.name }}</h2>
      <img :src="selectedPokemon.sprites.other['official-artwork'].front_default" :alt="selectedPokemon.name"
        class="w-48 h-48 mx-auto mb-4 transition-transform duration-500 ease-in-out hover:scale-110" />
      <div class="grid grid-cols-2 gap-2">
        <p><strong>Height:</strong> {{ selectedPokemon.height / 10 }} m</p>
        <p><strong>Weight:</strong> {{ selectedPokemon.weight / 10 }} kg</p>
        <p><strong>Base Experience:</strong> {{ selectedPokemon.base_experience }}</p>
        <p><strong>ID:</strong> {{ selectedPokemon.id }}</p>
      </div>
      <div class="mt-4 capitalize">
        <strong>Types:</strong>
        <span v-for="type in selectedPokemon.types" :key="type.type.name"
          class="inline-block px-3 py-1 mr-2 text-sm font-semibold text-white bg-gray-600 rounded-full">
          {{ type.type.name }}
        </span>
      </div>
      <div class="mt-4 capitalize">
        <strong>Abilities:</strong>
        <ul class="list-disc list-inside">
          <li v-for="ability in selectedPokemon.abilities" :key="ability.ability.name">
            {{ ability.ability.name }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const pokemons = ref([]);
    const allPokemons = ref([]);
    const page = ref(1);
    const totalPages = ref(0);
    const searchTerm = ref('');
    const selectedPokemon = ref(null);
    const showModal = ref(false);

    onMounted(async () => {
      const response = await axios.get('https://pokeapi.co/api/v2/pokemon?offset=0&limit=18');
      allPokemons.value = response.data.results;
      totalPages.value = Math.ceil(allPokemons.value.length / 18);
      updateDisplayedPokemons();
    });

    function updateDisplayedPokemons() {
      let filteredPokemons = allPokemons.value;
      if (searchTerm.value) {
        filteredPokemons = allPokemons.value.filter(
          (pokemon) =>
            pokemon.name.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
            pokemon.url.split('/')[6].includes(searchTerm.value)
        );
      }
      totalPages.value = Math.ceil(filteredPokemons.length / 18);
      page.value = 1;
      pokemons.value = filteredPokemons.slice(0, 18);
    }

    function debounce(func, timeout = 300) {
      let timer;
      return (...args) => {
        clearTimeout(timer);
        timer = setTimeout(() => {
          func.apply(this, args);
        }, timeout);
      };
    }

    const debouncedSearch = debounce(() => {
      updateDisplayedPokemons();
    });

    function changePage(newPage) {
      page.value = newPage;
      let filteredPokemons = allPokemons.value;
      if (searchTerm.value) {
        filteredPokemons = allPokemons.value.filter(
          (pokemon) =>
            pokemon.name.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
            pokemon.url.split('/')[6].includes(searchTerm.value)
        );
      }
      const startIndex = (page.value - 1) * 18;
      pokemons.value = filteredPokemons.slice(startIndex, startIndex + 18);
    }

    async function showPokemonDetails(pokemonUrl) {
      const id = pokemonUrl.split('/')[6];
      const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`);
      selectedPokemon.value = response.data;
      showModal.value = true;
    }

    function closeModal() {
      showModal.value = false;
      selectedPokemon.value = null;
    }

    return {
      pokemons,
      page,
      totalPages,
      searchTerm,
      selectedPokemon,
      showModal,
      debouncedSearch,
      changePage,
      showPokemonDetails,
      closeModal
    };
  }
};
</script>

<style>
@import 'tailwindcss/tailwind.css';
</style>