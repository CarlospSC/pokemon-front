<template>
  <PokemonBattle/>
</template>

<script>
import PokemonBattle from './components/PokemonBattle.vue'

export default {
  name: 'App',
  components: {
    PokemonBattle
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: #00CAF6;
}
body, h1 {
  margin: 0;
}

</style>
