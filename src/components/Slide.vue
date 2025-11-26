<template>
  <section class="relative">
    <div class="overflow-hidden">
      <!-- Slides -->
      <div
          class="whitespace-nowrap transition-transform duration-500 ease-out"
          :style="{ transform: `translateX(-${current * 100}%)` }"
      >
        <!-- Slide 1 -->
        <div class="inline-block w-full">
          <div class="relative flex items-center">
            <div class="w-full">
              <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
                <div class="grid grid-cols-12">
                  <div
                      class="col-span-12 md:col-span-10 md:col-start-2 lg:col-span-8 lg:col-start-3"
                  >
                    <div class="text-center py-16 sm:py-20 lg:py-28">
                      <h3
                          class="pt-16 sm:pt-20 text-base font-semibold tracking-wide uppercase text-gray-500"
                      >
                        Luiz Felipe,
                        Gustavo,
                        João
                      </h3>
                      <h1 class="mt-3 text-3xl sm:text-4xl lg:text-5xl font-bold">
                        Consistent Hashing
                      </h1>
                      <p class="mt-4 text-gray-600 max-w-2xl mx-auto">
                        Explicação e Código fonte
                      </p>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- BG opcional -->
            <div
                class="pointer-events-none absolute inset-0 -z-10 bg-gradient-to-br from-gray-50 to-white"
            ></div>
          </div>
        </div>

        <!-- Slide 2 -->
        <div class="inline-block w-full">
          <div class="relative flex items-center">
            <div class="w-full">
              <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
                <div
                    class="flex flex-col items-center justify-center py-20 sm:py-24 lg:py-28"
                >
                  <!-- Só a imagem, sem borda/fundo -->
                  <a
                      href="https://pt.wikipedia.org/wiki/Universidade_Estadual_de_Ponta_Grossa"
                      target="_blank"
                      rel="noopener noreferrer"
                  >
                    <img
                        :src="uepgLogo"
                        alt="Universidade Estadual de Ponta Grossa"
                        class="h-24 md:h-32 lg:h-40 w-auto object-contain"
                    />
                  </a>
                </div>
              </div>
            </div>

            <!-- BG opcional -->
            <div
                class="pointer-events-none absolute inset-0 -z-10 bg-gradient-to-br from-gray-50 to-white"
            ></div>
          </div>
        </div>
      </div>
    </div>

    <!-- Controles -->
    <button
        class="absolute left-3 top-1/2 -translate-y-1/2 inline-flex items-center justify-center p-2"
        @click="prev"
        aria-label="Slide anterior"
    >
      <i class="bi bi-chevron-left text-xl"></i>
    </button>
    <button
        class="absolute right-3 top-1/2 -translate-y-1/2 inline-flex items-center justify-center p-2"
        @click="next"
        aria-label="Próximo slide"
    >
      <i class="bi bi-chevron-right text-xl"></i>
    </button>

    <!-- Indicadores -->
    <div class="mt-4 flex items-center justify-center gap-2">
      <button
          v-for="i in total"
          :key="i"
          class="h-2.5 w-2.5 rounded-full"
          :class="current === i - 1 ? 'bg-gray-800' : 'bg-gray-300 hover:bg-gray-400'"
          @click="go(i - 1)"
          :aria-label="`Ir para slide ${i}`"
      />
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import uepgLogo from '../Images/Logo_State_University_of_Ponta_Grossa.png'

const current = ref(0)
const total = 2

function next () {
  current.value = (current.value + 1) % total
}

function prev () {
  current.value = (current.value - 1 + total) % total
}

function go (idx) {
  current.value = idx
}

let timer = null
onMounted(() => {
  timer = setInterval(next, 6000)
})
onBeforeUnmount(() => {
  if (timer) clearInterval(timer)
})
</script>
