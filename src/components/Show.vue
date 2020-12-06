<template>
  <div class="hello">
    <div class="ui top menu">
      <div class="item">
        <div class="ui search">
          <div class="ui icon input">
            <input class="prompt" type="text" placeholder="搜詢" v-model="myKey">
            <i class="search icon"></i>
          </div>
          <div class="results" v-show = "myKey">
            <div v-for = "d in countD()" :key="d.name + d.img">
              <img :src="d.img" class="ui mini image avatar">
              {{d.name}}
            </div>
          </div>
        </div>
      </div>
      <a class="item" :href="'https://ethercalc.net/' + this.$route.params.url" target="_blank">
        <img class="fat-only" src = "https://www.google.com/s2/favicons?domain=www.ethercalc.net"/> 資料
      </a>
    </div>
    <div class="ui top menu">
      <div class = "item ui input">
        <input type="text" name="" v-model = "year" />
      </div>
      <div class="item">
        <div class="ui checkbox">
          <input type="checkbox" name="" v-model="aliveOnly">
          <label>並世</label>
        </div>
      </div>
    </div>
    <div class="ui top menu">
      <div class="item ui button group">
        <a class="ui green button" @click = "speedup(-10)">
          <i class="angle double left icon"/>
        </a>
        <a class="ui green button" @click = "speedup(-1)">
          <i class="angle left icon"/>
        </a>
        <a class="ui red button" @click = "speed = 0">
          ||
        </a>
        <a class="ui green button" @click = "speedup(1)">
          <i class="angle right icon"/>
        </a>
        <a class="ui green button" @click = "speedup(10)">
          <i class="angle double right icon"/>
        </a>
      </div>
      <div class="item fat-only">
        {{speed}}
      </div>
    </div>
    <l-map ref = "myMap" style="height: 420px; width: 100%" :zoom="zoom" :center="center">
      <l-tile-layer :url="url"></l-tile-layer>
      <l-marker v-for = "d in countD()" :lat-lng="markerLatLng(d)" :key="d.name + d.img" v-if="d.name">
        <l-icon
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          :icon-url="d.img || 'https://icon-library.net/images/google-map-marker-icon/google-map-marker-icon-17.jpg'" >
        </l-icon>
        <l-tooltip>{{d.name}}
          ({{showAge(d)}})
        </l-tooltip>
        <l-popup :content="'<h3><a href=\'https://zh.wikipedia.org/wiki/' + d.name  + '\' target=\'_blank\'>' + d.name + '</a></h3>' + '<br/>' + showAge(d) +'<br/>' + '領域：' + d.fields + '<br/>' + '成就：' + d.works"></l-popup>
      </l-marker>
    </l-map>
    <div class="ui menu">
      <a class = "item striped" v-for = "y in range(102)" @click = "year = y*30-1000" :key="y"></a>
    </div>
  </div>
</template>

<script>

import * as L from 'leaflet'
import { LMap, LTileLayer, LMarker, LIcon, LTooltip, LPopup } from 'vue2-leaflet'

export default {
  name: 'Show',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
    LTooltip,
    LPopup
  },
  data () {
    return {
      myKey: '',
      aliveOnly: false,
      year: 2019,
      speed: 0,
      msg: 'Showing',
      data: [],
      url: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
      zoom: 2,
      center: [47.313220, -1.319482],
      latLng: [47.313220, -1.319482],
      iconSize: 50
    }
  },
  computed: {
    dynamicSize () {
      return [this.iconSize, this.iconSize * 1.15]
    },
    dynamicAnchor () {
      return [this.iconSize / 2, this.iconSize * 1.15]
    }
  },
  methods: {
    range (r) {
      var ans = []
      for (var i = 0; i < r; i++) {
        ans.push(i)
      }
      return ans
    },
    showAge (d) {
      if (!this.aliveOnly) {
        return (d.birth + '~' + d.death)
      } else {
        return (this.year - d.birth + '歲')
      }
    },
    countD () {
      var a = this.aliveOnly
      var y = this.year
      var ans = this.data.filter(function (d) {
        return !a || (d.birth < y && d.death > y)
      })

      if (this.myKey) {
        var k = this.myKey
        ans = ans.filter(function (d) {
          return JSON.stringify(d).match(new RegExp(k))
        })
      }

      return ans
    },
    speedup (n) {
      this.speed += n
    },
    go () {
      this.year = parseInt(this.year)
      this.year += this.speed
    },
    markerLatLng (d) {
      var ans = d.latlngColumn.replace('"', '').replace('"', '').split('?? ')

      if (!ans[1]) { return this.latLng }

      return L.latLng(parseFloat(ans[0]), parseFloat(ans[1]))
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
      return lines
    }
  },
  mounted () {
    this.$http.get('https://ethercalc.net/' + this.$route.params.url + '.csv').then((response) => {
      this.data = this.processData(response.data)
    }, (response) => {
      console.log(response)
    })
    this.timer = setInterval(this.go, 1000)
  },
  beforeDestroy () {
    clearInterval(this.timer)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.hello {

}

.leaflet-marker-icon {
  border-radius: 50%;
  position: relative; /* Affects Leaflet behaviour */
}

.striped {
  padding: 2px !important;
}
.striped:nth-child(even) {
  background-color: red !important;
}
.striped:nth-child(odd) {
  background-color: pink !important;
}

.ui.search > .results {
  display: block;
}

.ui.menu {
  margin: 0;
}

</style>
