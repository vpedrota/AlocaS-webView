<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="8">

        <div id="map" class="map"></div>
       
      </v-col>
      <v-col cols="12" md="4">
        <v-row>
          <v-col cols="12">
            <v-card
              class="mx-auto"
              variant="outlined"
            >
              <v-card-item>
                <div>
                  <div class="text-overline mb-1">
                    <h2>Ferramentas</h2>
                  </div>
                  <div>
                    <v-btn variant="tonal">s</v-btn>
                    <div>
                      <div class="text-overline mb-1">Selecionar Geometria</div>
                      <div class="opcoes">
                        <v-btn  @click="typeSelect = 'Polygon'" variant="tonal">
                          Polygon
                        </v-btn>
                        <v-btn @click="typeSelect = 'Circle'" variant="tonal">
                          Circle
                        </v-btn>
                        <v-btn @click="typeSelect = 'Point'" variant="tonal">
                          Point
                        </v-btn>
                      </div>
                    </div>
                  </div>
                </div>
              </v-card-item>
            </v-card>
          </v-col>
        </v-row>  
        <v-row>
          <v-col cols="12">
            <v-card
              class="mx-auto"
              variant="outlined"
            >
              <v-card-item>
                <div>
                  <div class="text-overline mb-1">
                    <h2>Geometrias inseridas</h2>
                    <h3 v-for="geo in geometrias">{{geo}}</h3>
                  </div>     
                </div>
              </v-card-item>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>

</template>

<script lang="ts">

  import 'ol/ol.css';
  import { Map, View } from 'ol';
  import {Draw, Modify, Snap} from 'ol/interaction.js';
  import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
  import { OSM, Vector as VectorSource } from 'ol/source';
  import { fromLonLat } from 'ol/proj';
  import { ref, onMounted, watch } from 'vue'

export default {
  
  setup() {

    let typeSelect = ref<string>("Polygon")
    let geometrias = ref(["s"])

    onMounted(() => {

      let newMap:any, draw:any, snap:any
      const source = new VectorSource();
      const vector = new VectorLayer({
        source: source,
        style: {
          'fill-color': 'rgba(255, 255, 255, 0.8)',
          'stroke-color': '#ffcc33',
          'stroke-width': 2,
          'circle-radius': 7,
          'circle-fill-color': '#ffcc33',
        },
      });

      newMap = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new OSM(),
          }),
          vector,
        ],
        view: new View({
          center: fromLonLat([-122.45, 37.75]),
          zoom: 14,
        }),
      });

      let modify = new Modify({source: source});
      newMap.addInteraction(modify);

      function addInteractions(type:any) {
        draw = new Draw({
          source: source,
          type: type,
        });
        newMap.addInteraction(draw);
        snap = new Snap({source: source});
        newMap.addInteraction(snap);
      }

      watch(typeSelect, () =>{
        newMap.removeInteraction(draw);
        newMap.removeInteraction(snap);
        addInteractions(typeSelect.value);

        draw.on('drawend', (event: any) => {
          console.log('Geometria modificada: ', event.features);
          geometrias.value.push("event.features")
        });
      })

      addInteractions(typeSelect.value);
    
      draw.on('drawend', (event: any) => {
        console.log('Geometria modificada: ', event.features);
        geometrias.value.push("event.features")
      });
      
      modify.on('modifyend', (event: any) => {
        console.log('Geometria modificada: ', event.features);
      });
    })

    return {typeSelect, geometrias}
  },
  
}
</script>

<style>
#map{
  height: 70vh;
  max-height: 800px;
  width: 100%;
}

.menu{
  display: flex;
  flex-direction: row;
  width: 100%;
  align-items: center;
  justify-content: center;
  column-gap: 30px;
  margin-top: 30px;
}

.opcoes{
  display: flex;
  column-gap: 5px;
}
</style>