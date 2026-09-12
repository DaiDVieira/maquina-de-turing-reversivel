<script setup>
defineProps({
  tipos: { type: Array, required: true },
})

const tipo = defineModel('tipo')
</script>

<template>
    <div class="container-navbar">
        <header class="header">
            <div class="navbar">
                <img class="navbar-logo" src="../../public/imagens/logo.png"></img>
                <h1 class="navbar-titulo">Simulador de Máquina de Turing</h1>
            </div>
        </header>
    </div>
</template>

<style scoped>
.container-navbar{
    background-color: #4155c8;
}

.header {
  display: flex;
  
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 18px 32px;
  margin: 0;
  background: var(--white);
  border-bottom: 1px solid var(--line);
}

.navbar {
  display: flex;
  align-items: center;
  gap: 12px;
}

.navbar-logo {
  font-size: 1.6rem;
  color: var(--accent);
  transform: scaleX(-1);
  width: 30px;
}

.navbar-titulo {
  display: flex;
  flex-direction: column;
  line-height: 1.25;
  font-size: 140%;
  color: #fcf5f5;
}

.navbar-titulo strong {
  font-size: 1.05rem;
}

.navbar-titulo span {
  font-size: 0.78rem;
  color: var(--ink-soft);
}

.seletor-maquina {
  display: flex;
  align-items: center;
  gap: 10px;
}

.seletor-rotulo {
  font-size: 0.8rem;
  color: var(--ink-soft);
  white-space: nowrap;
}

.seletor-maquina select {
  font-family: var(--font-display);
  font-size: 0.9rem;
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid var(--line);
  background: var(--paper-alt);
  color: var(--ink);
  cursor: pointer;
}

.seletor-maquina select:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

@media (max-width: 640px) {
  .navbar {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>