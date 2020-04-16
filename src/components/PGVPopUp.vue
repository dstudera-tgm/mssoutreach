


<template>
	<div id="popUp" class="ui-widget-content" style="position:absolute; background-color:red;" v-on:popup="setPopUpData">
		<p><button v-on:click="$emit('close-popup')">Close</button></p>
		<p v-text="station_id"></p>
		<p v-text="name"></p>
		<p v-text="network"></p>
		<p v-text="location"></p>
		<p v-text="coords"></p>
		<p v-text="utm_coords"></p>
		<p v-text="description"></p>
		<p><button v-on:click="$emit('add-popup')">Add Popup</button></p>
	</div>
</template>


<script>

require('../../public/assets/vue/js/jquery-ui-1.12.1/jquery-ui.min.js');
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
	padding:3px
	border: 1px solid black
	z-index:500
	
	
	
</style>