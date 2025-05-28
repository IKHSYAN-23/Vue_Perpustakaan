<template>
  <div class="chart-card">
    <header class="chart-header">
      <h2>Grafik Peminjaman per Bulan</h2>
    </header>
    <div class="chart-body">
      <canvas ref="chartCanvas"></canvas>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue';
import Chart from 'chart.js/auto';

export default {
  name: 'ChartPeminjaman',
  props: {
    peminjamanData: {
      type: Array,
      required: true
    }
  },
  setup(props) {
    const chartCanvas = ref(null);
    let chartInstance = null;

    const processChartData = () => {
      const monthlyData = {};
      props.peminjamanData.forEach(item => {
        const date = new Date(item.tanggal_peminjaman);
        const key = `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}`;
        monthlyData[key] = (monthlyData[key] || 0) + 1;
      });

      const sortedKeys = Object.keys(monthlyData).sort();
      const monthNames = [
        'Januari','Februari','Maret','April','Mei','Juni',
        'Juli','Agustus','September','Oktober','November','Desember'
      ];

      const labels = sortedKeys.map(key => {
        const [year, month] = key.split('-');
        return `${monthNames[parseInt(month, 10) - 1]} ${year}`;
      });
      const data   = sortedKeys.map(key => monthlyData[key]);

      return { labels, data };
    };

    const renderChart = () => {
      const { labels, data } = processChartData();
      const ctx = chartCanvas.value.getContext('2d');

      if (chartInstance) {
        chartInstance.destroy();
      }

      chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels,
          datasets: [{
            label: 'Jumlah Peminjaman',
            data,
            // warna default Chart.js untuk konsistensi tema
            backgroundColor: Chart.defaults.color,
            borderColor: Chart.defaults.borderColor,
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          layout: {
            padding: 20
          },
          scales: {
            x: {
              grid: { display: false },
              ticks: { maxRotation: 45, minRotation: 0 }
            },
            y: {
              beginAtZero: true,
              grid: { color: '#e0e0e0' },
              ticks: { precision: 0 }
            }
          },
          plugins: {
            legend: {
              display: false
            },
            tooltip: {
              backgroundColor: '#333',
              titleColor: '#fff',
              bodyColor: '#fff',
              padding: 10,
              callbacks: {
                label(ctx) {
                  return `Peminjaman: ${ctx.formattedValue}`;
                }
              }
            }
          }
        }
      });
    };

    onMounted(() => {
      if (props.peminjamanData.length) renderChart();
    });

    watch(() => props.peminjamanData, (newVal) => {
      if (newVal.length) renderChart();
    }, { deep: true });

    return { chartCanvas };
  }
};
</script>

<style scoped>
.chart-card {
  max-width: 960px;
  margin: 2rem auto;
  background: #fff;
  border-radius: 0.75rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  overflow: hidden;
}

.chart-header {
  padding: 1.5rem 2rem;
  background-color: #4caf50;
}

.chart-header h2 {
  margin: 0;
  font-size: 1.5rem;
  color: #fff;
  text-align: center;
  border-bottom: 2px solid rgba(255,255,255,0.5);
  padding-bottom: 0.5rem;
}

.chart-body {
  position: relative;
  height: 450px;
  padding: 1rem 2rem 2rem;
}

canvas {
  width: 100% !important;
  height: 100% !important;
}
</style>
