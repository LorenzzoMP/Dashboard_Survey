<script setup>
defineProps({
  semanas: Array
})

const diasDaSemana = ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'];
</script>

<template>
  <div>
    <div class="grid grid-cols-7 gap-2 text-center text-sm font-semibold text-slate-500 mb-2">
      <div v-for="dia in diasDaSemana" :key="dia">{{ dia }}</div>
    </div>

    <div class="grid grid-cols-7 gap-2">
      <template v-for="semana in semanas" :key="semana[0] ? semana[0].data : ''">

        <div v-for="dia in semana" :key="dia.data"
             class="bg-white border border-slate-200 rounded-lg p-2 min-h-[120px] flex flex-col"
             :class="{ 'bg-blue-50': dia.isHoje }">

          <div class="font-bold text-slate-600 mb-1" :class="{ 'text-blue-600': dia.isHoje }">
            {{ dia.numeroDia }}
          </div>

          <div v-if="dia.inspecoes.length > 0" class="space-y-1 overflow-y-auto text-xs">
            <div v-for="inspecao in dia.inspecoes" :key="inspecao.equipamentoNome"
                 class="bg-indigo-100 text-indigo-800 p-1 rounded truncate"
                 :title="`${inspecao.clienteNome} - ${inspecao.equipamentoNome}`">
              {{ inspecao.clienteNome }}
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>
