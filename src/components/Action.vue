<script setup lang="ts">
import { computed, defineProps } from 'vue'
import { AvailableActions } from '../../bindings/Action'
import type { Action } from '../../bindings/Action'

// const myAction: Action = { action: 'show_dialog', title: 'title', body: 'body' }

const { action, index } = defineProps<{ action: Action; index: number }>()
const actionParameters = computed(() =>
  Object.entries(action)
    .slice(1)
    .filter(([key, _value]) => key !== 'id')
    .map(([key, value]) => ({ key, value }))
)
</script>

<template>
  <div
    class="min-w-[350px] min-h-[90px] place-content-evenly m-6 p-6 max-w-lg mx-auto bg-white rounded-xl shadow shadow-gray-600 flex justify-evenly items-center space-x-4 cursor-grab transition-200 transition-all transition ease-in-out hover:bg-gray-100"
  >
    <div class="text-3xl text-black font-bold">
      {{ index }}
    </div>
    <div class="text-3xl text-black font-muted pl-1 pr-2 pb-1">|</div>
    <div>
      <div class="text-xl text-black font-bold">
        {{ AvailableActions[action.action].description }}
      </div>
      <ul v-if="actionParameters.length > 0">
        <li v-for="{ key } in actionParameters" :key="key" class="my-2">
          <span class="text-gray-600 font-bold tracking-wide capitalize">
            {{
              // @ts-ignore
              AvailableActions[action.action].args[key].description
            }} </span
          ><br />
          <input
            type="text"
            class="bg-gray-100 p-2 rounded shadow shadow-gray-400 mt-1"
            placeholder="Value"
            v-model="(action as any)[key]"
          />
          <!-- <span class="text-slate-500">{{ value }}</span> -->
        </li>
      </ul>
    </div>
  </div>
</template>
