<template>
        <!-- HTML HERE -->
        <v-container class="container" fluid justify="center" align="center">
            <v-row class="row" max-width="1200" justify="center"> 
                <v-col class="col col1" align="center">
                    <v-sheet class="rounded-t-lg mb-1" color="surface" elevation="0" max-width="800" width="100%">
                        <v-card class="text-secondary" title="LED Controls" color="surface" subtitle="Recent settings" variant="tonal" flat>
                        </v-card>
                    </v-sheet >
                    <v-sheet class="mb-1" color="surface" elevation="0" max-width="800" width="100%">
                        <v-card class="pt-5" color="surface" variant="tonal">
                            <v-slider class="pt-2 bg-surface" appendIcon="mdi:mdi-car-light-high" density="compact" thumbSize="16"
                            color="secondary" label="Brightness" direction="horizontal" min="0" max="250" step="10" showTicks thumbLabel="always" v-model="led.brightness">
                            </v-slider>
                        </v-card>
                    </v-sheet>
                    <v-sheet class="mb-1" color="surface" elevation="0" max-width="800" width="100%">
                        <v-card class="pt-5" color="surface" variant="tonal">
                            <v-slider class="pt-2 bg-surface" appendIcon="mdi:mdi-led-on" density="compact" thumbSize="16" color="secondary"
                            label="LED Nodes" direction="horizontal" min="1" max="7" step="1" showTicks thumbLabel="always" v-model="led.nodes">
                            </v-slider>
                        </v-card>
                    </v-sheet>
                    <v-sheet class="mb-1 pa-2" color="surface" elevation="0" max-width="800" width="100%" justify="center" align="center" border>
                        <VProgressCircular rotate="0" size="200" width="15" :model-value="led.nodes *15" :color="indicatorColor">
                            <span class="text-onSurface font-weight-bold">{{ led.nodes }} LED(s)</span>
                        </VProgressCircular>
                    </v-sheet>
                </v-col>
                <v-col class="col col2" align="center">
                    <VColorPicker v-model="led.color"></VColorPicker>
                </v-col>
            </v-row>

        </v-container>    
    
</template>

<script setup>
/** JAVASCRIPT HERE */
// WATCHERS
const led = reactive({"brightness":255,"nodes":1,"color":{ r: 255, g: 0, b: 255, a: 1 }});
let timer, ID = 1000;

watch(led,(controls)=>{
    clearTimeout(ID);

    ID = setTimeout(()=>{
        const message =
JSON.stringify({"type":"controls","brightness":controls.brightness,"leds":controls.nodes,"color": controls.color});
        Mqtt.publish("620155827_sub",message); // Publish to a topic subscribed to by the hardware
    },1000)
});
// IMPORTS

import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store 
import { storeToRefs } from "pinia";
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();  

const Mqtt        = useMqttStore(); 
const { payload, payloadTopic } = storeToRefs(Mqtt);



// FUNCTIONS

onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect(); // Connect to Broker located on the backend

    setTimeout(() => {
       // Subscribe to each topic
       Mqtt.subscribe("620155827");
       Mqtt.subscribe("620155827_pub");
   }, 3000);
 
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});

// COMPUTED PROPERTIES
const indicatorColor = computed(()=>{
    return `rgba(${led.color.r},${led.color.g},${led.color.b},${led.color.a})`
})
</script>


<style scoped>
/** CSS STYLE HERE */


</style>