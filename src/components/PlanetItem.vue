<template>
  <div class="planet-item">
    <div class="planet-item__cell">{{planet.name}}</div>
    <div class="planet-item__cell">{{planet.climate}}</div>
    <div class="planet-item__cell">{{planet.orbital_period}}</div>
    <div class="planet-item__cell">{{planet.population}}</div>
    <div class="planet-item__cell">{{planet.rotation_period}}</div>
    <div class="planet-item__cell">{{dateFormat(planet.created)}}</div>
  </div>
</template>

<script>
import moment from 'moment'
export default {
  name: 'PlanetItem',
  props: {
    planet: {}
  },
  methods: {
    dateFormat: function(dateString) {
      return moment(String(dateString)).format('DD/MM/YYYY')
    }
  }
}

</script>

<style scoped>
  .planet-item {
    display: block;
    border-bottom: 1px solid #CCC;
    padding: 10px 5px;
    margin-right: 10px;
    text-align: left;
    display: flex;
    flex-wrap: nowrap;
    justify-content: space-between;
    font-size: 12px;
  }
  .planet-item__cell {
    width: 16%;
  }
</style>
