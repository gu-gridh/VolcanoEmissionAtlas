<template>
    <div id="map"></div>
</template>

<script setup>
import { onMounted } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

onMounted(() => {
    const map = L.map('map').setView([51.505, -0.09], 3)

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
    }).addTo(map)

    //L.marker([51.505, -0.09]).addTo(map)
})
</script>

<style scoped>
#map {
    height: 100vh;
    width: 100vw;
}
</style>
