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
const faseExecucao = ref(1) //computacao = 1, copia = 2, reconstrucao = 3, concluido = 4
const estadoAtual = ref('1') //assume-se que sempre sera 1
const estadoAceitacao = computed(() => {
    return props.estados.length > 0 ? props.estados[props.estados.length - 1] : ''
})
const quadruplasCopia = ref([])
const quadruplasReversao = ref([])

const movimentoInverso = { R: 'L', L: 'R', S: 'S' }

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

function listaQuadruplasAtiva() {
    return faseExecucao.value === 1 ? props.quadruplas : quadruplasCopia.value;
}

function encontrarQuadruplaEscrita() {
    const simboloLido = fitaEntrada.value[posicaoCabecoteEntrada.value] ?? 'B';

    return listaQuadruplasAtiva().find(q => {
        return (q.tipo === 'escrever' && q.estadoOrigem === String(estadoAtual.value).trim() && q.simboloLido === String(simboloLido).trim());
    });
}

function encontrarQuadruplaMovimento(indiceTransicao) {
    return listaQuadruplasAtiva().find(q => {
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

function executarUmPasso() {
    if (faseExecucao.value === 1) {
        executarUmaTransicao();
    } else if (faseExecucao.value === 2) {
        if (quadruplasCopia.value.length === 0) {
            posicaoCabecoteEntrada.value = 0; //garante inicio da copia na posicao 0
            gerarQuadruplasCopia();
        }
        executarUmaTransicaoCopia();
    } else if (faseExecucao.value === 3) {
        if (quadruplasReversao.value.length === 0) {
            prepararReconstrucao();
        }
        executarUmaTransicaoReversao();
    }
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

function gerarQuadruplasCopia() {
    let n = 0;
    while (n < tamanhoFitaPadrao && fitaEntrada.value[n] !== 'B') {
        n++;
    }

    const novasQuadruplas = [];
    for (let i = 0; i <= n; i++) {
        const simbolo = fitaEntrada.value[i];
        const estadoOrigem = (i === 0) ? estadoAceitacao.value : `copia_${i}`;
        const estadoIntermediario = `copia_${i}`;
        const proximoEstado = (i === n) ? 'copia_fim' : `copia_${i + 1}`;
        const transicao = (i === n) ? 'L' : 'R';

        novasQuadruplas.push({
            tipo: 'escrever',
            indiceTransicao: `copia_${i}`,
            estadoOrigem,
            simboloLido: simbolo,
            simboloEscrito: simbolo, //re-escreve na fita de entrada (idempotente)
            simboloEscritoSaida: simbolo, //escreve na fita de saida
            estadoIntermediario,
            texto: `(${estadoOrigem}, ${simbolo}) = (${simbolo}, ${estadoIntermediario})`,
        });

        novasQuadruplas.push({
            tipo: 'mover',
            indiceTransicao: `copia_${i}`,
            estadoIntermediario,
            movimento: transicao,
            proximoEstado,
            textoPadrao: `(${estadoIntermediario}, {${simbolo}, B, ${simbolo}}) = (${transicao}, ${proximoEstado})`,
        });
    }

    quadruplasCopia.value = novasQuadruplas;
}

function executarUmaTransicaoCopia() {
    if (estadoAtual.value === 'copia_fim') {
        return false;
    }

    const quadruplaEscrita = encontrarQuadruplaEscrita();
    if (!quadruplaEscrita) {
        alert(`Nenhuma transição de cópia encontrada para o estado ${estadoAtual.value}.`);
        return false;
    }

    const quadruplaMovimento = encontrarQuadruplaMovimento(quadruplaEscrita.indiceTransicao);
    if (!quadruplaMovimento) {
        alert(`A transição de cópia ${quadruplaEscrita.indiceTransicao} não possui uma quadrupla de movimento.`);
        return false;
    }

    //escreve na fita de entrada (idempotente) e na fita de saida
    const novaFitaEntrada = [...fitaEntrada.value];
    novaFitaEntrada[posicaoCabecoteEntrada.value] = quadruplaEscrita.simboloEscrito;
    fitaEntrada.value = novaFitaEntrada;

    const novaFitaSaida = [...fitaSaida.value];
    novaFitaSaida[posicaoCabecoteSaida.value] = quadruplaEscrita.simboloEscritoSaida;
    fitaSaida.value = novaFitaSaida;

    estadoAtual.value = quadruplaEscrita.estadoIntermediario;

    //move os cabecotes de entrada e saida (historico nao se move na copia)
    const movimento = quadruplaMovimento.movimento;
    if (movimento === 'R') {
        posicaoCabecoteEntrada.value = Math.min(tamanhoFitaPadrao - 1, posicaoCabecoteEntrada.value + 1);
        posicaoCabecoteSaida.value = Math.min(tamanhoFitaPadrao - 1, posicaoCabecoteSaida.value + 1);
    }

    else if (movimento === 'L') {
        posicaoCabecoteEntrada.value = Math.max(0, posicaoCabecoteEntrada.value - 1);
        posicaoCabecoteSaida.value = Math.max(0, posicaoCabecoteSaida.value - 1);
    }

    estadoAtual.value = quadruplaMovimento.proximoEstado;

    if (estadoAtual.value === 'copia_fim') {
        faseExecucao.value = 3;
    }
    return true;
}

function executarFaseCopia(){
    if (faseExecucao.value !== 2) {
        return;
    }

    posicaoCabecoteEntrada.value = 0; //garante inicio da copia na posicao 0
    gerarQuadruplasCopia();

    if (quadruplasCopia.value.length === 0) {
        faseExecucao.value = 3;
        return;
    }

    let limitePassos = tamanhoFitaPadrao;
    let passosExecutados = 0;

    while (estadoAtual.value !== 'copia_fim' && passosExecutados < limitePassos) {
        const executou = executarUmaTransicaoCopia();
        if (!executou) {
            break;
        }
        passosExecutados++;
    }

    if (passosExecutados >= limitePassos && estadoAtual.value !== 'copia_fim') {
        alert('A fase de cópia excedeu o limite de passos permitido.');
        return;
    }
    console.log(`Cópia finalizada após ${passosExecutados} transições.`);
}

//inverte cada par de quadruplas da computacao: primeiro desfaz o movimento, depois a escrita
function gerarQuadruplasReversao() {
    const novasQuadruplas = [];

    props.quadruplas.filter(q => q.tipo === 'mover').forEach(quadruplaMovimento => {
        const quadruplaEscrita = props.quadruplas.find(q => {
            return (q.tipo === 'escrever' && q.indiceTransicao === quadruplaMovimento.indiceTransicao);
        });
        const estadoIntermediario = quadruplaMovimento.estadoIntermediario;
        const movimento = movimentoInverso[quadruplaMovimento.movimento];

        novasQuadruplas.push({
            tipo: 'mover',
            indiceTransicao: quadruplaMovimento.indiceTransicao,
            estadoOrigem: quadruplaMovimento.proximoEstado,
            simboloHistorico: estadoIntermediario,
            movimento,
            proximoEstado: estadoIntermediario,
            texto: `(${quadruplaMovimento.proximoEstado}, {*, ${estadoIntermediario}, *}) = (${movimento}, ${estadoIntermediario})`,
        });

        novasQuadruplas.push({
            tipo: 'escrever',
            indiceTransicao: quadruplaMovimento.indiceTransicao,
            estadoOrigem: estadoIntermediario,
            simboloLido: quadruplaEscrita.simboloEscrito,
            simboloEscrito: quadruplaEscrita.simboloLido,
            proximoEstado: quadruplaEscrita.estadoOrigem,
            texto: `(${estadoIntermediario}, ${quadruplaEscrita.simboloEscrito}) = (${quadruplaEscrita.simboloLido}, ${quadruplaEscrita.estadoOrigem})`,
        });
    });

    quadruplasReversao.value = novasQuadruplas;
}

//volta os cabecotes para onde a computacao terminou e gera as quadruplas de reversao
function prepararReconstrucao() {
    gerarQuadruplasReversao();

    let n = 0;
    while (n < tamanhoFitaPadrao && fitaHistorico.value[n] !== 'B') {
        n++;
    }

    //a posicao final do cabecote de entrada sai dos movimentos registrados no historico
    let posicao = 0;
    for (let i = 0; i < n; i++) {
        const quadruplaMovimento = props.quadruplas.find(q => {
            return (q.tipo === 'mover' && q.estadoIntermediario === fitaHistorico.value[i]);
        });
        if (quadruplaMovimento.movimento === 'R') {
            posicao = Math.min(tamanhoFitaPadrao - 1, posicao + 1);
        }
        else if (quadruplaMovimento.movimento === 'L') {
            posicao = Math.max(0, posicao - 1);
        }
    }

    posicaoCabecoteEntrada.value = posicao;
    posicaoCabecoteHistorico.value = Math.max(0, n - 1);
    estadoAtual.value = estadoAceitacao.value;
}

function executarUmaTransicaoReversao() {
    const simboloHistorico = fitaHistorico.value[posicaoCabecoteHistorico.value] ?? 'B';
    if (simboloHistorico === 'B') {
        faseExecucao.value = 4; //nada mais a desfazer
        return false;
    }

    //busca primeira quadrupla
    const quadruplaMovimento = quadruplasReversao.value.find(q => {
        return (q.tipo === 'mover' && q.estadoOrigem === String(estadoAtual.value).trim() && q.simboloHistorico === simboloHistorico);
    });
    if (!quadruplaMovimento) {
        alert(`Nenhuma transição de reversão encontrada para o estado ${estadoAtual.value} e histórico '${simboloHistorico}'.`);
        return false;
    }

    let novaPosicaoEntrada = posicaoCabecoteEntrada.value;
    if (quadruplaMovimento.movimento === 'R') {
        novaPosicaoEntrada = Math.min(tamanhoFitaPadrao - 1, novaPosicaoEntrada + 1);
    }
    else if (quadruplaMovimento.movimento === 'L') {
        novaPosicaoEntrada = Math.max(0, novaPosicaoEntrada - 1);
    }

    //busca segunda quadrupla
    const simboloLido = fitaEntrada.value[novaPosicaoEntrada] ?? 'B';
    const quadruplaEscrita = quadruplasReversao.value.find(q => {
        return (q.tipo === 'escrever' && q.estadoOrigem === quadruplaMovimento.proximoEstado && q.simboloLido === String(simboloLido).trim());
    });
    if (!quadruplaEscrita) {
        alert(`Nenhuma transição de reversão encontrada para o estado ${quadruplaMovimento.proximoEstado} e símbolo '${simboloLido}'.`);
        return false;
    }

    //executa primeira quadrupla: desfaz o movimento e apaga a entrada do historico
    posicaoCabecoteEntrada.value = novaPosicaoEntrada;

    const novaFitaHistorico = [...fitaHistorico.value];
    novaFitaHistorico[posicaoCabecoteHistorico.value] = 'B';
    fitaHistorico.value = novaFitaHistorico;
    posicaoCabecoteHistorico.value = Math.max(0, posicaoCabecoteHistorico.value - 1);

    estadoAtual.value = quadruplaMovimento.proximoEstado;

    //executa segunda quadrupla: restaura o simbolo que foi sobrescrito
    const novaFitaEntrada = [...fitaEntrada.value];
    novaFitaEntrada[posicaoCabecoteEntrada.value] = quadruplaEscrita.simboloEscrito;
    fitaEntrada.value = novaFitaEntrada;

    estadoAtual.value = quadruplaEscrita.proximoEstado;

    if (fitaHistorico.value[posicaoCabecoteHistorico.value] === 'B') {
        faseExecucao.value = 4;
    }
    return true;
}

function executarFaseReconstrucao(){
    if (faseExecucao.value !== 3) {
        return;
    }

    if (quadruplasReversao.value.length === 0) {
        prepararReconstrucao();
    }

    let limitePassos = tamanhoFitaPadrao;
    let passosExecutados = 0;

    while (faseExecucao.value === 3 && passosExecutados < limitePassos) {
        const executou = executarUmaTransicaoReversao();
        if (!executou) {
            break;
        }
        passosExecutados++;
    }

    if (passosExecutados >= limitePassos && faseExecucao.value !== 4) {
        alert('A fase de reconstrução excedeu o limite de passos permitido.');
        return;
    }

    if (faseExecucao.value === 4) {
        console.log(`Reconstrução finalizada após ${passosExecutados} transições.`);
    }
}

function executarMaq(){
    executarComputacaoCompleta()
    executarFaseCopia()
    executarFaseReconstrucao()
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

    quadruplasCopia.value = [];
    quadruplasReversao.value = [];

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
                <Botao class="botao" :class="{'ativo': faseExecucao >= 2}" texto="2. Cópia" @acao="executarFaseCopia"/>
                <span class="linha" :class="{'ativo': faseExecucao >= 3}"></span>
                <Botao class="botao" :class="{'ativo': faseExecucao >= 3}" texto="3. Reconstrução" @acao="executarFaseReconstrucao"/>
            </div>

            <p>Controle</p>
            <div class="container-botoes">
                <Botao texto="Executar" :class="{'ativo': 1}" @acao="executarMaq"/>
                <Botao texto="Passo a passo" :class="{'ativo': 1}" @acao="executarUmPasso"/>
                <Botao texto="Reiniciar" :class="{'ativo': 1}" @acao="reiniciarMaquina"/>
            </div>
        </div>
        <div>
            <p>Quádruplas de processamento</p>
            <div class="cont-process">
                <div class="process">
                    <h3>Cópia</h3>
                    <pre>{{ quadruplasCopia.map(q => q.texto ? q.texto : q.textoPadrao).join('\n') }}</pre>
                </div>
                <div class="process">
                    <h3>Reversão</h3>
                    <pre>{{ quadruplasReversao.map(q => q.texto).join('\n') }}</pre>
                </div>
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
.process{
    display: block;
    align-items: center;
    padding: 20px;
    background-color: #d9d9d9;
    border-radius: 8px;
    margin: 20px;
    text-align: center;
    border: 1px solid #000000;
}

.cont-process{
    display: grid;
    grid-template-columns: 1fr 1fr; 
    gap: 30px;

}
</style>