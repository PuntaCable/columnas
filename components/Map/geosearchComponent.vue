<template>
  <v-input hide-details>
    <v-autocomplete hide-details outlined item-text="label" filled hide-no-data dense :items="itemsAutocomplete"
      no-filter return-object @change="updateFinderPos" :search-input.sync="searchItems" label="Buscar ubicacion">
    </v-autocomplete>
    <v-btn fab small class="gradient-secondary rounded-sm ml-1 elevation-1" @click="findMyPosition">
      <v-icon color="white">ion-ios-locate</v-icon>
    </v-btn>
  </v-input>

</template>

<script>
  import {
    GoogleProvider
  } from 'leaflet-geosearch';

  export default {
    props:{
      value:Object,
    },
    data() {
      return {
        itemsAutocomplete: [{}],
        searchItems: ''

      }
    },
    methods: {
      findMyPosition() {
        navigator.geolocation.getCurrentPosition((e) => {
          this.$emit('input',{...this.value, ...{
            lat: e.latitude,
            lng: e.longitude
          }})
        })
      },
      async findPos(e) {
        const provider = new GoogleProvider({
          params: {
            key: 'AIzaSyD6dg8rpq7Z3fX_Lkrkhyk97yCpeF-bkbs',
          },
        });
        return await provider.search({
          query: e
        });
      },
      updateFinderPos(e) {
        this.$emit('input', {...this.value, ...{
          lat: e.y,
          lng: e.x
        }})
      },

    },
    watch: {
      async searchItems(e) {
        this.itemsAutocomplete = await this.findPos(e)
      },
    }
  }

</script>

<style>

</style>
