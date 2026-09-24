<script setup>
import { ref } from 'vue';
import Arquivo from './components/Arquivo.vue';
import MaquinaTuring from  './components/MaquinaTuring.vue'
import Navbar from './components/Barra.vue'
import Seletor from './components/Seletor.vue'

const dadosMaquina = ref({
  estados: [],
  alfEntrada: [],
  alfFita: [],
  quintuplas: [],
  entrada: '',
})

function atualizarDados(dados) {
  dadosMaquina.value = dados
}
</script>

<template>
  <Navbar/>
  <div class="site">
    <Seletor/>
    <Arquivo @dados-atualizados="atualizarDados" />
    <MaquinaTuring v-bind="dadosMaquina" />
  </div>
</template>

<style scoped>
.site{
  margin: 2%;
}
</style>
