<script setup lang="ts">
import { computed } from "vue"

const props = defineProps<{
  data: {
    name: string
    value: number
  }[]
}>()

const series = computed(() => [
  {
    name: "Compliance",
    data: props.data.map(i => i.value),
  },
])

const chartOptions = computed(() => ({
  chart: {
    type: "bar",
    toolbar: { show: false },
  },

  legend: {
    show: true,
    position: "top",
  },

  xaxis: {
    categories: props.data.map(i => i.name),
    title: {
      text: "Nilai"
    },
    labels: {
      rotate: -45, // biar muat kalau nama panjang
    }
  },

  yaxis: {
    title: {
      text: "Precentage (%)"
    },
    labels: {
      formatter: (val: number) => `${val}%`
    }
  },

  plotOptions: {
    bar: {
      borderRadius: 6,
      columnWidth: "50%",
    },
  },

  dataLabels: {
    enabled: true,
    formatter: (val: number) => `${val}%`,
  },

  tooltip: {
    y: {
      formatter: (val: number) => `${val}%`,
    },
  },
}))
</script>

<template>
  <apexchart
    type="bar"
    height="350"
    :options="chartOptions"
    :series="series"
  />
</template>
