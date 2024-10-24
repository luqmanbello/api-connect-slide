<!-- components/APIMonitoringDemo.vue -->
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { Clock, CheckCircle } from 'lucide-vue-next'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

const currentMetrics = ref({
  responseTime: 129,
  requests: 41,
  errorRate: 0.80
})

const chartData = ref({
  labels: [],
  datasets: [
    {
      label: 'Response Time (ms)',
      borderColor: '#4f46e5',
      data: [],
      yAxisID: 'y'
    },
    {
      label: 'Requests/sec',
      borderColor: '#22c55e',
      data: [],
      yAxisID: 'y1'
    }
  ]
})

const chartOptions = {
  responsive: true,
  interaction: {
    mode: 'index',
    intersect: false,
  },
  scales: {
    y: {
      type: 'linear',
      display: true,
      position: 'left',
      min: 0,
      max: 200
    },
    y1: {
      type: 'linear',
      display: true,
      position: 'right',
      min: 0,
      max: 100,
      grid: {
        drawOnChartArea: false
      }
    }
  }
}

onMounted(() => {
  // Initialize with some data
  const initialData = Array.from({ length: 10 }, (_, i) => {
    const time = new Date(Date.now() - (9 - i) * 1000)
    return {
      time: time.toLocaleTimeString('en-US', { 
        hour12: false,
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      }),
      responseTime: Math.random() * 40 + 80,
      requests: Math.random() * 20 + 30
    }
  })

  chartData.value.labels = initialData.map(d => d.time)
  chartData.value.datasets[0].data = initialData.map(d => d.responseTime)
  chartData.value.datasets[1].data = initialData.map(d => d.requests)

  // Update data every 2 seconds
  const interval = setInterval(() => {
    const now = new Date()
    const timeStr = now.toLocaleTimeString('en-US', { 
      hour12: false,
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit'
    })
    
    // Update chart data
    chartData.value.labels = [...chartData.value.labels.slice(1), timeStr]
    chartData.value.datasets[0].data = [...chartData.value.datasets[0].data.slice(1), Math.random() * 40 + 80]
    chartData.value.datasets[1].data = [...chartData.value.datasets[1].data.slice(1), Math.random() * 20 + 30]
    
    // Update current metrics
    currentMetrics.value = {
      responseTime: Math.round(chartData.value.datasets[0].data.slice(-1)[0]),
      requests: Math.round(chartData.value.datasets[1].data.slice(-1)[0]),
      errorRate: (Math.random() * 0.5 + 0.5).toFixed(2)
    }
  }, 2000)

  onUnmounted(() => clearInterval(interval))
})
</script>

<template>
  <div class="w-full max-w-4xl bg-white rounded-lg p-6">
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl font-semibold text-gray-900">Live API Monitoring Demo</h2>
      <CheckCircle 
        class="text-green-500"
        size="20"
      />
    </div>

    <div class="space-y-6">
      <div class="p-4 bg-gray-50 rounded-lg">
        <h3 class="text-lg font-semibold mb-4">Response Time & Requests</h3>
        <Line
          :data="chartData"
          :options="chartOptions"
          class="h-[300px]"
        />
      </div>

      <div class="grid grid-cols-3 gap-4">
        <div class="p-4 bg-blue-50 rounded-lg">
          <h4 class="text-sm font-medium text-gray-600 mb-2">Avg Response Time</h4>
          <div class="flex items-center">
            <Clock class="mr-2 text-gray-500" size="16" />
            <span class="text-xl font-semibold text-gray-900">
              {{ currentMetrics.responseTime }}ms
            </span>
          </div>
        </div>

        <div class="p-4 bg-green-50 rounded-lg">
          <h4 class="text-sm font-medium text-gray-600 mb-2">Current Requests</h4>
          <span class="text-xl font-semibold text-gray-900">
            {{ currentMetrics.requests }}/sec
          </span>
        </div>

        <div class="p-4 bg-yellow-50 rounded-lg">
          <h4 class="text-sm font-medium text-gray-600 mb-2">Error Rate</h4>
          <span class="text-xl font-semibold text-gray-900">
            {{ currentMetrics.errorRate }}%
          </span>
        </div>
      </div>
    </div>
  </div>
</template>