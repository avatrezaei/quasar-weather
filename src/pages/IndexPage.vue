<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        placeholder="Search"
        dark
        borderless
        @keyup.enter="getWeatherBySearch"
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          alt="Avat"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br />
          Weather
        </div>
        <q-btn class="col" @click="getLocation" flat>
          <q-icon left size="3em" name="my_location"></q-icon>
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lng: null,
      apiKey: "02d5a83f0206421e3e7f5d0d26bf3351",
      apiUrl: `https://api.openweathermap.org/data/2.5/weather`,
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      } else {
        return "";
      }
    },
  },
  methods: {
    getLocation() {
       this.$q.loading.show();
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.lat = position.coords.latitude;
            this.lng = position.coords.longitude;

            this.getWeather();
          },
          (error) => {
            console.log(error);
            this.$q.loading.hide();
          }
        );
      }
       this.$q.loading.hide();
    },
    getWeather() {
      this.$q.loading.show();
      const url = `${this.apiUrl}?lat=${this.lat}&lon=${this.lng}&appid=${this.apiKey}&units=metric`;
      fetch(url)
        .then((response) => response.json())
        .then((response) => {
          this.weatherData = response;
          this.$q.loading.hide();
        })
        .catch((error) => {
          console.log(error);
          this.$q.loading.hide();
        });
    },
    
    getWeatherBySearch() {
      this.$q.loading.show();
      const url = `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`;
      fetch(url)
        .then((response) => response.json())
        .then((response) => {
          this.weatherData = response;
          this.$q.loading.hide();
        })
        .catch((error) => {
          console.log(error);
          this.$q.loading.hide();
        });
    },
  },
});
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url('assets/images/skyline.png')
  background-size: contain
  background-position: 50% 110px
</style>
