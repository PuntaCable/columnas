<template>
  <v-slide-y-reverse-transition>
    <v-card elevation="24" width="90%" class="mx-auto rounded-xl" v-if="value">
      <v-card-title class="white--text">
        <v-icon class="mr-2">mdi-map-marker</v-icon>
        <span class="font-weight-bold">Distancia entre Col.</span>
        <v-spacer></v-spacer>
        <v-btn icon @click="closeDialog()">
            <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      <v-divider></v-divider>
      <v-card-text>
        <v-row v-for="(markerGroup,index) in markersGroup" :key="index">
          <v-col class="col-4 d-flex flex-column justify-center align-center">
            <img width="20" :src="`/icons/column-pin.png`">
            <span class="white--text font-weight-regular">{{ markerGroup[0].id }}</span>
          </v-col>
          <template v-if="markerGroup[1]!= undefined">
            <v-col class="col-4 d-flex flex-column justify-center align-center">
              <v-divider class="divider"></v-divider>
              <span class="font-weight-regular white--text">{{ getDistance(markerGroup) }} M</span>
            </v-col>
            <v-col class="col-4 d-flex flex-column justify-center align-center">
              <img width="20" :src="`/icons/column-pin.png`">
              <span class="white--text font-weight-regular">{{ markerGroup[1].id }}</span>
            </v-col>
          </template>
        </v-row>
      </v-card-text>
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
      points: {
        type: Array,
        default: () => []
      },
      marker: {
        type: Object,
        default: () => {}
      }
    },
    methods: {
      createConnection() {
        const updateConnections = async function (marker) {
          this.$axios.put('/api/columns/' + marker.id, {
            data: marker
          }).
          this.$emit('pincreated')

        }
      },
      connectMarker() {
        this.$emit('connectmarker')
      },
      getDistance(markers) {
        //getDistance in meters with leaflet 
        console.log(markers)
        if (markers[1] == undefined) return
        const distance = markers[0].latLng.distanceTo(markers[1].latLng)
        return Math.round(distance)
      },
      closeDialog(){
        this.$emit('points',[])
        this.$emit('input', false)
      }
    },
    computed: {
      markersGroup() {
        const groupsOfTwo = [];
        for (let i = 0; i < this.points.length; i++) {
          if (i === this.points.length - 1) {
            groupsOfTwo.push([this.points[i], this.points[i - 1]]);
          } else {
            groupsOfTwo.push(this.points.slice(i, i + 2));
          }
        }
        groupsOfTwo.pop()
        return groupsOfTwo
      }
    }
  }

</script>

<style>
  .divider {
    border-width: 2px !important;
    border-color: red !important;
    width:100%
  }

</style>
