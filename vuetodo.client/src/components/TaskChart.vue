<template>
  <div>
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import { Chart, registerables } from "chart.js";
Chart.register(...registerables);

const props = defineProps({
  completedTasks: Number,
  pendingTasks: Number,
});

const chart = ref(null);
const chartCanvas = ref(null);

function renderChart() {
  if (chart.value) {
    chart.value.destroy();
  }

  chart.value = new Chart(chartCanvas.value, {
    type: "doughnut",
    data: {
      labels: ["Виконані завдання", "Невиконані завдання"],
      datasets: [
        {
          data: [props.completedTasks, props.pendingTasks],
          backgroundColor: ["#42b983", "#d9534f"],
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
    },
  });
}

onMounted(() => {
  renderChart();
});

watch(() => [props.completedTasks, props.pendingTasks], renderChart);
</script>

<style scoped>
div {
  position: relative;
  height: 300px;
  width: 300px;
  margin: 20px auto;
}
</style>
