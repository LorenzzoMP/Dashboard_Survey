<script setup>
import { ref, computed, onMounted } from 'vue'

const clientes = ref([])

onMounted(async () => {
  try {
    const response = await fetch('/fire_inspection_mock.json')
    const data = await response.json()
    clientes.value = data.clientes
  } catch (error) {
    console.error('Erro ao carregar os dados de inspeção:', error)
  }
})

const todasAtividades = computed(() => {
  if (!clientes.value) return []

  let atividades = []
  clientes.value.forEach(cliente => {
    cliente.areas.forEach(area => {
      area.equipamentos.forEach(equipamento => {
        atividades.push({
          ...equipamento.atividade,
          clienteNome: cliente.nome,
          areaNome: area.nome,
          equipamentoNome: equipamento.nome,
        })
      })
    })
  })
  return atividades
})


const resumoStatus = computed(() => {
  const total = todasAtividades.value.length
  const concluidas = todasAtividades.value.filter(ativ => ativ.status === 'concluida').length
  const pendentes = total - concluidas
  return { total, concluidas, pendentes }
})


const atividadesPendentesUrgentes = computed(() => {
  return todasAtividades.value
    .filter(ativ => ativ.status === 'pendente' && ativ.alerta)
})


const proximasInspecoes = computed(() => {
  const hoje = new Date('2025-09-16')
  const futuro = new Date()
  futuro.setDate(hoje.getDate() + 30)

  return todasAtividades.value
    .filter(ativ => {
      if (!ativ.data_proxima_inspecao) return false
      const dataProxima = new Date(ativ.data_proxima_inspecao)
      return dataProxima >= hoje && dataProxima <= futuro
    })
    .sort((a, b) => new Date(a.data_proxima_inspecao) - new Date(b.data_proxima_inspecao));
})
</script>

<template>
  <div class="bg-gray-100 min-h-screen p-4 sm:p-8 font-sans">
    <div class="max-w-7xl mx-auto">
      <header class="mb-8">
        <h1 class="text-3xl font-bold text-gray-800">Dashboard de Inspeções</h1>
        <p class="text-gray-500">Acompanhamento de atividades e alertas</p>
      </header>

      <section id="resumo" class="grid grid-cols-1 sm:grid-cols-3 gap-6 mb-8">
        <div class="bg-white p-6 rounded-lg shadow-md text-center">
          <h3 class="text-lg font-semibold text-gray-500">Total de Atividades</h3>
          <p class="text-4xl font-bold text-blue-600">{{ resumoStatus.total }}</p>
        </div>
        <div class="bg-white p-6 rounded-lg shadow-md text-center">
          <h3 class="text-lg font-semibold text-gray-500">Concluídas</h3>
          <p class="text-4xl font-bold text-green-600">{{ resumoStatus.concluidas }}</p>
        </div>
        <div class="bg-white p-6 rounded-lg shadow-md text-center">
          <h3 class="text-lg font-semibold text-gray-500">Pendentes</h3>
          <p class="text-4xl font-bold text-orange-500">{{ resumoStatus.pendentes }}</p>
        </div>
      </section>

      <section id="alertas" class="bg-white p-6 rounded-lg shadow-md mb-8">
        <h2 class="text-2xl font-bold text-red-600 mb-4">Alertas Urgentes</h2>
        <div v-if="atividadesPendentesUrgentes.length > 0" class="space-y-4">
          <div v-for="(ativ, index) in atividadesPendentesUrgentes" :key="index" class="border-b last:border-b-0 pb-4">
            <p class="font-bold text-gray-700">{{ ativ.clienteNome }} - {{ ativ.equipamentoNome }}</p>
            <p class="text-sm text-gray-600">{{ ativ.descricao }}</p>
            <p class="text-sm font-semibold text-red-500 mt-1">Alerta: {{ ativ.alerta }}</p>
          </div>
        </div>
        <p v-else class="text-gray-500">Nenhum alerta urgente no momento.</p>
      </section>

      <section id="proximas-inspecoes" class="bg-white p-6 rounded-lg shadow-md">
        <h2 class="text-2xl font-bold text-gray-700 mb-4">Próximas Inspeções (Próximos 30 dias)</h2>
        <div v-if="proximasInspecoes.length > 0" class="space-y-4">
          <div v-for="(ativ, index) in proximasInspecoes" :key="index" class="border-b last:border-b-0 pb-4">
             <p class="font-bold text-gray-700">{{ ativ.clienteNome }} - {{ ativ.equipamentoNome }}</p>
             <p class="text-sm text-gray-600">Descrição: {{ ativ.descricao }}</p>
             <p class="text-sm font-semibold text-indigo-600 mt-1">Agendada para: {{ new Date(ativ.data_proxima_inspecao).toLocaleDateString() }}</p>
          </div>
        </div>
        <p v-else class="text-gray-500">Nenhuma inspeção agendada para os próximos 30 dias.</p>
      </section>

    </div>
  </div>
</template>
