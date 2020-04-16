<template>
	<div id="permaPopUp" style="position:absolute; background-color:blue;">
		<h1>Stations:</h1>
		<p><button v-on:click="$emit('close-perma')">Close</button></p>
		<div v-if="popUpStored.length">
			<PGVPopUp 
                       v-for="cur_popup in popUpStored"
                       v-bind:key="cur_popup.station_id"
                       v-bind:station_id="cur_popup.station_id" 
                       v-bind:name="cur_popup.name" 
                       v-bind:network="cur_popup.network"
                       v-bind:location="cur_popup.location"
                       v-bind:coords="cur_popup.coords"
                       v-bind:utm_coords="cur_popup.utm_coords"
                       v-bind:description="cur_popup.description"/>
		</div>
	
	</div>
</template>

<script>

import * as PGVPopUp from "../components/PGVPopUp.vue";
	
	
export default {
	name:"PGVPopUpPerma",
	
	components: {
		PGVPopUp,
	},
	
	props:{
		
	},
	
	data() {
		return {
			
		}
	},
	
	created() {
		console.log("Length: " +this.$store.popUpStored_length);
	},
	
	computed: {
		popUpStored: function() {
            return this.$store.getters.popUpStored;
        },
		
	},
}


</script>

<style scoped lang="sass">
div#permaPopUp
	height:100%
	width:300px
	right:0px
	top:0px
	z-index:800

</style>