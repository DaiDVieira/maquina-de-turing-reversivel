<script setup>
import {computed, watch, ref} from 'vue'
import Botao from './Botao.vue'

const props = defineProps({
    estados: { type: Array, default: () => [] },
    alfEntrada: { type: Array, default: () => [] },
    alfFita: { type: Array, default: () => [] },
    quintuplas: { type: Array, default: () => [] },
    quadruplas: { type: Array, default: () => [] },
    entrada: { type: String, default: '' },
})

const tamanhoFitaPadrao = 20;
const fitaEntrada = ref(Array(tamanhoFitaPadrao).fill('B'));
const fitaHistorico = ref(Array(tamanhoFitaPadrao).fill('B'));
const fitaSaida = ref(Array(tamanhoFitaPadrao).fill('B'));

const posicaoCabecoteEntrada = ref(0)
const posicaoCabecoteHistorico = ref(0)
const posicaoCabecoteSaida = ref(0)

//controle de execucao
const faseExecucao = ref(1) //computacao = 1, copia = 2, reconstrucao = 3
const estadoAtual = ref('1') //assume-se que sempre sera 1
const estadoAceitacao = computed(() => {
    return props.estados.length > 0 ? props.estados[props.estados.length - 1] : ''
})

watch(() => props.estados, (novosEstados) => {
        if(novosEstados && novosEstados.length > 0){
            estadoAtual.value = novosEstados[0];
        }
    },
    {
        immediate: true
    }
);

watch(() => props.entrada, () => {
        if (props.entrada) {
            reiniciarMaquina();
        }
    },
    {
        immediate: true
    }
);

function encontrarQuadruplaEscrita() {
    const simboloLido = fitaEntrada.value[posicaoCabecoteEntrada.value] ?? 'B';

    return props.quadruplas.find(q => {
        return (q.tipo === 'escrever' && q.estadoOrigem === String(estadoAtual.value).trim() && q.simboloLido === String(simboloLido).trim());
    });
}

function encontrarQuadruplaMovimento(indiceTransicao) {
    return props.quadruplas.find(q => {
        return (q.tipo === 'mover' && q.indiceTransicao === indiceTransicao);
    });
}

function executarUmaTransicao() {
    if(estadoAtual.value ===estadoAceitacao.value) {
        return false;
    }

    const simboloLido = fitaEntrada.value[posicaoCabecoteEntrada.value] ?? 'B';

    //busca primeira quadrupla
    const quadruplaEscrita = encontrarQuadruplaEscrita();
    if (!quadruplaEscrita) {
        alert(`Nenhuma transição encontrada para ` + `o estado ${estadoAtual.value} ` + `e símbolo '${simboloLido}'.`);
        return false;
    }

    //busca segunda quadrupla
    const quadruplaMovimento = encontrarQuadruplaMovimento(quadruplaEscrita.indiceTransicao);
    if (!quadruplaMovimento) {
        alert(`A transição ${quadruplaEscrita.indiceTransicao} ` + `não possui uma quadrupla de movimento.`);
        return false;
    }

    const simboloEscrito = quadruplaEscrita.simboloEscrito;
    const estadoIntermediario = quadruplaEscrita.estadoIntermediario;
    const movimento = quadruplaMovimento.movimento;
    const proximoEstado = quadruplaMovimento.proximoEstado;

    //executa primeira quadrupla
    const novaFitaEntrada = [...fitaEntrada.value];
    novaFitaEntrada[posicaoCabecoteEntrada.value] = simboloEscrito;
    fitaEntrada.value = novaFitaEntrada;

    estadoAtual.value = estadoIntermediario;

    //registra historico na fita
    const novaFitaHistorico = [...fitaHistorico.value];
    novaFitaHistorico[posicaoCabecoteHistorico.value] = estadoIntermediario;
    fitaHistorico.value = novaFitaHistorico;

    //executa segunda quadrupla
    if (movimento === 'R') {
        posicaoCabecoteEntrada.value = Math.min(tamanhoFitaPadrao - 1, posicaoCabecoteEntrada.value + 1);
        posicaoCabecoteHistorico.value = Math.min(tamanhoFitaPadrao - 1, posicaoCabecoteHistorico.value + 1);
    }

    else if (movimento === 'L') {
        posicaoCabecoteEntrada.value = Math.max(0, posicaoCabecoteEntrada.value - 1);
        posicaoCabecoteHistorico.value = Math.max(0, posicaoCabecoteHistorico.value + 1);
    }

    estadoAtual.value = proximoEstado;

    if(estadoAtual.value === estadoAceitacao.value) {
        faseExecucao.value = 2; //termina computacao
    }
    return true;
}

