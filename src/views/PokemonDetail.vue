<template>

</template>

<script setup>
import { computed, onMounted, ref, watch } from 'vue';
import { useRoute, useRouter } from '../router/shim';

const route = useRoute();
const router = useRouter();

const pokemon = ref(null);
const species = ref(null);
const description = computed(() => {
  const entries = species.value?.flavor_text_entries || [];
  const spanish = entries.find((entry) => entry.language.name === 'es');
  const english = entries.find((entry) => entry.language.name === 'en');
  const text = spanish?.flavor_text || english?.flavor_text || '';
  return text.replace(/\s+/g, ' ').replace(/\f/g, ' ').trim();
});

const capitalizedName = computed(() => {
  if (!pokemon.value) return '';
  return pokemon.value.name.charAt(0).toUpperCase() + pokemon.value.name.slice(1);
});

const displayedMoves = computed(() => {
  if (!pokemon.value) return [];
  return pokemon.value.moves.slice(0, 6);
});

const fetchPokemon = async (id) => {
  pokemon.value = null;
  species.value = null;
  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
    const data = await response.json();
    pokemon.value = data;
    const speciesResponse = await fetch(data.species.url);
    species.value = await speciesResponse.json();
  } catch (error) {
    console.error('Error al cargar el pokémon', error);
  }
};

const goBack = () => {
  router.push('/');
};

onMounted(() => {
  fetchPokemon(route.value.params.id);
});

watch(
  () => route.value.params.id,
  (newId) => {
    if (newId) {
      fetchPokemon(newId);
    }
  }
);
</script>
