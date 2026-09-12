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
    width: 150px;
    text-align: center;
    cursor: pointer;
    background-color: #4155c8;
    color: #fcf5f5;
    font-weight: bold;
    font-size: 15px;
    font-family: 'Times New Roman', Times, serif;
    display: inline-block;
    padding: 10px 20px;
    margin-left: 10px;
    border-radius: 6px;   
    border: 0;
}

.botao:hover{
    background-color: #214D71;
}
</style>