<script setup>
import Map from '@/components/Map.vue'
</script>

<template>
  <div class="home">
    <header class="app-header">
      <h1>Volcano Emission Atlas</h1>
    </header>
    <div class="map-slot">
      <Map />
    </div>
  </div>
</template>

<style>
html, body, #app {
  height: 100%;
  margin: 0;
  padding: 0;
  width: 100%;
}

.home {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
}

.app-header {
  flex: 0 0 auto;
  background: #262626;
  color: #727065;
  padding: 1rem .75rem;
}

.map-slot {
  flex: 1 1 auto;
  min-height: 0;           
  position: relative;      
  overflow: hidden;
  background-color: #262626;
}
</style>