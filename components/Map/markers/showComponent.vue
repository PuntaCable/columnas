<template>
    <v-slide-y-reverse-transition>
      <v-card elevation="24" width="90%" class="mx-auto rounded-xl" v-if="value">
        <v-card-title class="white--text">
          <v-icon class="mr-2">mdi-map-marker</v-icon>
          <span class="font-weight-bold">Ver marcador</span>
        </v-card-title>
        <v-divider></v-divider>
        <v-card-text>
          <v-row>
            <v-col class="col-12">
              <v-btn-toggle class="elevation-3 rounded-lg" color="primary white--text" v-model="marker.type"
                style="width:100%">
                <v-btn width="100%" active-class="primary" v-if="marker.type=='Column'" class="font-weight-bold" value="Column">
                  <img src="/icons/column-pin.png" width="30">
                  <span class="white--text">Columna</span>
                </v-btn>
                <v-btn width="100%" active-class="primary" v-else class="font-weight-bold" value="Building">
                  <img src="/icons/building-pin.png" width="30">
                  <span class="white--text">Edificio</span>
                </v-btn>
              </v-btn-toggle>
            </v-col>
            <v-col class="col-12">
              <formsFieldsTextComponent label-color="white--text" background-color="white" class="black--text"
                v-model="marker.reference" label="Referencia"></formsFieldsTextComponent>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-actions>
            <v-btn color="primary font-weight-regular rounded-lg" block @click="connectMarker()">
                Conectar marcador&nbsp;<img src="/icons/pin.png" width="40">
            </v-btn>
        </v-card-actions>
      </v-card>
    </v-slide-y-reverse-transition>
  
  </template>
  
  <script>
    export default {
      props: {
        latLng: {
          type: Object,
          default: () => {}
        },
        value: {
          type: Boolean,
          default: false
        },
        marker: {
          type: Object,
          default: () => {}
        }
      },
      methods: {
        connectMarker(){
            this.$emit('connectmarker')
        }
      }
    }
  
  </script>
  
  <style>
  
  </style>
  