


<template>
    <div id="popUp"   v-on:popup="setPopUpData">
        <div id="popUpFree" v-if="classicPopUp">
                <div style="text-align:right" id="buttonsContainer">
                <button v-on:click="$emit('add-popup')" data-open="off_canvas_perma" id="addButton" data-close="off_canvas_popup">
                    <i class="step fi-lock size-18"></i>
                </button>

                 <button class="close-button closeButton" aria-label="Close alert" type="button" data-close="off_canvas_popup">
                    <span aria-hidden="true">&times;</span>
                  </button>

                </div>
                <h3 id="content">{{name}}</h3>
                <p><b>latest PGV:</b> {{ (pgv * 1000).toFixed(3) }} mm/s</p> 
                <p><b>max. PGV:</b> {{ (pgv_max * 1000).toFixed(3) }} mm/s</p> 
                <h3 class="popUpHeading"  id="content">Stationsmetadaten</h3>
                <p id="content"><b>ID:</b> {{station_id}}</p>		
                <p id="content"><b>Netzwerk:</b> {{network}}</p>
                <p id="content"><b>Standort:</b> {{location}}</p>
                <p id="content"><b>Koordinaten:</b> {{coords}}</p>
                <p id="content"><b>UTM Koordinaten:</b> {{utm_coords}}</p>
                <p id="content"><b>Beschreibung:</b> {{description}}</p>
        </div>
        <div id="popUpFixed" v-if="!classicPopUp">
            <p>{{station_id}}</p>
        </div>


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
        classicPopUp:Boolean,   //Gibt an ob das PopUp am linken Rand als PopUp oder in der PermaPopUp Liste am rechten Rand ist
    },

    mounted() {
            /*
        //Jquery draggable und resizable für das PopUp aktivieren
        // eslint-disable-next-line
        $("#popUp").draggable({
            scroll: false
        });
        if(!this.atttached) {

        }*/



    },


    data() {
        return {
            close_icon_path:"/assets/vue/nrt/image/icons/close_popup.png",
            
        }
    },

    computed: {
        add_icon_path: function() {
            if(!this.attached) {
                return "/assets/vue/nrt/image/icons/lock_closed.png";
            }
            else {
                return "/assets/vue/nrt/image/icons/lock_open.png";
            }
        },

        pgv: function() {
            return this.$store.getters.current_pgv_by_station(this.$props.station_id);
        },

        pgv_max: function() {
            return this.$store.getters.disp_range_max_pgv_by_station(this.$props.station_id);
        },
    },

    methods: {
        setPopUpData() {
            this.logger.debug("SETPOPUPDATA");

        },
        attached(attach) {
            this.attached=attach;
        },
    },

}
</script>

<style scoped>
    #popUp{      
        direction:ltr;       
        font-size:15px;
        height:100%;
        height:100%;
        

    }
    #popUpFixed {
        background-color: #CCCCCC;
        height:100px;
        
    }
    
    .popUpHeading {
        background-color:LightGray;
        width:100%;
    }
    

    #closeButton{
        position:relative;
        right:20px;
    }
    #buttonsContainer {
        width:100%;
        position:absolute;
    }

    #addButton{
        position:relative;
        right:45px;
        top:10px;
    }
</style>


<style lang="sass">
    $attached:false	

    div#popUp
            
            padding:0.5em
            z-index:500


	
</style>
