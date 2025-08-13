<template>
    <div class="api-practice">
        <h2>API Practice Component</h2>

        <section class="api-section">
            <h3>Weather Info</h3>

            <button @click="fetchWeather" :disabled="loadingWeather" class="fetch-btn">
                {{ loadingWeather ? 'Loading...' : 'Get Weather' }}
            </button>

            <div v-if="loadingWeather" class="loading">Fetching weather...</div>
            <div v-if="weatherError" class="error">Error: {{ weatherError }}</div>

            <div v-if="weather" class="weather-card">
                <h4>London, UK</h4>
                <div class="weather-details">
                    <p>Temperature: {{ weather.current_weather?.temperature }}°C</p>
                    <p>Condition: {{ getWeatherDescription(weather.current_weather?.weathercode) }}</p>
                    <p>Wind Speed: {{ weather.current_weather?.windspeed }} km/h</p>
                    <p>Wind Direction: {{ weather.current_weather?.winddirection }}°</p>
                    <p>Last Updated: {{ new Date(weather.current_weather?.time).toLocaleString() }}</p>
                </div>
            </div>
        </section>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const weather = ref(null)         // Object to store weather data
const loadingWeather = ref(false) // Loading state for weather request
const weatherError = ref(null)    // Error state for weather request

const getWeatherDescription = (code) => {
    // convert the WMO code into text
    // https://www.nodc.noaa.gov/archive/arc0021/0002199/1.1/data/0-data/HTML/WMO-CODE/WMO4677.HTM
    const weatherCodes = {
        0: 'Clear sky',
        1: 'Mainly clear',
        2: 'Partly cloudy',
        3: 'Overcast',
        45: 'Fog',
        48: 'Depositing rime fog',
        51: 'Light drizzle',
        53: 'Moderate drizzle',
        55: 'Dense drizzle',
        61: 'Slight rain',
        63: 'Moderate rain',
        65: 'Heavy rain',
        71: 'Slight snow fall',
        73: 'Moderate snow fall',
        75: 'Heavy snow fall',
        95: 'Thunderstorm',
        96: 'Thunderstorm with slight hail',
        99: 'Thunderstorm with heavy hail'
    }
    return weatherCodes[code] || 'Unknown'
}

const fetchWeather = async () => {
    loadingWeather.value = true
    weatherError.value = null

    try {
        const response = await fetch(
            'https://api.open-meteo.com/v1/forecast?latitude=51.5074&longitude=-0.1278&current_weather=true&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m&timezone=Europe/London'
        )

        if (!response.ok) {
            throw new Error(`Received HTTP error! Status: ${response.status}`)
        }

        const data = await response.json()
        weather.value = data

        console.log('Response:', data)

    } catch (error) {
        weatherError.value = error.message
        console.error('Error fetching data:', error)
    } finally {
        loadingWeather.value = false
    }
}
</script>

<style scoped>
.api-practice {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
}

.api-practice h2 {
    text-align: center;
    color: var(--color-heading);
    margin-bottom: 2rem;
}

.api-section {
    margin-bottom: 3rem;
    padding: 1.5rem;
    border: 1px solid var(--color-border);
    border-radius: 8px;
    background: var(--color-background-soft);
}

.api-section h3 {
    color: var(--color-heading);
    margin-bottom: 1rem;
}

.fetch-btn {
    background: hsla(160, 100%, 37%, 1);
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9rem;
    margin-bottom: 1rem;
    transition: background-color 0.3s;
}

.fetch-btn:hover:not(:disabled) {
    background: hsla(160, 100%, 32%, 1);
}

.fetch-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
}

.loading {
    color: #666;
    font-style: italic;
    padding: 1rem;
}

.error {
    color: #e74c3c;
    background: #fdf2f2;
    padding: 0.5rem;
    border-radius: 4px;
    margin: 0.5rem 0;
}

.weather-card {
    background: var(--color-background);
    padding: 1rem;
    border-radius: 6px;
    border: 1px solid var(--color-border);
    margin-top: 1rem;
}

.weather-card h4 {
    color: var(--color-heading);
    margin-bottom: 1rem;
}

.weather-details p {
    margin: 0.5rem 0;
    color: var(--color-text);
}
</style>