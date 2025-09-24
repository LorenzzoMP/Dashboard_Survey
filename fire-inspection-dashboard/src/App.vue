<script setup>
import { ref, computed, onMounted } from 'vue'
import StatCard from './components/StatCard.vue'
import AlertsChart from './components/AlertsChart.vue'
import CalendarView from './components/CalendarView.vue'

const clientes = ref([])

// --- LÓGICA DE DADOS ---
onMounted(async () => {
  try {
    const response = await fetch('/fire_inspection_mock.json') //
    const data = await response.json()
    clientes.value = data.clientes //
  } catch (error) {
    console.error('Erro ao carregar os dados de inspeção:', error)
  }
})

const todasAtividades = computed(() => {
  if (!clientes.value) return []
  let atividades = []
  clientes.value.forEach(cliente => {
    cliente.areas.forEach(area => { //
      area.equipamentos.forEach(equipamento => { //
        atividades.push({
          ...equipamento.atividade, //
          clienteNome: cliente.nome, //
          areaNome: area.nome, //
          equipamentoNome: equipamento.nome, //
        })
      })
    })
  })
  return atividades
})

 // Calcula os totais para os cartões de resumo (StatCards).

const resumoStatus = computed(() => {
  const total = todasAtividades.value.length
  const concluidas = todasAtividades.value.filter(ativ => ativ.status === 'concluida').length //
  const pendentes = total - concluidas
  return { total, concluidas, pendentes }
})

 //Filtra apenas as atividades que estão pendentes E que possuem um alerta.

const atividadesPendentesUrgentes = computed(() => {
  return todasAtividades.value
    .filter(ativ => ativ.status === 'pendente' && ativ.alerta) //
})

 // Filtra as inspeções agendadas para os próximos 30 dias.

const proximasInspecoes = computed(() => {
  const hoje = new Date('2025-09-16')
  const futuro = new Date()
  futuro.setDate(hoje.getDate() + 30)

  return todasAtividades.value
    .filter(ativ => {
      if (!ativ.data_proxima_inspecao) return false //
      const dataProxima = new Date(ativ.data_proxima_inspecao) //
      return dataProxima >= hoje && dataProxima <= futuro
    })
    .sort((a, b) => new Date(a.data_proxima_inspecao) - new Date(b.data_proxima_inspecao)); //
})


 // Prepara os dados para o gráfico de alertas.

const resumoAlertasAgrupadosPorCliente = computed(() => {
  const tiposDeAlerta = [...new Set(atividadesPendentesUrgentes.value.map(a => a.alerta))];
  const nomesDeClientes = [...new Set(atividadesPendentesUrgentes.value.map(a => a.clienteNome))];

  const datasets = tiposDeAlerta.map((tipoAlerta, index) => {
    const data = nomesDeClientes.map(nomeCliente => {
      return atividadesPendentesUrgentes.value.filter(
        ativ => ativ.clienteNome === nomeCliente && ativ.alerta === tipoAlerta
      ).length;
    });

    const cores = ['#ef4444', '#f97316', '#eab308', '#3b82f6', '#8b5cf6'];

    return {
      label: tipoAlerta,
      backgroundColor: cores[index % cores.length],
      data: data,
    };
  });

  return {
    labels: nomesDeClientes,
    datasets: datasets,
  };
});

 // Prepara os dados para a vista de calendário.

const calendarioSemanas = computed(() => {
  if (proximasInspecoes.value.length === 0) return [];

  const semanas = [];
  const hoje = new Date('2025-09-16');
  const diaDaSemanaHoje = hoje.getDay();

  const inicioCalendario = new Date(hoje);
  inicioCalendario.setDate(hoje.getDate() - diaDaSemanaHoje);

  let semanaAtual = [];

  for (let i = 0; i < 28; i++) {
    const diaAtual = new Date(inicioCalendario);
    diaAtual.setDate(inicioCalendario.getDate() + i);

    const inspecoesDoDia = proximasInspecoes.value.filter(ativ => {
      const dataAtividade = new Date(ativ.data_proxima_inspecao);
      return dataAtividade.toDateString() === diaAtual.toDateString();
    });

    semanaAtual.push({
      data: diaAtual,
      numeroDia: diaAtual.getDate(),
      inspecoes: inspecoesDoDia,
      isHoje: diaAtual.toDateString() === hoje.toDateString(),
    });

    if (semanaAtual.length === 7) {
      semanas.push(semanaAtual);
      semanaAtual = [];
    }
  }

  return semanas;
});
</script>

<template>
  <div class="bg-slate-100 min-h-screen font-sans">
    <div class="max-w-7xl mx-auto p-4 sm:p-8">

      <header class="mb-10">
        <h1 class="text-4xl font-bold text-slate-800">Dashboard de Inspeções</h1>
        <p class="text-slate-500 mt-1">Acompanhamento de atividades e alertas da equipa</p>
      </header>

      <section id="resumo" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-10">
        <StatCard titulo="Total de Atividades" :valor="resumoStatus.total" cor-valor="text-blue-600" />
        <StatCard titulo="Concluídas" :valor="resumoStatus.concluidas" cor-valor="text-green-600" />
        <StatCard titulo="Pendentes" :valor="resumoStatus.pendentes" cor-valor="text-orange-500" />
        <StatCard titulo="Próximas (30d)" :valor="proximasInspecoes.length" cor-valor="text-indigo-600" />
      </section>

      <div class="grid grid-cols-1 xl:grid-cols-2 gap-10">

        <section id="alertas" class="bg-white p-6 rounded-xl border border-slate-200">
          <h2 class="text-2xl font-bold text-slate-700 mb-5">Alertas Urgentes por Cliente</h2>
          <div v-if="atividadesPendentesUrgentes.length > 0">
            <AlertsChart :chart-data="resumoAlertasAgrupadosPorCliente" />
          </div>
          <p v-else class="text-slate-500">Nenhum alerta urgente no momento.</p>
        </section>

        <section id="proximas-inspecoes" class="bg-white p-6 rounded-xl border border-slate-200">
          <h2 class="text-2xl font-bold text-slate-700 mb-5">Calendário de Próximas Inspeções</h2>
          <div v-if="proximasInspecoes.length > 0">
            <CalendarView :semanas="calendarioSemanas" />
          </div>
          <p v-else class="text-slate-500">Nenhuma inspeção agendada para os próximos 30 dias.</p>
        </section>

      </div>
    </div>
  </div>
</template>
