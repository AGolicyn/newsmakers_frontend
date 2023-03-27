<template>
<body class="d-flex flex-column min-vh-100">
  <header class="navbar navbar-expand-lg bd-navbar sticky-top">
    <nav class="container-xxl bd-gutter flex-wrap flex-lg-nowrap" aria-label="Main navigation">
      <div class="bd-navbar-toggle">
        <form @submit.prevent>
          <div class="main-menu">
            <span class="main-title">NEWSMAKERS&nbsp;</span>
            <label for="main-calendar">Date:&nbsp;</label>
            <input id="main-calendar" type="date" v-model="CurrentDate" @change="getEntities">
          </div>
          <div class="main-menu">
            <label for="main-country">Country: &nbsp;</label>
            <select v-model="selectedCountry" id="v-model-select" @change="getEntities">
              <option v-for="option in countrySelector" :value="option.country" :key="option.value">
                {{ option.country }}
              </option>
            </select>
          </div>
        </form>
      </div>
    </nav>
  </header>
  <div class="main">
    <div>
      <div v-if="frameTitles" class="container text-center">
        <label for="image-container"><strong>Wordcloud</strong></label>
        <div class="image-container" id="image-container">
          <picture>
            <img class="wordcloud" id="wordcloud-locations"
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
        <label for="text-container"><strong>Text</strong></label>
        <div class="row justify-content-md-center" id="text-container">
          <div class="col col-lg-2">
            <label for="main-locations">Locations</label>
            <div id="main-locations" class="list-group" v-for="(index, name) in entities.LOC" :key="index">
              <a @click="getNews(name, 'LOC', $event)" href="#" class="list-group-item d-flex list-group-item-action">
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
              <a @click="getNews(name, 'PER', $event)" href="#" class="list-group-item d-flex list-group-item-action">
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
              <a @click="getNews(name, 'ORG', $event)" href="#" class="list-group-item d-flex list-group-item-action">
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
              <a :href="title.href" class="list-group d-flex list-group-item-action" target="_blank">
                <div class="d-flex justify-content-between">
                  <small class="title-item">{{ title.title }}</small>
                </div>
              </a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <footer class="d-flex flex-wrap justify-content-between align-items-center mt-auto">
    <a href="https://github.com/CodNaoborot" target="_blank" class="contacts mb-3 me-2 mb-md-0 text-muted text-decoration-none lh-1">GitHub</a>
  </footer>
</body>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  created() {
    this.getEntities()
  },
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
      selectedCountry: "Russia",
      frameTitles: false,
    }
  },
  methods: {
    getPath(entity) {
      let year = this.CurrentDate.split('-')[0]
      let month = this.CurrentDate.split('-')[1]-0
      let day = this.CurrentDate.split('-')[2]-0

      let base_path = `images/${year}/${month}/${day}/${this.selectedCountry}/${entity}.webp`

      if (process.env.NODE_ENV === 'production') {
        return base_path
      }
      else {
        return require(`../../media/`+base_path)
      }
    },
    getEntities () {
      let url
      if (process.env.NODE_ENV === 'production') {
        url = 'https://newsmakers.online/api/country'
      }
      else {
        url = 'http://0.0.0.0:8000/country'
      }
      console.log(url)
      console.log(this.selectedCountry, this.CurrentDate)
      axios.post(url, {
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
    getNews(name, ent, event) {
      console.log(name, ent, event)
      event.preventDefault()
      console.log(Array.from(this.entities[ent][name]))
      let url
      if (process.env.NODE_ENV === 'production') {
        url = 'https://newsmakers.online/api/titles'
      }
      else {
        url = 'http://0.0.0.0:8000/titles'
      }
      axios.post(url, {
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
  padding: 10px 10px 4px;
  color: bisque;
  background-color: #303b44;
}
.navbar {
  padding-top: 0;
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
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.title-item {
  margin-left: 20px;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
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
footer {
  position: relative;
  height: 50px;
  bottom: 0;
  width: 100%;
  background-color: #303b44;
  color: bisque;
}
.title-item {
  overflow: hidden;
}
.main-title {
  margin-right: 100px;
  font-size: x-large;
}
.contacts {
  padding-left: 100px;
}
#text-container {
  margin-bottom: 20px;
}

</style>

