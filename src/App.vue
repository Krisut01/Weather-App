<template>
  <v-app>
    <v-container fluid>
      <v-row justify="center" class="mt-5">
        <v-col cols="12" md="8" lg="6">
          <v-card class="pa-4" elevation="20" style="background: rgba(255, 255, 255, 0.9); border-radius: 15px;">
            <v-card-title>
              <h2 class="text-center headline">Enhanced Weather App</h2>
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="city"
                label="Enter city"
                @keyup.enter="fetchWeather"
                @input="resetError"
                clearable
                :loading="loading"
                prepend-inner-icon="mdi-magnify"
                class="mt-3"
                outlined
                dense
                color="primary"
              ></v-text-field>
              <v-btn color="primary" @click="fetchWeather" :disabled="loading" class="mt-3">
                Get Weather
              </v-btn>
              <v-alert v-if="error" type="error" class="mt-2">{{ error }}</v-alert>
              <v-progress-circular
                v-if="loading"
                indeterminate
                color="yellow"
                class="mt-2"
                size="30"
              ></v-progress-circular>

              <v-list class="mt-4">
                <v-list-item-group>
                  <v-list-item
                    v-for="(weather, index) in weatherData"
                    :key="index"
                    class="mt-2"
                    style="background: rgba(240, 240, 240, 0.8); border-radius: 10px; padding: 15px; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);"
                  >
                    <v-list-item-content>
                      <v-list-item-title class="headline">
                        <strong>{{ weather.name }}, {{ weather.sys.country }}</strong>
                      </v-list-item-title>
                      <v-list-item-subtitle class="subtitle-1">
                        <v-icon>{{ getWeatherIcon(weather.weather[0].main) }}</v-icon>
                        {{ weather.weather[0].description }}
                      </v-list-item-subtitle>
                      <v-list-item-content>
                        <p class="body-2"><strong>Temperature:</strong> {{ weather.main.temp }} °C</p>
                        <p class="body-2"><strong>Humidity:</strong> {{ weather.main.humidity }}%</p>
                        <p class="body-2"><strong>Wind Speed:</strong> {{ weather.wind.speed }} m/s</p>
                        <p class="body-2"><strong>Pressure:</strong> {{ weather.main.pressure }} hPa</p>
                        <p class="body-2"><strong>Sunrise:</strong> {{ formatTime(weather.sys.sunrise) }}</p>
                        <p class="body-2"><strong>Sunset:</strong> {{ formatTime(weather.sys.sunset) }}</p>
                      </v-list-item-content>
                    </v-list-item-content>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
              <v-btn color="error" @click="clearWeatherData" class="mt-4">Clear Weather Data</v-btn>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: '',
      weatherData: [],
      error: '',
      loading: false,
      apiKey: process.env.VUE_APP_OPENWEATHER_API_KEY || '1d5eb6db2e311820d9a8c0c5f21f2b92' // Use environment variable
    };
  },
  methods: {
    async fetchWeather() {
      if (this.city.trim() === '') {
        this.error = 'Please enter a city name.';
        return;
      }

      const existingCity = this.weatherData.find(w => w.name.toLowerCase() === this.city.toLowerCase());
      if (existingCity) {
        this.error = 'Weather data for this city is already displayed.';
        return;
      }

      this.resetError();
      this.loading = true;

      try {
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(this.city)}&units=metric&appid=${this.apiKey}`
        );

        const data = await response.json();

        if (data.cod === 200) {
          this.weatherData.push(data);
          this.city = ''; // Clear input field
        } else {
          this.error = data.message || 'City not found.';
        }
      } catch (error) {
        this.error = 'Unable to fetch weather data. Please try again later.';
      } finally {
        this.loading = false;
      }
    },
    formatTime(timestamp) {
      return new Date(timestamp * 1000).toLocaleTimeString();
    },
    resetError() {
      this.error = '';
    },
    clearWeatherData() {
      this.weatherData = [];
    },
    getWeatherIcon(condition) {
      const icons = {
        Clear: 'mdi-sunglasses',
        Clouds: 'mdi-cloud',
        Rain: 'mdi-weather-rainy',
        Snow: 'mdi-snowflake',
        Thunderstorm: 'mdi-weather-lightning',
        Drizzle: 'mdi-weather-drizzle',
      };
      return icons[condition] || 'mdi-help-circle'; // Default icon
    }
  },
  mounted() {
    const storedCities = JSON.parse(localStorage.getItem('weatherCities')) || [];
    this.weatherData = storedCities;
  },
  watch: {
    weatherData(newData) {
      localStorage.setItem('weatherCities', JSON.stringify(newData));
    }
  }
};
</script>

<style>
body {
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(to bottom right, #e0f7fa, #ffffff); /* Light gradient background */
  backdrop-filter: blur(5px); /* Blur effect for transparency */
}
</style>