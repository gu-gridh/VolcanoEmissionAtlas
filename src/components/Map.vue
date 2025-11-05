<template>
  <div class="map-wrap">
    <div id="map" />

    <!-- Slide-over overlay -->
    <div
      v-if="selectedPlace"
      class="overlay"
      tabindex="0"
      @keydown.esc="close"
      ref="overlayEl"
    >
      <div class="panel" role="dialog" aria-modal="true" aria-labelledby="panel-title">
        <header class="panel-header">
          <h3 id="panel-title">{{ selectedPlace.title }}</h3>
          <button class="close-btn" @click="close" aria-label="Close">×</button>
        </header>

        <PlacePreview :place="selectedPlace" />
      </div>

      <div class="backdrop" @click="close" />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onBeforeUnmount, nextTick } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import PlacePreview from './PlacePreview.vue'

const selectedPlace = ref(null)

const year       = ref(1970)
const minYear    = 1970
const maxYear    = 2025
const tickMs     = 900
const intervalId = ref(null)

const overlayEl = ref(null)

let map
let geoLayer 

const close = () => { selectedPlace.value = null }

function setYear(newYear) {
  year.value = newYear
  const el = document.querySelector('.year-control')
  if (el) {
    const display = el.querySelector('.yc-display')
    const slider  = el.querySelector('.yc-slider')
    if (display) display.textContent = String(year.value)
    if (slider && Number(slider.value) !== year.value) slider.value = String(year.value)
  }
  // TODO: update layers based on year.value here
}

function startTick() {
  if (intervalId.value) return
  intervalId.value = setInterval(() => {
    setYear(year.value + 1 > maxYear ? minYear : year.value + 1)
  }, tickMs)
}

function stopTick() {
  if (!intervalId.value) return
  clearInterval(intervalId.value)
  intervalId.value = null
}

function getPlaceFromFeature(feature) {
  // Map your feature.properties into what PlacePreview expects
  const { name, altitude_m, description } = feature.properties || {}
  return {
    title: name ?? 'Unknown place',
    description:
      description ??
      (altitude_m != null
        ? `Altitude: ${altitude_m} m`
        : 'No description available'),
    raw: feature.properties
  }
}

onMounted(() => {
  const worldBounds = L.latLngBounds([[-60, -180], [75, 180]])

  map = L.map('map', {
    zoomControl: true,
    worldCopyJump: false,
    maxBounds: worldBounds,
    maxBoundsViscosity: 1.0
  }).setView([15, 60], 2)

  L.tileLayer(
    'https://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}.png',
    {
      attribution: '&copy; CARTO &copy; OSM',
      subdomains: 'abcd',
      maxZoom: 19,
      minZoom: 2,
      noWrap: true,
      bounds: worldBounds
    }
  ).addTo(map)

  // Year control
  const YearControl = L.Control.extend({
    onAdd() {
      const container = L.DomUtil.create('div', 'leaflet-bar year-control')
      container.innerHTML = `
        <div class="yc-row">
          <button class="yc-btn" aria-label="Play/Pause" title="Play/Pause">⏸</button>
          <div class="yc-display">${year.value}</div>
        </div>
        <input class="yc-slider" type="range" min="${minYear}" max="${maxYear}" step="1" value="${year.value}" />
      `
      L.DomEvent.disableClickPropagation(container)
      L.DomEvent.disableScrollPropagation(container)

      const btn    = container.querySelector('.yc-btn')
      const slider = container.querySelector('.yc-slider')

      btn.addEventListener('click', () => {
        if (intervalId.value) { stopTick(); btn.textContent = '▶️' }
        else { startTick(); btn.textContent = '⏸' }
      })

      slider.addEventListener('input', (e) => {
        stopTick()
        btn.textContent = '▶️'
        setYear(Number(e.target.value))
      })

      return container
    }
  })

  map.addControl(new YearControl({ position: 'bottomleft' }))

  // Load GeoJSON 
  fetch('output.geojson')
    .then(r => r.json())
    .then(data => {
      const volcanoStyle = {
        radius: 6,
        fillColor: '#FFD700',
        color: '#000',
        weight: 1,
        opacity: 1,
        fillOpacity: 0.8
      }

      geoLayer = L.geoJSON(data, {
        pointToLayer: (feature, latlng) => L.circleMarker(latlng, volcanoStyle),
        onEachFeature: (feature, layer) => {
          layer.on('click', () => {
            selectedPlace.value = getPlaceFromFeature(feature)
            nextTick(() => overlayEl.value?.focus())
          })
        }
      }).addTo(map)
    })

  setYear(year.value)
  startTick()
})

onBeforeUnmount(() => {
  stopTick()
  if (geoLayer) {
    geoLayer.remove()
    geoLayer = null
  }
  if (map) {
    map.remove()
    map = null
  }
})
</script>

<style scoped>
.map-wrap {
  height: 100%;
  width: 100%;
  position: relative;
}

#map {
  height: 100%;
  width: 100%;
  background: #262626;
}

/* --- Overlay / panel --- */
.overlay {
  position: absolute;
  inset: 0;
  z-index: 500;            
  outline: none;         
}

.backdrop {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,.45);
}

.panel {
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
  width: min(520px, 92vw);
  background: #111;
  color: #eee;
  box-shadow: -12px 0 30px rgba(0,0,0,.45);
  z-index: 1;
  display: flex;
  flex-direction: column;
  border-left: 1px solid #222;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  background: #161616;
  border-bottom: 1px solid #222;
}

.close-btn {
  appearance: none;
  border: 0;
  background: #222;
  color: #ddd;
  padding: 6px 10px;
  border-radius: 8px;
  cursor: pointer;
}
.close-btn:hover { background: #2a2a2a; }

.leaflet-control-attribution {
  background: rgba(0,0,0,.4);
  color: #ccc;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 12px;
}

/* --- Year control styles --- */
:deep(.year-control) {
  background: rgba(0,0,0,.6);
  color: #ffd700;
  padding: .5rem .75rem;
  border-radius: .5rem;
  width: 260px;
  box-shadow: 0 2px 10px rgba(0,0,0,.35);
  margin-bottom: 150px;
  margin-left: 20px;
}

:deep(.year-control .yc-row) {
  display: flex; align-items: center; gap: .5rem; margin-bottom: .35rem;
}

:deep(.year-control .yc-btn) {
  cursor: pointer;
  background: rgba(255,255,255,.1);
  border: 1px solid rgba(255,255,255,.2);
  color: #ffd700;
  border-radius: .5rem;
  padding: .25rem .5rem;
  font-size: 1rem;
}

:deep(.year-control .yc-display) {
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  font-size: 1.25rem;
  letter-spacing: .5px;
}

:deep(.year-control .yc-slider) {
  width: 100%;
  accent-color: #ffd700;  
}

:deep(.year-control .yc-minmax) {
  display: flex; justify-content: space-between;
  font-size: .75rem; color: #bbb; margin-top: .25rem;
}
</style>