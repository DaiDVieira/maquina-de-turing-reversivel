<script setup>
import {ref} from 'vue'
import Botao from './Botao.vue'
    const fitaEntrada = ref([])
    const fitaHistorico = ref([])
    const fitaSaida = ref([])
    const posicaoCabecoteEntrada = ref(0)
    const posicaoCabecoteHistorico = ref(0)
    const posicaoCabecoteSaida = ref(0)

    function escreveSimbolo(){

    }
</script>
    
<template>
    <div class="maquina">
        <div class="entrada-maq">
            <p>Digite a cadeia de entrada</p>
            <input type="text" v-model="fitaEntrada">
        </div>

        <div class="conteiner-maq">
            
            <div class="container-fita">
                <p class="tipo-fita">Fita de entrada</p>
                <div v-for="(celula, index) in fitaEntrada" :key="index" :class="['celula', {'cabeçote-ativo': index == posicaoCabecoteEntrada}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Fita de histórico</p>
                <div v-for="(celula, index) in fitaHistorico" :key="index" :class="['celula', {'cabeçote-ativo': index == posicaoCabecoteHistorico}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Fita de saída</p>
                <div v-for="(celula, index) in fitaSaida" :key="index" :class="['celula', {'cabeçote-ativo': index == posicaoCabecoteSaida}]">
                    {{ celula || '_' }}
                </div>
            </div>
        </div>
    </div>
    <div class="container-botoes">
        <p>Execução</p>
        <Botao class="botao" texto="Passo a passo" />
        <Botao class="botao" texto="Imediata"/>
    </div>
</template>

<style scoped>
.entrada-maq{
    margin-bottom: 20px;
}

.container-fita{
    display: flex;
    align-items: center;
    overflow-x: auto;
    padding: 20px;
    background-color: #f5f5f5;
    border-radius: 8px;
    margin-bottom: 20px;
}

.tipo-fita{
    padding: 10px;
    margin-right: 20px;
}

.celula{
    min-width: 40px;
    height: 25px;
    border: 2px solid;
    border-color: #ccc;
    align-items: center;
    justify-content: center;
    justify-items: center;
    text-align: center;
    text-justify: center;
    font-family: monospace;
    font-size: 1.2rem;
    background-color: white;
    transition: all 0.2s ease;
}

.container-botoes{
    display: flex;
}

.botao{
    margin: 10px;
}
</style>