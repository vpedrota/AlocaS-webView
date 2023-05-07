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
                    <v-btn variant="tonal">Satélite</v-btn>
                    <div>
                      <div class="text-overline mb-1">Selecionar Geometria</div>
                      <div class="opcoes">
                        <v-btn :class="{ active_polygon: typeSelect == 'Polygon' }" @click="typeSelect = 'Polygon'" variant="tonal">
                          Polygon
                        </v-btn>
                        <v-btn :class="{ active_circle: typeSelect == 'Circle' }" @click="typeSelect = 'Circle'" variant="tonal">
                          Circle
                        </v-btn>
                        <v-btn :class="{ active_point: typeSelect == 'Point' }" @click="typeSelect = 'Point'" variant="tonal">
                          Point
                        </v-btn>
                      </div>
                    </div>
                    <div>
                      <div class="text-overline mb-1">
                        Remover
                      </div>
                      <div>
                        <v-btn  @click="typeSelect = 'Polygon'" variant="tonal">
                          Remover Todos
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
                    <h3 v-if="geometrias.length == 0">Não há geometrias inseridas</h3>
                    <v-container>

                      <div v-for="geo in geometrias">
                        <div>
                            <v-row no-gutters>
                              <v-col  class="infos" cols="1" md="1">{{ geo.id }}</v-col>
                              <v-col  class="infos" cols="5"  md="5">
                                <v-text-field 
                                  class="input"
                                  density="compact"
                                  label="População"
                                  required
                                ></v-text-field>
                              </v-col>
                              <v-col  class="infos" cols="5" md="5">
                                <v-text-field 
                                  class="input"
                                  density="compact"
                                  label="Impacto"
                                  required
                                ></v-text-field>
                              </v-col>
                              <v-col  class="infos" cols="1" md="1">
                                <v-btn @click="remover(geo.id)" variant="tonal">x</v-btn>
                              </v-col>
                            </v-row>
                        </div>
                      </div>
                    </v-container>
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

interface geom {
  id: number;
}

export default {
  
  setup() {

    let typeSelect = ref<string>("Polygon")
    let geometrias = ref<geom[]>([])
    let numberGeo = 1;

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
        console.log(draw)
        newMap.addInteraction(draw);
        snap = new Snap({source: source});
        newMap.addInteraction(snap);
      }

      watch(typeSelect, () =>{
        newMap.removeInteraction(draw);
        newMap.removeInteraction(snap);
        addInteractions(typeSelect.value);
        //console.log(draw)
        draw.on('drawend', (event: any) => {
        
          geometrias.value.push({"id":numberGeo++})
        });
      })

      addInteractions(typeSelect.value);
    
      draw.on('drawend', (event: any) => {
        console.log('Geometria modificada: ', event.features);
        geometrias.value.push({"id":numberGeo++, "geomtry": draw})
      });
      
      modify.on('modifyend', (event: any) => {
        console.log('Geometria modificada: ', event.features);
      });
    })

    // Função utlizada para remover a geomtria do layer e da lista de geometrias
    function remover(value) {
      let itemIndex = -1;

      geometrias.value.forEach(function(elemento) {
        if(elemento.id == value){
          itemIndex++
        }
      });

      console.log(itemIndex)
      if(itemIndex != -1){
        geometrias.value.splice(itemIndex, 1);
      }
    }

    return {typeSelect, geometrias, remover}
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
  margin-bottom: 5px;
}

.input{
  max-width: 150px;
}

.infos{
  display: flex;
  align-items: center;
  justify-content: center;
}

.active_polygon{
  background-color: #FF0000;
  color: #FFFFFF; /* Define a cor do texto do botão como branco */
}

.active_circle{
  background-color: #eab676;
  color: #FFFFFF; /* Define a cor do texto do botão como branco */
}

.active_point{
  background-color:  #93ea76;
  color: #FFFFFF; /* Define a cor do texto do botão como branco */
}
</style>