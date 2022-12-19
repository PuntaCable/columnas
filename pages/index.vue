<template>
  <v-container fluid class="pa-0">
    <v-card class="fill-height p-relative">
      <l-map ref="map" :center="latLng(mapCenter.lat, mapCenter.lng)" :zoom="mapZoom">
        <div class="p-relative">
          <l-tile-layer :url="url" />
          <l-marker z-index="1000" v-for="(pos,index) in geoMultiple" :key="index" @click="setMarker(pos)"
            :lat-lng="latLng(pos.lat, pos.lng)">
            <l-icon :icon-size="[40, 40]" class="d-flex align-center">
              <img style="width:40px!important" :src="`/icons/${pos.type.toLowerCase()}-pin.png`">
              <span class="font-weight-regular black--text">{{ pos.id }}</span>
            </l-icon>
          </l-marker>
          <l-marker draggable z-index="100" @dragend="setPos" :lat-lng="latLng(myPosition.lat, myPosition.lng)"
            :duration="1200">
            <l-icon :icon-size="[70, 70]" icon-url="/icons/pin.png"></l-icon>
          </l-marker>

          <div class="slidePet d-flex align-center flex-column z-index">
            <mapMarkersCreateConnectionComponent :marker="marker" :points="points" v-model="infoConnectMarker">
            </mapMarkersCreateConnectionComponent>
            <mapMarkersShowComponent :marker="marker" @connectmarker="connectMarker" v-model="showMarker">
            </mapMarkersShowComponent>
            <mapMarkersCreateComponent :latLng="myPosition" @pincreated="getMarkers()" :value="modal">
            </mapMarkersCreateComponent>
            <v-btn rounded color="primary font-weight-bold" class="mt-4" @click.prevent="modal = !modal"
              v-show="!showMarker && !infoConnectMarker">
              <template v-if="modal">
                Cerrar
                <v-icon color="white">mdi-close</v-icon>
              </template>
              <template v-else>
                Agregar marcador
                <img src="/icons/pin.png" width="30">
              </template>
            </v-btn>
          </div>
          <v-btn fab absolute small right top color="blue" class="mt-14 z-index" @click="setLocationUser()">
            <v-icon color="white">mdi-list</v-icon>
          </v-btn>
        </div>
      </l-map>

    </v-card>
  </v-container>
</template>

<script>
  const L = require('leaflet');
  var Routing = require('leaflet-routing-machine/src/');

  import {
    latLng
  } from 'leaflet';
  import {
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
  } from 'vue2-leaflet';
  import 'leaflet/dist/leaflet.css';
  import LMovingMarker from 'vue2-leaflet-movingmarker'

  export default {
    layout: 'empty',
    components: {
      LMap,
      LTileLayer,
      LIcon,
      LMarker,
      LMovingMarker,
    },
    data() {
      return {
        url: 'https://api.maptiler.com/maps/streets/256/{z}/{x}/{y}.png?key=oqNjPRZr4BmLmnzKyL60',
        latLng: latLng,
        mapZoom: 18,
        modal: false,
        showMarker: false,
        infoConnectMarker: false,
        marker: {},
        myPosition: {
          lat: -34.9185,
          lng: -54.9716,
        },
        geoMultiple: [],
        points: [],
        mapCenter: {
          lat: -34.9185,
          lng: -54.9716,
        },
        waypoints: [],
        findPet: null,
      }
    },
    created() {},
    mounted() {
      console.log(Routing)
      this.getMarkers()
    },
    methods: {
      getMarkers() {
        this.$axios.get('/api/columns/?populate=*')
          .then((response) => {
            this.geoMultiple = response.data.data
          })
          .catch((error) => {
            console.log(error)
          })
      },
      setMarker(e) {
        if (this.infoConnectMarker) {
          this.points.push({
            id: e.id,
            latLng: (new L.LatLng(e.lat, e.lng))
          })
          console.log(this.points.map(point => point.latLng))
          var firstpolyline = new L.polyline(this.points.map(point => point.latLng), {
            color: 'red',

            weight: 3,
            opacity: 0.5,
            smoothFactor: 1

          });
          L.Lin
          firstpolyline.addTo(this.$refs.map.mapObject);

        } else {
          this.marker = e
          this.showMarker = true
        }
      },
      connectMarker() {
        this.points = []
        this.points.push({
          id: this.marker.id,
          latLng: new L.LatLng(this.marker.lat, this.marker.lng)
        })
        this.showMarker = false
        this.infoConnectMarker = true
      },
      setPos(e) {
        console.log(e)
        this.myPosition.lat = e.target._latlng.lat
        this.myPosition.lng = e.target._latlng.lng
      },
      getPosition() {
        var vm = this
        navigator.geolocation.getCurrentPosition(() => {
          vm.mapCenter = vm.myPosition
        })
        navigator.geolocation.watchPosition(function (position) {
          vm.myPosition.lat = position.coords.lat
          vm.myPosition.lng = position.coords.lng
          vm.$forceUpdate()
        });
      },
      setLocationUser() {
        this.mapCenter = this.myPosition
        this.mapZoom = 15
      },

      setMapZoom() {
        var markerArray = []

        for (let position in this.geoMultiple) {
          var marker = L.marker(new L.LatLng(this.geoMultiple[position].lat, this.geoMultiple[position].lng));
          markerArray.push(marker);
        }

        if (this.myPosition.lat)
          markerArray.push(L.marker(new L.LatLng(this.myPosition.lat, this.myPosition.lng)));

        var group = L.featureGroup(markerArray);
        this.$refs.map.fitBounds(group.getBounds())
      },
    },
    watch: {
      infoConnectMarker(val) {
        if (!val) {
          for (i in this.$refs.map._layers) {
            if (this.$refs.map._layers[i].options.format == undefined) {
              try {
                this.$refs.map.removeLayer(map._layers[i]);
              } catch (e) {
                console.log("problem with " + e + map._layers[i]);
              }
            }
          }

        }
      }
    },
  }

</script>

<style scoped>
  .card-pet {
    position: absolute;
    bottom: 20px;
    left: 0;
    right: 0;
    width: 80%;
    margin: 0 auto;
  }

  .title-map {
    background-color: #f3f3f3;
    position: absolute;
    z-index: 1;
    top: 0;
    left: 0;
    right: 0;
  }

  .p-relative {
    position: relative;
    height: 100%;
  }

  .fill-height {
    height: calc(100vh - 48px) !important;
  }

  .slidePet {
    position: absolute;
    bottom: 5%;
    margin: 0 auto;
    left: 0;
    right: 0;

  }

  .cardPet {
    margin: 0 auto;
    width: 70vw;
  }

  .z-index {
    z-index: 1000000;
  }

</style>
