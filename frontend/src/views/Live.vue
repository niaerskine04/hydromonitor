<!-- MY CODE -->
<template>
    <v-container class="bg-surface" fluid align="center" >
        <v-row class="row row1" max-width="1200px">
            <v-col class="col col1" cols="9">
                <figure class="highcharts-figure">
                    <div id="container"></div>
                </figure>
            </v-col>
            <v-col class="col col2" cols="3">
                <v-card class="mb-5" max-width="500" color="primaryContainer" subtitle="Temperature">
                    <v-card-item>
                        <span class="text-onPrimaryContainer">{{ temperature }}</span>
                    </v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="tertiaryContainer" subtitle="Heat Index(Feels like)">
                    <v-card-item>
                        <span class="text-onTertiaryContainer">{{heatindex }}</span>
                    </v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="secondaryContainer" subtitle="Humidity">
                    <v-card-item>
                        <span class="text-onSecondaryContainer">{{ humidity }}</span>
                    </v-card-item>
                </v-card>
            </v-col>
        </v-row>
        <v-row class="row row2" max-width="1200px" justify="start">
            <v-col class="col col1" cols="9">
                <figure class="highcharts-figure">
                    <div id="container1"></div>
                </figure>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from "pinia";
import Highcharts from "highcharts";
import more from "highcharts/highcharts-more";
import Exporting from "highcharts/modules/exporting";
Exporting(Highcharts);
more(Highcharts);

//COMPUTED PROPERTIES
const temperature = computed(() => {
  if (!!payload.value) {
    return `${payload.value.temperature.toFixed(2)} °C`;
  }
});
const heatindex = computed(() => {
  if (!!payload.value) {
    return `${payload.value.heatindex.toFixed(2)} °C`;
  }
});
const humidity = computed(() => {
  if (!!payload.value) {
    return `${payload.value.humidity.toFixed(2)} %`;
  }
});


// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const Mqtt        = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.
const tempHiChart = ref(null); // Chart object
const humChart = ref(null); // Chart object -->

// FUNCTIONS
onMounted(()=>{
// THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED -->
    CreateCharts();
    CreateCharts2();

    Mqtt.connect();
    setTimeout(() => {
        // subscribes to the relevant topics 
        Mqtt.subscribe("620155827");
        Mqtt.subscribe("620155827_pub");
    }, 3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});


const CreateCharts = async () => {
  // TEMPERATURE CHART
  tempHiChart.value = Highcharts.chart("container", {
    chart: { zoomType: "x" },
    title: { text: "Air Temperature and Heat Index Analysis", align: "left" },
    yAxis: {
      title: {
        text: "Air Temperature & Heat Index",
        style: { color: "#000000" },
      },
      labels: { format: "{value} °C" },
    },
    xAxis: {
      type: "datetime",
      title: { text: "Time", style: { color: "#000000" } },
    },
    tooltip: { shared: true },
    series: [
      {
        name: "Temperature",
        type: "spline",
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[0],
      },
      {
        name: "Heat Index",
        type: "spline",
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[1],
      },
    ],
  });
};

const CreateCharts2 = async () => {
  // Humidity CHART
  humChart.value = Highcharts.chart("container1", {
    chart: { zoomType: "x" },
    title: { text: "Humidity Analysis", align: "left" },
    yAxis: {
      title: {
        text: "Humidity",
        style: { color: "#000000" },
      },
      labels: { format: "{value} %" },
    },
    xAxis: {
      type: "datetime",
      title: { text: "Time", style: { color: "#000000" } },
    },
    tooltip: { shared: true },
    series: [
      {
        name: "Humidity",
        type: "spline",
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[0],
      },
    ],
  });
};

//WATCHERS
watch(payload,(data)=> {
    if(points.value > 0){ points.value -- ; }
    else{ shift.value = true; }

 tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);

 tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);

humChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);
})
</script>