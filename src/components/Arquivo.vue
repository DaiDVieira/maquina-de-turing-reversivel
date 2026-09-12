<template>
    <div class="container-input">
        <p>Escolha o arquivo de funções de transição</p>
        <label for="arquivo" class="botao-arquivo">
            Selecionar arquivo
        </label>
        <input id="arquivo" class="input-arq" type="file" accept=".txt" @change="lerArquivo"/>
        <span class="nome-arq" v-if="nomeArquivo"> Arquivo carregado: {{ nomeArquivo }}</span>
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
import Botao from './Botao.vue'

const nomeArquivo = ref('')
const conteudo = ref('')

function lerArquivo(evento){
    const arquivo = evento.target.files[0]
    if(!arquivo) return

    const leitor = new FileReader()
    leitor.onload = (e) => {
        conteudo.value = e.target.result;
        nomeArquivo.value = arquivo.name
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
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
    background-color: #d9d9d9;
}

.nome-arq{
    margin-left: 10px;
}

.botao-arquivo{
    display: inline-block;
    padding: 10px 20px;
    margin-left: 10px;
    background-color: #4155c8;
    color: white;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
}

.botao-arquivo:hover{
    background-color: #214D71;
}

.arq{
    display: block;
    align-items: center;
    padding: 20px;
    background-color: #d9d9d9;
    border-radius: 8px;
    margin: 20px;
    text-align: center;
    border: 1px solid #000000;
}

.cont-arq{
    display: grid;
    grid-template-columns: 1fr 1fr; 
    gap: 30px;

}
</style>