<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'
import draggable from 'vuedraggable'
import _defaultConfiguration from '../shortcut-hero.example.json'
import ActionComponent from './components/Action.vue'
import CodeEditor from './components/CodeEditor.vue'
import Chevron from './components/Chevron.vue'
import H1 from './components/H1.vue'
import H2 from './components/H2.vue'
import H3 from './components/H3.vue'
import Kbd from './components/Kbd.vue'
import { KeybdKey } from '../bindings/KeybdKeyDef'
import { AvailableActions, type Action } from '../bindings/Action'

/** Inject an `id` property to each action */
const defaultConfiguration = {
  ..._defaultConfiguration,
  keyboard_shortcuts: _defaultConfiguration.keyboard_shortcuts.map(shortcut => ({
    ...shortcut,
    actions: shortcut.actions.map(action => ({ ...action, id: Math.random() })),
  })),
}

const drag = ref(false)
const newKeyInput = ref('')
const configuration = reactive(defaultConfiguration)
const editedShortcutIndex = ref<number | null>(0)
const addActionSelect = ref<keyof typeof AvailableActions>('debug')

// load from local storage on mount
const stateConfiguration = localStorage.getItem('state-configuration')
if (stateConfiguration) {
  const parsed = JSON.parse(stateConfiguration)
  configuration.keyboard_shortcuts = parsed.keyboard_shortcuts
  configuration.openai_api_key = parsed.openai_api_key
}
const stateEditedShortcutIndex = localStorage.getItem('state-editedShortcutIndex')
if (stateEditedShortcutIndex) {
  editedShortcutIndex.value = parseInt(stateEditedShortcutIndex)
}

// save to local storage on change
watch(configuration, (newConfig, _oldConfig) => localStorage.setItem('state-configuration', JSON.stringify(newConfig)))
watch(editedShortcutIndex, (newIndex, _oldIndex) => localStorage.setItem('state-editedShortcutIndex', newIndex + ''))

const dragOptions = computed(() => ({
  animation: 200,
  group: 'description',
  disabled: false,
  ghostClass: 'ghost',
}))

const variablesInShortcut = computed(() =>
  editedShortcutIndex.value === null
    ? []
    : configuration.keyboard_shortcuts[editedShortcutIndex.value].actions
        .filter(action => action.action === 'set_variable' && (action as any).name !== '')
        .map(action => (action as any).name)
        .reduce((acc, name) => {
          if (!acc.includes(name)) acc.push(name)
          return acc
        }, [] as string[])
)

/** Remove the `id` injected property */
const configurationJson = computed(() => {
  const clone = JSON.parse(JSON.stringify(configuration))
  clone.keyboard_shortcuts = clone.keyboard_shortcuts.map((shortcut: any) => ({
    ...shortcut,
    actions: shortcut.actions.map((action: any) => {
      const { id, ...rest } = action
      return rest
    }),
  }))
  return clone
})

const addShortcut = () => {
  configuration.keyboard_shortcuts.push({
    description: 'Open the browser',
    keys: [],
    actions: [],
  })
}

const addKeyToShortcut = (shortcutIndex: number, key: string) => {
  // Check if key exists
  if (!Object.values(KeybdKey).includes(key as any)) return
  configuration.keyboard_shortcuts[shortcutIndex].keys.push(key)
}

const addActionToShortcut = (
  shortcutIndex: number,
  actionId: keyof typeof AvailableActions,
  where: 'START' | 'END'
) => {
  const newAction: Action & { id: any } = {
    ...Object.values(AvailableActions[actionId].args).reduce((acc, arg) => {
      acc[arg.name] = ''
      return acc
    }, {}),
    id: Math.random(),
    action: actionId,
  }

  if (where === 'START') configuration.keyboard_shortcuts[shortcutIndex].actions.unshift(newAction)
  else if (where === 'END') configuration.keyboard_shortcuts[shortcutIndex].actions.push(newAction)

  // configuration.keyboard_shortcuts[shortcutIndex].actions.push(newAction)
}
</script>

