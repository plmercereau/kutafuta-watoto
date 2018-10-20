<template lang="pug">
  div
    //- div(style="height: 10%; overflow: auto;")
    //-   span(v-if="loading") Loading...
    //-   label(for="checkbox") GeoJSON Visibility
    //-   input(id="checkbox", v-model="show", type="checkbox")
    //-   span {{zoom}}
    //-   br
    el-row
      el-date-picker(v-model="dateRange"
        type="daterange"
        :picker-options="datePickerOptions"
        range-separator="To"
        start-placeholder="Start date"
        end-placeholder="End date"
        v-on:change="changeDate"
        align="right")
    el-row
      el-col(:span="18")
        l-map(:zoom="zoom",
            :maxZoom="maxZoom",
            :minZoom="minZoom",
            :center="center",
            style="height: 500px;",
            :max-bounds="maxBounds")
          l-marker-cluster(:options="clusterOptions")
            l-marker(v-for="c in cases", :key="c.id", :lat-lng="c.latlng", @click="showDetails(c)")
              l-popup(:content="c.text")
          l-tile-layer(:url="url", :attribution="attribution", zoomControl="false")
          l-geo-json(v-if="show", :geojson="geojson", :options="options", :optionsStyle="styleFunction")
      el-col(:span="6") Child details
        div Id: {{ caseDetail.id }}
        div Admission date: {{ caseDetail.admission | moment("DD/MM/YYYY") }}
</template>

<script>
import L from 'leaflet'
import { LMap, LTileLayer, LMarker, LGeoJson, LPopup } from 'vue2-leaflet'
import axios from 'axios'
function rand (n) {
  let max = n + 0.03
  let min = n - 0.03
  return Math.random() * (max - min) + min
}
let casesData = []
for (let i = 0; i < 100; i++) {
  var date = new Date()
  date = date.setTime(date.getTime() - 3600 * 1000 * 24 * (Math.random() * 50))
  casesData.push({
    id: i,
    latlng: L.latLng(rand(-6.82653), rand(39.25925)),
    text: 'Child ' + i,
    admission: date
  })
}

export default {
  name: 'Example',
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LMarker,
    LPopup
  },
  data () {
    const end = new Date()
    const start = new Date()
    start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
    return {
      loading: false,
      show: true,
      zoom: 13,
      minZoom: 10,
      maxZoom: 18,
      center: [-6.82653, 39.25925],
      datePickerOptions: {
        shortcuts: [{
          text: 'Last week',
          onClick (picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: 'Last month',
          onClick (picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: 'Last 3 months',
          onClick (picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      },
      dateRange: [start, end],
      maxBounds: L.latLngBounds([[-6.7837, 39.2065], [-6.8915, 39.3328]]),
      geojson: null,
      url: 'http://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png',
      attribution: '',
      //   marker: L.latLng(-6.82653, 39.25925),
      cases: casesData,
      caseDetail: {},
      clusterOptions: {}
    }
  },
  computed: {
    options () {
      return {
        onEachFeature: this.onEachFeatureFunction
      }
    },
    styleFunction () {
      const fillColor = this.fillColor // important! need touch fillColor in computed for re-calculate when change fillColor
      return () => {
        return {
          weight: 2,
          color: '#ECEFF1',
          opacity: 1,
          fillColor: fillColor,
          fillOpacity: 1
        }
      }
    },
    onEachFeatureFunction () {
      if (!this.enableTooltip) {
        return () => {}
      }
      return (feature, layer) => {
        layer.bindTooltip('<div>code:' + feature.properties.code + '</div><div>nom: ' + feature.properties.nom + '</div>', { permanent: false, sticky: true })
      }
    }
  },
  methods: {
    showDetails (details) {
      this.caseDetail = details
    },
    changeDate (range) {
      this.cases = casesData.filter((c) => (c.admission >= range[0] && c.admission <= range[1]) )
    }
  },
  created () {
    this.loading = true
    axios.get('https://rawgit.com/gregoiredavid/france-geojson/master/regions/pays-de-la-loire/communes-pays-de-la-loire.geojson').then(response => {
      this.geojson = response.data
      this.loading = false
    })
  },
  mounted () {
    setTimeout(() => {
      this.$nextTick(() => {
        this.clusterOptions = { disableClusteringAtZoom: 17 }
      })
    }, 5000)
    this.changeDate(this.dateRange)
  }
}
</script>
<style>
@import "~leaflet.markercluster/dist/MarkerCluster.css";
@import "~leaflet.markercluster/dist/MarkerCluster.Default.css";
.flex-container {
  display: flex;
}
</style>
