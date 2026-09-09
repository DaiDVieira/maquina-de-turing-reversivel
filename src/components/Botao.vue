<script setup lang="js">
import { ref } from 'vue';

defineProps({
    texto: String,
    tipo: {type: String, default: 'padrão'}
})

const emite = defineEmits(['acao'])

function clique(){
    console.log('Clique no botao')
}
</script>

<template>
    <button class="botao" :class="tipo" @click="clique">{{ texto }}</button>
</template>

<style scoped lang="css">
.botao{
    padding: 8px;
    border-radius: 8px;
    width: 150px;
    text-align: left;
    cursor: pointer;
    background: #4155c8;
    color: #d6d4d7;
}
</style>