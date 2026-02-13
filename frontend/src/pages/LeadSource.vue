<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-900 py-8">
    <div class="mx-auto max-w-4xl px-4">
      <!-- Header -->
      <div class="mb-6 flex items-center gap-3">
        <FeatherIcon name="check-square" class="h-6 w-6 text-gray-700 dark:text-gray-300" />
        <h1 class="text-2xl font-semibold text-gray-900 dark:text-gray-100">Lead Tasks</h1>
      </div>

      <!-- Controls -->
      <div class="mb-6 flex flex-wrap items-center gap-2">
        <!-- Sort Buttons -->
        <div class="flex overflow-hidden rounded border border-gray-300 dark:border-gray-600">
          <button
            @click="sortOrder = 'latest'"
            :class="[
              'px-4 py-2 text-sm font-medium transition-colors',
              sortOrder === 'latest'
                ? 'bg-blue-500 text-white dark:bg-blue-600'
                : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-700'
            ]"
          >
            Latest First
          </button>
          <button
            @click="sortOrder = 'oldest'"
            :class="[
              'px-4 py-2 text-sm font-medium transition-colors border-l border-gray-300 dark:border-gray-600',
              sortOrder === 'oldest'
                ? 'bg-blue-500 text-white dark:bg-blue-600'
                : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-700'
            ]"
          >
            Oldest First
          </button>
        </div>

        <!-- Source Filter Buttons -->
        <div class="flex overflow-hidden rounded border border-gray-300 dark:border-gray-600">
          <button
            v-for="(filter, index) in sourceFilters"
            :key="filter.value"
            @click="activeSource = filter.value"
            :class="[
              'px-4 py-2 text-sm font-medium transition-colors',
              index > 0 ? 'border-l border-gray-300 dark:border-gray-600' : '',
              activeSource === filter.value
                ? 'bg-blue-500 text-white dark:bg-blue-600'
                : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-700'
            ]"
          >
            {{ filter.label }}
          </button>
        </div>
      </div>

      <!-- Task List -->
      <div class="flex flex-col gap-3">
        <div
          v-for="task in filteredTasks"
          :key="task.id"
          class="rounded border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-800 p-4 shadow-sm hover:shadow-md dark:hover:shadow-lg transition-shadow"
        >
          <div class="flex items-start justify-between">
            <div class="flex-1">
              <div class="flex items-center gap-2 flex-wrap mb-2">
                <h3 class="font-medium text-gray-900 dark:text-gray-100 text-base">{{ task.title }}</h3>
                <span
                  v-if="task.isNew"
                  class="px-2 py-0.5 text-xs font-medium bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-200 rounded-full"
                >
                  NEW
                </span>
                <span
                  v-if="task.overdueDays"
                  class="px-2 py-0.5 text-xs font-medium bg-red-100 dark:bg-red-900 text-red-800 dark:text-red-200 rounded-full"
                >
                  {{ task.overdueDays }} days overdue
                </span>
              </div>
              
              <div class="flex items-center gap-4 flex-wrap text-sm text-gray-600 dark:text-gray-400">
                <span class="font-mono text-xs text-gray-600 dark:text-gray-400">{{ task.leadId }}</span>
                <span :class="['px-2 py-0.5 rounded text-xs font-medium', getSourceClass(task.source)]">
                  {{ getSourceIcon(task.source) }} {{ task.source }}
                </span>
                <span class="flex items-center gap-1">
                  <FeatherIcon name="user" class="h-3.5 w-3.5" />
                  {{ task.raisedBy }}
                </span>
                <span class="flex items-center gap-1">
                  <FeatherIcon name="clock" class="h-3.5 w-3.5" />
                  {{ task.timeAgo }}
                </span>
              </div>
            </div>
          </div>
        </div>

        <p
          v-if="filteredTasks.length === 0"
          class="py-12 text-center text-gray-500 dark:text-gray-400"
        >
          No leads from this source.
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { FeatherIcon } from 'frappe-ui'

const leadTasks = [
  {
    id: '1',
    title: 'Follow up with lead - interested in premium package',
    leadId: 'FB-LEAD-2026-00451',
    source: 'facebook',
    raisedBy: 'Sales Team',
    timeAgo: '2 hours ago',
    isNew: true,
    overdueDays: undefined,
  },
  {
    id: '2',
    title: 'Schedule demo call - requested product walkthrough',
    leadId: 'FB-LEAD-2026-00448',
    source: 'facebook',
    raisedBy: 'Admin',
    timeAgo: '1 day ago',
    overdueDays: 3,
  },
  {
    id: '3',
    title: 'Send pricing details - enquiry about enterprise plan',
    leadId: 'IG-LEAD-2026-00322',
    source: 'instagram',
    raisedBy: 'Akhil Kumar',
    timeAgo: '3 days ago',
    isNew: true,
    overdueDays: 2,
  },
  {
    id: '4',
    title: 'Respond to product inquiry - wants bulk pricing',
    leadId: 'IG-LEAD-2026-00318',
    source: 'instagram',
    raisedBy: 'Ramesh Kumar',
    timeAgo: '5 days ago',
    overdueDays: 5,
  },
  {
    id: '5',
    title: 'Qualify lead - asked about delivery timelines',
    leadId: 'WA-LEAD-2026-00189',
    source: 'whatsapp',
    raisedBy: 'Sales Team',
    timeAgo: '1 hour ago',
    isNew: true,
  },
  {
    id: '6',
    title: 'Follow up on quote sent last week',
    leadId: 'WA-LEAD-2026-00185',
    source: 'whatsapp',
    raisedBy: 'Admin',
    timeAgo: '1 week ago',
    overdueDays: 7,
  },
  {
    id: '7',
    title: 'Re-engage cold lead - previously interested in starter plan',
    leadId: 'FB-LEAD-2026-00440',
    source: 'facebook',
    raisedBy: 'Ramesh Kumar',
    timeAgo: '2 weeks ago',
    overdueDays: 11,
  },
  {
    id: '8',
    title: 'Send catalogue to new prospect',
    leadId: 'WA-LEAD-2026-00180',
    source: 'whatsapp',
    raisedBy: 'Akhil Kumar',
    timeAgo: '3 days ago',
    isNew: true,
    overdueDays: 1,
  },
]

const sourceFilters = [
  { label: 'All Sources', value: 'all' },
  { label: 'Facebook', value: 'facebook' },
  { label: 'Instagram', value: 'instagram' },
  { label: 'WhatsApp', value: 'whatsapp' },
]

const sortOrder = ref('latest')
const activeSource = ref('all')

const filteredTasks = computed(() => {
  let tasks = leadTasks.filter(
    (t) => activeSource.value === 'all' || t.source === activeSource.value
  )
  
  if (sortOrder.value === 'oldest') {
    tasks = [...tasks].reverse()
  }
  
  return tasks
})

const getSourceClass = (source) => {
  const classes = {
    facebook: 'bg-blue-100 dark:bg-blue-900 text-blue-800 dark:text-blue-200',
    instagram: 'bg-pink-100 dark:bg-pink-900 text-pink-800 dark:text-pink-200',
    whatsapp: 'bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-200',
  }
  return classes[source] || 'bg-gray-100 dark:bg-gray-700 text-gray-800 dark:text-gray-200'
}

const getSourceIcon = (source) => {
  const icons = {
    facebook: '📘',
    instagram: '📸',
    whatsapp: '💬',
  }
  return icons[source] || '📋'
}
</script>