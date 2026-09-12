<script setup>
import {ref} from 'vue'
import Botao from './Botao.vue'

const fitaEntrada = ref(["_", "_", "_", "_", "_", "_", "_", "_", "_", "_"])
//ver se símbolo de branco sempre será o último na definição do alfabeto. mudar na inicialização das fitas
const fitaHistorico = ref(["_", "_", "_", "_", "_", "_", "_", "_", "_", "_"])
const fitaSaida = ref(["_", "_", "_", "_", "_", "_", "_", "_", "_", "_"])
const posicaoCabecoteEntrada = ref(0)
const posicaoCabecoteHistorico = ref(0)
const posicaoCabecoteSaida = ref(0)

//controle de execucao para mudar opacidade de botoes e linhas
const faseExecucao = ref(1)

function separaElementosEntrada(){

}

function escreveSimbolo(){

}
</script>
    
<template>
    <div class="maquina">
        <div class="entrada-maq">
            <p>Cadeia de entrada</p>
           <!--Aqui vai a string lida na última linha do arquivo-->
        </div>
        <!--
        <div class="entrada-maq">
            <p>Digite a cadeia de entrada</p>
            <input type="text" v-model="fitaEntrada">
        </div>
    -->
        <div class="container-controle">
            <p>Execução por fases</p>
            <div class="container-botoes">
                
                <!--
                <Botao class="botao" texto="Passo a passo" />
                <Botao class="botao" texto="Completa"/>
                -->
                <Botao class="botao" :class="{'ativo': faseExecucao >= 1}" texto="1. Computação" />
                <span class="linha" :class="{'ativo': faseExecucao >= 2}"></span>
                <Botao class="botao" :class="{'ativo': faseExecucao >= 2}" texto="2. Cópia"/>
                <span class="linha" :class="{'ativo': faseExecucao >= 3}"></span>
                <Botao class="botao" :class="{'ativo': faseExecucao >= 3}" texto="3. Reconstrução"/>
            </div>

            <p>Controle</p>
            <div class="container-botoes">
                <Botao texto="Executar" :class="{'ativo': 1}" />
                <Botao texto="Passo a passo" :class="{'ativo': 1}"/>
                <Botao texto="Reiniciar" :class="{'ativo': 1}"/>
            </div>
        </div>

        <div class="conteiner-maq">
            <div class="container-fita">
                <p class="tipo-fita">Fita de entrada</p>
                <div v-for="(celula, index) in fitaEntrada" :key="index" :class="['celula', {'cabecote-ativo': index == posicaoCabecoteEntrada}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Fita de histórico</p>
                <div v-for="(celula, index) in fitaHistorico" :key="index" :class="['celula', {'cabecote-ativo': index == posicaoCabecoteHistorico}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Fita de saída</p>
                <div v-for="(celula, index) in fitaSaida" :key="index" :class="['celula', {'cabecote-ativo': index == posicaoCabecoteSaida}]">
                    {{ celula || '_' }}
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.entrada-maq{
    margin-bottom: 20px;
}

.container-fita{
    border: 1px solid #763232;
    display: flex;
    align-items: center;
    overflow-x: auto;
    padding: 20px;
    background-color: #d9d9d9;
    border-radius: 8px;
    margin-bottom: 20px;
}

.tipo-fita{
    padding: 10px;
    margin-right: 20px;
}

.celula{
    position: relative;
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

.cabecote-ativo{
    border-color: #98633F;
    box-shadow: 0 0 10px 2px rgba(152, 99, 63, 0.35);
}

.cabecote-ativo::after{
    content: '';
    position: absolute;
    top: -10px;
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 0;
    border-left: 8px solid transparent;
    border-right: 8px solid transparent;
    border-top: 8px solid #98633F;
}

.container-botoes{
    display: flex;
    justify-content: space-between;
    margin: 2% 5% 2% 5%;
    align-items: center;
}

.linha{
    background-color: #4155c898;
    height: 10px;
    flex-grow: 1;
    display: block;
}

.botao{
    margin: 0;
    flex-shrink: 0;
    background-color: #4155c898;
}

.botao.ativo,
.linha.ativo{
    background-color: #4155c8;
}

.botao:hover{
    background-color: #214d7198;
}

.botao.ativo:hover{
    background-color: #214d71;
}
</style>