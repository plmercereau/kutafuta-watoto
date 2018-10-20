<template lang="pug">
  div
    div(style="height: 10%; overflow: auto;")
      span(v-if="loading") Loading...
      label(for="checkbox") GeoJSON Visibility
      input(id="checkbox", v-model="show", type="checkbox")
      label(for="checkboxTooltip") Enable tooltip
      input(id="checkboxTooltip", v-model="enableTooltip", type="checkbox")
      input(type="color", v-model="fillColor")
      br
    l-map(:zoom="zoom", :center="center", style="height: 450px")
      l-tile-layer(:url="url", :attribution="attribution")
      l-geo-json(v-if="show", :geojson="geojson", :options="options", :optionsStyle="styleFunction")
      l-marker(:lat-lng="marker")
</template>

<script>
import L from 'leaflet'
import { LMap, LTileLayer, LMarker, LGeoJson } from 'vue2-leaflet'
import axios from 'axios'
export default {
  name: 'Example',
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LMarker
  },
  data () {
    return {
      loading: false,
      show: true,
      enableTooltip: true,
      zoom: 15,
      center: [-6.82653, 39.25925],
      geojson: null,
      fillColor: '#e4ce7f',
      url: 'http://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker: L.latLng(-6.82653, 39.25925)
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
  created () {
    this.loading = true
    axios.get('https://rawgit.com/gregoiredavid/france-geojson/master/regions/pays-de-la-loire/communes-pays-de-la-loire.geojson').then(response => {
      this.geojson = response.data
      this.loading = false
    })
  }
}
</script>
