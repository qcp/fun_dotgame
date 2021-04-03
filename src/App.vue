<template>
	<section>
		<aside>
			<label for="name">Имя:</label>
			<input id="name" v-model="name" :disabled="ready">
			<label for="game">№ игры:</label>
			<input id="game" type="number" v-model="game" :disabled="ready">
			<label v-if="name.length > 5 && game">Готов к игре:
				<input type="checkbox" v-model="ready" />	
			</label>
		</aside>
		<Map v-if="ready" :usrname="name" :gameid="game"></Map>
	</section>
</template>

<script setup>
import Map from './components/Map.vue'
import { ref } from 'vue'

let game = ref('')
let name = ref('')
let ready = ref(false)

</script>

<style>
	section {
		display: flex;
		align-items: center;
		justify-content: space-around;
		height: 100vh;
		margin: 0;
		min-width: 800px;
	}	
	aside {
		width: calc(100vw - 660px);
		max-width: 400px;
		min-width: 200px;
	}
</style>