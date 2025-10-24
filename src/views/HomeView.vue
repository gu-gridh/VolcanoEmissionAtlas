<script setup>
import Map from '@/components/Map.vue'
</script>

<template>
  <div class="home">
    <header class="app-header">
      <h1>Volcano Emission Atlas</h1>
      <p>This is a prototype application to visualize volcanic emissions data.</p>
    </header>

    <!-- Map area fills all remaining height -->
    <div class="map-slot">
      <Map />
    </div>
  </div>
</template>

<style>
/* Make the whole app 100% height so flex works */
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

/* Header in normal flow (on top) */
.app-header {
  flex: 0 0 auto;
  background: #262626;
  color: #fff;
  text-align: center;
  padding: 1rem .75rem;
}

/* Map area fills the rest of the screen */
.map-slot {
  flex: 1 1 auto;
  min-height: 0;           /* important so child can shrink within flex container */
  position: relative;      /* gives the child a positioned ancestor if needed */
  overflow: hidden;
  background-color: #262626;
}
</style>