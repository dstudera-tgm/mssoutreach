<template>
    <div id="permaPopUp">
        <h1>Stations</h1>
        <button class="close-button" aria-label="Close menu" type="button" data-close>
            <span aria-hidden="true">&times;</span>
        </button>
        <div v-if="popUpStored.length" id="popUpContainer">
            <template v-for="(cur_popup,index) in popUpStored">
                <component v-bind:is="getPopUp(index)"
                     v-bind:key="stationID(cur_popup.station_id)"
                     v-bind:station_id="cur_popup.station_id" 
                     v-bind:name="cur_popup.name" 
                     v-bind:network="cur_popup.network"
                     v-bind:location="cur_popup.location"
                     v-bind:coords="cur_popup.coords"
                     v-bind:utm_coords="cur_popup.utm_coords"
                     v-bind:description="cur_popup.description"
                     v-bind:classicPopUp="cur_popup.classicPopUp"
                     ></component>
                
            </template>


        </div>

    </div>
</template>

<script>
import Vue from 'vue';
import * as PGVPopUp from "../components/PGVPopUp.vue";
import * as log from 'loglevel';
import * as log_prefix from 'loglevel-plugin-prefix';

export default {
    name:"PGVPopUpPerma",

    components: {
        PGVPopUp,
    },

    props:{

    },
    
    created() {
        

        
        this.logger = log.getLogger(this.$options.name)
        this.logger.setLevel(this.$store.getters.log_level);
        log_prefix.apply(this.logger,
            this.$store.getters.prefix_options);  
    },
    
    data() {
        return {
            logger: undefined,
        }
    },
    
    
    mounted() {
        this.logger.debug("PopUpStored Length: " +this.popUpStored.length);
    },
    methods: {
        getPopUp: function(index) {
            return this.popUps[index];
        },
        stationID:function(station_id) {
          return "perma_"+station_id;
        },
    },

    computed: {
        popUpStored: function() {
            return this.$store.getters.popUpStored;
        },
        popUps: function() {
            var popUps=[];
            for(var i=0;i<this.popUpStored.length;i++) {
                var name="pop-up-perma-"+i;
                Vue.component(name,Vue.extend(PGVPopUp.default))
                popUps.push(name);

            }
            return popUps;
        },


    },
    watch: {
  
    },
}


</script>
<style scoped>
    h1{
        padding-left:10px;
    }
    #permaPopUp{
        background-color:aliceblue;
        height:100%;
    }
</style>

<style scoped lang="sass">


</style>
