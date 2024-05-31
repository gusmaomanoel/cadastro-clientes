<template>
  <div id="app">
    <div class="app-content">
      <NavBar />
      <router-view />
    </div>
  </div>
</template>
<script setup>
import NavBar from "./components/NavBar/NavBar.vue";
</script>

<style lang="scss">
@import "@/shared/css/_general.scss";
.app-content {
  width: 100%;
  height: 100%;
  background-color: $blue-101;
}
</style>
