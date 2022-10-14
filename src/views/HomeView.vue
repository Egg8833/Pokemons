<template>
  <div class="w-full flex justify-center">
    <input
      type="text"
      placeholder="Enter Pokemon here"
      class="mt-5 p-2 border-blue-500 border-2"
      v-model="text"
      ref="pokemonText"
    />
  </div>
  <div class="mt-10 p-5 flex flex-wrap justify-center">
    <div
      class="ml-4 mb-2 text-2x text-blue-500 text-center border-2 shadow-1xl"
      v-for="(pokemon, idx) in filteredPokemon"
      :key="idx"
    >
      <router-link :to="`/about/${urlIdLookup[pokemon.name]}`">
        {{ pokemon.name }}
        <img :src="pokemon.png" alt="" />
      </router-link>
    </div>
    <div v-if="isLoad" class="w-40 flex justify-center">
      <div v-for="(item, index) in newpokemons" v-show="imgI == index">
        <h3 class="text-2xl text-green-900 text-center">{{ item.name }}</h3>
        <img class="w-40" :src="item.png" alt="" />
      </div>
    </div>
  </div>
</template>

<script>
import { computed, onMounted, reactive, toRefs, ref } from "vue";
export default {
  name: "HomeView",
  setup() {
    const pokemonText = ref(null);
    const state = reactive({
      newpokemons: [],
      pokemonPng: [],
      pokemons: [],
      urlIdLookup: {},
      text: "",
      filteredPokemon: computed(() => updatePokemon()),
    });
    const isLoad = ref(true);
    const imgI = ref(0);

    function updatePokemon() {
      state.newpokemons = state.pokemons.map((item, i) => {
        return { ...item, png: state.pokemonPng.at(i) };
      });

      if (!state.text) {
        isLoad.value = true;
        return [];
      }
      isLoad.value = false;
      console.log(state);

      return state.newpokemons.filter((pokemon) => {
        return pokemon.name.includes(state.text);
      });
    }

    onMounted(() => {
      fetch("https://pokeapi.co/api/v2/pokemon?offset=0")
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          console.log(data);
          state.pokemons = data.results;
          state.urlIdLookup = data.results.reduce(
            (acc, cur, idx) => (acc = { ...acc, [cur.name]: idx + 1 }),
            {}
          );
          console.log(state);
        });

      // 拿每隻寶可夢的照片
      for (let i = 1; i <= 20; i++) {
        fetch(`https://pokeapi.co/api/v2/pokemon-form/${i}/`)
          .then((res) => res.json())
          .then((data) => {
            state.pokemonPng.push(data.sprites.front_shiny);
          });
      }
      console.log(state);
      pokemonText.value.focus();

      // 每個寶可夢切換
      setInterval(() => {
        if (imgI.value < 19) {
          imgI.value++;
        } else {
          imgI.value = 0;
        }
      }, 4000);
    });

    return { ...toRefs(state), pokemonText, isLoad, imgI };
  },
};
</script>
