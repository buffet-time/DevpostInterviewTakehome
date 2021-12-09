<!-- The new Script Setup 

Could drastically be improved in terms of how sloppy this was due to time constraint

Given more time I would've implemented the bonus section by making using a form of localcache per user
in this i'd store the weather data and before going out to update quickly check to see if the hour has
changed since the data was last updated. Given this exmaple you onyl really need to care about hourly
data if it's 1pm and you make a request you get new data at 1:50pm you still use it, but at 
2:00pm you update the data so the "today's hourly temps" section shows the current hour first. 

i didnt see a need given the scope of this exercise to use Vuex so i stuck with standard 
Vue 3.2x+ script setup syntax https://v3.vuejs.org/api/sfc-script-setup.html, 
it'd become more useful if i were to expand this out and make it more scalable for larger amounts of users

a major thing i'd do with more time is create types for the endpoints response
this would make handling the API much easier
 -->
<script setup lang="ts">
/* eslint-disable @typescript-eslint/no-explicit-any */
import { onMounted, ref } from 'vue'
import ApiKey from '../assets/apiKey'

interface CityLatAndLon {
	lat: string
	lon: string
}

const apiKey = ApiKey
const baseCurrentWeatherUrl = 'https://api.openweathermap.org/data/2.5/weather'
const baseOneCallUrl = 'https://api.openweathermap.org/data/2.5/onecall'
const listOfExcludedParams = 'minutely,alerts'
const unitsToUse = 'imperial'
const defaultCity = 'New York City'

const hourlyTemps = ref([] as any[]) // 48 hours of data
const dailyWeather = ref([] as any[])
const currentWeather = ref()
const inputCity = ref(defaultCity)
const selectedDay = ref()
const render = ref(false)

onMounted(() => updateCity(defaultCity))

async function updateCity(city: string): Promise<boolean> {
	const weatherData = await getWeatherDataByCity(city)
	if (!weatherData) return false
	currentWeather.value = weatherData.current
	// this may return 8 days worth of data if its later at night so reduce it to 7
	dailyWeather.value = weatherData.daily.slice(0, 7)
	hourlyTemps.value = weatherData.hourly.slice(0, 12)
	render.value = true //
	return true
}

// Various helper functions
function timestampToHour(timestamp: number): string {
	const date = new Date(timestamp * 1000) // to ms
	let hours = date.getHours()
	let amPm = 'am'
	if (hours === 0) return `12am`
	else if (hours === 12) return `12pm`
	else if (hours > 12) {
		amPm = 'pm'
		hours -= 12
	}

	return `${hours}${amPm}`
}

function timestampToDayOfWeek(timestamp: number): string {
	// prettier-ignore
	const days = [
		'Sunday', 'Monday', 'Tuesday',
		'Wednesday', 'Thursday', 'Friday', 'Saturday'
	]
	const dayIndexFromTimestamp = new Date(timestamp * 1000).getDay()
	return days[dayIndexFromTimestamp]
}

// https://openweathermap.org/api/one-call-api
// eslint-disable-next-line @typescript-eslint/ban-types
async function getWeatherDataByCity(city: string) {
	let returnData: any
	const cityCoords = await getLatAndLongitudeByCityName(city)
	if (!cityCoords) {
		inputCity.value = 'Error retrieving'
		return null
	}

	await fetch(
		`${baseOneCallUrl}?lat=${cityCoords.lat}&lon=${cityCoords.lon}
			&exclude=${listOfExcludedParams}&appid=${apiKey}&units=${unitsToUse}`
	)
		.then(async (response) => {
			if (!response.ok) {
				inputCity.value = 'Error retrieving!!!'
				returnData = null
			} else {
				returnData = await response.json()
			}
		})
		.catch(() => (returnData = null))

	return returnData
}