function executarFaseComputacao() {
    executarUmaTransicao();
}

function executarComputacaoCompleta() {
    if (faseExecucao.value !== 1) {
        return;
    }
    let limitePassos = tamanhoFitaPadrao * 100; //evitar loop infinito
    let passosExecutados = 0;

    while (estadoAtual.value !== estadoAceitacao.value && passosExecutados < limitePassos) {
        const executou = executarUmaTransicao();
        if (!executou) {
            break;
        }
        passosExecutados++;
    }

    if(passosExecutados >= limitePassos && estadoAtual.value !== estadoAceitacao.value) {
        alert('A computação excedeu o limite de passos ' + 'permitido. Verifique se a máquina possui ' + 'um ciclo infinito.');
        return;
    }
    console.log(`Computação finalizada após ` +`${passosExecutados} transições.`);
}

function executarFaseCopia(){

}

function executarFaseReconstrucao(){

}

function executarMaq(){
    executarComputacaoCompleta()
    //criar outras funcoes para executar cada fase
}

//limpa todas as fitas e inicializa a entrada
function reiniciarMaquina() {
    const novaFitaEntrada = Array(tamanhoFitaPadrao).fill('B');
    const caracteres = props.entrada.trim().split('');
    caracteres.forEach((char, index) => {
        if (index < tamanhoFitaPadrao) {
            novaFitaEntrada[index] = char;
        }
    });
    fitaEntrada.value = novaFitaEntrada;

    fitaHistorico.value = Array(tamanhoFitaPadrao).fill('B');

    fitaSaida.value = Array(tamanhoFitaPadrao).fill('B');

    posicaoCabecoteEntrada.value = 0;
    posicaoCabecoteHistorico.value = 0;
    posicaoCabecoteSaida.value = 0;

    estadoAtual.value = props.estados.length > 0 ? props.estados[0] : '';

    faseExecucao.value = 1;
}
</script>
    
<template>
    <div class="maquina">
        <div class="entrada-maq">
            <p>Cadeia de entrada</p>
            <p>{{ props.entrada }}</p>
        </div>

        <div class="container-controle">
            <p>Execução por fases</p>
            <div class="container-botoes">
                <Botao class="botao" :class="{'ativo': faseExecucao >= 1}" texto="1. Computação" @acao="executarComputacaoCompleta"/>
                <span class="linha" :class="{'ativo': faseExecucao >= 2}"></span>
                <Botao class="botao" :class="{'ativo': faseExecucao >= 2}" texto="2. Cópia"/>
                <span class="linha" :class="{'ativo': faseExecucao >= 3}"></span>
                <Botao class="botao" :class="{'ativo': faseExecucao >= 3}" texto="3. Reconstrução"/>
            </div>

            <p>Controle</p>
            <div class="container-botoes">
                <Botao texto="Executar" :class="{'ativo': 1}" @acao="executarMaq"/>
                <Botao texto="Passo a passo" :class="{'ativo': 1}" @acao="executarFaseComputacao"/>
                <Botao texto="Reiniciar" :class="{'ativo': 1}" @acao="reiniciarMaquina"/>
            </div>
        </div>

        <p>Fitas</p>
        <p class="paragrafo">Estado atual: {{ estadoAtual }}</p>
        <div class="conteiner-maq">
            <div class="container-fita">
                <p class="tipo-fita">Entrada</p>
                <div v-for="(celula, index) in fitaEntrada" :key="index" :class="['celula', {'cabecote-ativo': index == posicaoCabecoteEntrada}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Histórico</p>
                <div v-for="(celula, index) in fitaHistorico" :key="index" :class="['celula', {'cabecote-ativo': index == posicaoCabecoteHistorico}]">
                    {{ celula || '_' }}
                </div>
            </div>

            <div class="container-fita">
                <p class="tipo-fita">Saída</p>
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
    min-width: 110px;
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

.paragrafo{
    margin-bottom: 2%;
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