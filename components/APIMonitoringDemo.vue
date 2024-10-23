<!-- components/APIMonitoringDemo.vue -->
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { AlertTriangle, CheckCircle, Clock } from 'lucide-vue-next'

const metrics = ref([])
const status = ref('healthy')

let interval
onMounted(() => {
  interval = setInterval(() => {
    const now = new Date()
    const newMetric = {
      timestamp: now.toLocaleTimeString(),
      responseTime: Math.random() * 100 + 50,
      errorRate: Math.random() * 2,
      requests: Math.floor(Math.random() * 50 + 30)
    }
    
    metrics.value = [...metrics.value.slice(-10), newMetric]
    status.value = newMetric.errorRate > 1.5 ? 'degraded' : 'healthy'
  }, 2000)
})

onUnmounted(() => {
  clearInterval(interval)
})
</script>

<template>
  <div class="w-full max-w-4xl p-6 bg-white/10 rounded-xl">
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl font-bold">Live API Monitoring Demo</h2>
      <div>
        <CheckCircle v-if="status === 'healthy'" class="text-green-500" />
        <AlertTriangle v-else class="text-yellow-500" />
      </div>
    </div>

    <div class="space-y-6">
      <div class="grid grid-cols-3 gap-4">
        <div class="p-4 bg-blue-500/10 rounded-lg">
          <h4 class="font-semibold mb-2">Avg Response Time</h4>
          <div class="flex items-center">
            <Clock class="mr-2" />
            <span class="text-2xl">
              {{ metrics.length ? Math.round(metrics[metrics.length - 1].responseTime) : 0 }}ms
            </span>
          </div>
        </div>

        <div class="p-4 bg-green-500/10 rounded-lg">
          <h4 class="font-semibold mb-2">Current Requests</h4>
          <span class="text-2xl">
            {{ metrics.length ? metrics[metrics.length - 1].requests : 0 }}/sec
          </span>
        </div>

        <div class="p-4 bg-yellow-500/10 rounded-lg">
          <h4 class="font-semibold mb-2">Error Rate</h4>
          <span class="text-2xl">
            {{ metrics.length ? metrics[metrics.length - 1].errorRate.toFixed(2) : 0 }}%
          </span>
        </div>
      </div>
    </div>
  </div>
</template>