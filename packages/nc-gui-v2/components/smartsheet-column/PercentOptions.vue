<!-- File not in use for now -->

<script setup lang="ts">
import { precisions } from '@/utils/percentUtils'

interface Props {
  value: Record<string, any>
}

const props = defineProps<Props>()
const emit = defineEmits(['update:value'])
const vModel = useVModel(props, 'value', emit)

if (!vModel.value.meta) vModel.value.meta = {}
if (!vModel.value.meta?.negative) vModel.value.meta.negative = false
if (!vModel.value.meta?.default) vModel.value.meta.default = null
if (!vModel.value.meta?.precision) vModel.value.meta.precision = precisions[0].id
</script>

<template>
  <div class="flex flex-col mt-2 gap-2">
    <div class="flex flex-row space-x-2">
      <a-form-item class="flex w-1/2" label="Precision">
        <a-select v-model:value="vModel.meta.precision">
          <a-select-option v-for="(precision, i) of precisions" :key="i" :value="precision.id">
            <div class="flex flex-row items-center">
              <div class="text-xs">
                {{ precision.title }}
              </div>
            </div>
          </a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="Default Number (%)">
        <a-input v-model:value="vModel.meta.default" name="default" type="number" />
      </a-form-item>
    </div>
    <div class="flex flex-row mt-2">
      <a-form-item>
        <div class="flex flex-row space-x-2 items-center">
          <a-switch v-model:checked="vModel.meta.negative" name="negative" />
          <div class="text-xs">Allow negative numbers</div>
        </div>
      </a-form-item>
    </div>
  </div>
</template>
