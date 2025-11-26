<!-- NavBar.vue -->
<template>
  <nav class="bg-gray-900 text-white">
    <div class="relative px-4 py-4 flex items-center justify-between gap-4">
      <!-- Logo -->
      <RouterLink
          class="text-2xl font-bold"
          :to="{ name: 'home' }"
          @click="closeMenu"
      >
        Site Pesquisa
      </RouterLink>

      <!-- Search alinhado no meio -->
      <!-- div
          class="relative flex items-center bg-white overflow-hidden
                 rounded-lg h-11
                 w-full max-w-xs md:max-w-md lg:max-w-lg
                 flex-1"
      >
        <div class="grid place-items-center h-full w-10 text-gray-400">
          <i class="bi bi-search"></i>
        </div>

        <input
            class="peer h-full w-full outline-none text-sm text-gray-700 pr-3"
            type="text"
            id="search"
            placeholder="Search something..."
        />
      </div -->

      <!-- Botão Hamburguer -->
      <button
          class="md:hidden text-white text-3xl p-2 -mr-2"
          @click="open = !open"
          :aria-expanded="open.toString()"
          aria-label="Abrir/fechar menu"
      >
        <i :class="open ? 'bi bi-x-lg' : 'bi bi-list'"></i>
      </button>

      <!-- Menu -->
      <ul
          class="md:flex md:items-center gap-5 absolute left-0 top-full w-full md:w-auto bg-gray-900
               md:static transition-transform duration-300 z-40 px-4 md:px-6 pb-4 md:pb-0"
          :class="[open ? 'translate-x-0' : '-translate-x-full md:translate-x-0']"
      >
        <li v-for="item in items" :key="item" class="my-4 md:my-0">
          <!-- Git: link externo -->
          <a
              v-if="item === 'Git'"
              href="https://github.com/LuizFelipeDias/Estrutura-Dados-ConsistenHashing"
              target="_blank"
              rel="noopener noreferrer"
              class="text-xl hover:text-green-500 duration-300"
              @click="closeMenu"
          >
            {{ item }}
          </a>

          <!-- Demais itens: mesmo comportamento original -->
          <button
              v-else
              class="text-xl hover:text-green-500 duration-300"
              @click="closeMenu"
          >
            {{ item }}
          </button>
        </li>
      </ul>
    </div>
  </nav>
</template>

<script setup>
import { ref } from 'vue'
import { RouterLink } from 'vue-router'

const open = ref(false)
const items = ['Conteudo', 'Sobre', 'Git']

function closeMenu () {
  open.value = false
}
</script>
