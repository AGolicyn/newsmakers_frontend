<template>
  <header class="navbar navbar-expand-lg bd-navbar sticky-top">
    <nav class="container-xxl bd-gutter flex-wrap flex-lg-nowrap" aria-label="Main navigation">
        <div class="bd-navbar-toggle">
          <h1>NEWSMAKERS</h1>
        </div>
    </nav>
  </header>
  <div class="main">
    <form @submit.prevent>
      <div>
        <label for="main-calendar">Date:&nbsp;</label>
        <input id="main-calendar" type="date" v-model="CurrentDate">
      </div>
      <div class="main-menu">
        <label for="main-country">Country: &nbsp;</label>
        <select v-model="selectedCountry" id="v-model-select">
          <option disabled value="">Choose country</option>
          <option v-for="option in countrySelector" :value="option.country" :key="option.value">
            {{ option.country }}
          </option>
        </select>
      </div>
      <button v-if="selectedCountry" class="main-menu" @click="getEntities">Submit</button>
      <button disabled v-else class="main-menu">Submit</button>
    </form>
    <div>
      <div v-if="frameTitles" class="container text-center">
        <label for="text-container"><strong>Text</strong></label>
        <div class="row justify-content-md-center" id="text-container">
          <div class="col col-lg-2">
            <label for="main-locations">Locations</label>
            <div id="main-locations" class="list-group" v-for="(index, name) in entities.LOC" :key="index">
              <a @click="getNews(name, 'LOC')" href="#" class="list-group-item d-flex list-group-item-action">
                <div class="d-flex w-100 justify-content-between">
                  <small class="mb-1">{{ name }}</small>
                  <span class="badge bg-primary rounded-pill">{{ getLenEntity(name, 'LOC') }}</span>
                </div>
              </a>
            </div>
          </div>
          <div class="col col-lg-2">
            <label for="main-persons">Persons</label>
            <div id="main-persons" class="list-group" v-for="(index, name) in entities.PER" :key="index">
              <a @click="getNews(name, 'PER')" href="#" class="list-group-item d-flex list-group-item-action">
                <div class="d-flex w-100 justify-content-between">
                  <small class="mb-1">{{ name }}</small>
                  <span class="badge bg-primary rounded-pill">{{ getLenEntity(name, 'PER') }}</span>
                </div>
              </a>
            </div>
          </div>
          <div class="col col-lg-2">
            <label for="main-organizations">Organizations</label>
            <div id="main-organizations" class="list-group" v-for="(index, name) in entities.ORG" :key="index">
              <a @click="getNews(name, 'ORG')" href="#" class="list-group-item d-flex list-group-item-action">
                <div class="d-flex w-100 justify-content-between">
                  <small class="mb-1">{{ name }}</small>
                  <span class="badge bg-primary rounded-pill">{{ getLenEntity(name, 'ORG') }}</span>
                </div>
              </a>
            </div>
          </div>
          <div class="col col-xl">
            <label for="main-titles">News/Titles</label>
            <div id="main-titles" class="list-group" v-for="title in titles.data" :key="title">
              <a :href="title.href" class="list-group d-flex list-group-item-action">
                <small>{{ title.title }}</small>
              </a>
            </div>
          </div>
        </div>
        <label for="image-container"><strong>Wordcloud</strong></label>
        <div class="image-container" id="image-container">
          <picture>
            <img class="wordcloud"
                 v-bind:src="getPath('LOC')"
                 alt="LOCATIONS"
                 title="LOCATIONS WORDCLOUD">
            <img class="wordcloud"
                 v-bind:src="getPath('PER')"
                 alt="PERSONS"
                 title="PERSONS WORDCLOUD">
            <img class="wordcloud"
                 v-bind:src="getPath('ORG')"
                 alt="ORGANIZATIONS"
                 title="ORGANIZATIONS WORDCLOUD">
          </picture>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      entities: {},
      titles: [],
      CurrentDate: new Date().toISOString().slice(0,10),
      countrySelector: [
        {"value": 'RU', "country": 'Russia'},
        {"value": 'DE', "country": 'Germany'},
        {"value": 'US', "country": 'USA'},
      ],
      selectedCountry: '',
      frameTitles: false,
    }
  },
  methods: {
    getPath(entity) {
      let year = this.CurrentDate.split('-')[0]
      let month = this.CurrentDate.split('-')[1]-0
      let day = this.CurrentDate.split('-')[2]-0

      console.log(year, month, day)
      return require(`../../../media/images/${year}/${month}/${day}/${this.selectedCountry}/${entity}.webp`)
    },
    getEntities () {
      console.log(this.selectedCountry, this.CurrentDate)
      axios.post('http://0.0.0.0:8000/country', {
        country: this.selectedCountry,
        date: this.CurrentDate
      })
          .then((res) => {
            console.log(res)
            this.entities = res.data
            this.frameTitles = true
              })
          .catch((e) => {
            console.error(e)
          })
    },
    getNews(name, ent) {
      console.log(name, ent)
      console.log(Array.from(this.entities[ent][name]))
      axios.post('http://0.0.0.0:8000/titles', {
        entities: Array.from(this.entities[ent][name]),
      })
        .then((res) => {
          this.titles = res
          console.log(res)
            })
        .catch((e) => {
          console.error(e)
        })
    },
    getLenEntity(name, ent) {
      return (Array.from(this.entities[ent][name])).length
    },
  },
}

</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
.list-group-item {
  font-size: medium;
  width: 200px;
  padding-top: 4px;
  padding-bottom: 4px;

}
.mb-1 {
  margin-bottom: 0;
  font-size: 1em;
}
.main {

}
.main-menu {
  margin: 5px;
  display: inline-block;
}
.wordcloud {
  width: 30%;
}
</style>
