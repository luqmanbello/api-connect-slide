<!-- components/APIMonitoringDemo.vue -->
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { Clock, CheckCircle } from 'lucide-vue-next'
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend } from 'recharts'

const metrics = ref([])
const currentMetrics = ref({
  responseTime: 129,
  requests: 31,
  errorRate: 0.89
})
const status = ref('healthy')

// Helper to format time for X-axis
const formatTime = (date) => {
  return date.toLocaleTimeString('en-US', { 
    hour12: false,
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// Initialize data
onMounted(() => {
  // Create initial data points
  const now = new Date()
  const initialData = Array.from({ length: 6 }, (_, i) => ({
    time: new Date(now - (5 - i) * 2000).getTime(),
    responseTime: Math.random() * 40 + 80,
    requests: Math.random() * 20 + 30
  }))
  metrics.value = initialData

  // Update data periodically
  const interval = setInterval(() => {
    const now = new Date()
    const newMetric = {
      time: now.getTime(),
      responseTime: Math.random() * 40 + 80,
      requests: Math.random() * 20 + 30
    }
    
    metrics.value = [...metrics.value.slice(1), newMetric]
    currentMetrics.value = {
      responseTime: Math.round(newMetric.responseTime),
      requests: Math.round(newMetric.requests),
      errorRate: (Math.random() * 0.5 + 0.5).toFixed(2)
    }
    status.value = currentMetrics.value.errorRate > 0.9 ? 'degraded' : 'healthy'
  }, 2000)

  onUnmounted(() => clearInterval(interval))
})
</script>

<template>
  <div class="w-full max-w-4xl bg-white rounded-lg p-6">
    <!-- Header -->
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl font-semibold text-gray-900">Live API Monitoring Demo</h2>
      <CheckCircle 
        :class="status === 'healthy' ? 'text-green-500' : 'text-yellow-500'"
        size="20"
      />
    </div>

    <!-- Chart Section -->
    <div class="mb-6">
      <h3 class="text-lg font-semibold text-gray-700 mb-4">Response Time & Requests</h3>
      <LineChart 
        :width="600"
        :height="300"
        :data="metrics"
      >
        <CartesianGrid 
          strokeDasharray="3 3"
          stroke="#e5e7eb"
        />
        <XAxis 
          dataKey="time"
          :tickFormatter="time => formatTime(new Date(time))"
          stroke="#6b7280"
        />
        <YAxis 
          yAxisId="left"
          stroke="#6b7280"
          :domain="[0, 160]"
          :ticks="[0, 40, 80, 120, 160]"
        />
        <YAxis 
          yAxisId="right"
          orientation="right"
          stroke="#6b7280"
          :domain="[0, 80]"
          :ticks="[0, 20, 40, 60, 80]"
        />
        <Tooltip />
        <Legend />
        <Line
          yAxisId="left"
          type="monotone"
          dataKey="responseTime"
          stroke="#4f46e5"
          name="Response Time (ms)"
          :dot="false"
        />
        <Line
          yAxisId="right"
          type="monotone"
          dataKey="requests"
          stroke="#22c55e"
          name="Requests/sec"
          :dot="false"
        />
      </LineChart>
    </div>

    <!-- Metrics Grid -->
    <div class="grid grid-cols-3 gap-4">
      <!-- Response Time -->
      <div class="bg-blue-50 rounded-lg p-4">
        <h4 class="text-sm font-medium text-gray-600 mb-2">Avg Response Time</h4>
        <div class="flex items-center">
          <Clock class="mr-2 text-gray-500" size="16" />
          <span class="text-xl font-semibold text-gray-900">
            {{ currentMetrics.responseTime }}ms
          </span>
        </div>
      </div>

      <!-- Current Requests -->
      <div class="bg-green-50 rounded-lg p-4">
        <h4 class="text-sm font-medium text-gray-600 mb-2">Current Requests</h4>
        <span class="text-xl font-semibold text-gray-900">
          {{ currentMetrics.requests }}/sec
        </span>
      </div>

      <!-- Error Rate -->
      <div class="bg-yellow-50 rounded-lg p-4">
        <h4 class="text-sm font-medium text-gray-600 mb-2">Error Rate</h4>
        <span class="text-xl font-semibold text-gray-900">
          {{ currentMetrics.errorRate }}%
        </span>
      </div>
    </div>
  </div>
</template>