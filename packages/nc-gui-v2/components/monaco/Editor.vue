<script setup lang="ts">
import * as monaco from 'monaco-editor'
import JsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker'
import EditorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'
import TypescriptWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker'
import { onMounted } from '#imports'
import { deepCompare } from '~/utils'

interface Props {
  modelValue: string | Record<string, any>
  hideMinimap?: boolean
  lang?: string
  validate?: boolean
  disableDeepCompare?: boolean
  readOnly?: boolean
}

const { hideMinimap, lang = 'json', validate = true, disableDeepCompare = false, modelValue, readOnly } = defineProps<Props>()

const emits = defineEmits(['update:modelValue'])

let vModel = $computed<string>({
  get: () => {
    if (typeof modelValue === 'object') {
      return JSON.stringify(modelValue, null, 2)
    } else {
      return modelValue
    }
  },
  set: (newVal: string | Record<string, any>) => {
    if (typeof modelValue === 'object') {
      try {
        emits('update:modelValue', typeof newVal === 'object' ? newVal : JSON.parse(newVal))
      } catch (e) {
        console.error(e)
      }
    } else {
      emits('update:modelValue', newVal)
    }
  },
})

const isValid = ref(true)

/**
 * Adding monaco editor to Vite
 *
 * @ts-expect-error */
self.MonacoEnvironment = {
  getWorker(_: any, label: string) {
    switch (label) {
      case 'json':
        return new JsonWorker()
      case 'typescript':
        return new TypescriptWorker()
      default:
        return new EditorWorker()
    }
  },
}

const root = ref<HTMLDivElement>()
let editor: monaco.editor.IStandaloneCodeEditor

const format = () => {
  editor.setValue(JSON.stringify(JSON.parse(editor?.getValue() as string), null, 2))
}

defineExpose({
  format,
  isValid,
})

onMounted(() => {
  if (root.value && lang) {
    const model = monaco.editor.createModel(vModel, lang)

    if (lang === 'json') {
      // configure the JSON language support with schemas and schema associations
      monaco.languages.json.jsonDefaults.setDiagnosticsOptions({
        validate: validate as boolean,
      })
    }

    editor = monaco.editor.create(root.value, {
      model,
      theme: 'vs',
      foldingStrategy: 'indentation',
      selectOnLineNumbers: true,
      scrollbar: {
        verticalScrollbarSize: 8,
        horizontalScrollbarSize: 8,
      },
      tabSize: 2,
      automaticLayout: true,
      readOnly,
      minimap: {
        enabled: !hideMinimap,
      },
    })

    editor.onDidChangeModelContent(async () => {
      try {
        isValid.value = true

        if (disableDeepCompare) {
          vModel = editor.getValue()
        } else {
          const obj = JSON.parse(editor.getValue())
          if (!obj || !deepCompare(vModel, obj)) vModel = obj
        }
      } catch (e) {
        isValid.value = false
        console.log(e)
      }
    })
  }
})

watch($$(vModel), (v) => {
  if (!editor || !v) return

  const editorValue = editor?.getValue()
  if (!disableDeepCompare) {
    if (!editorValue || !deepCompare(JSON.parse(v), JSON.parse(editorValue))) {
      editor.setValue(v)
    }
  } else {
    if (editorValue !== v) editor.setValue(v)
  }
})
</script>

<template>
  <div ref="root"></div>
</template>
