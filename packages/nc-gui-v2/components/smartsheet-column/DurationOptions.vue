<script setup lang="ts">
import { durationOptions } from '@/utils'

interface Props {
  value: Record<string, any>
}

const props = defineProps<Props>()
const emit = defineEmits(['update:value'])
const vModel = useVModel(props, 'value', emit)

const durationOptionList =
  durationOptions.map((o) => ({
    ...o,
    // h:mm:ss (e.g. 3:45, 1:23:40)
    title: `${o.title} ${o.example}`,
  })) || []

// set default value
vModel.value.meta = {
  duration: 0,
  ...vModel.value.meta,
}
</script>

<template>
  <a-row>
    <a-col :span="24">
      <span class="prose-sm mt-2">A duration of time in minutes or seconds (e.g. 1:23).</span>
    </a-col>
    <a-col :span="24">
      <a-form-item label="Duration Format">
        <a-select v-model:value="vModel.meta.duration" class="w-52">
          <a-select-option v-for="(duration, i) of durationOptionList" :key="i" :value="duration.id">
            {{ duration.title }}
          </a-select-option>
        </a-select>
      </a-form-item>
    </a-col>
  </a-row>
</template>
