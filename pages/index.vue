<template>
  <v-container fluid class="pa-0">

    <v-card class="fill-height p-relative">
      <v-card-title>
        <mapExportComponent :points="geoMultiple"></mapExportComponent>
        <formsFieldsTextButtonComponent class="mt-2" v-model="search" dense background-color="white" label="Buscar..." icon="mdi-magnify"></formsFieldsTextButtonComponent>
      </v-card-title>
      <l-map ref="map" @update:zoom="checkZoom" :center="latLng(mapCenter.lat, mapCenter.lng)" :zoom="mapZoom">
        <div class="p-relative">
          <l-tile-layer :url="url" />
          <l-marker z-index="1000" v-for="(pos,index) in geoMultiple" :draggable="pos.draggable" @dragend="setMarkerPos"
            :key="index" @click="setMarker(pos)" :lat-lng="latLng(pos.lat, pos.lng)">
            <l-icon :icon-size="[40, 40]" class="d-flex align-center">
              <img style="width:40px!important" :src="`/icons/${pos.type.toLowerCase()}-pin.png`">
              <span class="font-weight-regular black--text">{{ pos.id }}</span>
            </l-icon>
          </l-marker>
          <template v-if="showPadrones">
            <l-marker z-index="1000" v-for="(m,index) in markers" :key="'P'+index" :lat-lng="latLng(m.lat, m.lng)">
              <l-icon :icon-size="[70, 70]" class="pt-4">
                <div class="pt-6 pl-3">
                  <span class="black--text">{{ m.padron }}</span>
                </div>
              </l-icon>
            </l-marker>
          </template>

          <l-marker :zIndexOffset="10000" draggable @dragend="setPos" :lat-lng="latLng(myPosition.lat, myPosition.lng)"
            :duration="1200">
            <l-icon :icon-size="[70, 70]" icon-url="/icons/pin.png">
            </l-icon>
          </l-marker>

          <div class="slidePet d-flex align-center flex-column z-index">
            <mapMarkersCreateConnectionComponent :marker="marker" :points="points" v-model="infoConnectMarker">
            </mapMarkersCreateConnectionComponent>
            <mapMarkersShowComponent :marker="marker" @connectmarker="connectMarker"
              @updateModifiedMarker="updateModifiedMarker" @updatemarker="setMarkerDragable" @pindeleted="getMarkers();showMarker=false" v-model="showMarker">
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
          <v-btn absolute small right top color="blue" class="z-index font-weight-regular"
            @click="showPadrones = !showPadrones">
            <v-icon v-if="showPadrones">mdi-eye-off</v-icon>
            <v-icon v-else>mdi-eye</v-icon>
            padrones
          </v-btn>
          <v-btn absolute small right top color="blue" style="margin-top:40px" class="z-index font-weight-regular"
            @click="follow = !follow">
            <v-icon v-if="follow">mdi-check-circle</v-icon>
            <v-icon v-else>mdi-close-circle</v-icon>
            Seguir
            </v-btn>
            <v-btn absolute small @click="setLocationUser()" right fab top color="blue" class="z-index font-weight-regular" style="margin-top:110px">
            <!--map center icon -->
            <v-icon>mdi-crosshairs-gps</v-icon>
          </v-btn>

        </div>
      </l-map>

    </v-card>
  </v-container>
</template>

