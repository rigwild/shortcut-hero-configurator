<script setup lang="ts">
import { computed, defineProps } from 'vue'
import { snakToTitleCase } from '../utils'
import type { Action } from '../../bindings/Action'

// const myAction: Action = { action: 'show_dialog', title: 'title', body: 'body' }

const { action } = defineProps<{ action: Action }>()
const actionParameters = computed(() =>
  Object.entries(action)
    .slice(1)
    .filter(([key, value]) => key !== 'id')
    .map(([key, value]) => ({ key, value }))
)
</script>

<template>
  <div class="m-6 p-6 max-w-lg mx-auto bg-white rounded-xl shadow shadow-gray-600 flex items-center space-x-4">
    <div>
      <div class="text-xl text-black font-bold mb-2">{{ snakToTitleCase(action.action) }}</div>
      <ul v-if="actionParameters.length > 0">
        <li v-for="{ key, value } in actionParameters" :key="key">
          <span class="text-gray-600 font-bold tracking-wide capitalize mr-1">{{ key }}</span
          ><br />
          <span class="text-slate-500">{{ value }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>
