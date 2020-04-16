


<template>
	<div id="popUp" class="ui-widget-content" style="position:absolute; background-color:white;" v-on:popup="setPopUpData">
		<p style="text-align:right"><button v-on:click="$emit('close-popup')">x</button></p>
		<h2>{{name}}</h2>
		<h3 style="background-color:LightGray">Stationsmetadaten</h3>
		<p><b>ID:</b> {{station_id}}</p>		
		<p><b>Netzwerk</b> {{network}}</p>
		<p><b>Standort:</b> {{location}}</p>
		<p><b>Koordinaten:</b> {{coords}}</p>
		<p><b>UTM Koordinaten:</b> {{utm_coords}}</p>
		<p><b>Beschreibung:</b> {{description}}</p>
		<p><button v-on:click="$emit('add-popup')">Add Popup</button></p>
		
	</div>
</template>


<script>


export default {

	
	name:"PGVPopUp",
	
	props: {
		station_id: String,
		name: String,
		network: String,
		location: String,
		coords: String,
		utm_coords: String,
		description: String,
	},
	
	mounted() {
		// eslint-disable-next-line
		$("#popUp").draggable({
			scroll: false
		});
		
	},
	
	
	data() {
		return {	
		}
	},
	
	computed: {
		
	},
	
	methods: {
		setPopUpData() {
			console.log("SETPOPUPDATA");

		},
	},
	
}
</script>

<style scoped lang="sass">
div#popUp
	position:relative
	top: 200px
	width:400px
	height:600px
	margin-left:10px
	padding:10px
	border: 2px solid black
	z-index:500

div#addPopUp
	position:absolute
	bottom:0
	right:0
	
</style>