<!--
    /*****************************************************************************
    * LICENSE
    *
    * This file is part of mss_vis.
    * 
    * If you use mss_vis in any program or publication, please inform and
    * acknowledge its authors.
    * 
    * mss_vis is free software: you can redistribute it and/or modify
    * it under the terms of the GNU General Public License as published by
    * the Free Software Foundation, either version 3 of the License, or
    * (at your option) any later version.
    * 
    * mss_vis is distributed in the hope that it will be useful,
    * but WITHOUT ANY WARRANTY; without even the implied warranty of
    * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    * GNU General Public License for more details.
    * 
    * You should have received a copy of the GNU General Public License
    * along with mss_vis. If not, see <http://www.gnu.org/licenses/>.
    *
    * Copyright 2019 Stefan Mertl
    *****************************************************************************/
-->


<template>
    <!--<div id="mapcontainer" @click.ctrl="capture_map">-->
    <div id="mapcontainer" >


        <div id="mapid">

        </div>

        <!-- SVG templates are added to the leaflet SVG overlay in the mounted
            function. -->
        <svg id="svg_template_pgv_marker">
            <g id="current_pgv_marker">
                <PGVMapMarker v-on:open-popup="setPopUp($event)"
                                 v-for="cur_station in stations"
                                 v-bind:key="cur_station.id"
                                 v-bind:station_id="cur_station.id"
                                 v-bind:x="cur_station.x"
                                 v-bind:y="cur_station.y"/>
            </g>
        </svg>
        
        
        <svg id="svg_template_archive_plot" >
            <ArchiveEventPlot />
        </svg>

        <svg id="svg_template_event_monitor" >
            <EventMonitorPlot/>
        </svg>
        
        

        <div id="off_canvas_popup" class="off-canvas-absolute position-left" data-off-canvas data-transition="overlap">
            <component v-bind:is="popUp" 
                     v-on:add-popup="addPopUp()"
                     v-bind:station_id="popUpData.station_id" 
                     v-bind:name="popUpData.name" 
                     v-bind:network="popUpData.network"
                     v-bind:location="popUpData.location"
                     v-bind:coords="popUpData.coords"
                     v-bind:utm_coords="popUpData.utm_coords"
                     v-bind:description="popUpData.description"
                     v-bind:classicPopUp="popUpData.classicPopUp"
                     ref="currentPopUp"></component>
        </div>

        

        <svg id="svg_legend" width="300px" height="140" v-show="show_legend">
            <PGVLegend name="map_legend" />
        </svg>


        <div id="map_info"  v-show="show_event_monitor">last data: {{ data_time_range[1] }} UTC<br>
            first data: {{ data_time_range[0] }} UTC<br>
            server state: {{ server_state }}<br><br>
            <b>event monitor</b><br>
            <div id="current_event">
                start: {{ current_event_start }}<br>
                end: {{ current_event_end }}<br>
                state: {{ current_event_state }}<br>
                max PGV: {{ (current_event_max_pgv * 1000).toFixed(3) + ' mm/s'}}<br><br>
            </div>

            <b>archived events</b><br>
            <ArchiveEvent v-for="(cur_event, index) in event_archive"
                          v-bind:key="cur_event.start_time"
                          v-bind:id="cur_event.start_time"
                          v-bind:pos="index"/>

        </div>
    </div>


</template>


<script>
import $ from 'jquery';	
import Vue from 'vue';
import PGVMapMarker from '../components/PGVMapMarker.vue';
import PGVLegend from '../components/PGVLegend.vue';
//import PGVEventVoronoi from '../components/PGVEventVoronoi.vue';

import ArchiveEvent from '../components/ArchiveEvent.vue';
import ArchiveEventPlot from '../components/ArchiveEventPlot.vue';
import EventMonitorPlot from '../components/EventMonitorPlot.vue';
import * as PGVPopUp from "../components/PGVPopUp.vue";
import * as d3 from "d3";
import domtoimage from 'dom-to-image';
import * as log from 'loglevel';
import * as log_prefix from 'loglevel-plugin-prefix';
import L from 'leaflet';
import 'leaflet-easybutton';

