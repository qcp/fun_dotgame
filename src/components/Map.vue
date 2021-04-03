<template>
	<div class="map">
		<div v-for="el in mapArray" 
			class="point" 
			:style="'background-color: '+el.COLOR" 
			:title="el.USER" 
			@click="step(el)">
		</div>
	</div>
</template>

<script setup>
import { reactive, defineProps, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({	
  gameid: String,
	usrname: String
})

let colorMap = reactive({
	[props.usrname]: '#9b4dca'
})

let mapArray = reactive([...Array(40)].map((e, i) => ([...Array(40)].map((e, j) => ({"ROW":i+1, "COL":j+1})))).flat())

function getColor(name){
	if(!colorMap[name])
		colorMap[name] = '#' + [...Array(6)].map(() => '0123456789ABCDEF'[Math.floor(Math.random() * 16)]).join('')
	return colorMap[name]
}

async function loadMap() {
	let response = await fetch('https://dotgameWrapper.qcp.repl.co/get?game='+props.gameid)	
	let arr = await response.json()

	for(let el of mapArray.filter(e => !!e['USER'])){
		el['USER'] = null	
		el['COLOR'] = null		
	}
	for(let el of arr){
		el['COLOR'] = getColor(el['USER'])
		mapArray[ (el['ROW'] - 1)*40 + (el['COL'] - 1) ] = el;
	}	
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
			Game: +props.gameid,
			User: props.usrname
		})
	});	
	let res = await response.json()

	if(res.error_code) 
		alert(res.error)
	
	loadMap()
}

const timer = setInterval(loadMap, 2000)
loadMap()

onBeforeUnmount(() => {
	clearInterval(timer)
})

</script>

<style>
.map {
	width: 560px;
	height: 560px;

	display: grid; 
	grid-template-rows: repeat(40, 14px);
	grid-template-columns: repeat(40, 14px);

	padding: 2px;
}
.point {
	background-color: lightgray;

	width: 10px;
	height: 10px;	
	border-radius: 50%;
}
.point:hover {
	transition: .2s;
	transform: scale(1.8, 1.8);
	box-shadow: 0 0 15px 5px white; 
}
</style>