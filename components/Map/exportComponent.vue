<template>
  <v-btn block color="primary" class="font-weight-regular" @click="exportMap()">EXPORTAR</v-btn>
</template>

<script>
  var GeoJSON = require('geojson');
  const tokml = require('tokml');
  export default {
    props: {
      points: {
        type: Array,
        default: () => []
      }
    },
    methods: {
      exportMap() {
        const points = this.points.map((p)=>{
            return {
                latitude: p.lat,
                longitude: p.lng
            }
        })

        const geojsonObject = GeoJSON.parse(points, {
          Point: ['latitude', 'longitude'],
        }, {
          documentName: 'map',
          documentDescription: 'KML Export'
        })

        const response = tokml(geojsonObject);
        console.log(response)

        var pom = document.createElement('a');
        var bb = new Blob([response], {
          type: 'text/plain'
        });

        pom.setAttribute('href', window.URL.createObjectURL(bb));
        pom.setAttribute('download', 'map.xml');

        pom.dataset.downloadurl = ['text/plain', pom.download, pom.href].join(':');
        pom.draggable = true;
        pom.classList.add('dragout');

        pom.click();


      }
    }
  }

</script>

<style>

</style>
