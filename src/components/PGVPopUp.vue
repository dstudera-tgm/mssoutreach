


<template>
	<div id="popUp" class="ui-widget-content" style="position:absolute; background-color:red;" v-on:popup="setPopUpData">
		<p v-text="station_id"></p>
		<p v-text="name"></p>
		<p v-text="network"></p>
		<p v-text="location"></p>
		<p v-text="coords"></p>
		<p v-text="utm_coords"></p>
		<p v-text="description"></p>
	</div>
</template>

<script>
import * as d3 from "d3";
	
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
	top: 40px
	width:400px
	height:600px
	background-color:red
	padding:3px
	border: 1px solid black
	z-index:500
	
	
	
</style>