/*
the free API oneCallApi can't search by city name and there is no other way
to get 7 day forecast in free so i need to make a quick current weather call
as a workaround as to not use a different API (at least that was what i gathered from this exercise)
of course in a real world scenario this wouldn't suffice but this should be
fine for this project i think.
*/
async function getLatAndLongitudeByCityName(
	city: string
): Promise<CityLatAndLon | null> {
	let returnedData: any

	await fetch(`${baseCurrentWeatherUrl}?q=${city}&appid=${apiKey}`)
		.then(async (response) => {
			if (!response.ok) {
				returnedData = null
			} else {
				returnedData = await response.json()
			}
		})
		.catch(() => (returnedData = null))

	return returnedData
		? {
				lon: returnedData.coord.lon,
				lat: returnedData.coord.lat
		  }
		: null
}
</script>

<!-- 
	Given more time I'd likely break down some of these into child-components 
	of this parent but for times sake i did it all in one large component 
-->
<template>
	<div class="card bg-dark weather-card-div">
		<div v-if="render" class="card-body">
			<!-- first row -->
			<div class="card-row top-row">
				<div class="todays-temp">
					<h1>{{ currentWeather.temp.toFixed() }}F</h1>
					<div class="wind-and-humidity">
						<div>Wind: {{ currentWeather.wind_gust.toFixed() }}mph</div>
						<div>Humidity: {{ currentWeather.humidity }}%</div>
					</div>
				</div>
				<div class="mb-3 weather-input">
					<input
						id="cityInput"
						v-model="inputCity"
						class="form-control"
						@keypress.enter="updateCity(inputCity)"
					/>
					<div class="today-div">
						{{ timestampToDayOfWeek(currentWeather.dt) }}
					</div>
				</div>
			</div>

			<!-- second row -->
			<div class="card-row">
				<h5 class="card-title">Today's Hourly Temperatures</h5>
				<div class="row-of-hourly-temps">
					<div v-for="hourlyObject in hourlyTemps" :key="hourlyObject.dt">
						<div>{{ timestampToHour(hourlyObject.dt) }}</div>
						<h4>{{ hourlyObject.temp.toFixed() }}F</h4>
					</div>
				</div>
			</div>

			<!-- third row -->
			<div class="card-row">
				<h5 class="card-title">7 Day Forecast</h5>
				<div class="forecast-div">
					<div
						v-for="dailyObject in dailyWeather"
						:key="dailyObject.dt"
						class="forecast-day"
						@click="selectedDay = dailyObject"
					>
						<div>{{ timestampToDayOfWeek(dailyObject.dt) }}</div>
						<div>
							<h5>{{ dailyObject.temp.max.toFixed() }}f</h5>
							{{ dailyObject.temp.min.toFixed() }}f
						</div>
					</div>
				</div>
			</div>

			<!-- fourth row -->
			<div v-if="selectedDay" class="card-row">
				<h6 class="card-title">Tuesday</h6>
				<div class="current-day-div">
					<div>
						<h4>{{ selectedDay.weather[0].main }}</h4>
						<h6>{{ selectedDay.weather[0].description }}</h6>
					</div>
					<div class="wind-and-humidity-bottom-row">
						<h6>Wind: {{ selectedDay.wind_speed.toFixed() }}mph</h6>
						<h6>Humidity: {{ selectedDay.humidity }}%</h6>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<!-- with more time this'd be using more values and be better structured of course -->
<style scoped>
.weather-card-div {
	display: flex;
	align-items: center;
	justify-content: center;
	width: calc(100% - 240px);
	height: 100%;
	margin-left: 100px;
	margin-right: 100px;
}
.card-body {
	width: 100%;
	padding: 24px;
	gap: 24px;
}
.weather-input {
	display: flex;
	flex-direction: column;
	margin-left: auto;
}
.card-row {
	display: flex;
	flex-direction: column;
	margin-bottom: 32px;
}
.top-row {
	flex-direction: row;
}
.today-div {
	text-align: right;
}
.todays-temp {
	display: flex;
	flex-direction: row;
}
.wind-and-humidity {
	margin-left: 16px;
	flex-direction: row;
}
.row-of-hourly-temps {
	display: flex;
	gap: 32px;
	max-width: 100%;
	overflow: hidden;
}
.current-day-div {
	display: flex;
}
.wind-and-humidity-bottom-row {
	margin-left: auto;
}
.forecast-div {
	display: flex;
	gap: 32px;
	max-width: 100%;
	overflow: hidden;
}
.forecast-day {
	cursor: pointer;
}
</style>
