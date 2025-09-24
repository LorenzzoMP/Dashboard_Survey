<script setup>
defineProps({
  titulo: String,
  valor: Number,
  corValor: {
    type: String,
    default: 'text-gray-800'
  }
})
</script>

<template>
  <div class="bg-white p-6 rounded-xl border border-slate-200">
    <h3 class="font-semibold text-slate-500">{{ titulo }}</h3>
    <p class="text-5xl font-bold mt-2" :class="corValor">{{ valor }}</p>
  </div>
</template>
