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

const selectedPlace = ref(null)
let map

const places = [
  { id: 1, title: 'London', lat: 51.505, lng: -0.09, info: 'Some data for London' },
  { id: 2, title: 'Paris',  lat: 48.8566, lng: 2.3522, info: 'Some data for Paris' },
]

const close = () => { selectedPlace.value = null }

onMounted(() => {
  map = L.map('map', { zoomControl: true }).setView([51.505, -0.09], 4)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19
  }).addTo(map)

  // Add markers and set place selected
  places.forEach(p => {
    const m = L.marker([p.lat, p.lng]).addTo(map)
    m.on('click', () => {
      selectedPlace.value = p
      //zoom to place if not already zoomed in
        map.setView([p.lat, p.lng], Math.max(map.getZoom(), 6), { animate: true })
    })
  })
  map.on('click', close)
})

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