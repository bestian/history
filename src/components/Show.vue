<template>
  <div class="hello">
    <h1>{{ msg }} {{ $route.params.url }}</h1>
      <l-map ref = "myMap" style="height: 400px; width: 100%" :zoom="zoom" :center="center">
        <l-tile-layer :url="url"></l-tile-layer>
        <l-marker v-for = "(d, index) in data" :lat-lng="markerLatLng(d)">        
          <l-icon
            :icon-size="dynamicSize"
            :icon-anchor="dynamicAnchor"
            :icon-url="d.img" >
          </l-icon>
        </l-marker>
      </l-map>
      <div class="ui container">  
      <table class = "ui fixed single line celled striped table">
        <tr v-for = "(d, index) in data" :key = "d" v-show="index > 0">
          <img class="ui small image" :src="d.img" />
          {{ d.name }} （公元 {{d.birth}} ~ {{d.death}}）
        </tr>
      </table>
    </div>
  </div>
</template>

<script>

import {LMap, LTileLayer, LMarker, LIcon } from 'vue2-leaflet'

export default {
  name: 'Show',
  components: {
      LMap,
      LTileLayer,
      LMarker,
      LIcon
  },
  data () {
    return {
      msg: 'Showing',
      data: '',
      url: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
      zoom: 2,
      center: [47.313220, -1.319482],
      latLng: [47.313220, -1.319482]
    }
  },
  computed: {
    dynamicSize () {
      return [this.iconSize, this.iconSize * 1.15];
    },
    dynamicAnchor () {
      return [this.iconSize / 2, this.iconSize * 1.15];
    }
  },
  methods: {
    markerLatLng (d) {
      console.log(d.latlngColumn.replace('"', '').replace('"', '').split('?? '))
      return (d.latlngColumn.replace('"', '').replace('"', '').split('?? '))
    },
    processData (allText) {
      var allTextLines = allText.split(/\r\n|\n/)
      var headers = allTextLines[0].split(',')
      var lines = []

      for (var i = 1; i < allTextLines.length; i++) {
        var data = allTextLines[i].split(',')
        if (data.length === headers.length) {
          var tarr = {}
          for (var j = 0; j < headers.length; j++) {
            tarr[headers[j]] = data[j]
          }
          lines.push(tarr)
        }
      }
      console.log(lines)
      return lines
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.$refs.myMap.mapObject.ANY_LEAFLET_MAP_METHOD();
    })
    this.$http.get('https://ethercalc.org/' + this.$route.params.url + '.csv').then((response) => {
      console.log(response)
      this.data = this.processData(response.data)
    }, (response) => {
      console.log(response)
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.hello {

}

</style>
