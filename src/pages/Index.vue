<template>
  <q-page class="flex column">
    <!-- header 搜索栏 -->
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBysearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBysearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherDate">
      <link
        rel="stylesheet"
        href="https://cdn.jsdelivr.net/npm/qweather-icons@1.1.0/font/qweather-icons.css"
      />
      <div class="col text-white text-center">
        <!-- UI显示城市 -->
        <div class="text-h4 text-weight-light">{{ weatherDate2.name }}</div>
        <!-- <div class="text-h4 text-weight-light">{{ weatherDate.adm2 }}</div> -->
        <!-- UI显示天气 -->
        <div class="text-h6 text-weight-light">{{ weatherDate.now.text }}</div>
        <!-- <div class="text-h6 text-weight-light">{{ weatherDate.now.text }}</div> -->
        <!-- UI显示温度 -->
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ weatherDate.now.temp }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <!-- UI显示天气图标 -->
      <div class="col text-center">
        <!-- <img src="https://www.fillmurray.com/100/100" alt="Bill" /> -->
        <i :class="`qi-${weatherDate.now.icon}`"></i>
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          我爱你 <br />
          方婷！！
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <!-- UI显示底部图案 -->
    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import axios from "axios";
// Vue.use(VueAxios, axios);

export default defineComponent({
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherDate: null,
      weatherDate2: null,
      flag: false,
      lat: null,
      lon: null,
      location: null,
      // apiUrl: "https://restapi.amap.com/v3/weather/weatherInfo",
      weatherApiUrl: "https://devapi.qweather.com/v7/weather/now",
      cityApiUrl: "https://geoapi.qweather.com/v2/city/lookup",
      // apiKey: "96ead9a3b3653dda9078b745d30364cd",
      apiKey: "2c911a104cf14135af8dc1f9f9b01fc4",
    };
  },
  methods: {
    getLocation() {
      // 获取当前位置的经纬度信息
      navigator.geolocation.getCurrentPosition((position) => {
        // lat 纬度 接口需要保留两位小数
        this.lat = position.coords.latitude.toFixed(2);
        // lon 经度 接口需要保留两位小数
        this.lon = position.coords.longitude.toFixed(2);
        // this.location = this.lon + "," + this.lat;
        this.getWeatherBycoords();
      });
    },
    getWeatherBycoords() {
      // 第一次请求天气状况、温度、icon等数据
      axios
        .get(`${this.weatherApiUrl}`, {
          params: {
            // location: `${this.location}`,
            location: `${this.lon},${this.lat}`,
            // location: `120.22,31.54`,
            key: this.apiKey,
          },
        })
        .then((res) => {
          this.weatherDate = res.data;
          console.log(this.weatherDate);
        })
        .catch((error) => {
          console.log(error);
        });
      // 第二次利用经纬度请求城市名称数据
      axios
        .get(`${this.cityApiUrl}`, {
          params: {
            location: `${this.lon},${this.lat}`,
            key: this.apiKey,
          },
        })
        .then((res) => {
          this.weatherDate2 = res.data.location[0];
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getWeatherBysearch() {
      axios
        .get(`${this.cityApiUrl}`, {
          params: {
            location: `${this.search}`,
            key: this.apiKey,
          },
        })
        .then((res) => {
          this.lat = res.data.location[0].lat;
          this.lon = res.data.location[0].lon;
          this.getWeatherBycoords();
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
});
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to right, #654ea3, #eaafc8)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../../public/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
