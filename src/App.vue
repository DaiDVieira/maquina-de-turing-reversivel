<script setup>
import Arquivo from './components/Arquivo.vue';
import MaquinaTuring from  './components/MaquinaTuring.vue'
</script>

<template>
  <div class="titulo">
    <h1>Simulador de Máquina de Turing Reversível</h1>
  </div>
  <Arquivo />
  <MaquinaTuring />
  
</template>

<style scoped>
.titulo{
  text-align: center;
}
</style>
