<template>
  <div id="app">
    <PlanetsList page="1"/>
  </div>
</template>

<script>
import PlanetsList from './components/PlanetsList.vue'

export default {
  name: 'App',
  components: {
    PlanetsList
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
  margin-top: 60px;
}
</style>
