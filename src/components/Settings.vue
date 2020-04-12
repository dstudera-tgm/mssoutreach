
<template>
	<div id="settings_container">
		<button id="btn_open_settings" v-on:click="activate_settings()" v-if="!settings.show_settings" width="50px" height="50px">
			Settings
		</button>

		<div id="settings" v-if="settings.show_settings">
			<h1>Einstellung</h1>
			
			<button id="btn_close_settings" v-on:click="close_settings()">
				Close
			</button>
			
			<form>
              <input type="radio" @change="change_map($event)" name="some-radios" value="osm">Openstreetmap<br>
              <input type="radio" @change="change_map($event)" name="some-radios" value="basemap">Basemap<br>
			</form>
		</div>
	</div>
	
</template>

<script>
import $ from 'jquery';	
export default {
	name: "Settings",
	
	props: {

	},
	
	data() {
		return {
			settings: {
				show_settings: false,
				map_type: "",
			},
		}
	},
	
	mounted() {
		this.settings=$.extend(true,{},this.$store.getters.settings);
	},
	
	methods: {
		activate_settings() {
			this.settings.show_settings=true;
			$(".leaflet-control-zoom").css("visibility", "hidden");
		},
		
		close_settings() {
			this.settings.show_settings=false;
			$(".leaflet-control-zoom").css("visibility", "visible");
		},
		
		change_map(event) {
			this.settings.map_type=event.target.value;
		},
	},
	
}
</script>

<style scoped>
	#settings {
		background-color:red;
		width:400px;
		height:500px;
		
	}
	
	#btn_open_settings {
		position:absolute;
		margin-left:50px;
		margin-top:25px;
	}
	
	#settings_container {
		position:absolute;
		z-index:500;
	}
</style>
