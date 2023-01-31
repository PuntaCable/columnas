<template>
  <v-slide-y-reverse-transition>
    <v-card elevation="24" width="90%" class="mx-auto rounded-xl" v-if="value">
      <v-card-title class="white--text">
        <v-icon class="mr-2">mdi-map-marker</v-icon>
        <span class="font-weight-bold">Ver marcador</span>

        <v-spacer></v-spacer>
        <v-btn icon @click="$emit('input', false)">
          <v-icon>mdi-close</v-icon>
        </v-btn>

      </v-card-title>
      <v-divider></v-divider>
      <template v-if="!updateMarkerContent">
        <v-card-text>
          <v-row no-gutters>
            <v-col class="col-12">
              <v-btn-toggle class="elevation-3 rounded-lg" color="primary white--text" v-model="marker.type"
                style="width:100%">
                <v-btn width="100%" active-class="primary" v-if="marker.type=='Column'" class="font-weight-bold"
                  value="Column">
                  <img src="/icons/column-pin.png" width="30">
                  <span class="white--text">Columna</span>
                </v-btn>
                <v-btn width="100%" active-class="primary" v-else-if="marker.type=='Building'" class="font-weight-bold" value="Building">
                  <img src="/icons/building-pin.png" width="30">
                  <span class="white--text">Edificio</span>
                </v-btn>
                <v-btn width="100%" active-class="primary" v-else class="font-weight-bold" value="Home">
                  <img src="/icons/home-pin.png" width="30">
                  <span class="white--text">Casa</span>
                </v-btn>

              </v-btn-toggle>
            </v-col>
            <v-col class="col-12">
              <formsFieldsTextComponent readonly label-color="white--text" background-color="white" class="black--text"
                v-model="marker.reference" label="Referencia"></formsFieldsTextComponent>
            </v-col>
            <template v-if="marker.type == 'Building'">
            <v-col class="col-12">
              <formsFieldsTextComponent readonly label-color="white--text" background-color="white" class="black--text"
                v-model="marker.floor" type="number" label="Piso"></formsFieldsTextComponent>
            </v-col>
            <v-col class="col-12">
              <formsFieldsTextComponent readonly label-color="white--text" background-color="white" class="black--text"
                v-model="marker.apto" type="number" label="Apto"></formsFieldsTextComponent>
            </v-col>
            </template>
            <v-col class="col-12">
              <formsFieldsTextComponent type="number"  v-if="marker.type=='House'"  label-color="white--text" background-color="white" class="black--text"
                v-model="marker.permisor" label="Permisor"></formsFieldsTextComponent>
            </v-col>


          </v-row>
        </v-card-text>
        <v-card-actions>
          <v-btn color="red" class="rounded-xl" width="30%" @click="deleteMarker()">
            Eliminar
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary font-weight-regular rounded-xl" width="30%" @click="connectMarker()">
            Conectar&nbsp;<img src="/icons/pin.png" width="30">
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary font-weight-regular rounded-xl" width="30%" @click="updateMarker()">
            Actualizar UBI
          </v-btn>

        </v-card-actions>

      </template>
      <template v-else>
        <v-card-text>
          Mueve el marcador para cambiarle la posicion
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-btn color="primary font-weight-regular rounded-xl" width="45%" @click="updateMarker()">
            Cancelar
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary font-weight-regular rounded-xl" width="45%" @click="updateMarkerPos()">
            Actualizar&nbsp;<img src="/icons/pin.png" width="30">
          </v-btn>

        </v-card-actions>
      </template>
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
    data() {
      return {
        updateMarkerContent: false
      }
    },
    methods: {
      connectMarker() {
        this.$emit('connectmarker')
      },
      updateMarker() {
        this.$emit('updatemarker')
        this.updateMarkerContent = !this.updateMarkerContent
      },
      updateMarkerPos(){
        this.$axios.put('/api/columns/'+this.marker.id, {
          data:{
            lat: this.marker.lat,
            lng: this.marker.lng
          }
        }).then(res => {
          this.$emit('updateModifiedMarker')
          this.updateMarkerContent = !this.updateMarkerContent
        }).catch(err => {
          console.log(err)
        })
      },
      deleteMarker(){
        this.$axios.delete('/api/columns/'+this.marker.id).then(res => {
          this.$emit('pindeleted')
        }).catch(err => {
          console.log(err)
        })
      }

    }
  }

</script>

<style>

</style>
