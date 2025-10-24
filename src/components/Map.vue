<template>
  <div class="map-wrap">
    <div id="map"></div>

    <!-- Slide-over overlay -->
    <div v-if="selectedPlace" class="overlay" @keydown.esc="close">
      <div class="panel" role="dialog" aria-modal="true">
        <header class="panel-header">
          <h3>{{ selectedPlace.title }}</h3>
          <button class="close-btn" @click="close" aria-label="Close">×</button>
        </header>

        
        <PlacePreview :place="selectedPlace" />

      </div>
      <div class="backdrop" @click="close" />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onBeforeUnmount } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import PlacePreview from './PlacePreview.vue'
//import placesData from '@/assets/output.geojson'

const selectedPlace = ref(null)
let map

const close = () => { selectedPlace.value = null }

onMounted(() => {

  // Define world bounds to prevent panning too far
  const worldBounds = L.latLngBounds([[-60, -180], [75, 180]]);
  map = L.map('map', { zoomControl: true,
    worldCopyJump: false,          // don't jump between wrapped worlds
    maxBounds: L.latLngBounds([[-85, -180], [85, 180]]),        // hard pan limit
    maxBoundsViscosity: 1.0,        // 0..1; 1 = sticky at edges
    zoomControl: true
  }).setView([15, 60], 2);



// Dark basemap (CartoDB Dark Matter — free and fast)
L.tileLayer(
  'https://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}.png',
  {
    attribution:
      '&copy; <a href="https://carto.com/attributions">CARTO</a> | &copy; <a href="https://www.openstreetmap.org/copyright">OSM</a>',
    subdomains: 'abcd',
    maxZoom: 19,
    minZoom: 2,
    noWrap: true,                 // <- key: stop horizontal repetition
    bounds: worldBounds
  }
).addTo(map);

// Load the volcano GeoJSON
fetch(`output.geojson`)
  .then(res => res.json())
  .then(data => {
    // Style function for yellow circle markers
    const volcanoStyle = {
      radius: 6,
      fillColor: '#FFD700', // bright yellow
      color: '#000',         // thin black border
      weight: 1,
      opacity: 1,
      fillOpacity: 0.8
    };

    L.geoJSON(data, {
      pointToLayer: (feature, latlng) => L.circleMarker(latlng, volcanoStyle),
      onEachFeature: (feature, layer) => {
        const { name, altitude_m } = feature.properties;
        layer.bindPopup(`<strong>${name}</strong><br>Alt: ${altitude_m} m`);
      }
    }).addTo(map);
  })
  .catch(console.error);
});

onBeforeUnmount(() => {
  if (map) {
    map.off('click', close)
    map.remove()
  }
})
</script>

<style scoped>
.map-wrap {
  height: 100%;
  width: 100%;
}

/* Let Leaflet size to the container */
#map {
  height: 100%;
  width: 100%;
  background: #262626;
}

/* Keep controls readable on dark basemap */
.leaflet-control-attribution {
  background: rgba(0,0,0,.4);
  color: #ccc;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 12px;
}
</style>