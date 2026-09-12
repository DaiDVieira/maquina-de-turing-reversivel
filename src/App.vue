<script setup>
import Arquivo from './components/Arquivo.vue';
import MaquinaTuring from  './components/MaquinaTuring.vue'
import Navbar from './components/Barra.vue'
import Seletor from './components/Seletor.vue'

</script>

<template>
  <Navbar/>
  <div class="site">
    <Seletor/>
    <Arquivo />
    <MaquinaTuring />
  </div>
</template>

<style scoped>
.site{
  margin: 2%;
}
</style>
