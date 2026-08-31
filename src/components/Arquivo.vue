<template>
    <div class="container-input">
        <p>Escolha o arquivo de funções de transição</p>
        <input class="input-arq" type="file" accept=".txt" @change="lerArquivo">
    </div>
    <div>
        <p>Conteúdo do arquivo</p>
        <pre> {{ conteudo }}</pre>
    </div>
</template>

<script setup>
import {ref} from 'vue'

const conteudo = ref('')

function lerArquivo(evento){
    const arquivo = evento.target.files[0]
    if(!arquivo) return

    const leitor = new FileReader()
    leitor.onload = (e) => {
        conteudo.value = e.target.result
    }

    leitor.readAsText(arquivo)
}
</script>

<style>
.container-input{
    display: flex;
    align-items: center;
}

.input-arq{
    margin-left: 20px;
}
</style>