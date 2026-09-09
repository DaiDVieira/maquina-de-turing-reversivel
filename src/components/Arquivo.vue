<template>
    <div class="container-input">
        <p>Escolha o arquivo de funções de transição</p>
        <input class="input-arq" type="file" accept=".txt" @change="lerArquivo">
    </div>

    <div class="cont-arq">
        <div class="arq">
            <h3>Quintuplas</h3>
            <pre> {{ conteudo }}</pre>
        </div>
        
        <div class="arq">
            <h3>Quadruplas</h3>
        </div>
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

.arq{
    display: block;
    align-items: center;
    padding: 20px;
    background-color: #f5f5f5;
    border-radius: 8px;
    margin: 20px;
    width: 900px;
    text-align: center;
}

.cont-arq{
    display: flex;
}
</style>