<template>
  <codemirror
    v-model="code"
    placeholder="Code goes here..."
    :style="{ height: '78vh' }"
    :autofocus="false"
    :indent-with-tab="true"
    :tab-size="4"
    :extensions="extensions"
    @ready="handleReady"
    @change="log('change', $event)"
    @focus="log('focus', $event)"
    @blur="log('blur', $event)"
    class="m-12"
  />
</template>

<script lang="ts">
import { defineComponent, shallowRef } from 'vue'
import { Codemirror } from 'vue-codemirror'
import { json } from '@codemirror/lang-json'
import { oneDark } from '@codemirror/theme-one-dark'

export default defineComponent({
  components: {
    Codemirror,
  },
  props: {
    code: {
      type: Object,
      default: `console.log('Hello, world!')`,
    },
  },
  setup(props) {
    const extensions = [json(), oneDark]

    // Codemirror EditorView instance ref
    const view = shallowRef()
    const handleReady = (payload: any) => {
      view.value = payload.view
    }

    // Status is available at all times via Codemirror EditorView
    // const getCodemirrorStates = () => {
    //   const state = view.value.state
    //   const ranges = state.selection.ranges
    //   const selected = ranges.reduce((r, range) => r + range.to - range.from, 0)
    //   const cursor = ranges[0].anchor
    //   const length = state.doc.length
    //   const lines = state.doc.lines
    //   // more state info ...
    //   // return ...
    // }

    return {
      code: JSON.stringify(props.code, null, 4),
      extensions,
      handleReady,
      log: console.log,
    }
  },
})
</script>
