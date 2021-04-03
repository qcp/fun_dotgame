<template>
	<section>
		<aside>
			<div v-if="!ready">
				<label for="name">Имя:</label>
				<input id="name" v-model="name">
				<label for="game">№ игры:</label>
				<input id="game" type="number" v-model="game">
			</div>

			<label v-if="name.length > 5 && game">Готов к игре:
				<input type="checkbox" v-model="ready" />	
			</label>

			<ul>
				<li v-for="(value, name) in colorMap">
					<span 
						class="point" 
						:style="'background-color: '+value"
						:title="name"  >
					</span>
					<span>&emsp;{{name}}</span>
				</li>
			</ul>
		</aside>

		<main v-if="ready" class="map">			
			<span v-for="el in mapArray" 
				class="point" 
				:style="'background-color: '+el.COLOR" 
				:title="el.USER" 
				@click="step(el)">
			</span>
		</main>
	</section>
</template>

<script setup>
import { ref, reactive, watch } from 'vue'

let game = ref('')
let name = ref('')
let ready = ref(false)

let colorMap = reactive({ })
let mapArray = reactive([...Array(40)].map((e, i) => ([...Array(40)].map((e, j) => ({"ROW":i+1, "COL":j+1})))).flat())


function getColor(name){
	if(!colorMap[name])
		colorMap[name] = '#' + [...Array(6)].map(() => '0123456789ABCDEF'[Math.floor(Math.random() * 16)]).join('')
	return colorMap[name]
}


async function loadMap() {
	if(!ready.value)
		return;

	let response = await fetch(`https://dotgameWrapper.qcp.repl.co/get?game=${game.value}`)	
	let arr = await response.json()

	for(let el of mapArray.filter(e => !!e['USER'])){
		el['USER'] = null	
		el['COLOR'] = null		
	}
	for(let el of arr){
		el['COLOR'] = getColor(el['USER'])
		mapArray[ (el['ROW'] - 1)*40 + (el['COL'] - 1) ] = el;
	}	
	console.log('load')
}


async function step(el){
	let response = await fetch('https://dotgameWrapper.qcp.repl.co/add', {
		method: 'POST',
		headers: {
			'Accept': 'application/json',
			'Content-Type': 'application/json'
		},
		body: JSON.stringify({
			Col: +el['COL'], 
			Row: +el['ROW'],
			Game: +game.value,
			User: name.value
		})
	});	

	loadMap()

	let res = await response.json()

	if(res.error_code) 
		alert(res.error)
}



let timer = null
watch(ready, (now, prev) => {	
	if(now) {
		timer = setInterval(loadMap, 2000)
		loadMap()
	}
	else {
		clearInterval(timer)
		for(let prop in colorMap) 
			delete colorMap[prop]
	}
})

</script>

<style>
	section {
		display: flex;
		align-items: center;
		justify-content: space-around;
		height: 100vh;
		margin: auto;
		max-width: 1000px;

	}	
	aside {
		width: calc(100vw - 660px);
		max-width: 400px;
		min-width: 200px;
	}
	.map {
		width: 560px;
		height: 560px;

		display: grid; 
		grid-template-rows: repeat(40, 14px);
		grid-template-columns: repeat(40, 14px);
	}
	.point {
		display: inline-block;
		background-color: lightgray;

		width: 10px;
		height: 10px;	
		margin: 2px;
		border-radius: 50%;
	}
	.point:hover {
		transition: .2s;
		transform: scale(1.8, 1.8);
		box-shadow: 0 0 15px 5px white; 
	}
	ul {
		list-style-type: none;
	}
</style>