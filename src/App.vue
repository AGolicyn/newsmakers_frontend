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
          <div class="main-menu">
            <button v-if="showWordCloud" @click="showWordCloud = !showWordCloud" class="btn btn-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-cloud" aria-expanded="false" aria-controls="PersonsCollapse OrganizationsCollapse">Hide word-clouds</button>
            <button v-else @click="showWordCloud = !showWordCloud" class="btn btn-outline-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-cloud" aria-expanded="false" aria-controls="PersonsCollapse OrganizationsCollapse">Show word-clouds</button>
            <button v-if="showChartBar" @click="showChartBar = !showChartBar" class="btn btn-outline-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-chart" aria-expanded="false" aria-controls="ChartCollapse" ref="btn">Show chart</button>
            <button v-else @click="showChartBar = !showChartBar" class="btn btn-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-chart" aria-expanded="false" aria-controls="ChartCollapse">Hide chart</button>
            <button v-if="showTextInfo" @click="showTextInfo = !showTextInfo" class="btn btn-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-text" aria-expanded="false" aria-controls="TextCollapse">Hide text-info</button>
            <button v-else @click="showTextInfo = !showTextInfo" class="btn btn-outline-light wordcloud-button" type="button" data-bs-toggle="collapse" data-bs-target=".multi-collapse-text" aria-expanded="false" aria-controls="TextCollapse">Show text-info</button>
          </div>
        </form>
      </div>
    </nav>
  </header>
  <div class="main">
    <div>
      <div v-if="frameTitles" class="container text-center">
      <div v-if="showWordCloud" class="text-secondary"><h5>Today's word-clouds</h5></div>
      <div class="row">
        <div class="col">
          <div aria-expanded="true" class="collapse multi-collapse-cloud show" id="LocationCollapse">
            <div class="card card-body">
              <span class="cloud-title">States, cities</span>
              <div class="image-container" id="image-container">
                <picture>
                  <img class="wordcloud" id="wordcloud-locations"
                 v-bind:src="getPath('LOC')"
                 alt="LOCATIONS"
                 title="LOCATIONS WORDCLOUD">
                </picture>
              </div>
            </div>
          </div>
        </div>
        <div class="col">
          <div class="collapse multi-collapse-cloud show" id="PersonsCollapse">
            <div class="card card-body">
              <span class="cloud-title">People</span>
              <div class="image-container" id="image-container">
                <picture>
                  <img class="wordcloud"
                    v-bind:src="getPath('PER')"
                    alt="PERSONS"
                    title="PERSONS WORDCLOUD">
                </picture>
              </div>
            </div>
          </div>
        </div>
        <div class="col">
          <div class="collapse multi-collapse-cloud show" id="OrganizationsCollapse">
            <div class="card card-body">
              <span class="cloud-title">Organizations</span>
              <div class="image-container" id="image-container">
                <picture>
                  <img class="wordcloud"
                    v-bind:src="getPath('ORG')"
                    alt="ORGANIZATIONS"
                    title="ORGANIZATIONS WORDCLOUD">
                </picture>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-if="!showChartBar" class="text-secondary"><h5>Entity monthly trend</h5></div>
      <div class="row">
        <div class="col">
          <div aria-expanded="true" class="collapse multi-collapse-chart" id="ChartCollapse">
            <div class="card card-body">
              <div v-if="showChart" class="chart-container">
                <div class="chart-view">
                  <Line :data="chartData" :options="chartOptions" ref="line"/>
                  <form @submit.prevent class="form-inline">
                      <div class="form-row row">
                        <div class="form-group col-md-3">
                          <input v-model="token" placeholder="Enter token, ex.: Москва" class="form-control input-size">
                        </div>
                        <div class="form-group col-md-3">
                          <button v-if="tokenLength < 2" disabled @click="addDatasetChart(token)" class="btn btn-secondary mb-2 btn-chart">Add</button>
                          <button v-else @click="addDatasetChart(token)" class="btn btn-secondary mb-2 btn-chart">Add</button>
                          <button type="submit" @click="deleteChart" class="btn btn-secondary mb-2 btn-chart">Delete</button>
                        </div>
                      </div>
                  </form>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-if="showTextInfo" class="text-secondary"><h5>Main entities</h5></div>
      <div class="row">
        <div class="col">
          <div aria-expanded="true" class="collapse multi-collapse-text show" id="TextCollapse">
            <div class="card card-body">
              <div class="row justify-content-md-center" id="text-container">
                <div class="col col-lg-2">
                  <label class="cloud-title" for="main-locations">Locations</label>
                  <div id="main-locations" class="list-group" v-for="(index, name) in entities.LOC" :key="index">
                    <a @click="getNews(name, 'LOC', $event)"
                       href="#"
                       class="list-group-item d-flex list-group-item-action">
                      <div class="d-flex w-100 justify-content-between">
                        <small class="mb-1">{{ name }}</small>
                        <span class="badge bg-secondary rounded-pill">{{ getLenEntity(name, 'LOC') }}</span>
                      </div>
                    </a>
                  </div>
                </div>
                <div class="col col-lg-2">
                  <label class="cloud-title" for="main-persons">Persons</label>
                  <div id="main-persons" class="list-group" v-for="(index, name) in entities.PER" :key="index">
                    <a @click="getNews(name, 'PER', $event)" href="#" class="list-group-item d-flex list-group-item-action">
                      <div class="d-flex w-100 justify-content-between">
                        <small class="mb-1">{{ name }}</small>
                        <span class="badge bg-secondary rounded-pill">{{ getLenEntity(name, 'PER') }}</span>
                      </div>
                    </a>
                  </div>
                </div>
                <div class="col col-lg-2">
                  <label class="cloud-title" for="main-organizations">Organizations</label>
                  <div id="main-organizations" class="list-group" v-for="(index, name) in entities.ORG" :key="index">
                    <a @click="getNews(name, 'ORG', $event)" href="#" class="list-group-item d-flex list-group-item-action">
                      <div class="d-flex w-100 justify-content-between">
                        <small class="mb-1">{{ name }}</small>
                        <span class="badge bg-secondary rounded-pill">{{ getLenEntity(name, 'ORG') }}</span>
                      </div>
                    </a>
                  </div>
                </div>
                <div class="col col-xl">
                  <label class="cloud-title" for="main-titles">News/Titles</label>
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
      </div>
    </div>
  </div>
  </div>
  <footer class="d-flex flex-wrap justify-content-between align-items-center mt-auto">
    <a href="https://github.com/AGolicyn" target="_blank" class="contacts mb-3 me-2 mb-md-0 text-muted text-decoration-none lh-1">GitHub</a>
  </footer>
