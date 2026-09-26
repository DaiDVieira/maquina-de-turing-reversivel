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
            <pre>{{ quintuplas.join('\n') }}</pre>
        </div>
        
        <div class="arq">
            <h3>Quadruplas</h3>
            <pre>{{ quadruplas.map(q => q.gerarTexto ? q.textoPadrao : q.texto).join('\n') }}</pre>
        </div>
    </div>

</template>

<script setup>
import {ref} from 'vue'

const nomeArquivo = ref('')
const conteudo = ref('')
const estados = ref([])
const alfEntrada = ref([])
const alfFita = ref([])
const quintuplas = ref([])
const quadruplas = ref([])
const entrada = ref('')
const emitir = defineEmits(['dados-atualizados'])

function lerArquivo(evento){
    const arquivo = evento.target.files[0]
    if(!arquivo) return

    const leitor = new FileReader()
    leitor.onload = (e) => {
        conteudo.value = e.target.result
        nomeArquivo.value = arquivo.name
        dividirElementos()
    }
    leitor.readAsText(arquivo)
}

function dividirElementos(){
    const linhas = conteudo.value.split(/\r?\n/).map(linha => linha.trim()).filter(linha => linha.length > 0)
    const cabecalho = linhas[0].split(/\s+/)
    const quantidadeQuintuplas = Number(cabecalho[3])

    estados.value = linhas[1].split(/\s+/).filter(Boolean)
    console.log('Estados:', estados.value)
    alfEntrada.value = linhas[2].split(/\s+/).filter(Boolean)
    console.log('Alfabeto da entrada:', alfEntrada.value)
    alfFita.value = linhas[3].split(/\s+/).filter(Boolean)
    console.log('Alfabeto da fita:', alfFita.value)
    quintuplas.value = linhas.slice(4, 4 + quantidadeQuintuplas)
    entrada.value = linhas[4 + quantidadeQuintuplas] ?? ''

    quadruplas.value = converterQuintuplasParaQuadruplas(quintuplas.value)

    emitir('dados-atualizados', {
        estados: estados.value,
        alfEntrada: alfEntrada.value,
        alfFita: alfFita.value,
        quintuplas: quintuplas.value,
        quadruplas: quadruplas.value,
        entrada: entrada.value,
    })
}

function converterQuintuplasParaQuadruplas(quintuplasBrutas) {
    const quadruplasFormatadas = [];

    quintuplasBrutas.forEach((qStr, indiceTransicao) => {
        const regex = /\(\s*(\d+)\s*,\s*([^)]+)\s*\)\s*=\s*\(\s*(\d+)\s*,\s*([^)]+)\s*,\s*([LRS])\s*\)/;
        const match = qStr.match(regex);

        if (!match) {
            quadruplasFormatadas.push({
                tipo: 'outro',
                texto: qStr
            });
            return;
        }

        const [
            ,
            estadoAtual,
            simboloLido,
            proximoEstado,
            simboloEscrito,
            movimento
        ] = match;


        const estado = estadoAtual.trim();
        const lido = simboloLido.trim();
        const escrito = simboloEscrito.trim();
        const proximo = proximoEstado.trim();
        const direcao = movimento.trim();

        const estadoIntermediario =`${estado}_${proximo}_${lido}`; //o que sera escrito na fita de historico

       //primeira quadrupla
        quadruplasFormatadas.push({
            tipo: 'escrever',
            indiceTransicao,
            estadoOrigem: estado,
            simboloLido: lido,
            simboloEscrito: escrito,
            estadoIntermediario,
            texto:
                `(${estado}, ${lido}) = ` +
                `(${escrito}, ${estadoIntermediario})`
        });

        //segunda quadrupla
        quadruplasFormatadas.push({
            tipo: 'mover',
            indiceTransicao,
            estadoIntermediario,
            simboloEscrito: escrito,
            movimento: direcao,
            proximoEstado: proximo,
            estadoOrigem: estado,
            simboloLido: lido,

            //texto da div de quadruplas considerando as 3 fitas
            gerarTexto: (fitaE, posE, fitaH, posH, fitaS, posS) => {
                const valE = fitaE?.[posE] ?? 'B';
                const valH = fitaH?.[posH] ?? 'B';
                const valS = fitaS?.[posS] ?? 'B';
                return (
                    `(${estadoIntermediario}, ` +
                    `{${valE}, ${valH}, ${valS}}) = ` +
                    `(${direcao}, ${proximo})`
                );
            },

            textoPadrao:
                `(${estadoIntermediario}, ` +
                `{${lido}, B, B}) = ` +
                `(${direcao}, ${proximo})`
        });

    });
    return quadruplasFormatadas;
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