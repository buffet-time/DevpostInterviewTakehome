<script setup lang="ts">
import Weather from './components/Weather.vue'
</script>

<template>
	<Weather />
</template>

<style lang="scss">
// Using Bootstrap to make it pretty faster because of time constraints :)

// a la carte importing of Bootstrap 5
@import 'node_modules/bootstrap/scss/functions';
@import 'node_modules/bootstrap/scss/variables';
@import 'node_modules/bootstrap/scss/mixins';
@import 'node_modules/bootstrap/scss/reboot';

@import 'node_modules/bootstrap/scss/card';
@import 'node_modules/bootstrap/scss/forms';

body {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	background-color: #201f1d;
	color: white;
	margin-top: 60px;
}

.card {
	margin: 20px;
	color: #fff;
	text-align: left;
	background-color: #424242 !important;
}
</style>
