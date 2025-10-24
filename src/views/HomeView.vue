<script setup>
import Map from '@/components/Map.vue'
</script>

<template>
  <div class="home">
    <h1>Volcano Emission Atlas</h1>
    <p>This is a prototype application to visualize volcanic emissions data.</p>
  <div class="mapView">
    <Map />
  </div>
  </div>
</template>

<style scoped>

.home {
    text-align: center;
    width: 100%;
    margin-bottom: 2rem;
}

.mapView {
    width: 80%;
    height: 70vh;
    margin: 0 auto;
    border: 2px solid #ccc;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

</style>

