<script setup>
import {ref} from 'vue'
    const tipos = ref([
        {id: 1, nome: 'Reversível'}
    ])

    const tipoSelecionado = ref('Reversível')
</script>

<template>
    <div class="container-seletor">
        <label for="tipo-maquina" class="seletor-maquina">Tipo de Máquina de Turing</label>
        <select id="tipo-maquina" class="seletor" v-model="tipoSelecionado">
            <option v-for="t in tipos" :key="t.id" :value="t.nome">
                {{ t.nome }}
            </option>
        </select>
    </div>
</template>

<style scoped>
.container-seletor{
    display: flex;
    gap: 8px;
    margin-bottom: 2%;
    align-items: center;
}

.seletor {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #000000;
  background-color: #d9d9d9;
  font-size: 14px;
}
</style>