<script>
  const L = require('leaflet');
  var Routing = require('leaflet-routing-machine/src/');
  const esri = require('esri-leaflet');
  var qs = require('qs');

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
        mapZoom: 20,
        modal: false,
        showMarker: false,
        infoConnectMarker: false,
        showPadrones: true,
        follow: true,
        esri: esri,
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
        polylines: L.featureGroup(),
        findPet: null,
        loading: false,
        markers: [],
        search:""
      }
    },
    created() {
      this.getPosition()
    },
    mounted() {
      this.getMarkers()
      const labelClass = {
        // autocasts as new LabelClass()
        symbol: {
          type: "text", // autocasts as new TextSymbol()
          color: "green",
          font: { // autocast as new Font()
            family: "Playfair Display",
            size: 12,
            weight: "bold"
          }
        },
        labelPlacement: "above-center",
        labelExpressionInfo: {
          expression: "$feature.objectid",
          value: "-"
        },
        "maxScale": 14864,
        "minScale": 0,


      };

      this.setAllPadrones()
      this.esri.dynamicMapLayer({
        url: "https://gis.maldonado.gub.uy/arcgis/rest/services/Servicios_AGOL/Maldonado_Base/MapServer/",
      }).addTo(this.$refs.map.mapObject);
        this.$refs.map.mapObject.panTo(this.latLng(this.myPosition.lat, this.myPosition.lng))

    },


    methods: {
      setAllPadrones() {
        var vm = this
        var esri = this.esri.featureLayer({
          url: " https://gis.maldonado.gub.uy/arcgis/rest/services/Servicios_AGOL/Maldonado_Base/MapServer/11",
          resultRecordCount: 100,
          minZoom: 18,
          style: function () {
            return {
              color: "transparent",
              weight: 2
            };
          },

        }).addTo(this.$refs.map.mapObject);
        var markers = []
        esri.on('createfeature', function (e) {
          var bounds = new L.LatLngBounds(e.feature.geometry.coordinates);
          markers.push({
            padron: e.feature.properties.padron,
            lat: bounds.getCenter().lng,
            lng: bounds.getCenter().lat
          })
        })

        esri.on('load', () => {
          vm.markers = []
          const mapBounds = this.$refs.map.mapObject.getBounds()
          vm.markers = markers.filter((m) => {
            if (mapBounds.contains([m.lat, m.lng])) {
              return m
            }
          })
        })


      },
      checkZoom(e) {
        if (e < 18) {
          this.markers = []
        } else {
          this.setAllPadrones()
        }
      },
      getMarkers() {
        this.modal = false
        this.$axios.get('/api/columns/?populate=*',{
        })
          .then((response) => {
            this.geoMultiple = response.data.data
          })
          .catch((error) => {
            console.log(error)
          })
      },
      searchMarkers() {
        this.modal = false
        let filters = null
        if(this.search !=""){
          filters ={
            filters:{
              reference:{
                $contains:this.search
              }
            },
          }
        }
        this.$axios.get('/api/columns/?populate=*',{
          params:filters,
          paramsSerializer: params => {
          return qs.stringify(params, {
            arrayFormat: 'brackets'
          })
          }


        })
          .then((response) => {
            if(response.data.data[0]){
              this.mapCenter = response.data.data[0]
            }
          })
          .catch((error) => {
            console.log(error)
          })
      },


      setMarkerDragable() {
        let indexMarker = this.geoMultiple.findIndex(marker => marker.id === this.marker.id)
        const marker = this.geoMultiple[indexMarker]
        let draggable = (marker.draggable) ? false : true
        this.$set(this.geoMultiple, indexMarker, {
          ...this.marker,
          draggable: draggable
        })
      },
      setMarkerPos(e) {
        console.log(e.target)
        this.marker.lat = e.target._latlng.lat
        this.marker.lng = e.target._latlng.lng
      },
      updateModifiedMarker() {
        let indexMarker = this.geoMultiple.findIndex(marker => marker.id === this.marker.id)
        this.$set(this.geoMultiple, indexMarker, this.marker)
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
          this.polylines.addLayer(firstpolyline)
          this.$refs.map.mapObject.addLayer(this.polylines)

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
        this.loading = true
        navigator.geolocation.getCurrentPosition((position) => {
          vm.myPosition.lat = position.coords.latitude
          vm.myPosition.lng = position.coords.longitude
        })
        navigator.geolocation.watchPosition(function (position) {
          if(vm.follow){
            vm.myPosition.lat = position.coords.latitude
            vm.myPosition.lng = position.coords.longitude
            vm.mapCenter.lat = position.coords.latitude
            vm.mapCenter.lng = position.coords.longitude
          }
        });
        vm.$forceUpdate()
      },
      setLocationUser() {
        this.$refs.map.mapObject.panTo(new L.LatLng(this.myPosition.lat, this.myPosition.lng));
        this.$forceUpdate()
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
      search(){
        this.searchMarkers()
      },
      mapZoom(val) {
        console.log(val)
        if (val < 18) {
          this.markers = []
        }
      },
      infoConnectMarker(val) {
        if (!val) {
          this.polylines.clearLayers()
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
    height: calc(100vh - 100px) !important;
  }

  .slidePet {
    position: absolute;
    bottom: 15%;
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
