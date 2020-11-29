<template>
  <div class="container">
    <div class="planets-page__header">
      <input type="text" class="planets-page__search" placeholder="Поиск по названию..." v-model="searchQuery">
      <div class="planets-page__filters">
        <div class="planets-page__filters-item">
          <strong>Climate:</strong>
          <label v-for="(climateVar , index) in filtersClimate" :key="index">
            <input 
              type="checkbox"               
              :value="climateVar.name" 
              @change="setFilter($event)"
              />{{climateVar.name}}
          </label>
        </div>
      </div>
    </div>    
    <div class="planets-page__list">
      <div class="planet-page__list-header">
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'name'}" @click="sortColumn('name')">Name</div>
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'climate'}" @click="sortColumn('climate')">Climate</div>
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'orbital_period'}" @click="sortColumn('orbital_period')">Orbital period</div>
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'population'}" @click="sortColumn('population')">Population</div>
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'rotation_period'}" @click="sortColumn('rotation_period')">Rotation</div>
        <div class="planet-item__list-header-cell" :class="{ 'planet-item__list-header-cell--active' : sortedField == 'created'}" @click="sortColumn('created')">Created</div>
      </div>
      <div class="planets-page__list-loader">
        <Loader v-if="loader" />
      </div>
      <PlanetItem v-for="planet in planets" :key="planet.name" :planet = "planet" />
    </div>
    <div class="planets-page__paginator" v-if="showPaginator">
      <div v-for="pageNum in pagesCount" 
          :key="pageNum" 
          class="planets-page__paginator-item" 
          :class="{'planets-page__paginator-item--active' : currentPage == pageNum}"
          @click="changePage(pageNum)"
          >
        {{pageNum}}
      </div>
    </div>
  </div>
</template>

<script>
import _debounce from 'lodash.debounce'
import axios from 'axios'
import PlanetItem from '../components/PlanetItem'
import Loader from '../components/Loader'

const ITEMS_ON_PAGE = 10

export default {
  name: 'PlanetsList',
  components: { PlanetItem, Loader },
  props: {
    page: String
  },
  data: function() {
    return {
      loadedPlanets: [],
      planets: [],
      filtersClimate: [],
      searchQuery: '',
      loader: true,
      currentPage: this.page,
      pagesCount: 0,
      sortedField: '',
      showPaginator: true
    }
  },
  created () {
    const getAllPlanets = () => {
      axios.get(`http://swapi.dev/api/planets/?page=${this.page}`).then(res => {
        this.planets = res.data.results
        this.loadedPlanets = res.data.results
        this.loader = false
        this.pagesCount = Math.round(res.data.count / ITEMS_ON_PAGE )
        this.getClimateFilters()
        console.log(res.data)        
      }, err => {
        throw err
      })    
    }
    getAllPlanets()    
  },
  methods: {
    getClimateFilters: function() {
      this.filtersClimate = []
      let filtersClimateNames = []
      this.planets.map(item => {
        let climateArr = item.climate.split(',')
        climateArr.map(climateItem => {
          if(filtersClimateNames.indexOf(climateItem) == -1) {
            filtersClimateNames.push(climateItem.trim())
            this.filtersClimate.push({name: climateItem.trim(), chacked: false})
          }
        })
      })
    },
    sortColumn: function(columnName) {
      let sortedPlanets = this.planets.sort((a, b) => {
        if(['orbital_period', 'rotation_period', 'population'].indexOf(columnName) != -1) {
          return parseInt(a[columnName]) > parseInt(b[columnName]) ? 1 : -1
        } else {
          return a[columnName].toUpperCase() > b[columnName].toUpperCase() ? 1 : -1
        }
        
      })
      this.planets = sortedPlanets
      this.sortedField = columnName
    },
    setFilter: function(e) {
      let changedFilters = this.filtersClimate
      changedFilters.map((item, index )=> {
        if(item.name == e.target.value) {
          this.filtersClimate[index] = {
            name: item.name, checked: e.target.checked
          }
        }
      })
      this.filtersClimate = changedFilters
      
      let filterArr = []
      this.filtersClimate.map(item => {
        if(item.checked) { filterArr.push(item.name) }
      })

      if(filterArr.length > 0) {
        let filteredPlanets = this.planets
        this.planets = filteredPlanets.filter(planetProperty => { 
          let planetClimats = planetProperty.climate.split(',')
          let isFound = false
          console.log(planetClimats)
          planetClimats.map(item => {
            if(filterArr.indexOf(item.trim()) != -1) {
              isFound = true
            }
          })
          return isFound
        })
      } else {
        this.planets = this.loadedPlanets
        if(this.sortedField) {
          this.sortColumn(this.sortedField)
        }
      }
    },
    search: _debounce(function () {
      this.showPaginator = false
      axios.get(`https://swapi.dev/api/planets/?search=${this.searchQuery}`).then(res => {
        this.planets = res.data.results
        this.getClimateFilters()
      }, err => {
        throw err
      })
    }, 200),    
    clearResults: function(){
      this.showPaginator = true
      this.planets = this.loadedPlanets
      if(this.sortedField) {
        this.sortColumn(this.sortedField)
      }
      this.getClimateFilters()
    },
    changePage: function(pageNumber) {
      axios.get(`http://swapi.dev/api/planets/?page=${pageNumber}`).then(res => {        
        this.planets = res.data.results
        this.loadedPlanets = res.data.results
        this.currentPage = pageNumber
        this.getClimateFilters()
        this.sortedField = ''
      }, err => {
        throw err
      }) 
      this.getClimateFilters()
    }
  },
  watch: {
    searchQuery (value) {
      if (value) this.search()
      else this.clearResults()
    },
    selectedEntity () {
      if (this.query) this.search()
    }
  }
}
</script>

<style scoped>
  .container {
    width: 800px;
    margin: 0 auto;  
  }
  .planets-page__list {
    width: 100%;
  }
  .planet-page__list-header {
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
  .planet-item__list-header-cell {
    width: 16%;    
    font-weight: bold;
    cursor: pointer;
  }
  .planet-item__list-header-cell--active {
    color: #CCC;
  }
  .planets-page__paginator {
    padding: 10px 0;
  }
  .planets-page__paginator-item {
    display: inline-block;
    margin-left: 10px;
    cursor: pointer;
  }
  .planets-page__paginator-item--active {
    font-weight: bold;
  }
  .planets-page__search {
    padding: 5px 10px;
    border: 1px solid #CCC;
    border-radius: 3px;
    width: 50%;
    margin-bottom: 10px;
  }
  .planets-page__filters {
    display: flex;
  }
  .planets-page__filters-item {
    width: 30%;
    text-align: left;
  }
  .planets-page__filters-item label {
    display: block;
    cursor: pointer;
  }
</style>
