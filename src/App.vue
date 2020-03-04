<template>
  <div id="app">
    <main>
      <div class="search-box">
        <input 
          type="text" 
          class="search-bar" 
          placeholder="Search..." 
          v-model="query"
          @keyup.enter="fetchGeocode"
        />
      </div>

      <transition name="fade">
      <div class="weather-wrap" v-if="typeof weather.currently != 'undefined'">
        <div class="weather-box">
          <div class="section-one">
            <p class="day">Right Now</p>
            <p class="temp"><b>{{ Math.round(weather.currently.temperature) }}°</b></p>
          </div>
          <div class="section-two">
            <div class="weather">{{ weather.currently.summary }}</div>
          </div>
          <div class="section-three">
            <div class="feels-like">
              <p><b>{{ Math.round(weather.currently.apparentTemperature) }}°</b></p>
              <p>Feels like</p>  
            </div>
            <div class="humidity">
              <p><b>{{ Math.round(weather.currently.humidity * 100) }}%</b></p>
              <p>Humidity</p>
            </div>
            <div class="wind-speed">
              <p><b>{{ Math.round(weather.currently.windSpeed) }} mph</b></p>
              <p>Wind Speed</p>
            </div>
          </div>
        </div>
        <div class="forecast-box" v-for="forecast in 6" v-bind:key="forecast.id">
          <div class="section-one">
            <p class="day" v-if="forecast == 1">Tomorrow</p>
            <p class="day" v-else>{{ dateBuilder(weather.daily.data[forecast].time).day }}</p>
            <p class="temp">{{ Math.round(weather.daily.data[forecast].temperatureLow) }}° <b>{{ Math.round(weather.daily.data[forecast].temperatureHigh) }}°</b></p>
          </div>
          <div class="section-two">
            <div class="weather">{{ weather.daily.data[forecast].summary }}</div>
          </div>
        </div>
      </div>
      </transition>
    </main>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      api_key: '296d05f58b634ea5cabbba7d273ccbff',
      api_base: 'https://api.darksky.net/forecast/',
      query: '',
      location: '',
      weather: {},
      forecast: {}
    }
  },
  methods: {
    async fetchGeocode () {
      this.weather.currently = undefined
      const { features } = await fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/'${encodeURIComponent(this.query)}.json?access_token=pk.eyJ1Ijoid29sZmZwYyIsImEiOiJjazNxY3dnOXgwMGNvM2JsZXd0emV2YXhzIn0.imKrKKD3A1691epL-GfehA&limit=1`)
        .then(response => response.json())
      this.location = features[0].place_name
      this.query = this.location
      this.fetchWeather(features[0].center[1], features[0].center[0])
    },
    async fetchWeather (latitude, longitude) {
      const weather = await fetch(`https://cors-anywhere.herokuapp.com/${this.api_base}${this.api_key}/${latitude},${longitude}`)
        .then(response => response.json())
      this.weather = weather
    },
    setResults (results) {
      this.weather = results
    },
    dateBuilder (n) {
      let d = new Date(n * 1000);
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]

      let day = days[d.getDay()]
      let date = d.getDate()
      let month = months[d.getMonth()]
      let year = d.getFullYear()

      let myDate = { day, date, month, year }

      return myDate
    } 
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'montserrat', sans-serif;
}

#app {
  background-color: #1b1b1b;
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}

main {
  max-width: 850px;
  margin: 0 auto;
  min-height: 100vh;
  padding: 25px;
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 20px;

  color: #ffffff;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;

  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 8px 8px 8px 8px;
  transition: 0.4s;
}

.search-box .search-bar:focus {
  background-color: rgba(255, 255, 255, 0.3);
}

.weather-box {
  display: flex;
  flex-direction: column;
  color: #ffffff;
  background-color: #575d81;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.forecast-box {
  display: flex;
  flex-direction: column;
  color: #ffffff;
  background-color: #7885cb;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.section-one {
  display: flex;
  justify-content: space-between;
}

.section-three {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.day {
  font-size: 32px;
  font-weight: 700;
}

.temp {
  font-size: 32px;

}

.weather {
  font-size: 20px;
  font-weight: 100;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
