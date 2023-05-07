<script setup lang="ts">
import { computed, defineProps } from 'vue'
import { AvailableActions } from '../../bindings/Action'
import type { Action } from '../../bindings/Action'

// const myAction: Action = { action: 'show_dialog', title: 'title', body: 'body' }

const { action, index } = defineProps<{ action: Action; index: number }>()
const actionParameters = computed(() =>
  Object.entries(action)
    .filter(([key, _value]) => key !== 'id' && key !== 'action')
    .map(([key, value]) => ({ key, value }))
)
</script>

<template>
  <div
    class="min-w-[380px] min-h-[90px] place-content-evenly m-6 p-6 max-w-lg mx-auto bg-white rounded-xl shadow shadow-gray-600 flex justify-between items-center space-x-4 cursor-grab transition-200 transition-all transition ease-in-out hover:bg-gray-100"
  >
    <div class="text-3xl text-black font-bold flex">
      <div class="mt-1 mr-3">
        {{ index }}
      </div>
      <div class="text-3xl text-black font-muted pl-1 pr-2 mb-">|</div>
    </div>
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
    <button @click="$emit('delete', index)" class="text-gray-800 ml-2 cursor-pointer">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
        />
      </svg>
    </button>
  </div>
</template>
