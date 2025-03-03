<script setup lang="ts">
import type { UITypes } from 'nocodb-sdk'
import { computed } from '#imports'

interface Props {
  value: Record<string, any>
}

const props = defineProps<Props>()
const emit = defineEmits(['update:value'])
const vModel = useVModel(props, 'value', emit)

const { sqlUi } = useProject()

const { onAlter, onDataTypeChange, validateInfos } = useColumnCreateStoreOrThrow()

// todo: 2nd argument of `getDataTypeListForUiType` is missing!
const dataTypes = computed(() => sqlUi?.value?.getDataTypeListForUiType(vModel.value as { uidt: UITypes }, '' as any))

// to avoid type error with checkbox
vModel.value.rqd = !!vModel.value.rqd
vModel.value.pk = !!vModel.value.pk
vModel.value.un = !!vModel.value.un
vModel.value.ai = !!vModel.value.ai
vModel.value.au = !!vModel.value.au
</script>

<template>
  <div class="p-4 border-[2px] radius-1 border-grey w-full flex flex-col gap-2">
    <div class="flex justify-between w-full gap-1">
      <a-form-item label="NN">
        <a-checkbox
          v-model:checked="vModel.rqd"
          :disabled="vModel.pk || !sqlUi.columnEditable(vModel)"
          class="nc-column-checkbox-NN"
          @change="onAlter"
        />
      </a-form-item>
      <a-form-item label="PK">
        <a-checkbox
          v-model:checked="vModel.pk"
          :disabled="!sqlUi.columnEditable(vModel)"
          class="nc-column-checkbox-PK"
          @change="onAlter"
        />
      </a-form-item>
      <a-form-item label="AI">
        <a-checkbox
          v-model:checked="vModel.ai"
          :disabled="sqlUi.colPropUNDisabled(vModel) || !sqlUi.columnEditable(vModel)"
          class="nc-column-checkbox-AI"
          @change="onAlter"
        />
      </a-form-item>
      <a-form-item label="UN" :disabled="sqlUi.colPropUNDisabled(vModel) || !sqlUi.columnEditable(vModel)" @change="onAlter">
        <a-checkbox v-model:checked="vModel.un" class="nc-column-checkbox-UN" />
      </a-form-item>
      <a-form-item label="AU" :disabled="sqlUi.colPropAuDisabled(vModel) || !sqlUi.columnEditable(vModel)" @change="onAlter">
        <a-checkbox v-model:checked="vModel.au" class="nc-column-checkbox-AU" />
      </a-form-item>
    </div>
    <a-form-item :label="$t('labels.databaseType')" v-bind="validateInfos.dt">
      <a-select v-model:value="vModel.dt" @change="onDataTypeChange">
        <a-select-option v-for="type in dataTypes" :key="type" :value="type">
          {{ type }}
        </a-select-option>
      </a-select>
    </a-form-item>
    <a-form-item :label="$t('labels.lengthValue')">
      <a-input
        v-model:value="vModel.dtxp"
        :disabled="sqlUi.getDefaultLengthIsDisabled(vModel.dt) || !sqlUi.columnEditable(vModel)"
        @input="onAlter"
      />
    </a-form-item>
    <a-form-item v-if="sqlUi.showScale(vModel)" label="Scale">
      <a-input v-model="vModel.dtxs" :disabled="!sqlUi.columnEditable(vModel)" @input="onAlter" />
    </a-form-item>
    <a-form-item :label="$t('placeholder.defaultValue')">
      <a-textarea v-model:value="vModel.cdf" auto-size @input="onAlter(2, true)" />
      <span class="text-gray-400 text-xs">{{ sqlUi.getDefaultValueForDatatype(vModel.dt) }}</span>
    </a-form-item>
  </div>
</template>
