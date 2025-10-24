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

const year       = ref(1970)
const minYear    = 1970
const maxYear    = 2025
const tickMs     = 900
const intervalId = ref(null)

let map

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

  // --- Year control (slider + play/pause + display) ---
  const YearControl = L.Control.extend({
    onAdd() {
      const container = L.DomUtil.create('div', 'leaflet-bar year-control')
      container.innerHTML = `
        <div class="yc-row">
          <button class="yc-btn" aria-label="Play/Pause" title="Play/Pause">⏸</button>
          <div class="yc-display">${year.value}</div>
        </div>
        <input class="yc-slider" type="range" min="${minYear}" max="${maxYear}" step="1" value="${year.value}" />
        <div class="yc-minmax"><span>${minYear}</span><span>${maxYear}</span></div>
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

      slider.addEventListener('keydown', (e) => {
        if (e.key === 'ArrowLeft' || e.key === 'ArrowDown') {
          e.preventDefault(); setYear(Math.max(minYear, year.value - 1))
        } else if (e.key === 'ArrowRight' || e.key === 'ArrowUp') {
          e.preventDefault(); setYear(Math.min(maxYear, year.value + 1))
        } else if (e.key === ' ') {
          e.preventDefault(); btn.click()
        }
      })

      return container
    }
  })

  const ctl = new YearControl({ position: 'bottomleft' })
  map.addControl(ctl)

  //Load geojson data
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
      L.geoJSON(data, {
        pointToLayer: (feature, latlng) => L.circleMarker(latlng, volcanoStyle),
        onEachFeature: (feature, layer) => {
          const { name, altitude_m } = feature.properties
          layer.bindPopup(`<strong>${name}</strong><br>Alt: ${altitude_m} m`)
        }
      }).addTo(map)
    })

  
  setYear(year.value)
  startTick()
})

onBeforeUnmount(() => {
  stopTick()
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

#map {
  height: 100%;
  width: 100%;
  background: #262626;
}

.leaflet-control-attribution {
  background: rgba(0,0,0,.4);
  color: #ccc;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 12px;
}

/* Year control styling */
:deep(.year-control) {
  background: rgba(0,0,0,.6);
  color: #ffd700;
  padding: .5rem .75rem;
  border-radius: .5rem;
  width: 260px;
  box-shadow: 0 2px 10px rgba(0,0,0,.35);
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
  accent-color: #ffd700;   /* modern browsers */
}

:deep(.year-control .yc-minmax) {
  display: flex; justify-content: space-between;
  font-size: .75rem; color: #bbb; margin-top: .25rem;
}
</style>