</body>
</template>

<script>
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

import { Line } from 'vue-chartjs'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

import axios from 'axios';

export default {
  name: 'App',
  created() {
    this.getEntities()
    document.cookie = "new=Cook"
  },
  computed: {
     chartData() {return this.cData},
     chartOptions() {return this.cOptions},
     tokenLength() {return this.token.length},
  },
  components: {
    Line
  },
  data() {
    return {
      COLORS: [ '#4dc9f6', '#f67019', '#f53794', '#537bc4', '#acc236',
                '#166a8f', '#00a950', '#58595b', '#8549ba' ],
      token: '',
      showWordCloud: true,
      showTextInfo: true,
      showChart: true,
      showChartBar: true,
      chart_axes: [],
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
      cOptions: {
        responsive: true,
        maintainAspectRatio: false,
        type: Object,
        default: () => {}
      },
      cData: {
        datasets: [
          {
            data: []
          }
        ]
      }
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
    async getChartData(name) {
      this.token = ''
      let url
      if (process.env.NODE_ENV === 'production') {
        url = 'https://newsmakers.online/api/trend'
      }
      else {
        url = 'http://0.0.0.0:8000/trend'
      }
      console.log(url)
      return axios.post(url, {
        country: this.selectedCountry,
        date: this.CurrentDate,
        token: name,
        day_offset: 30,
      })
        .then((res) => {
            this.chart_axes.push(res.data)
            console.log(res)
            })
        .catch((e) => {
          console.error(e)
        })
    },
    async createChart(token) {
      if (!(this.$refs.btn === null)) {
          this.$refs.btn.click()
      }
      this.chart_axes = []
      console.log("try to get data")
      await this.getChartData(token)
      console.log("New AXES", this.chart_axes[0])

      let chart_labels = []
      let chart_axis = []
      for (let item of this.chart_axes[0]) {
            chart_axis.push(item["weight"])
            chart_labels.push(item["date"])
          }
      chart_axis.reverse()
      chart_labels.reverse()
      console.log(chart_labels)
      console.log(chart_axis)

      this.cData = {
        ...this.cData,
        labels: chart_labels,
        datasets: [{
            "data": chart_axis,
            "label": token,
            "borderColor": this.COLORS[0],
            "backgroundColor": this.COLORS[0],
        }]
      }
      console.log("NEW CHART", this.cData)
      this.showChart = true
      return this.cData

    },
    async addDatasetChart(token) {
        this.showChartBar = false
        if (this.cData.labels === undefined) {
            return this.createChart(token)
        }
        await this.getChartData(token)
        let chart_axis = []
        let len = this.chart_axes.length - 1
        for (let item of this.chart_axes[len]) {
              chart_axis.push(item["weight"])
            }
        chart_axis.reverse()
        console.log(chart_axis)

        this.cData = {
          ...this.cData,
          datasets: this.cData.datasets.concat({
              "data": chart_axis,
              "label": token,
              "borderColor": this.COLORS[len],
              "backgroundColor": this.COLORS[len],
          }),
        }
        console.log("NEW CHART", this.cData)
        this.showChart = true
        return this.cData
    },
    deleteChart() {
        this.cData = {
            ...this.cData,
            datasets: this.cData.datasets.slice(0, this.cData.datasets.length - 1)
        }
        this.chart_axes.pop()

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
            })
        .catch((e) => {
          console.error(e)
        })
      this.createChart(name)
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
  width: 100%;
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
.chart-view {
    display: block;
    box-sizing: border-box;
    height: 250px;
    width: 100%;
}
.wordcloud-button {
    position: relative;
    left: 30%;
}
.btn-secondary {
    --bs-btn-bg: #303b44;
}
.btn-light {
    margin-right: 10px;
    padding: 3px 6px;
}
.btn-outline-light {
    margin-right: 10px;
    padding: 3px 6px;
}
.chart-container {
    margin: 0 0 30px 0;
    padding-bottom: 10px;
}
.col {
    margin-bottom: 20px;
}
.cloud-title {
    background-color: #6c757d;
    color: bisque;
    width: 100%;
}
.btn-chart {
    margin-left: 10px;
    width: 100px;
}
</style>

