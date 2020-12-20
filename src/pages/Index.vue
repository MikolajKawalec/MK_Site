<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Szukaj miejscowości"
        dark
        borderless
        >
        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="my_location" 
          />
        </template>

        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch"
            round
            dense
            flat
            icon="search" />
        </template>
      </q-input>
    </div>

  <template v-if="weatherData">

    <template>
      <div class="q-pa-md">

        <div class="row">
          <div class="col">
             <q-input
               rounded
               filled
               bg-color="white"
               v-model="lowerbound_temp"
               @keyup.enter="possibletable"
               label="Minimalna temperatura" />
          </div>
          <div class="col">
            <q-input
               rounded
               filled
               bg-color="white"
               v-model="upperbound_temp"
               @keyup.enter="possibletable"
               label="Maksymalna temperatura" />
          </div>
          <div class="col">
            <q-input
               rounded
               filled
               bg-color="white"
               v-model="max_wind_speed"
               @keyup.enter="possibletable"
               label="Maksymalna prędkosć wiatru w m/s" />
          </div>
        </div>

      </div>
    </template>

    <div class="col text-white text-center">
      <div class="text-h4 text-weight-light" style="padding-bottom: 20px">
        {{ CurrData.name }}
      </div>

      <div class="text-h6 text-weight-light" style="padding-bottom: 20px">
        Możliwe godziny w których mozna prowadzić oprysk dla podancyh parametrów. Zakres temperatury od {{ lowerbound_temp }} do {{ upperbound_temp }} oraz prędkośc wiatru nie przekraczająca {{ max_wind_speed }} m/s oraz brak opadów na dwie godziny wcześniej i trzy godziny później.
      </div>

      <template>
        <div class="q-pa-md q-gutter-sm">
          <q-btn
            color="white"
            text-color="black"
            label="Generuj tabelę"
            @click="possibletable" />
        </div>
      </template>

      <template>
        <div class="q-pa-md">
          <q-table
            class="my-sticky-dynamic"
            :data="EligbleTable"
            row-key="name"
            :loading="loading"
            hide-bottom
            virtual-scroll
            :virtual-scroll-item-size="96"
            :virtual-scroll-sticky-size-start="96"
            :pagination="pagination"
            :rows-per-page-options="[0]"
            :class="fillClass"
          />
        </div>
      </template>
    </div>

  </template>

  <template v-else>
    <div class="col column text-center text-white">
      <div class="col text-h2 text-weight-thin">
        Quasar<br>Aplikacja
      </div>
      <q-btn
        @click="getLocation"
        class="col"
        flat>
        <q-icon left size="3em" name="my_location" />
        <div>Znajdź moją lokalizację</div>
    </q-btn>
    </div>
  </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      Data: null,
      CurrData: null,

      
      lat: null,
      lon: null,


      exclude: 'minutely,daily,current',
      apiUrl: 'https://api.openweathermap.org/data/2.5/onecall',
      CurrUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: 'b589a59d5c71789007ab80e38d633e05',


      lowerbound_temp: 5,
      upperbound_temp: 30,
      max_wind_speed: 5,

      EligbleTable: []
    }
  },
  computed: {
    bgClass() {
      if(this.weatherData) {
        if(this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        else {
          return 'bg-day'
        }
      }
    },
    fillClass () {
      if (this.EligbleTable.eligible) {
        return 'fill-g'
      }
      else {
        return 'fill-r'
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()
      navigator.geolocation.getCurrentPosition(position => {
        console.log('position: ', position)
        this.lat = position.coords.latitude
        this.lon = position.coords.longitude
        this.getWeatherByCoords()
      })
      
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`${ this.apiUrl }?lat=${ this.lat }&lon=${ this.lon }&exclude =${ this.exclude }&appid=${ this.apiKey }&units=mertic&lang=pl`).then(responce => {
        console.log('responce: ', responce)
        this.weatherData = responce.data.current
        this.Data = responce.data
        this.$q.loading.hide()

        console.log('responce_data: ', this.data)

        this.$axios(`${ this.CurrUrl }?lat=${ this.lat }&lon=${ this.lon }&appid=${ this.apiKey }&units=mertic&lang=pl`).then(responce => {
          console.log('responce: ', responce)
          this.CurrData = responce.data
          this.$q.loading.hide()

          console.log('responce_data_curr: ', this.data)

          this.possibletable()
        })
        .catch(err=> {
        console.log(err)
        this.$q.loading.hide()
        })
      })

      
    },

    getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(`${ this.CurrUrl }?q=${ this.search }&appid=${ this.apiKey }&units=mertic&lang=pl`).then(responce => {
        console.log('responce: ', responce)
        this.CurrData = responce.data
        this.$q.loading.hide()
  
        console.log('responce_data_curr: ', this.data)

        let local_lat = this.CurrData.coord.lat
        let local_lon = this.CurrData.coord.lon

        this.$axios(`${ this.apiUrl }?lat=${ local_lat }&lon=${ local_lon }&exclude =${ this.exclude }&appid=${ this.apiKey }&units=mertic&lang=pl`).then(responce => {
          console.log('responce: ', responce)
          this.weatherData = responce.data.current
          this.Data = responce.data
          this.$q.loading.hide()

          console.log('responce_data: ', this.data)

          this.possibletable()
        })
      })
      .catch(err=> {
        console.log(err)
        this.$q.loading.hide()
      })
    },

    possibletable: function() {
      this.lowerbound_temp = parseFloat(this.lowerbound_temp)
      this.upperbound_temp = parseFloat(this.upperbound_temp)
      this.max_wind_speed = parseFloat(this.max_wind_speed)

      this.EligbleTable = []
      console.log('insertTimeTable called')
      for (var i = 2; i < this.Data.hourly.length - 3; i++) {
        let unix = this.Data.hourly[i].dt
        let date = new Date(unix * 1000)
        let hours = date.getHours()
        let minutes = date.getMinutes()
        let format_time = hours + ':' + minutes + '0'
        if (this.lowerbound_temp > (this.Data.hourly[i].temp - 273.15)) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if ((this.Data.hourly[i].temp - 273.15) > this.upperbound_temp) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i].wind_speed > this.max_wind_speed) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i + 1].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i + 2].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i + 3].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i - 1].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        if (this.Data.hourly[i - 2].rain) {
          this.EligbleTable.push({ date: format_time, eligible: false })
          continue
        }
        else {
          this.EligbleTable.push({ date: format_time, eligible: true })
        }

      }
      console.log('possible called')
    }
  }
}
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #0083B0, #00B4DB)
  &.bg-night
     background: linear-gradient(to top, #232526, #414345)
  &.bg-day
      background: linear-gradient(to top, #134e5e, #71b280)
.q-table
  &.fill-g
    color: #00ff00
  &.fill-r
    color: #ff0000
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../statics/skyline.png)
  background-size: contain
  background-position: center bottom
</style>