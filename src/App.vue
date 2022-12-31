<template>
  <div class="app">
    <h1>Sensação Térmica</h1>
    <p v-if="loading">Carregando...</p>
    <p v-if="error">Erro ao carregar dados.</p>
    <div v-if="temp">
      <h2>Salvador</h2>
      <p class="temp" v-bind:class="tempClassSalvador">
        A sensação térmica em Salvador é de {{ temp }}°C
      </p>
    </div>
    <div v-if="tempCamacari">
      <h2>Camaçari</h2>
      <p class="temp" v-bind:class="tempClassCamacari">
        A sensação térmica em Camaçari é de {{ tempCamacari }}°C
      </p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      temp: null,
      tempCamacari: null,
      loading: true,
      error: false,
      tempClassSalvador: "",
      tempClassCamacari: "",
      cacheExpiration: 3600,
    };
  },
  created() {
    this.fetchData();
    this.fetchDataCamacari();
  },
  methods: {
    async fetchData() {
      try {
        // Verifica se os dados estão no cache e ainda não expiraram
        const cache = localStorage.getItem("tempCacheSalvador");
        if (cache) {
          const cacheData = JSON.parse(cache);
          const expirationTime = cacheData.timestamp + this.cacheExpiration;
          if (expirationTime > Date.now()) {
            this.temp = cacheData.temp;
            this.loading = false;
            return;
          }
        }
        // Se os dados não estão no cache ou já expiraram, faz a chamada à API
        const response = await axios.get(
          "https://api.openweathermap.org/data/2.5/weather?q=Salvador,BR&units=metric&appid=510e66aba7f8a7e4f574a4445b7ab0d5"
        );
        this.temp = Math.round(response.data.main.feels_like);
        this.tempClassSalvador = this.temp > 30 ? "temp-red" : "temp-orange";
        this.loading = false;

        // Atualiza o cache com os novos dados
        const cacheData = {
          timestamp: Date.now(),
          temp: this.temp,
        };
        localStorage.setItem("tempCacheSalvador", JSON.stringify(cacheData));
      } catch (error) {
        this.error = true;
      }
    },
    async fetchDataCamacari() {
      try {
        // Verifica se os dados estão no cache e ainda não expiraram
        const cache = localStorage.getItem("tempCacheCamacari");
        if (cache) {
          const cacheData = JSON.parse(cache);
          const expirationTime = cacheData.timestamp + this.cacheExpiration;
          if (expirationTime > Date.now()) {
            this.temp = cacheData.temp;
            this.loading = false;
            return;
          }
        }
        // Se os dados não estão no cache ou já expiraram, faz a chamada à API
        const response = await axios.get(
          "https://api.openweathermap.org/data/2.5/weather?q=Camacari,BR&units=metric&appid=510e66aba7f8a7e4f574a4445b7ab0d5"
        );
        this.tempCamacari = Math.round(response.data.main.feels_like);
        this.tempClassCamacari = this.tempCamacari > 30 ? "temp-red" : "temp-orange";
        this.loading = false;

        // Atualiza o cache com os novos dados
        const cacheData = {
          timestamp: Date.now(),
          temp: this.tempCamacari,
        };
        localStorage.setItem("tempCacheCamacari", JSON.stringify(cacheData));
      } catch (error) {
        this.error = true;
      }
    },
  },
};
</script>

<style>
body {
  margin: 0;
  text-align: center;
}

h1 {
  font-size: 36px;
  font-weight: bold;
  color: #333;
}

h2 {
  font-size: 24px;
  font-weight: bold;
  color: #666;
}

p {
  font-size: 18px;
  color: #999;
}

.red {
  background-color: #ff0000;
}

.orange {
  background-color: #ffa500;
}

.container {
  max-width: 1000px;
  margin: 0 auto;
}

.temp {
  color: #ff0000;
}

.temp-orange {
  color: #ffa500;
}
</style>