export default {
    name: 'PGVMap',

    props: {
        title: String,
        geoJson: FileList,
    },

    components: {
        PGVMapMarker,
        // eslint-disable-next-line
        PGVLegend,
        // eslint-disable-next-line
        EventMonitorPlot,
        // eslint-disable-next-line
        ArchiveEvent,
        // eslint-disable-next-line
        ArchiveEventPlot,
        // eslint-disable-next-line
        PGVPopUp,
    },

    data() {
        return {
            leaflet_map: 'undefined',

            popUp: '',
            popUpData: {
                station_id:"",
                name:"",
                network:"",
                location: "",
                coords: "",
                utm_coords: "",
                description: "",
                classicPopUp:false,
            },
            allOptions: 'undefined',
            map_image: 'undefined',
            //map_image_url: '/assets/vue/nrt/image/mss_map_with_stations.jpg',
            map_image_url: '/assets/vue/nrt/image/mss_map_clean.jpg',
            logger: undefined,
        };
    },

    beforeCreate() {
    },

    created() {
        this.show_event_monitor=true;
        this.logger = log.getLogger(this.$options.name)
        this.logger.setLevel(this.$store.getters.log_level);
        log_prefix.apply(this.logger,
            this.$store.getters.prefix_options);
    },

    mounted() {
        //Leaflet map initialisieren
        this.leaflet_map=L.map("mapid");
        this.map_image = new Image;
        this.init_map();
        //window.addEventListener('resize', this.on_resize);
        //this.$watch('radius', this.plot_stations);
        this.$store.commit("LOAD_STATION_METADATA");
        const vm = this;
        var checkExist = setInterval(function() {
            if (vm.$store.getters.stations_imported == true) {
                clearInterval(checkExist);
                vm.logger.debug("Stations: " + vm.$store.getters.station_meta.length);
                vm.updateMarkers();
            }
        }, 100);


        //window.addEventListener('resize', this.on_resize);
        this.leaflet_map.on("moveend", this.updateMarkers);

        // Get the svg marker template and add it to the leaflet svg overlay.
        $('#svg_template_pgv_marker').find('#current_pgv_marker').appendTo("#svg_overlay");
	$('#svg_template_pgv_marker').remove();

        // Get the svg archive event plot and add it to the leaflet svg overlay.
        $('#svg_template_archive_plot').find('*').appendTo("#svg_overlay");
	$('#svg_template_archive_plot').remove();

        // Get the svg event monitor plot and add it to the leaflet svg overlay.
        $('#svg_template_event_monitor').find('*').appendTo("#svg_overlay");
	$('#svg_template_event_monitor').remove();


    },

    updated() {
        //this.on_resize();
    },

    computed: {
        popUpStored: function() {
            return this.$store.getters.popUpStored;
        },
        
        show_event_monitor: {
            get() {
                return this.$store.getters.map_control.show_event_monitor;
            },

            set(value) {
                var payload = {property: 'show_event_monitor',
                    value: value}
                this.$store.commit('set_map_control', payload);
            }
        },
        
        show_legend: {
            get() {
                return this.$store.getters.map_control.show_legend;
            },

            set(value) {
                var payload = {property: 'show_legend',
                    value: value}
                this.$store.commit('set_map_control', payload);
            }
        },
        
        data_time_range: function() {
            return this.$store.getters.data_time_range;
        },

        server_state: function() {
            return this.$store.getters.server_state;
        },

        stations: function() {
            return this.$store.getters.station_meta;
        },

        scales: function() {
            return this.$store.getters.scales;
        },

        current_event: function() {
            return this.$store.getters.current_event;
        },

        current_event_max_pgv: function() {
            var max_pgv = this.$store.getters.current_event_max_pgv;
            if (max_pgv)
            {
                return max_pgv;
            }
            else
            {
                return 0;
            }
        },

        current_event_start: function() {
            if ('start_time' in this.current_event)
            {
                return this.current_event.start_time;
            }
            else
            {
                return "";
            }
        },

        current_event_end: function() {
            if ('end_time' in this.current_event)
            {
                return this.current_event.end_time;
            }
            else
            {
                return "";
            }
        },

        current_event_state: function() {
            if ('state' in this.current_event)
            {
                return this.current_event.state;
            }
            else
            {
                return "No active event.";
            }
        },

        event_archive: function() {
            return this.$store.getters.event_archive;
        },
    },

    methods: {
        init_map() {

            
            var oe3d = L.tileLayer('/assets/vue/nrt/data/map/oe3d/{z}/{x}/{y}.jpg', 
                {
                    minZoom: 10,
                    maxZoom: 13,
                    tms: false,
                    attribution: 'Map based on OE3D and OpenStreetMap. Generated with QGis.',
                });

            var osm=L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw',
                {
                    minZoom:10,
                    maxZoom: 13,
                    attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
                    '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
                    'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
                    id: 'mapbox/streets-v11',
                    tileSize: 512,
                    zoomOffset: -1,
                });


            var allOptions= {
                "MSS OE3D": oe3d,
                "Open Streetmap": osm,	
            }

            oe3d.addTo(this.leaflet_map);

            L.control.layers(allOptions, null, {position: 'topleft', autoZIndex:false }).addTo(this.leaflet_map);
            L.easyButton('<span style="width: 36px; height: 36px; border-radius: 5px; display: inline-block; font-size: 36px; background-color: white;"><i class="fi-widget size-12" style="color:#696969;"></span>', function(){$('#off_canvas_settings').foundation('open');}).addTo(this.leaflet_map);
            
            var vm = this;
            L.easyButton({
              id: 'open-perma-button',  // an id for the generated button
              position: 'topright',      // inherited from L.Control -- the corner it goes in
              type: 'replace',          // set to animate when you're comfy with css
              leafletClasses: true,     // use leaflet classes to style the button?
              states:[{                 // specify different icons and responses for your button
                stateName: 'open-perma',
                onClick: function(button,vm){
                    
                        $('#off_canvas_perma').foundation('open');
                    
                  
                },
                title: 'Open perma',
                icon: '<span style="width: 35px; height: 35px; border-radius: 5px; display: inline-block; font-size: 36px; background-color: white;"><i class="fi-indent-less size-8" style="color:#696969;"></span>'
              }]
            }).addTo(this.leaflet_map);

            

            this.leaflet_map.setView([47.8972,16.3507], 10);
            
            this.leaflet_map.createPane('markers');
            this.leaflet_map.getPane('markers').style.zIndex = 700;
            
            var svg=L.svg({pane:'markers'});
            svg.addTo(this.leaflet_map);
            
            this.leaflet_map.createPane('geoJson');
            this.leaflet_map.getPane('geoJson').style.zIndex = 650;
            
            
            //this.logger.debug("Overlays: "+d3.select(".leaflet-overlay-pane").count());
            d3.select(".leaflet-markers-pane")
                .select("svg")
                .attr("id","svg_overlay");

            
            //this.show_image();
            //this.on_resize();
        },

        updateMarkers() {
            this.leaflet_map.invalidateSize();
            var stations=this.stations;

            for(var i=0;i<stations.length;i++)	{
                var latlng=new L.LatLng(stations[i].y, stations[i].x);
                var elementid=stations[i].id.replace(/\./g, '-');
                var stationMax=elementid+"_max";
                var stationCurrent=elementid+"_current";

                d3.select("*[id$="+stationMax+"]").attr("cx",this.leaflet_map.latLngToLayerPoint(latlng).x);
                d3.select("*[id$="+stationMax+"]").attr("cy",this.leaflet_map.latLngToLayerPoint(latlng).y);

                d3.select("*[id$="+stationCurrent+"]").attr("cx",this.leaflet_map.latLngToLayerPoint(latlng).x);
                d3.select("*[id$="+stationCurrent+"]").attr("cy",this.leaflet_map.latLngToLayerPoint(latlng).y);
            }
        },
        setPopUp(station_id) {
            this.logger.debug("setPopUp: "+station_id);

            var curStation="undefined";
            var stations=this.stations;
            for(var i=0;i<this.stations.length;i++) {
                if(stations[i].id==station_id) {
                    curStation=stations[i];
                }
            }
            this.logger.debug("Station ID: "+curStation.id);

            this.popUpData.popUpId="pop-"+curStation.id;
            this.popUpData.station_id=curStation.id;
            this.popUpData.name=curStation.name;
            this.popUpData.network=curStation.network;
            this.popUpData.location=curStation.location;
            this.popUpData.coords="x: "+curStation.x+" y: "+curStation.y+" z: "+curStation.z;
            this.popUpData.utm_coords="x_utm: "+curStation.x_utm+" y_utm: "+curStation.y_utm;
            this.popUpData.description=curStation.description;
            this.popUpData.classicPopUp=curStation.classicPopUp;

            Vue.component("pop-up",Vue.extend(PGVPopUp.default));
            this.popUp="pop-up";
        },



        //Hängt das offene Popup an die Perma anzeige an
        addPopUp() {            
            var newPermaPopUp=[];
            
            newPermaPopUp.popUpId=this.popUpData.popUpId;
            newPermaPopUp.station_id=this.popUpData.station_id;
            newPermaPopUp.name=this.popUpData.name;
            newPermaPopUp.network=this.popUpData.network;
            newPermaPopUp.location=this.popUpData.location;
            newPermaPopUp.coords=this.popUpData.coords;
            newPermaPopUp.utm_coords=this.popUpData.utm_coords;
            newPermaPopUp.description=this.popUpData.description;
            newPermaPopUp.classicPopUp=false;
            
            this.$store.commit("add_pop_up",newPermaPopUp);
            this.logger.debug("Length: " +this.popUpStored.length);

        },
        
        loadGeoJSON(event) {
            console.log("Files: "+event.target.files);
        },


        calculate_path() {
            const scale = this.get_scales();
            const path = d3.line()
                .x((d, i) => scale.x(i))
                .y(d => scale.y(d));
            this.line = path(this.data)
        },

        show_image() {
            let self = this;
            var map_svg = d3.select("#map");
            this.map_image.onload = function()
            {
                self.logger.debug("Map image loaded.")
                map_svg.append("svg:image")
                    .attr('width', 4000)
                    .attr('height', 2500)
                    .attr('id', 'map_image')
                    .attr("xlink:href", self.map_image_url);
                d3.select('#map_image').lower();
                //self.on_resize();
            }
            this.logger.debug("Loading map image: " + this.map_image_url);
            this.map_image.src = this.map_image_url;
        },

        on_resize() {
            this.logger.debug("resize");
            var map_container = d3.select("#mapcontainer").node();
            var map_svg = d3.select("#svg_template");

            //const width = map_container.clientWidth;
            //const height = map_container.clientHeight;
            //this.logger.debug("Width: "+width);

            var map_bounds = map_container.getBoundingClientRect();
            map_svg.attr("width", map_bounds.width)
                .attr("height", window.innerHeight)
                .style("left", map_bounds.left + "px")
                .style("top", map_bounds.top + "px");

            //map_svg.attr("viewBox", "0 0 "+map_bounds.width+" "+window.innerHeight)

            this.logger.debug("width attr" + map_bounds.width);

            this.updateMarkers();


            //const scale = height / this.map_image.height;

            //const map_image = d3.select("#map_image");
            //map_image
            //    .attr('width', this.map_image.width * scale)
            //    .attr('height', this.map_image.height * scale)
        },

        capture_map() {
            // TODO: SVG images are not rendered by the domtoimage export.
            this.logger.debug('Capturing the map.');
            domtoimage.toPng(document.getElementById('mapcontainer'), {height: 2000})
                .then(function (dataUrl) {
                    var link = document.createElement('a');
                    link.download = 'my-image-name.png';
                    link.href = dataUrl;
                    link.click();
                });
        },

        on_click() {
            this.logger.debug("Clicked the map.");
        },
    },
    watch: {
        geoJson: function () {
            let stringData="";
            let vm=this;
            let fileReader
            if(this.geoJson.length) {   
                
                for(var i=0;i<this.geoJson.length;i++) {
                    fileReader=new FileReader();
                    fileReader.onload = function(progressEvent) {
                        stringData = fileReader.result;
                        L.geoJSON(jQuery.parseJSON(stringData),{pane:'geoJson'}).addTo(vm.leaflet_map);  
                    }

                    fileReader.readAsText(this.geoJson[i], "UTF-8");
                    
                    //json=this.geoJson[i].text();
                    
                     
                }
                L.geoJSON.bringToBack();
            }
            
        }
      },
}
</script>
<style>
    #svg_legend {
        background-color:rgba(181,181,181,0.62);
        position: absolute; 
        bottom:20px; 
        right:5px; 
        z-index:500;
        border-radius: 25px;
    }

    #off_canvas_popup{
        top:155px;
        z-index:1000;
        height:70%;
        width:20%;
        margin:10px;
        overflow-x: hidden;
        border-radius: 25px;
        direction:rtl;
        background-color: #E87B10;
    }

    #map_info{
        background-color:rgba(181,181,181,0.62);
        position: absolute; 
        top:5px; 
        right:5px; 
        z-index:500;
        border-radius: 25px;
    }


</style>

<style scoped lang="sass">
$breakpoint-mobile-width: 700px
$breakpoint-mobile-height: 350px

div#mapid
    height: 100%
    width: 100%
    cursor: default

div#mapcontainer
    height: 100%
    position: relative
    font-family: Helvetica, sans-serif

div#map_info
    position: absolute
    text-align: right
    font-size: 10pt
    font-family: Helvetica, sans-serif
    padding: 5px
    z-index:500

div#map_config
    position: absolute
    text-align: left
    font-size: 10pt
    font-family: Helvetica, sans-serif
    padding: 5px

svg#map
    border: none

@media (max-width: $breakpoint-mobile-width), (max-height: $breakpoint-mobile-height)
div#map_info
    font-size: 8pt
</style>
