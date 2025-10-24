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

// triangle icon as an SVG string
const triangleSVG = `
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24">
  <polygon points="12,2 22,22 2,22"
           fill="orange"
           stroke="darkorange"
           stroke-width="2"
           filter="url(#shadow)" />
  <defs>
    <filter id="shadow" x="-20%" y="-20%" width="140%" height="140%">
      <feDropShadow dx="1" dy="1.5" stdDeviation="1.5" flood-color="rgba(0,0,0,0.4)" />
    </filter>
  </defs>
</svg>
`;



onMounted(() => {
  map = L.map('map', { zoomControl: true }).setView([-20, 60], 2)

// Dark basemap (CartoDB Dark Matter — free and fast)
L.tileLayer(
  'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',
  {
    attribution:
      '&copy; <a href="https://carto.com/attributions">CARTO</a> | &copy; <a href="https://www.openstreetmap.org/copyright">OSM</a>',
    subdomains: 'abcd',
    maxZoom: 19
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
.map-wrap { position: relative; }
#map {
  height: 100vh;
  width: 100vw;
}

/* Overlay */
.overlay {
  position: absolute; /* sits above the map but inside the same stacking context */
  inset: 0;
  z-index: 1001;      /* Leaflet controls use z-index ~1000 */
  pointer-events: none; /* allow map interactions except where panel/backdrop are */
}

.panel {
  position: absolute;
  top: 0;
  right: 0;
  width: min(420px, 90vw);
  height: 100%;
  background: #fff;
  box-shadow: -8px 0 24px rgba(0,0,0,.15);
  display: flex;
  flex-direction: column;
  gap: 0;
  pointer-events: auto;  /* re-enable interactions on the panel */
  overflow: auto;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: .75rem 1rem;
  border-bottom: 1px solid #eee;
}

.close-btn {
  background: transparent;
  border: 0;
  font-size: 1.5rem;
  line-height: 1;
  cursor: pointer;
}

.backdrop {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,.25);
  pointer-events: auto; /* captures clicks to close */
}
</style>