<template>
  <div class="p-4">
    <H1>Shortcut Hero</H1>
    <div class="p-3 grid grid-cols-2 gap-8">
      <div>
        <H2>General</H2>
        <H3>OpenAI API Key</H3>
        <input
          type="text"
          class="w-full p-2 rounded-md border border-gray-300 bg-gray-100 text-gray-800"
          v-model="configuration.openai_api_key"
        />

        <H2>Shortcuts</H2>
        <button
          class="bg-gray-100 hover:bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"
          @click="addShortcut"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-6 h-6 mr-2"
          >
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 6v6m0 0v6m0-6h6m-6 0H6" class="text-gray-800" />
          </svg>
          <span class="text-gray-800">New Shortcut</span>
        </button>

        <div
          v-for="({ description, keys, actions }, index) in configuration.keyboard_shortcuts"
          :key="`${index}-${actions.map(x => x.id).join('-')}`"
          class="my-4 p-5 bg-white rounded-lg shadow shadow-gray-600 flex items-center justify-between transition ease-in-out hover:-translate-y-1 hover:shadow-xl hover:bg-gray-100 cursor-pointer"
          :class="{
            // 'shadow-xl': editedShortcutIndex === index,
            'bg-gradient-to-r from-orange-200 to-pink-200': editedShortcutIndex === index,
          }"
          @click="editedShortcutIndex = index"
        >
          <div class="flex flex-col items-left justify-center gap-7">
            <div class="text-xl font-bold">Shortcut #{{ index + 1 }}</div>
            <div class="text-lg">{{ description }}</div>
            <div>
              <Kbd v-for="key in keys" :key="key">{{ key }}</Kbd>
            </div>
          </div>

          <button
            @click.stop="
              () => {
                editedShortcutIndex = null
                configuration.keyboard_shortcuts.splice(index, 1)
              }
            "
            class="text-gray-800 ml-2 cursor-pointer"
          >
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

        <H2>Configuration File</H2>
        <CodeEditor :code="configurationJson" :key="configurationJson" />
      </div>
      <div v-if="editedShortcutIndex !== null">
        <H2>Edit Shortcut #{{ editedShortcutIndex + 1 }}</H2>

        <H3>Description</H3>
        <textarea
          type="text"
          class="w-full p-2 rounded-md border border-gray-300 bg-gray-100 text-gray-800"
          rows="4"
          v-model="configuration.keyboard_shortcuts[editedShortcutIndex].description"
        />

        <H3>Keys</H3>
        <div>
          <div class="my-4">
            <Kbd
              v-for="(key, index) in configuration.keyboard_shortcuts[editedShortcutIndex].keys"
              :key="`${key}-${index}`"
              class="text-xl"
              deletable
              @click="configuration.keyboard_shortcuts[editedShortcutIndex].keys.splice(index, 1)"
            >
              {{ key }}
            </Kbd>
          </div>
          <div class="mt-6 flex">
            <input
              list="add-keys-list"
              class="bg-gray-100 p-2 rounded shadow shadow-gray-400 mr-4"
              id="add-keys-choice"
              name="add-keys-choice"
              placeholder="Add a key..."
              v-model="newKeyInput"
            />

            <datalist id="add-keys-list">
              <option v-for="key in Object.values(KeybdKey)" :key="key" :value="key"></option>
            </datalist>

            <button
              class="bg-gray-100 hover:bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded flex items-center"
              @click="addKeyToShortcut(editedShortcutIndex, newKeyInput)"
            >
              <span class="text-gray-800">Add</span>
            </button>
          </div>
        </div>

        <H2>Actions</H2>

        <div v-if="variablesInShortcut.length > 0">
          <H3>Variables</H3>
          <ul class="list-disc text-gray-300">
            <li v-for="variable in variablesInShortcut" :key="variable" class="ml-6 mt-1">
              <code>{{ variable }}</code>
            </li>
          </ul>
        </div>

        <H3>Flow</H3>
        <div class="flex items-center justify-center my-4">
          <select class="bg-gray-100 p-2 rounded shadow shadow-gray-400 mr-4" v-model="addActionSelect">
            <option
              v-for="(action, index) in AvailableActions"
              :key="`${index}-${action.description}`"
              :value="action.action"
            >
              {{ action.description }}
            </option>
          </select>

          <button
            class="bg-gray-100 hover:bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"
            @click="addActionToShortcut(editedShortcutIndex, addActionSelect, 'START')"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-6 h-6 mr-2"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M12 6v6m0 0v6m0-6h6m-6 0H6"
                class="text-gray-800"
              />
            </svg>
            <span class="text-gray-800">Add Action</span>
          </button>
        </div>

        <draggable
          class="flex items-center justify-center flex-col"
          v-model="configuration.keyboard_shortcuts[editedShortcutIndex].actions"
          v-bind="dragOptions"
          @start="drag = true"
          @end="drag = false"
          item-key="id"
        >
          <template #item="{ element, index }">
            <div class="flex flex-col items-center justify-center">
              <ActionComponent
                @delete="configuration.keyboard_shortcuts[editedShortcutIndex].actions.splice(index, 1)"
                :action="element"
                :index="index"
              />
              <Chevron v-if="index < configuration.keyboard_shortcuts[editedShortcutIndex].actions.length - 1" />
            </div>
          </template>
        </draggable>

        <div
          v-if="configuration.keyboard_shortcuts[editedShortcutIndex].actions.length > 0"
          class="flex items-center justify-center my-4"
        >
          <select class="bg-gray-100 p-2 rounded shadow shadow-gray-400 mr-4" v-model="addActionSelect">
            <option
              v-for="(action, index) in AvailableActions"
              :key="`${index}-${action.description}`"
              :value="action.action"
            >
              {{ action.description }}
            </option>
          </select>

          <button
            class="bg-gray-100 hover:bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"
            @click="addActionToShortcut(editedShortcutIndex, addActionSelect, 'END')"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-6 h-6 mr-2"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M12 6v6m0 0v6m0-6h6m-6 0H6"
                class="text-gray-800"
              />
            </svg>
            <span class="text-gray-800">Add Action</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
