
<template>
    <div id="settings_container">

        
         <!-- Your menu or Off-canvas content goes here -->
                <button class="close-button" aria-label="Close menu" type="button" data-close>
                    <span aria-hidden="true">&times;</span>
                </button>
                <h4>Settings</h4>
                <ul class="accordion" data-accordion>
                    <li class="accordion-item is-active" data-accordion-item>
                        <!-- Accordion tab title -->
                        <a href="#" class="accordion-title">Display</a>

                        <!-- Accordion tab content: it would start in the open state due to using the `is-active` state class. -->
                        <div class="accordion-content" data-tab-content>
                            <div>
                                <label>Show event monitor</label>
                            </div>
                            <div class="switch">
                                <input class="switch-input" id="switchEM" type="checkbox" name="switchEM" v-model="show_event_monitor">
                                <label class="switch-paddle" for="switchEM">
                                    <span class="show-for-sr">Show event monitor</span>
                                </label>
                            </div>
                            
                            <div>
                                <label>Show legend</label>
                            </div>
                            <div class="switch">
                                <input class="switch-input" id="switchL" type="checkbox" name="switchL" v-model="show_legend">
                                <label class="switch-paddle" for="switchL">
                                    <span class="show-for-sr">Show legend</span>
                                </label>
                            </div>
                        </div>
                    </li>
                    <li class="accordion-item" data-accordion-item>
                        <!-- Accordion tab title -->
                        <a href="#" class="accordion-title">Other</a>

                        <!-- Accordion tab content: it would start in the open state due to using the `is-active` state class. -->
                        <div class="accordion-content" data-tab-content>
                            Content Other
                        </div>
                    </li>
                </ul>
    </div>

</template>

<script>
import $ from 'jquery';	

export default {
    name: "Settings",

    props: {

    },

    created() {
    },

    data() {
        return {

        }
    },

    mounted() {
        this.settings=$.extend(true,{},this.$store.getters.settings);
    },

    methods: {
        activate_settings() {
            this.$store.commit("show_settings",true);
            $(".leaflet-control-zoom").css("visibility", "hidden");
            $(".leaflet-control-layers").css("visibility", "hidden");
        },

        close_settings() {
            this.$store.commit("show_settings",false);
            $(".leaflet-control-zoom").css("visibility", "visible");
            $(".leaflet-control-layers").css("visibility", "visible");
        },


    },
    computed: {
        settings: function() {
            return this.$store.getters.settings;
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
        
    },

}
</script>

<style scoped>

h4 {
    margin:10px;
}

#btn_open_settings {
    position:absolute;
    margin-left:50px;
    margin-top:25px;
}


</style>
