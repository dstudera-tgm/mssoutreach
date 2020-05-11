


<template>
    <div id="popUp" class="ui-widget-content position-left off-canvas"  v-on:popup="setPopUpData" data-off-canvas data-transition="overlap">

        <div style="text-align:right" id="buttonsContainer">
            <button v-on:click="$emit('add-popup')" id="addButton">
                <i class="step fi-lock size-18"></i>
            </button>

             <button v-on:click="$emit('close-popup')" class="close-button closeButton" aria-label="Close alert" type="button" data-close>
                <span aria-hidden="true">&times;</span>
              </button>

        </div>
        <h2 id="content">{{name}}</h2>
        <p><b>latest PGV:</b> {{ (pgv * 1000).toFixed(3) }} mm/s</p> 
        <p><b>max. PGV:</b> {{ (pgv_max * 1000).toFixed(3) }} mm/s</p> 
        <h3 class="popUpHeading"  id="content">Stationsmetadaten</h3>
        <p id="content"><b>ID:</b> {{station_id}}</p>		
        <p id="content"><b>Netzwerk</b> {{network}}</p>
        <p id="content"><b>Standort:</b> {{location}}</p>
        <p id="content"><b>Koordinaten:</b> {{coords}}</p>
        <p id="content"><b>UTM Koordinaten:</b> {{utm_coords}}</p>
        <p id="content"><b>Beschreibung:</b> {{description}}</p>

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
            attached:false,
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
    },

}
</script>

<style scoped>
    #popUp{
        display:inline-block;
        width:30%;
        height:450px;
        left:250px;
        top: 245px;
        font-size:15px;
        
        overflow-x:hidden;
        margin: 0px auto;
        background-color: #E87B10;

    }
    
    .popUpHeading {
        background-color:LightGray;
        width:90%;
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
