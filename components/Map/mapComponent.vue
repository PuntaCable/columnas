<template>
  <no-ssr>
    <v-row>
      <v-col class="col-12">

      </v-col>
      <v-col class="col-12">
        <l-map ref="map" @dragend="checkPos" style="height:400px;z-index: 1;" :center="latLng(value.lat, value.lng)" :zoom="zoom">
          <l-tile-layer :url="url" />
          <l-marker :draggable="draggable" @dragend="updatePos" :lat-lng="latLng(value.lat, value.lng)">
            <l-icon :icon-size="[50, 50]" icon-url="/marker.png" />
          </l-marker>
        </l-map>
      </v-col>
    </v-row>
  </no-ssr>
</template>

<script>
  import {
    latLng
  } from 'leaflet';
  import {
    LMap,
    LTileLayer,
    LMarker,
    LIcon
  } from 'vue2-leaflet';
  import 'leaflet/dist/leaflet.css';

  export default {
    components: {
      LMap,
      LIcon,
      LMarker,
      LTileLayer,
    },
    props: {
      draggable: {
        default: false,
        type: Boolean
      },
      zoom: {
        default: 18,
        type: Number
      },
    },
    data() {
      return {
        url: 'https://api.maptiler.com/maps/streets/256/{z}/{x}/{y}.png?key=oqNjPRZr4BmLmnzKyL60',
        latLng: latLng,
        mapZoom: 18,
        searchItems: '',
        value:{
          lat: -34.9057331,
          lng: -54.9513517
        }
      }
    },
    methods: {
      updatePos(e) {
        this.$emit('input', e.target['_latlng'])
      },
      checkPos(e){
        console.log(e)
        //console.log(this.$refs.map.getCenter())
      }
    },
    watch: {
      myPosition: {
        handler(e) {
          this.$emit('input', e.target)
        },
        deep: true
      },
    }
  }

</script>
